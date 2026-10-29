# Hola mundo con GitHub Agentic Workflows

# Objetivos

En este laboratorio aprenderás a crear y ejecutar tu primer agente utilizando GitHub Agentic Workflows.

* Crear un repositorio en GitHub.
* Abrir el repositorio utilizando GitHub Codespaces.
* Instalar GitHub Agentic Workflows.
* Crear un Personal Access Token para utilizar GitHub Copilot.
* Configurar el token como secreto del repositorio.
* Crear un Agentic Workflow mediante un archivo Markdown.
* Compilar el archivo Markdown en un workflow de GitHub Actions.
* Ejecutar el agente manualmente.
* Verificar el resultado generado por el agente.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Copilot.
* Acceso a GitHub Codespaces.
* Conocimientos básicos de Markdown.
* Tiempo estimado: 20 a 30 minutos.

# Pasos

## Crear un repositorio

* [BROWSER] GitHub
  * Iniciar sesión con una cuenta de GitHub.
  * [BUTTON] New
  * En `Repository name`, ingresar:

```text
hello-agentic-workflows
```

  * Seleccionar `Public`.
  * Seleccionar `Add a README file`.
  * [BUTTON] Create repository

* Esperar hasta que se muestre el nuevo repositorio.

## Abrir el repositorio en GitHub Codespaces

* [BROWSER] Repositorio `hello-agentic-workflows`
  * [BUTTON] Code
  * Seleccionar la pestaña `Codespaces`.
  * [BUTTON] Create codespace on main

* Esperar hasta que GitHub Codespaces termine de iniciar.
* Se abrirá Visual Studio Code directamente en el navegador.
* Si aparece una solicitud de confianza:
  * [BUTTON] I Trust
* Verificar que en el panel izquierdo aparezcan los archivos del repositorio.
* Verificar que exista el archivo:

```text
README.md
```

* Verificar que la terminal esté disponible en la parte inferior de Visual Studio Code.

> [!NOTE]
> GitHub Codespaces proporciona un entorno de desarrollo dentro del navegador. Los comandos de los siguientes pasos se ejecutarán desde la terminal integrada de este entorno.

## Modificar el archivo README

* [FILE] `README.md`
* Reemplazar su contenido por:

```markdown
# Hello Agentic Workflows

Este repositorio se utiliza para aprender cómo funcionan los GitHub Agentic Workflows.

## Contenido

El repositorio contiene un ejemplo sencillo para aprender a ejecutar un agente con GitHub Actions.

## Objetivo

El objetivo es crear nuestro primer agente y permitir que analice este archivo.
```

* Guardar el archivo.

## Verificar GitHub CLI

* [CONSOLE]

```bash
gh --version
```

* Verificar que se muestre la versión instalada de GitHub CLI.

* [CONSOLE]

```bash
gh auth status
```

* Verificar que GitHub CLI indique que existe una sesión autenticada en GitHub.

> [!NOTE]
> GitHub Agentic Workflows utiliza GitHub CLI para realizar varias de sus operaciones.

## Instalar GitHub Agentic Workflows

* [CONSOLE]

```bash
gh extension install github/gh-aw
```

* Esperar hasta que finalice la instalación.

* [CONSOLE]

```bash
gh aw version
```

* Verificar que se muestre la versión instalada.

> [!NOTE]
> `gh-aw` es la extensión de GitHub CLI utilizada para trabajar con GitHub Agentic Workflows.

## Crear el token de GitHub Copilot

El agente necesita autenticarse para utilizar GitHub Copilot durante la ejecución del workflow.

Para este laboratorio se utilizará un Fine-grained Personal Access Token.

* [BROWSER] GitHub
  * Hacer clic en la imagen de perfil ubicada en la esquina superior derecha.
  * [BUTTON] Settings
  * En el menú lateral, desplazarse hasta la parte inferior.
  * [BUTTON] Developer settings
  * [BUTTON] Personal access tokens
  * [BUTTON] Fine-grained tokens
  * [BUTTON] Generate new token

* En `Token name`, ingresar:

```text
COPILOT_GITHUB_TOKEN
```

* En `Description`, ingresar:

```text
GitHub Agentic Workflows - Copilot engine authentication
```

* En `Expiration`, seleccionar el período de validez del token.

* En `Resource owner`, seleccionar la cuenta utilizada para realizar el laboratorio.

* Configurar el acceso correspondiente al repositorio del laboratorio.

* En la sección `Permissions`, abrir:

```text
Account permissions
```

* [BUTTON] Add permissions

* Buscar y agregar:

```text
Copilot Requests
```

* Configurar el acceso requerido para `Copilot Requests`.

* [BUTTON] Generate token

* Copiar el valor del token generado.

> [!WARNING]
> El token es una credencial y debe tratarse como una contraseña.
>
> No guardarlo en archivos del repositorio, código fuente, Issues, Pull Requests ni mensajes de Copilot Chat.
>
> GitHub solamente mostrará el valor completo del token al momento de crearlo.

## Crear el secreto del repositorio

Ahora el token se almacenará como un secreto de GitHub Actions.

* [BROWSER] Repositorio `hello-agentic-workflows`
  * [BUTTON] Settings
  * En el menú lateral, seleccionar `Secrets and variables`.
  * [BUTTON] Actions
  * [BUTTON] New repository secret

* En `Name`, ingresar:

```text
COPILOT_GITHUB_TOKEN
```

* En `Secret`, pegar el token generado anteriormente.
* [BUTTON] Add secret

* Verificar que aparezca el secreto:

```text
COPILOT_GITHUB_TOKEN
```

> [!NOTE]
> El token queda almacenado como un secreto del repositorio y puede ser utilizado durante la ejecución de GitHub Actions sin incluir su valor en los archivos del repositorio.

## Configurar los permisos de GitHub Actions

El agente generará un Issue como resultado de su ejecución.

* [BROWSER] Repositorio `hello-agentic-workflows`
  * [BUTTON] Settings
  * [BUTTON] Actions
  * [BUTTON] General

* Buscar la sección `Workflow permissions`.
* Seleccionar:

```text
Read and write permissions
```

* [BUTTON] Save

## Crear el primer Agentic Workflow

Volver a GitHub Codespaces.

* Crear la siguiente carpeta si todavía no existe:

```text
.github/workflows
```

* Dentro de `.github/workflows`, crear el archivo:

```text
hello-agent.md
```

La estructura del repositorio debería ser similar a:

```text
hello-agentic-workflows
│
├── .github
│   └── workflows
│       └── hello-agent.md
│
└── README.md
```

* [FILE] `.github/workflows/hello-agent.md`
* Agregar el siguiente contenido:

```markdown
---
name: hello-agent

on:
  workflow_dispatch:

safe-outputs:
  create-issue:
---

# Hello Agent

Read the `README.md` file from this repository.

Create an issue with the title:

Hello from my Agentic Workflow

In the body of the issue:

- Write a short summary of the repository.
- Include three bullet points describing the repository.
- Finish with the sentence: "Generated by my first GitHub Agentic Workflow."
```

* Guardar el archivo.

## Comprender la definición del agente

El archivo creado contiene dos partes principales.

### Frontmatter

La sección delimitada por `---` configura el Agentic Workflow.

```yaml
---
name: hello-agent

on:
  workflow_dispatch:

safe-outputs:
  create-issue:
---
```

`name` define el nombre del workflow.

`workflow_dispatch` permite ejecutarlo manualmente desde GitHub Actions.

`safe-outputs` define las operaciones de escritura que el agente tiene permitido realizar.

En este ejemplo:

```yaml
safe-outputs:
  create-issue:
```

permite que el agente genere un Issue como resultado de su ejecución.

### Instrucciones del agente

Todo lo que aparece después del frontmatter son instrucciones escritas en lenguaje natural.

Por ejemplo:

```text
Read the README.md file from this repository.

Create an issue...
```

Estas instrucciones indican al agente qué información debe analizar y qué resultado debe generar.

## Compilar el Agentic Workflow

El archivo Markdown todavía no es un workflow ejecutable directamente por GitHub Actions.

GitHub Agentic Workflows debe compilarlo.

* [CONSOLE]

```bash
gh aw compile .github/workflows/hello-agent.md
```

* Esperar hasta que la compilación finalice correctamente.
* Verificar que se haya creado:

```text
.github/workflows/hello-agent.lock.yml
```

Ahora existirán dos archivos:

```text
.github/workflows/hello-agent.md
.github/workflows/hello-agent.lock.yml
```

El archivo:

```text
hello-agent.md
```

contiene la configuración y las instrucciones del agente.

El archivo:

```text
hello-agent.lock.yml
```

es el workflow generado que podrá ejecutar GitHub Actions.

El proceso puede resumirse como:

```text
hello-agent.md
      ↓
gh aw compile
      ↓
hello-agent.lock.yml
      ↓
GitHub Actions
      ↓
Agente
```

> [!IMPORTANT]
> No editar manualmente el archivo `.lock.yml`.
>
> Los cambios se realizan sobre el archivo `.md` y luego se vuelve a ejecutar `gh aw compile`.

## Verificar los archivos modificados

* [CONSOLE]

```bash
git status
```

* Verificar que aparezcan los archivos creados o modificados.

## Guardar los cambios en Git

* [CONSOLE]

```bash
git add .
```

* [CONSOLE]

```bash
git commit -m "Create my first agentic workflow"
```

* [CONSOLE]

```bash
git push
```

* Esperar hasta que finalice el envío de los archivos al repositorio de GitHub.

## Verificar el workflow en GitHub

* [BROWSER] Repositorio `hello-agentic-workflows`
  * Actualizar la página.
  * [BUTTON] Actions
  * Buscar el workflow `hello-agent`.
  * Seleccionar `hello-agent`.

## Ejecutar el agente

* [BROWSER] GitHub Actions
  * Seleccionar `hello-agent`.
  * [BUTTON] Run workflow
  * Verificar que la rama seleccionada sea `main`.
  * [BUTTON] Run workflow

* Esperar hasta que comience la ejecución.
* Abrir la ejecución para observar su progreso.
* Esperar hasta que la ejecución finalice correctamente.

> [!NOTE]
> La ejecución del agente puede tardar algunos minutos.

## Revisar el resultado generado por el agente

* [BROWSER] Repositorio `hello-agentic-workflows`
  * [BUTTON] Issues

* Verificar que exista un nuevo Issue con el título:

```text
Hello from my Agentic Workflow
```

* Abrir el Issue.
* Verificar que el agente haya:
  * leído el archivo `README.md`;
  * generado un resumen del repositorio;
  * generado tres puntos sobre su contenido;
  * incluido la frase:

```text
Generated by my first GitHub Agentic Workflow.
```

## ¿Qué ocurrió?

Durante la ejecución:

1. GitHub Actions inició el workflow.
2. El agente recibió las instrucciones definidas en `hello-agent.md`.
3. El agente leyó el contenido de `README.md`.
4. GitHub Copilot procesó la información y generó el contenido solicitado.
5. `safe-outputs` permitió crear el resultado de forma controlada.
6. El agente creó un Issue en el repositorio.

El archivo Markdown define **qué queremos que haga el agente**.

El archivo `.lock.yml` contiene **el workflow ejecutable por GitHub Actions**.

GitHub Copilot funciona como **motor de inteligencia artificial**.

GitHub Actions proporciona **el entorno donde se ejecuta el agente**.

Safe Outputs controla **las operaciones que el agente puede realizar como resultado de su ejecución**.

# Conclusiones

En este laboratorio aprendimos a:

* Crear un repositorio en GitHub.
* Utilizar GitHub Codespaces como entorno de trabajo.
* Instalar GitHub Agentic Workflows.
* Crear un Fine-grained Personal Access Token para GitHub Copilot.
* Guardar el token como secreto de GitHub Actions.
* Crear un agente mediante un archivo Markdown.
* Utilizar lenguaje natural para definir el comportamiento del agente.
* Compilar el Agentic Workflow.
* Ejecutar el agente mediante GitHub Actions.
* Utilizar Safe Outputs.
* Generar automáticamente un Issue como resultado de la ejecución del agente.

El flujo básico de GitHub Agentic Workflows puede resumirse como:

```text
Instrucciones en Markdown
        ↓
Compilación
        ↓
GitHub Actions
        ↓
Agente de IA
        ↓
Safe Output
        ↓
Resultado
```
