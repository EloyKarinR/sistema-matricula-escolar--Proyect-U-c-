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
| Interfaz gráfica | Qt 6 (Widgets) |
| Sistema de build | CMake |
| Base de datos | PostgreSQL 16 |
| Driver BD | QtSql con `QPSQL` |
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
   │ (Secretaría)│   │ (Docente)  │    │ (Acudiente)│
   └─────┬──────┘    └─────┬──────┘    └─────┬──────┘
         │                 │                 │
         └─────────────────┼─────────────────┘
                           │ TCP/IP
                  ┌────────▼────────┐
                  │   PostgreSQL    │
                  │     Servidor    │
                  └─────────────────┘
```

Varios clientes Qt se conectan a un único servidor PostgreSQL central. La lógica de negocio reside en la aplicación cliente; la base de datos se ocupa de la integridad referencial, restricciones y auditoría a nivel de fila.

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
