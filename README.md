# Sistema de Matrícula Escolar

Sistema cliente-servidor para la gestión de matrícula en escuelas y colegios de Panamá (educación K-12).

> **Estado:** En desarrollo · **Entrega final:** 20 de junio de 2026

---

## Descripción

Aplicación de escritorio multiusuario que permite a una institución educativa panameña administrar el ciclo completo de matrícula de sus estudiantes: desde la inscripción inicial al grado y paralelo, pasando por el cobro de aranceles y el registro de calificaciones por trimestre, hasta la generación de reportes y certificaciones.

El sistema modela la estructura del sistema educativo panameño (Preescolar, Primaria, Premedia y Media diversificada), opera con la escala de evaluación oficial (1.0–5.0, mínimo aprobatorio 3.0) y el calendario en trimestres.

## Stack tecnológico

| Capa | Tecnología |
|---|---|
| Lenguaje | C++17 |
| **API server** | **Crow (C++, header-only REST framework)** |
| **Auth** | **JWT (jwt-cpp)** |
| **Driver BD (server)** | **libpqxx (C++ nativo)** |
| Interfaz gráfica | Qt 6 (Widgets) |
| **HTTP client (Qt)** | **QNetworkAccessManager** |
| Sistema de build | CMake |
| Base de datos | PostgreSQL 16 |
| Reportes | QPrinter / QPdfWriter + QtCharts |
| IDE | Qt Creator (código) · VSCode (docs) |
| Compilador | MinGW (Windows) |
| Diagramas | PlantUML |
| Documentación | Doxygen |
| Control de versiones | Git + GitHub |

## Arquitectura

```
  ┌────────────┐    ┌────────────┐    ┌────────────┐
  │ Cliente Qt │    │ Cliente Qt │    │ Cliente Qt │
  │ (Secretaría)│   │ (Docente)  │    │ (Admin)    │
  └─────┬──────┘    └─────┬──────┘    └─────┬──────┘
        │                 │                 │
        └─────────────────┼─────────────────┘
                          │ HTTP/JSON + JWT
                 ┌────────▼────────┐
                 │  Crow REST API  │  ← lógica de negocio
                 │   Server C++    │    validaciones, reglas
                 └────────┬────────┘
                          │ SQL (libpqxx)
                 ┌────────▼────────┐
                 │   PostgreSQL    │  ← integridad, auditoría
                 └─────────────────┘
```

**Arquitectura de 3 capas:** los clientes Qt nunca acceden directamente a la base de datos — toda la comunicación pasa por el API REST en C++ (Crow). Esto desacopla la presentación de la lógica y permite escalar el servidor horizontalmente.

Ver [arquitectura escalable](docs/arquitectura-escalable.md) para el diseño de producción con load balancer, réplicas y multi-tenancy.

## Estructura del repositorio

```
matricula-proyectoC++/
├── docs/             Documentación del proyecto (propuesta, manual de usuario)
├── diagrams/         Diagramas UML y modelo ER (PlantUML)
├── sql/              Scripts SQL: schema, seeds, migraciones
├── src/              Código fuente C++
└── README.md
```

## Documentación

- [Propuesta del proyecto](docs/propuesta.md)
- [Diagrama de casos de uso](diagrams/use-cases.puml)
- [Modelo Entidad-Relación](diagrams/er-model.puml)

## Autor

Eloy Rivadeneira · Proyecto final · 2026
