# Arquitectura Escalable — Sistema de Matrícula Escolar

## Visión general

Este documento describe dos niveles de arquitectura:

1. **Lo que implementamos** — arquitectura de 3 capas con REST API en C++ (Crow), suficiente para demostrar un sistema multiusuario desacoplado y escalable.
2. **Arquitectura de producción** — lo que se agregaría si el sistema tuviera que servir a cientos de escuelas simultáneamente.

---

## Nivel 1 — Lo que implementamos (proyecto)

### Arquitectura de 3 capas

```
 ┌──────────────────────────────────────────────┐
 │              CAPA DE PRESENTACIÓN            │
 │                                              │
 │   Qt Desktop Client (C++ + Qt Widgets)       │
 │   • Interfaz gráfica por rol                 │
 │   • Llama endpoints REST vía HTTP            │
 │   • Parsea respuestas JSON                   │
 │   • Almacena JWT token en memoria            │
 └───────────────────┬──────────────────────────┘
                     │ HTTP/JSON + JWT
                     ↓
 ┌──────────────────────────────────────────────┐
 │            CAPA DE LÓGICA DE NEGOCIO         │
 │                                              │
 │   Crow REST API Server (C++17)               │
 │   • Expone endpoints REST                    │
 │   • Valida JWT en cada request               │
 │   • Aplica reglas de negocio                 │
 │     (validaciones de matrícula, cálculo      │
 │      de promedios, verificación de cupos…)   │
 │   • Único punto que accede a la BD           │
 └───────────────────┬──────────────────────────┘
                     │ SQL (libpqxx)
                     ↓
 ┌──────────────────────────────────────────────┐
 │               CAPA DE DATOS                  │
 │                                              │
 │   PostgreSQL 16                              │
 │   • Esquema relacional normalizado           │
 │   • Restricciones de integridad              │
 │   • Triggers de auditoría                    │
 │   • Row Level Security (preparado)           │
 └──────────────────────────────────────────────┘
```

### Stack implementado

| Componente | Tecnología | Rol |
|---|---|---|
| Cliente desktop | Qt 6 + C++17 | Interfaz gráfica por rol |
| HTTP client | QNetworkAccessManager (Qt) | Llamadas REST desde el cliente |
| API server | Crow (C++17, header-only) | Servidor REST que expone la lógica |
| JSON | nlohmann/json + QJsonDocument | Serialización en server y client |
| Auth | JWT (jwt-cpp) | Tokens sin estado, válidos N horas |
| DB driver | libpqxx | Conexión C++ nativa a PostgreSQL |
| Base de datos | PostgreSQL 16 | Persistencia central |
| Build | CMake 3.20+ | Build unificado de server + client |

### Ventajas sobre cliente-BD directo

| Aspecto | Cliente → BD directo | 3 capas con API |
|---|---|---|
| Seguridad | Credenciales de BD en cada cliente | Solo el server conoce la BD |
| Escalabilidad | N clientes = N conexiones a BD | N clientes = 1 pool del server |
| Mantenimiento | Cambio de BD requiere recompilar todos los clientes | Cambio transparente para el cliente |
| Plataformas | Solo Qt desktop | Mismo server puede servir web, móvil, desktop |
| Lógica de negocio | Dispersa en cada cliente | Centralizada en el server |

### Endpoints REST del API (diseño inicial)

```
POST   /api/auth/login              → devuelve JWT
POST   /api/auth/logout

GET    /api/estudiantes             → lista paginada
POST   /api/estudiantes             → crear
GET    /api/estudiantes/:id
PUT    /api/estudiantes/:id
GET    /api/estudiantes/:id/matriculas

GET    /api/acudientes
POST   /api/acudientes
GET    /api/acudientes/:id

GET    /api/grados
GET    /api/grados/:id/paralelos
GET    /api/paralelos/:id/estudiantes

POST   /api/matriculas              → matricular (valida prerrequisitos, cupo, estado financiero)
GET    /api/matriculas/:id
DELETE /api/matriculas/:id          → anular

GET    /api/pagos
POST   /api/pagos
GET    /api/pagos/:id/recibo        → PDF

POST   /api/calificaciones
GET    /api/estudiantes/:id/calificaciones

GET    /api/reportes/lista-clase/:paralelo_id
GET    /api/reportes/record/:estudiante_id
GET    /api/reportes/estadisticas

GET    /api/auditoria               → solo Admin
```

---

## Nivel 2 — Arquitectura de producción (multi-escuela)

Si el sistema creciera para servir a cientos de escuelas simultáneamente:

```
Clientes (desktop, web, móvil)
         │
         ↓
┌─────────────────────────────────────────────────────────┐
│                   LOAD BALANCER                          │
│              (nginx / AWS ALB / Cloudflare)              │
│  Distribuye tráfico, SSL termination, rate limiting      │
└────────────────────┬────────────────────────────────────┘
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
 ┌──────────┐  ┌──────────┐  ┌──────────┐
 │ API      │  │ API      │  │ API      │  ← escalado horizontal
 │ Server 1 │  │ Server 2 │  │ Server 3 │    (más instancias según demanda)
 │ (Drogon) │  │ (Drogon) │  │ (Drogon) │    Drogon en Linux para producción
 └──────────┘  └──────────┘  └──────────┘
       │             │             │
       └─────────────┼─────────────┘
                     │
              ┌──────▼──────┐
              │  PgBouncer  │  ← pool de conexiones
              │             │    500 clientes → 50 conexiones reales a BD
              └──────┬──────┘
          ┌──────────┼──────────┐
          ↓                     ↓
   ┌────────────┐        ┌────────────┐
   │  BD        │───────→│  Réplica   │  ← escritura en primaria
   │  Primaria  │        │  (lectura) │     lectura en réplica
   └────────────┘        └────────────┘
          │
   ┌────────────┐
   │   Redis    │  ← caché de sesiones JWT + datos frecuentes
   └────────────┘
```

### Multi-tenancy (varias escuelas en un sistema)

Cada escuela tiene su propio **schema** en PostgreSQL:

```sql
-- Escuela A: Colegio San José
CREATE SCHEMA escuela_001;
CREATE TABLE escuela_001.estudiantes (...);

-- Escuela B: Instituto Panamá
CREATE SCHEMA escuela_002;
CREATE TABLE escuela_002.estudiantes (...);
```

El API server selecciona el schema según el token JWT del usuario:

```
JWT payload: { "escuela_id": "001", "rol": "secretaria", "sub": "42" }
```

Con Row Level Security de PostgreSQL, se puede ir un paso más allá y tener todo en las mismas tablas con aislamiento automático a nivel de BD:

```sql
ALTER TABLE estudiantes ENABLE ROW LEVEL SECURITY;
CREATE POLICY escuela_isolation ON estudiantes
  USING (id_escuela = current_setting('app.escuela_id')::INT);
```

### Componentes adicionales en producción

| Componente | Propósito |
|---|---|
| **Docker + Kubernetes** | Despliegue del server en contenedores, autoescalado |
| **CI/CD (GitHub Actions)** | Tests automáticos + deploy en cada push a main |
| **Redis** | Caché de JWTs inválidos (logout), rate limiting, sesiones |
| **PgBouncer** | Pool de conexiones frente a PostgreSQL |
| **Réplica de lectura** | Separar tráfico de lectura (reportes) de escritura |
| **Drogon** | Reemplaza Crow: async/await nativo, ORM, más performance |
| **Monitoreo** | Prometheus + Grafana para métricas del server y la BD |
| **S3/Object storage** | Almacenamiento de fotos y PDFs generados |

### Decisiones de diseño (Crow vs Drogon)

Durante el desarrollo se usa **Crow** (header-only, ideal para Windows) por su facilidad de configuración. En un despliegue de producción sobre Linux se usaría **Drogon** por sus ventajas:

- I/O completamente asíncrono (event loop)
- ORM integrado con PostgreSQL async
- Connection pooling nativo
- Soporte de WebSockets para notificaciones en tiempo real
- Benchmarks entre los más altos de cualquier framework web

La migración Crow → Drogon es directa: los endpoints tienen la misma estructura, solo cambia el framework que los registra.

---

## Decisiones tomadas para este proyecto

| Decisión | Alternativa descartada | Razón |
|---|---|---|
| Crow en Windows | Drogon | Drogon requiere setup complejo en Windows; Crow es header-only |
| JWT stateless | Sesiones en BD | JWT no requiere consulta a BD en cada request |
| libpqxx en el server | QtSql | El server no usa Qt; libpqxx es la librería C++ nativa de PostgreSQL |
| QNetworkAccessManager en el client | Libcurl | Ya viene incluido en Qt, sin dependencia extra |
| Un schema por escuela | RLS | Más fácil de respaldar, migrar y aislar por institución |
