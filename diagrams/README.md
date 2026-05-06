# Diagramas

Los diagramas se mantienen en formato **PlantUML** (`.puml`), texto plano que se puede versionar en Git y diff-ear como código.

## Cómo visualizarlos

### Opción 1 — VSCode (recomendada)

1. Instalar la extensión **PlantUML** de jebbs (`jebbs.plantuml`).
2. Instalar **Java JRE** (PlantUML lo necesita): https://adoptium.net/
3. Abrir cualquier archivo `.puml` y presionar `Alt+D` para previsualizar.
4. Para exportar a PNG/SVG: paleta de comandos (`Ctrl+Shift+P`) → `PlantUML: Export Current Diagram`.

### Opción 2 — En línea (sin instalar nada)

1. Copiar el contenido del `.puml`.
2. Pegarlo en https://www.plantuml.com/plantuml/uml/

### Opción 3 — Herramienta gráfica

Si preferís editar visualmente, podés importar el `.puml` en https://app.diagrams.net/ (draw.io) o en StarUML.

## Diagramas disponibles

| Archivo | Descripción | Estado |
|---|---|---|
| [use-cases.puml](use-cases.puml) | Casos de uso por actor | Preliminar |
| [er-model.puml](er-model.puml) | Modelo Entidad-Relación de la base de datos | Preliminar |
| [deployment.puml](deployment.puml) | Despliegue: Qt Client → Crow API → PostgreSQL | Preliminar |

## Pendientes

- [ ] Diagrama de clases del dominio (server + client)
- [ ] Diagrama de secuencia: login con JWT
- [ ] Diagrama de secuencia: matriculación (flujo HTTP)
- [ ] Diagrama de secuencia: registro de pago
- [ ] Diagrama de componentes (arquitectura de 3 capas)
