# Propuesta de Proyecto Final

## Sistema de Matrícula Escolar para Escuelas y Colegios de Panamá

---

### 1. Información general

| Campo | Valor |
|---|---|
| Nombre del sistema | Sistema de Matrícula Escolar |
| Tipo | Aplicación de escritorio cliente-servidor |
| Dominio | Educación K-12 (Panamá) |
| Autor | Eloy Rivadeneira |
| Inicio | 5 de mayo de 2026 |
| Entrega | 20 de junio de 2026 |
| Modalidad | Trabajo individual |

---

### 2. Objetivo general

Diseñar e implementar un sistema multiusuario que permita a una institución educativa panameña administrar de forma integral el proceso de matrícula de sus estudiantes, desde la inscripción al año lectivo hasta la emisión de reportes oficiales, respetando la estructura, el calendario y la escala de evaluación del sistema educativo panameño.

---

### 3. Objetivos específicos

1. Modelar la estructura académica panameña: niveles (Preescolar, Primaria, Premedia, Media), grados, paralelos y materias por grado.
2. Implementar la matriculación de estudiantes en grado y paralelo de un año lectivo, con validaciones automáticas (aprobación previa, cupo, documentación, estado financiero).
3. Gestionar el registro de **acudientes** y su vínculo con uno o varios estudiantes.
4. Permitir a los docentes registrar calificaciones por materia y trimestre, en escala 1.0–5.0.
5. Administrar pagos de aranceles y emitir comprobantes y estados de cuenta.
6. Generar reportes oficiales exportables a PDF (lista de clase, récord académico, certificados).
7. Implementar autenticación basada en roles (Administrador, Secretaría, Docente, Acudiente) con control granular de permisos.
8. Mantener un registro de auditoría de las operaciones críticas del sistema.

---

### 4. Justificación

**¿Por qué un sistema de matrícula escolar?**
La matrícula es el proceso administrativo más sensible de una institución educativa: vincula al estudiante con su grado, condiciona los pagos, habilita el registro de notas y es la base de toda reportería oficial. Automatizarlo elimina errores manuales, agiliza la atención al acudiente y deja una trazabilidad legible ante auditorías del MEDUCA.

**¿Por qué C++ con Qt?**
- Requisito del curso: el proyecto debe desarrollarse en C++ o en un framework basado en C++/Rust.
- Qt 6 es el framework C++ multiplataforma más maduro para aplicaciones de escritorio, con licencia LGPL para uso libre.
- Provee de fábrica los componentes que el proyecto necesita: GUI declarativa, módulo SQL, generación de PDF, gráficos y patrón Model/View.

**¿Por qué PostgreSQL en lugar de SQLite?**
- El sistema es **multiusuario**: secretaría, dirección, docentes y acudientes acceden concurrentemente. SQLite no soporta concurrencia real de escritura.
- PostgreSQL permite definir vistas, restricciones avanzadas, triggers de auditoría y roles por base de datos, lo que demuestra dominio relacional.
- Para defender el proyecto como un sistema "real" y no un prototipo escolar, PostgreSQL es el estándar.

---

### 5. Alcance funcional (módulos)

#### 5.1 Autenticación y control de acceso
- Inicio de sesión con usuario y contraseña (hash + salt).
- Cuatro roles: **Administrador**, **Secretaría**, **Docente**, **Acudiente**.
- Bloqueo de cuenta tras intentos fallidos.

#### 5.2 Gestión de personas
- **Estudiantes**: datos personales, cédula juvenil, foto, vínculo con acudiente(s).
- **Acudientes**: datos completos, parentesco con el estudiante, contacto.
- **Docentes**: datos personales, materias que dicta, paralelos asignados.

#### 5.3 Estructura académica
- **Institución**: datos del colegio (nombre, código MEDUCA, dirección, logo).
- **Niveles**: Preescolar, Primaria, Premedia, Media diversificada.
- **Grados**: 1° a 6° de Primaria, 7° a 9° de Premedia, 10° a 12° de Media (con bachillerato).
- **Paralelos**: A, B, C, … por grado.
- **Materias**: asignadas a cada grado, con docente responsable por paralelo.

#### 5.4 Año lectivo y trimestres
- Apertura/cierre del año lectivo.
- Definición de los tres trimestres con sus fechas.
- Bloqueo automático de operaciones fuera del período habilitado.

#### 5.5 Matriculación (núcleo del sistema)
- Inscripción del estudiante a un grado + paralelo del año lectivo activo.
- Validaciones automáticas:
  - Aprobación del grado anterior (o ingreso nuevo).
  - Cupo disponible en el paralelo.
  - Documentación entregada (partida de nacimiento, certificado anterior, foto, copia de cédula del acudiente).
  - Estado de cuenta del acudiente.
- Generación del comprobante de matrícula en PDF.

#### 5.6 Pagos y aranceles
- Configuración de aranceles por nivel/grado (matrícula, mensualidad, otros).
- Registro de pagos parciales y totales.
- Estado de cuenta en tiempo real.
- Emisión de recibo en PDF.

#### 5.7 Calificaciones
- Captura de notas por el docente, por materia y trimestre, en escala 1.0–5.0.
- Cálculo automático de promedio trimestral y final.
- Bloqueo de modificación una vez cerrado el trimestre (excepto Administrador con auditoría).
- Estado de aprobación según mínimo de 3.0.

#### 5.8 Reportes
- Lista oficial de clase por paralelo.
- Récord académico del estudiante.
- Certificado de matrícula.
- Reporte de pagos pendientes.
- Estadísticas por grado/nivel/año.
- Todos exportables a PDF con encabezado institucional.

#### 5.9 Auditoría
- Registro de quién hizo qué, cuándo y desde qué cliente.
- Inmutable: solo se puede consultar, no editar ni borrar.

---

### 6. Arquitectura

**Modelo cliente-servidor de tres capas:**

1. **Capa de presentación** — Cliente Qt 6 con Widgets, ejecutándose en cada PC del personal autorizado.
2. **Capa de lógica de negocio** — Clases C++ de dominio (Estudiante, Matrícula, Calificación…) y servicios (MatriculaService, PagoService…) dentro del cliente.
3. **Capa de datos** — Servidor PostgreSQL único, accesible vía red local. La aplicación accede mediante `QSqlDatabase` con driver `QPSQL` y prepared statements para prevenir inyección SQL.

**Patrones aplicados:**
- DAO (Data Access Object) para aislar SQL del resto de la aplicación.
- Model/View de Qt para listas y tablas.
- Repositorio + Servicio para la lógica de negocio.
- Singleton para la conexión a BD (con cuidado de hilos).

---

### 7. Cronograma

| Semana | Fechas | Hitos |
|---|---|---|
| 0 | 6 may | Propuesta, casos de uso, ER preliminar, repo en GitHub. **Presentación de avances.** |
| 1 | 7–13 may | Instalación de Qt y PostgreSQL. UML y ER finales. Esqueleto del proyecto. Módulo de autenticación. |
| 2 | 14–20 may | CRUDs base: estudiantes, acudientes, docentes, niveles, grados, paralelos, materias. |
| 3 | 21–27 may | **Núcleo: Matriculación** con todas sus validaciones. |
| 4 | 28 may–3 jun | Pagos y calificaciones. |
| 5 | 4–10 jun | Reportes en PDF y auditoría. |
| 6 | 11–17 jun | Pulido, manual de usuario, video demo, pruebas. |
| Buffer | 18–20 jun | Imprevistos y ensayo de defensa. |

---

### 8. Documentación a entregar

1. Este documento de propuesta.
2. Diagrama de casos de uso (UML).
3. Diagrama de clases (UML).
4. Diagramas de secuencia para los flujos críticos (matrícula, registro de pago, registro de notas).
5. Modelo Entidad-Relación.
6. Diagrama de despliegue (cliente-servidor).
7. Manual de usuario por rol.
8. Manual de instalación y configuración.
9. Documentación del código generada con Doxygen.
10. Repositorio en GitHub con historial de commits y issues como evidencia del proceso.

---

### 9. Riesgos identificados y mitigación

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Curva de aprendizaje de Qt y PostgreSQL | Alto | Empezar setup en semana 1 con tutoriales oficiales. |
| Subestimar la complejidad de matriculación | Alto | Asignarle una semana completa (semana 3) con todas las validaciones. |
| Problemas de despliegue del servidor PostgreSQL | Medio | Tener ambiente local funcional desde semana 1; documentar instalación. |
| Falta de tiempo para pulido y manual | Medio | Reservar semana 6 + buffer exclusivamente para esto. |
