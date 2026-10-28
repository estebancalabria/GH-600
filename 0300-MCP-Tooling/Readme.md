# GitHub Agents, MCP y Agentic Workflows

## Introducción

Los agentes de software modernos no trabajan de forma aislada. Para realizar tareas útiles necesitan interactuar con:

- Repositorios
- APIs
- Workflows
- Herramientas externas
- Entornos de ejecución controlados

A medida que los agentes adquieren mayor autonomía, por ejemplo para modificar código, crear ramas, generar Pull Requests o ejecutar automatizaciones, resulta fundamental definir:

- **Qué pueden hacer**
- **A qué recursos pueden acceder**
- **Dónde se ejecutan**
- **Qué permisos tienen**
- **Qué acciones requieren revisión humana**

En GitHub, estas capacidades se apoyan principalmente en:

- GitHub APIs
- GitHub Actions
- Model Context Protocol (MCP)
- GitHub Agentic Workflows
- Permisos y tokens
- Controles de seguridad y revisión humana

---

# 1. Cómo interactúan los agentes con GitHub

Los agentes pueden trabajar dentro del contexto de un repositorio para completar tareas de desarrollo.

Por ejemplo, un agente puede:

1. Analizar y comprender un repositorio.
2. Planificar los cambios necesarios.
3. Crear o modificar archivos.
4. Trabajar sobre una rama.
5. Crear commits.
6. Preparar cambios para revisión mediante Pull Requests.
7. Utilizar workflows para validar los cambios.

Los agentes no evitan los mecanismos normales de GitHub. Utilizan las mismas estructuras que los desarrolladores:

- Repositories
- Branches
- Commits
- Pull Requests
- Issues
- GitHub Actions

Esto permite que las acciones realizadas por agentes permanezcan visibles, controladas y auditables.

---

# 2. GitHub APIs

Las APIs de GitHub permiten interactuar programáticamente con la plataforma.

Un agente puede utilizarlas para realizar operaciones como:

- Leer información del repositorio.
- Crear ramas.
- Crear commits.
- Modificar archivos.
- Abrir o actualizar Pull Requests.
- Consultar Issues.
- Ejecutar determinadas automatizaciones.

Las solicitudes a las APIs deben autenticarse mediante mecanismos como:

- Personal Access Token (PAT)
- GitHub App token
- `GITHUB_TOKEN`

De esta forma, las acciones están condicionadas por los permisos asociados al token utilizado.

La idea central es:

> **El agente no obtiene permisos propios. Actúa utilizando credenciales y permisos explícitamente controlados por GitHub.**

---

# 3. Flujo típico de cambios realizados por un agente

Cuando un agente modifica un repositorio, el flujo general sigue el modelo de desarrollo habitual de GitHub:

1. Seleccionar una rama base.
2. Crear una rama de trabajo.
3. Analizar el repositorio.
4. Crear o modificar archivos.
5. Generar commits.
6. Preparar los cambios para revisión.
7. Ejecutar workflows de validación.
8. Revisar los resultados.
9. Aprobar, solicitar modificaciones o descartar los cambios.

La utilización de ramas y Pull Requests permite mantener los cambios aislados antes de incorporarlos a la rama principal.

---

# 4. GitHub Actions como capa de ejecución

Los agentes necesitan un entorno donde ejecutar tareas.

GitHub proporciona estos entornos mediante **GitHub Actions**.

Un workflow tradicional se define mediante YAML y puede ejecutar jobs como respuesta a diferentes eventos.

Por ejemplo:

```yaml
on:
  push:
  pull_request:
  workflow_dispatch:
```

Los workflows pueden utilizarse para:

- Ejecutar pruebas.
- Validar cambios.
- Compilar aplicaciones.
- Analizar código.
- Ejecutar scripts.
- Automatizar tareas.
- Implementar aplicaciones.

Cada ejecución utiliza un runner y queda registrada mediante logs.

Por lo tanto, GitHub Actions funciona como una **capa de ejecución controlada** para muchas tareas automáticas y agentic.

---

# 5. GitHub Actions tradicional vs. GitHub Agentic Workflows

Los workflows tradicionales y los Agentic Workflows representan dos modelos diferentes de automatización.

## GitHub Actions tradicional

Normalmente es:

- Determinista.
- Definido en YAML.
- Basado en pasos explícitos.
- Diseñado indicando exactamente qué debe ejecutarse.

Ejemplo conceptual:

```yaml
steps:
  - checkout
  - install dependencies
  - run tests
  - build
```

El desarrollador define el procedimiento.

## GitHub Agentic Workflows

Los **GitHub Agentic Workflows** permiten describir principalmente el **resultado esperado** utilizando Markdown.

El agente interpreta esa intención y decide cómo completar la tarea dentro de los límites establecidos.

Esto permite automatizar tareas menos deterministas, como:

- Triage de Issues.
- Generación de reportes.
- Mantenimiento de documentación.
- Análisis de fallos de CI.
- Revisión de código.
- Mejoras controladas del repositorio.

Los Agentic Workflows **no reemplazan CI/CD ni GitHub Actions**.

Los complementan incorporando automatización basada en agentes.

---

# 6. Estructura de un GitHub Agentic Workflow

Un Agentic Workflow combina dos elementos principales:

1. **Frontmatter**
2. **Instrucciones en Markdown**

El frontmatter establece las restricciones y configuración del workflow.

El Markdown describe el objetivo.

Ejemplo conceptual:

```markdown
---
on:
  schedule: daily

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  create-issue:
    title-prefix: "[repo-status] "
    labels:
      - report

tools:
  github:
---

# Daily Repository Status Report

Create a daily report for maintainers.

Include:

- Recent activity
- Issues and Pull Requests
- Important commits
- Risks
- Recommended next steps

Keep the report concise and link to relevant issues and pull requests.
```

En este modelo:

**Frontmatter**

Define:

- Cuándo se ejecuta.
- Qué permisos posee.
- Qué herramientas puede utilizar.
- Qué outputs están permitidos.

**Markdown**

Define:

- El objetivo.
- Las instrucciones.
- El resultado esperado.

La idea fundamental es:

> **Markdown expresa la intención. El frontmatter establece los límites. El agente decide cómo alcanzar el objetivo dentro de esos límites.**

---

# 7. Model Context Protocol (MCP)

**Model Context Protocol (MCP)** es un protocolo que permite que aplicaciones y agentes de IA interactúen con herramientas y servicios mediante una interfaz estandarizada.

En lugar de crear una integración específica para cada herramienta, un cliente compatible con MCP puede conectarse a servidores MCP que exponen herramientas.

MCP proporciona un modelo común para:

- Descubrir herramientas.
- Invocar herramientas.
- Enviar solicitudes estructuradas.
- Recibir resultados estructurados.
- Integrar diferentes sistemas mediante un mecanismo consistente.

---

# 8. MCP Server

Un **MCP Server** expone herramientas que un cliente o agente puede utilizar.

Conceptualmente:

```text
Agente
  ↓
Cliente MCP
  ↓
MCP Server
  ↓
API / Servicio / Base de datos / Herramienta
```

El servidor se encuentra entre el agente y el sistema que proporciona la capacidad real.

Puede:

- Exponer herramientas.
- Recibir solicitudes.
- Validar parámetros.
- Ejecutar operaciones.
- Devolver resultados estructurados.

Un MCP Server puede ejecutarse:

- Localmente.
- Remotamente.
- Cerca de recursos locales.
- Como puente hacia APIs o servicios externos.

El **GitHub MCP Server**, por ejemplo, permite que clientes compatibles con MCP interactúen con capacidades de GitHub.

---

# 9. MCP Server local y remoto

## Local MCP Server

Se ejecuta en el equipo del desarrollador.

Puede ser útil cuando se necesita:

- Acceder a archivos locales.
- Interactuar con herramientas locales.
- Ejecutar servicios personalizados.
- Mantener control directo sobre el entorno.

Por ejemplo:

```text
http://localhost:3000
```

## Remote MCP Server

Se ejecuta como un servicio remoto accesible a través de la red.

Ventajas:

- Menor configuración local.
- Administración centralizada.
- Posibilidad de compartir el mismo servicio entre varios entornos.

Un agente puede utilizar herramientas expuestas por servidores MCP tanto locales como remotos dependiendo de su configuración y de las políticas aplicadas.

---

# 10. MCP Registry

Un **MCP Registry** funciona como un catálogo de servidores MCP.

Su función principal es facilitar:

- Descubrimiento.
- Distribución.
- Configuración.
- Administración de servidores MCP.

En lugar de configurar individualmente cada servidor, los desarrolladores pueden utilizar un registro para descubrir qué servidores están disponibles.

Esto resulta especialmente útil en organizaciones donde se quiere proporcionar un conjunto controlado de herramientas.

Conceptualmente:

```text
MCP Registry
    │
    ├── MCP Server A
    ├── MCP Server B
    ├── MCP Server C
    └── MCP Server D
```

El registry permite descubrir servidores.

Pero descubrir un servidor no necesariamente significa que esté permitido utilizarlo.

Para eso existe otra capa de gobierno.

---

# 11. MCP Allow List

Una **Allow List** determina qué servidores MCP están permitidos.

Esto es importante porque MCP puede ampliar considerablemente las capacidades de un agente.

Una organización puede decidir:

- Permitir MCP.
- Impedir MCP.
- Permitir cualquier MCP Server.
- Permitir únicamente servidores aprobados.

Por ejemplo:

```text
MCP Registry
      │
      ├── Server A ✅
      ├── Server B ✅
      ├── Server C ❌
      └── Server D ❌
```

La relación entre los conceptos es:

| Componente | Función |
|---|---|
| MCP Server | Expone herramientas |
| MCP Registry | Permite descubrir servidores |
| MCP Allow List | Determina qué servidores pueden utilizarse |

Por lo tanto:

> **El Server proporciona capacidades, el Registry permite descubrirlas y la Allow List controla cuáles están autorizadas.**

---

# 12. MCP y gobierno organizacional

MCP incrementa las capacidades disponibles para un agente, pero también aumenta la necesidad de gobierno.

Una estrategia empresarial puede utilizar:

```text
Organization
    ↓
MCP Registry
    ↓
Approved MCP Servers
    ↓
Allow List
    ↓
Developer / Agent
```

Esto permite combinar:

- Extensibilidad.
- Descubrimiento.
- Seguridad.
- Gobierno.
- Control organizacional.

El objetivo no es solamente conectar herramientas, sino controlar qué herramientas pueden ser utilizadas por los agentes.

---

# 13. Execution Context

El **Execution Context** define dónde opera un agente y qué puede hacer.

En GitHub puede estar determinado por:

- Repository.
- Branch.
- Workflow.
- Runner.
- Permissions.
- Tokens.
- Tools disponibles.

Estos elementos forman los límites dentro de los cuales funciona el agente.

Conceptualmente:

```text
Agent
  ↓
Repository
  ↓
Branch
  ↓
Workflow
  ↓
Runner
  ↓
Permissions
  ↓
Tools
```

Cada nivel reduce o controla el espacio de acción del agente.

---

# 14. Repository Scope

El repositorio constituye uno de los principales límites.

El agente trabaja dentro del contexto del repositorio al cual fue asignado.

Dentro de ese contexto puede interactuar, según sus permisos, con elementos como:

- Código.
- Issues.
- Pull Requests.
- Branches.
- Workflows.

El acceso adicional debe concederse explícitamente.

Esto contribuye a evitar que un agente tenga acceso indiscriminado a recursos que no necesita.

---

# 15. Branch Isolation

Los cambios realizados por agentes pueden mantenerse aislados mediante ramas.

Un flujo típico es:

```text
main
  │
  └── agent-branch
          │
          ├── modificaciones
          ├── commits
          └── Pull Request
```

Este modelo permite:

- Mantener `main` sin modificaciones directas.
- Ejecutar validaciones antes del merge.
- Revisar cambios.
- Solicitar correcciones.
- Rechazar cambios.
- Mantener trazabilidad.

La rama funciona así como una barrera adicional entre la acción autónoma del agente y el código principal.

---

# 16. Workflow Boundaries

Los workflows también funcionan como límites de ejecución.

Un workflow determina:

- Qué evento inicia la ejecución.
- Qué runner se utiliza.
- Qué jobs se ejecutan.
- Qué comandos pueden ejecutarse.
- Qué permisos recibe el token.
- Qué secretos pueden utilizarse.

Esto permite crear un entorno controlado y reproducible para las tareas del agente.

---

# 17. Permission Boundaries

Los permisos determinan qué acciones están autorizadas.

Por ejemplo:

```yaml
permissions:
  contents: read
  issues: read
  pull-requests: read
```

La recomendación general es utilizar el principio de:

> **Least Privilege**

Es decir, proporcionar únicamente los permisos necesarios para completar la tarea.

Los permisos pueden controlar capacidades como:

- Leer contenido.
- Modificar contenido.
- Crear Pull Requests.
- Interactuar con Issues.
- Utilizar determinados recursos.
- Ejecutar operaciones específicas del workflow.

---

# 18. Guardrails en GitHub Agentic Workflows

Los Agentic Workflows incorporan un modelo basado en múltiples capas de protección.

Entre los mecanismos importantes se encuentran:

- Tokens de solo lectura por defecto.
- Safe outputs.
- Separación entre razonamiento y acciones con efectos.
- Ejecución aislada.
- Control de acceso a herramientas.
- Controles de red.
- Revisión de outputs antes de determinadas operaciones.
- Revisión humana.

La filosofía general es:

```text
Agent proposes
      ↓
Guardrails validate
      ↓
GitHub applies controlled action
      ↓
Human reviews
```

La autonomía no significa ausencia de control.

---

# 19. Safe Outputs

Los **Safe Outputs** permiten controlar acciones generadas por el agente.

En lugar de proporcionar acceso de escritura irrestricto, puede definirse qué tipos de resultados están permitidos.

Por ejemplo:

```yaml
safe-outputs:
  create-issue:
    title-prefix: "[repo-status] "
    labels:
      - report
```

En este ejemplo el agente puede proponer una acción específica dentro de límites previamente establecidos.

Esto reduce la necesidad de otorgar permisos amplios al proceso del agente.

---

# 20. Protecciones de ejecución

Los sistemas basados en agentes deben considerar situaciones donde la ejecución no funciona según lo esperado.

Entre las principales estrategias se encuentran:

## Error handling

Los workflows deben detectar y registrar errores.

El objetivo es evitar que una tarea continúe después de producirse un fallo crítico.

---

## Retries

Algunos errores son temporales.

Se pueden utilizar estrategias de retry para:

- Errores de red.
- Servicios temporalmente no disponibles.
- Rate limits.
- Fallos transitorios.

Los retries deben estar limitados para evitar loops infinitos.

---

## Rollback

Si un agente genera cambios incorrectos, el modelo basado en Git permite revertirlos.

Las principales estrategias incluyen:

- Descartar una rama.
- Cerrar un Pull Request.
- Revertir commits.

La combinación de ramas y Pull Requests constituye una protección natural frente a cambios incorrectos.

---

## Escalation

Cuando el agente no puede completar una tarea con suficiente seguridad, debe existir un mecanismo para transferir la decisión a una persona.

Por ejemplo:

- Requerir revisión del Pull Request.
- Asignar reviewers.
- Notificar maintainers.
- Solicitar aprobación antes de continuar.

---

# 21. Traceability y Accountability

Las acciones realizadas por agentes deben poder auditarse.

GitHub proporciona diferentes mecanismos para ello:

- Workflow logs.
- Commit history.
- Branch history.
- Pull Request discussions.
- Review history.

Esto permite responder preguntas como:

- ¿Qué hizo el agente?
- ¿Cuándo lo hizo?
- ¿Qué archivos modificó?
- ¿Qué workflow se ejecutó?
- ¿Quién revisó el resultado?
- ¿Qué cambios fueron finalmente aceptados?

La trazabilidad es fundamental cuando aumenta el nivel de autonomía.

---

# 22. Environment Protections

Cuando los cambios generados por agentes participan en procesos de deployment, los **GitHub Environments** pueden agregar controles adicionales.

Los environments pueden utilizarse para:

- Exigir aprobaciones.
- Restringir secretos.
- Controlar deployments.
- Limitar destinos de implementación.

Esto permite separar:

```text
Agent genera cambio
        ↓
Pull Request
        ↓
Validación
        ↓
Approval
        ↓
Deployment
```

La generación automática de código no implica necesariamente una implementación automática en producción.

---

# 23. Human in the Loop

Un principio central del modelo es mantener puntos explícitos de intervención humana.

La persona puede:

- Revisar modificaciones.
- Solicitar cambios.
- Aprobar Pull Requests.
- Rechazar cambios.
- Autorizar deployments.
- Modificar instrucciones.

Por lo tanto, la arquitectura combina:

```text
Automation
    +
AI Agent
    +
Guardrails
    +
Human Review
```

La revisión humana sigue siendo especialmente importante antes de operaciones con impacto significativo.

---

# 24. Flujo End-to-End de un agente en GitHub

Un flujo completo puede verse de la siguiente manera:

```text
Task
  ↓
Agent
  ↓
Repository Context
  ↓
Repository Analysis
  ↓
Plan
  ↓
GitHub APIs / MCP Tools
  ↓
Working Branch
  ↓
Changes
  ↓
Commit
  ↓
GitHub Actions
  ↓
Validation
  ↓
Pull Request
  ↓
Human Review
  ↓
Merge
```

Durante todo el proceso existen controles sobre:

- Scope.
- Permissions.
- Tools.
- Execution.
- Network.
- Outputs.
- Review.

---

# 25. GitHub Agentic Workflows dentro del modelo general

Los Agentic Workflows agregan una nueva capa sobre el modelo tradicional de automatización.

El cambio conceptual puede resumirse así:

### Automatización tradicional

```text
Developer
   ↓
YAML
   ↓
Explicit Steps
   ↓
GitHub Actions
```

### Automatización agentic

```text
Developer
   ↓
Intent in Markdown
   ↓
Coding Agent
   ↓
Tools
   ↓
GitHub Actions
   ↓
Controlled Output
```

El desarrollador pasa de definir únicamente **cómo ejecutar cada paso** a poder definir también **qué resultado desea obtener**.

El agente determina cómo alcanzar ese resultado respetando los límites establecidos.

---

# 26. Conceptos principales

| Concepto | Función |
|---|---|
| GitHub API | Permite interactuar programáticamente con GitHub |
| GitHub Actions | Proporciona automatización y entornos de ejecución |
| Agent | Analiza una tarea y decide cómo ejecutarla |
| Agentic Workflow | Describe resultados utilizando Markdown y guardrails |
| MCP | Estandariza la conexión entre agentes y herramientas |
| MCP Server | Expone herramientas al agente |
| MCP Registry | Facilita el descubrimiento de MCP Servers |
| MCP Allow List | Define qué MCP Servers están autorizados |
| Repository Scope | Limita dónde puede operar el agente |
| Branch Isolation | Mantiene los cambios separados de la rama principal |
| Permissions | Controlan qué acciones puede realizar |
| Safe Outputs | Restringen los efectos que puede producir el agente |
| Pull Request | Proporciona un punto de revisión |
| Human Review | Mantiene decisiones importantes bajo control humano |

---

# 27. Idea central

Los agentes en GitHub no deben entenderse como procesos que operan libremente sobre un repositorio.

Funcionan dentro de una arquitectura compuesta por:

```text
Agent
  ↓
Tools / MCP
  ↓
GitHub APIs
  ↓
Repository + Branch
  ↓
GitHub Actions
  ↓
Permissions + Guardrails
  ↓
Pull Request
  ↓
Human Review
```

La capacidad de un agente surge de las herramientas que puede utilizar.

Su seguridad surge de los límites que se aplican a esas herramientas y al entorno de ejecución.

GitHub combina **APIs, MCP, workflows, permisos, aislamiento, safe outputs y revisión humana** para permitir automatizaciones basadas en agentes manteniendo control sobre sus acciones.

---

## Resumen

Los conceptos fundamentales son:

- **GitHub APIs** permiten que los agentes interactúen con la plataforma.
- **GitHub Actions** proporciona la infraestructura donde se ejecutan automatizaciones.
- **Agentic Workflows** permiten describir resultados en Markdown en lugar de programar exclusivamente pasos deterministas.
- **MCP** proporciona una forma estandarizada de conectar agentes con herramientas.
- **MCP Servers** exponen capacidades.
- **MCP Registries** permiten descubrir servidores.
- **Allow Lists** controlan qué servidores pueden utilizarse.
- **Execution Context** define dónde y bajo qué condiciones opera un agente.
- **Repository Scope** y **Branch Isolation** limitan el alcance de los cambios.
- **Permissions** aplican el principio de mínimo privilegio.
- **Guardrails y Safe Outputs** limitan las acciones autónomas.
- **Pull Requests y revisión humana** mantienen control sobre los cambios antes de incorporarlos al código principal.

> **Los agentes amplían la automatización de GitHub, pero siguen operando dentro de límites definidos por repositorios, herramientas, workflows, permisos y procesos de revisión.**
