# Creación de un equipo de agentes con GitHub Copilot CLI

# Objetivos

En este laboratorio aprenderás a utilizar GitHub Copilot CLI para coordinar un equipo de agentes personalizados que planificará, diseñará, implementará y validará el dashboard Project Pulse.

* Utilizar GitHub Copilot CLI desde GitHub Codespaces.
* Inspeccionar agentes personalizados almacenados en `.github/agents/`.
* Utilizar un agente Orchestrator para coordinar agentes especializados.
* Utilizar un agente Planner para crear un plan de implementación.
* Delegar el diseño del dashboard a un agente Designer.
* Delegar la implementación del dashboard a un agente Coder.
* Crear una aplicación estática con HTML, CSS y JSON.
* Crear una configuración de ejecución para Visual Studio Code.
* Validar el dashboard desde GitHub Codespaces.
* Documentar el trabajo realizado por los diferentes agentes.
* Crear commits y publicar los cambios en el repositorio.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Copilot.
* Acceso a GitHub Codespaces.
* Conocimientos básicos de repositorios, commits y Markdown.
* Conocimientos básicos sobre el uso de una terminal integrada.
* Tiempo estimado: menos de 45 minutos.

# Pasos

## Crear el repositorio del laboratorio

* [BROWSER] [https://github.com/skills/agent-orchestration-build-your-ai-dream-team](https://github.com/skills/agent-orchestration-build-your-ai-dream-team)
  * Iniciar sesión con una cuenta de GitHub.
  * Desplazarse hasta la sección del ejercicio.
  * [BUTTON] Copy Exercise
  * Mantener la configuración predeterminada.
  * [BUTTON] Create repository
  * Esperar entre 15 y 20 segundos.
  * Actualizar la página.
  * [BUTTON] Go to exercise
  * Hacer clic con el botón derecho en `Open in GitHub Codespaces`.
  * Abrir el enlace en una pestaña nueva.
  * Confirmar que el repositorio seleccionado sea la copia personal del ejercicio.
  * [BUTTON] Create Codespace

* Esperar hasta que Visual Studio Code termine de cargar en el navegador.
  * [BUTTON] I Trust, si aparece la solicitud de confianza.
  * Verificar que la terminal esté disponible en la parte inferior.

> La copia personal del repositorio se crea dentro de la cuenta del alumno. El repositorio original pertenece a la organización `github`.

## Iniciar GitHub Copilot CLI

* [CONSOLE]

```bash
copilot --allow-all --enable-all-github-mcp-tools
```

* Esperar hasta que se inicie el modo interactivo de GitHub Copilot CLI.
* Si se solicita autenticación, ejecutar:

```text
/login
```

* Completar el proceso de inicio de sesión con la cuenta de GitHub.

> El comando inicia GitHub Copilot CLI con los permisos requeridos por el laboratorio y habilita las herramientas de GitHub MCP disponibles en el entorno.

* Para copiar y pegar contenido en la terminal de GitHub Codespaces:
  * Utilizar los atajos estándar del navegador.
  * Seleccionar texto para copiarlo automáticamente, si el navegador lo permite.
  * Hacer clic con el botón derecho en la terminal para pegar.

## Inspeccionar el equipo de agentes personalizados

Los agentes personalizados utilizados en el laboratorio se encuentran en el siguiente directorio:

```text
.github/agents/
```

El equipo está compuesto por:

* `Orchestrator`: coordina el trabajo de los demás agentes.
* `Planner`: crea el plan de implementación y determina las dependencias.
* `Designer`: define la estructura visual y la experiencia del dashboard.
* `Coder`: implementa los archivos de la aplicación.

* [COPILOT CLI] Ingresar el siguiente prompt:

```text
Inspect .github/agents/ and summarize the custom agent team I will use to build Mona's Project Pulse dashboard.
```

* Revisar la respuesta generada por GitHub Copilot CLI.

* [FILE] `docs/agent-team.md`
  * Crear o actualizar el archivo.
  * Documentar los agentes `Orchestrator`, `Planner`, `Designer` y `Coder`.
  * Incluir el modelo asignado a cada agente.
  * Describir la responsabilidad de cada agente.
  * Indicar el archivo correspondiente dentro de `.github/agents/`.
  * Explicar cómo trabajarán juntos para construir Project Pulse.
  * Guardar el archivo.

> Los modelos asignados deben obtenerse de las definiciones existentes en `.github/agents/`. No deben completarse manualmente ni suponerse.

## Publicar la documentación del equipo

* [COPILOT CLI] Ingresar el siguiente prompt:

```text
Stage docs/agent-team.md.

Commit it with the message "Document the Project Pulse agent team".

Push the commit.
```

* Permitir la ejecución de los comandos, si GitHub Copilot CLI solicita autorización.
* Verificar que el archivo haya sido agregado al área de preparación.
* Verificar que se haya creado el commit.
* Verificar que el commit se haya publicado en el repositorio remoto.

## Crear el plan de implementación

El agente Orchestrator coordinará al agente Planner para crear un plan antes de comenzar la implementación.

El plan debe contemplar los siguientes archivos:

```text
app/index.html
app/styles.css
app/project-data.json
.vscode/launch.json
```

* [CONSOLE] Si GitHub Copilot CLI no se encuentra en ejecución:

```bash
copilot --allow-all --enable-all-github-mcp-tools
```

* [COPILOT CLI]

```text
/agent
```

* Seleccionar el agente `Orchestrator`.

* [COPILOT CLI] Ingresar el siguiente prompt:

```text
Ask the Planner to create an implementation plan for the Project Pulse dashboard.

Save the plan in docs/project-pulse-plan.md.

Include app/index.html, app/styles.css, app/project-data.json,
and .vscode/launch.json in the file assignments.

Include Designer and Coder responsibilities.

Include dependencies.

Include parallel work decisions.

Include validation expectations.
```

* Esperar hasta que el Orchestrator coordine la creación del plan.

* [FILE] `docs/project-pulse-plan.md`
  * Verificar que incluya el objetivo de Project Pulse.
  * Verificar que incluya las fases de implementación.
  * Verificar que asigne los archivos correspondientes.
  * Verificar que describa las responsabilidades del Designer.
  * Verificar que describa las responsabilidades del Coder.
  * Verificar que identifique las dependencias entre los archivos.
  * Verificar que explique qué tareas pueden realizarse en paralelo.
  * Verificar que incluya las expectativas de validación.

## Publicar el plan de implementación

* [COPILOT CLI] Ingresar el siguiente prompt:

```text
Stage docs/project-pulse-plan.md.

Commit it with the message "Plan the Project Pulse dashboard".

Push the commit.
```

* Permitir la ejecución de los comandos, si se solicita autorización.
* Verificar que se haya creado el commit.
* Verificar que el commit se haya publicado en el repositorio remoto.

## Diseñar y construir Project Pulse

En esta etapa, el Orchestrator utilizará el plan para delegar el trabajo a los agentes Designer y Coder.

El Designer definirá la estructura y la experiencia visual del dashboard.

El Coder implementará los archivos estáticos de la aplicación y la configuración para ejecutarla.

* [COPILOT CLI]

```text
/agent
```

* Seleccionar el agente `Orchestrator`.

* [COPILOT CLI] Ingresar el siguiente prompt:

```text
Use the plan in docs/project-pulse-plan.md.

Ask the Designer to create the dashboard design and structure.

Ask the Coder to implement app/index.html, app/styles.css, and app/project-data.json
based on the Designer's work.

Have the Coder create .vscode/launch.json with a configuration named "Run Project Pulse Dashboard".
```

* Esperar hasta que los agentes completen las tareas delegadas.

* [FILE] Verificar que se hayan creado los siguientes archivos:

```text
app/index.html
app/styles.css
app/project-data.json
.vscode/launch.json
```

* [FILE] `app/index.html`
  * Verificar que contenga la estructura del dashboard.

* [FILE] `app/styles.css`
  * Verificar que contenga los estilos visuales del dashboard.

* [FILE] `app/project-data.json`
  * Verificar que contenga los datos utilizados por el dashboard.

* [FILE] `.vscode/launch.json`
  * Verificar que incluya una configuración denominada:

```text
Run Project Pulse Dashboard
```

## Publicar el dashboard

* [COPILOT CLI] Ingresar el siguiente prompt:

```text
Stage app/ and .vscode/launch.json.

Commit it with the message "Build Project Pulse dashboard".

Push the commit.
```

* Permitir la ejecución de los comandos, si se solicita autorización.
* Verificar que los archivos hayan sido agregados al área de preparación.
* Verificar que se haya creado el commit.
* Verificar que el commit se haya publicado en el repositorio remoto.

## Ejecutar Project Pulse

* [BROWSER] GitHub Codespaces
  * [BUTTON] Run and Debug
  * Seleccionar `Run Project Pulse Dashboard`.
  * [BUTTON] Start Debugging
  * Esperar hasta que se inicie el dashboard.
  * Abrir el puerto reenviado, si GitHub Codespaces muestra una notificación.
  * Verificar que Project Pulse se muestre correctamente.

* Verificar que:
  * El archivo `app/index.html` se cargue sin errores.
  * Los estilos de `app/styles.css` se apliquen correctamente.
  * Los datos de `app/project-data.json` estén disponibles para la aplicación.
  * La estructura visual coincida con el trabajo definido por el Designer.
  * La configuración `Run Project Pulse Dashboard` permita iniciar la aplicación.

> Si la aplicación no se inicia, revisar el contenido de `.vscode/launch.json` y verificar que la configuración coincida con la estructura real del repositorio.

## Validar el dashboard y crear la entrega final

El Orchestrator debe cerrar el ciclo de trabajo explicando:

* Qué agentes participaron.
* Cómo se utilizó el plan.
* Qué aportó el Designer.
* Qué implementó el Coder.
* Qué validaciones se realizaron.
* Cuál es el resultado final.

* [CONSOLE] Si GitHub Copilot CLI no se encuentra en ejecución:

```bash
copilot --allow-all --enable-all-github-mcp-tools
```

* [COPILOT CLI]

```text
/agent
```

* Seleccionar el agente `Orchestrator`.

* [COPILOT CLI] Ingresar el siguiente prompt:

```text
Review docs/agent-team.md, docs/project-pulse-plan.md, the files in app/,
and .vscode/launch.json.

Validate the Project Pulse dashboard.

Write a final handoff in docs/final-handoff.md.

Include these exact agent names: Orchestrator, Planner, Designer, and Coder.

Include these exact file paths: app/index.html, app/styles.css,
and app/project-data.json.

Include the exact launch name "Run Project Pulse Dashboard".

Include the exact launch file path .vscode/launch.json.

Include a section heading that contains the lowercase word "validation".

Include a section heading that contains the lowercase word "handoff".
```

* Esperar hasta que el Orchestrator complete la validación y genere la documentación final.

* [FILE] `docs/final-handoff.md`
  * Verificar que incluya los agentes `Orchestrator`, `Planner`, `Designer` y `Coder`.
  * Verificar que describa el resultado final de Project Pulse.
  * Verificar que mencione `app/index.html`.
  * Verificar que mencione `app/styles.css`.
  * Verificar que mencione `app/project-data.json`.
  * Verificar que mencione `.vscode/launch.json`.
  * Verificar que incluya la configuración `Run Project Pulse Dashboard`.
  * Verificar que contenga un encabezado con la palabra `validation` en minúsculas.
  * Verificar que contenga un encabezado con la palabra `handoff` en minúsculas.
  * Verificar que incluya los próximos pasos o las limitaciones identificadas.

## Publicar la entrega final

* [COPILOT CLI] Ingresar el siguiente prompt:

```text
Stage docs/final-handoff.md.

Commit it with the message "Validate and hand off Project Pulse".

Push the commit.
```

* Permitir la ejecución de los comandos, si se solicita autorización.
* Verificar que se haya creado el commit.
* Verificar que el commit se haya publicado en el repositorio remoto.

## Verificar el resultado en GitHub

* [BROWSER] Repositorio del laboratorio
  * Actualizar la página.
  * Revisar el historial de commits.
  * Verificar la existencia de los siguientes commits:
    * `Document the Project Pulse agent team`
    * `Plan the Project Pulse dashboard`
    * `Build Project Pulse dashboard`
    * `Validate and hand off Project Pulse`

* Verificar que el repositorio contenga los siguientes archivos:

```text
.github/agents/
.vscode/launch.json
app/index.html
app/styles.css
app/project-data.json
docs/agent-team.md
docs/project-pulse-plan.md
docs/final-handoff.md
```

## Explorar las funciones de GitHub Copilot CLI

* [COPILOT CLI] Explorar los siguientes comandos:

```text
/agent
```

* Permite seleccionar el agente que atenderá la solicitud.

```text
/model
```

* Permite revisar o seleccionar el modelo utilizado.

```text
/tasks
```

* Permite revisar las tareas administradas durante la sesión.

```text
/fleet
```

* Permite explorar flujos de trabajo con agentes y subagentes.

> La disponibilidad y el comportamiento exacto de estos comandos pueden depender de la versión de GitHub Copilot CLI y de la configuración habilitada en el entorno.

# Conclusiones

En este laboratorio aprendimos a:

* Utilizar GitHub Copilot CLI desde GitHub Codespaces.
* Inspeccionar agentes personalizados definidos en `.github/agents/`.
* Coordinar agentes especializados mediante un agente Orchestrator.
* Utilizar un agente Planner para crear un plan antes de implementar.
* Separar las responsabilidades de planificación, diseño y desarrollo.
* Delegar el diseño del dashboard a un agente Designer.
* Delegar la implementación a un agente Coder.
* Crear una aplicación estática con HTML, CSS y JSON.
* Crear una configuración de ejecución en `.vscode/launch.json`.
* Ejecutar y validar Project Pulse desde GitHub Codespaces.
* Documentar el equipo de agentes y el plan de implementación.
* Generar una entrega final con los resultados de la validación.
* Crear commits y publicar los cambios en el repositorio remoto.
* Aplicar un patrón de orquestación reutilizable en otros proyectos de software.
