# Automatización del sitio de Mona con GitHub Agentic Workflows

# Objetivos

En este laboratorio aprenderás a crear un agente mediante GitHub Agentic Workflows para actualizar automáticamente el contenido de un sitio web.

* Inicializar Agentic Workflows en un repositorio.
* Configurar los permisos y secretos necesarios.
* Definir las instrucciones del agente en un archivo Markdown.
* Compilar el archivo Markdown en un workflow YAML de GitHub Actions.
* Ejecutar el agente para analizar información y modificar archivos.
* Generar un Pull Request con los cambios propuestos.
* Utilizar Safe Outputs para evitar modificaciones directas en la rama `main`.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Copilot.
* Acceso a GitHub Codespaces.
* Conocimientos básicos de repositorios, ramas, commits y Pull Requests.
* Conocimientos básicos de Markdown.
* Conocimientos básicos sobre el uso de una terminal integrada.
* Tiempo estimado: 45 minutos.

# Pasos

## Crear el repositorio del laboratorio

* [BROWSER] https://github.com/skills/agentic-workflows-that-read-the-room
  * Iniciar sesión con una cuenta de GitHub.
  * [BUTTON] Copy Exercise
  * Mantener la configuración predeterminada.
  * [BUTTON] Create repository
  * Esperar entre 15 y 20 segundos.
  * Actualizar la página.
  * [BUTTON] Go to exercise
  * [BUTTON] Open in GitHub Codespaces
  * Confirmar que el repositorio seleccionado sea la copia personal.
  * [BUTTON] Create Codespace

* Esperar hasta que Visual Studio Code termine de cargar en el navegador.
  * [BUTTON] I Trust, si aparece la solicitud de confianza.
  * Verificar que la terminal esté disponible en la parte inferior.

## Ejecutar el sitio web

* [BROWSER] GitHub Codespaces
  * [BUTTON] Run and Debug
  * Seleccionar `Mona Astro: Dev Server`.
  * [BUTTON] Start Debugging
  * Esperar hasta que se inicie el servidor.
  * Abrir el puerto reenviado `4321`.
  * Verificar que el sitio GitHub Info se muestre correctamente.

<br>

## Instalar GitHub Agentic Workflows

<br>

* [CONSOLE]

```bash
curl -fsSL https://raw.githubusercontent.com/github/gh-aw/main/install-gh-aw.sh | bash
```

* Esperar hasta que finalice la instalación.

* [CONSOLE]

```bash
gh aw version
```

* Verificar que se muestre la versión instalada.

<br>

## Crear el token de GitHub Copilot

<br>

* [BROWSER] https://github.com/settings/personal-access-tokens/new?name=COPILOT_GITHUB_TOKEN&description=GitHub+Agentic+Workflows+-+Copilot+engine+authentication&user_copilot_requests=read
  * En `Token name`, ingresar `COPILOT_GITHUB_TOKEN`.
  * En `Description`, ingresar `GitHub Agentic Workflows - Copilot engine authentication`.
  * Configurar el acceso requerido al repositorio.
  * Configurar el permiso `Copilot Requests: Read`.
  * [BUTTON] Generate token
  * Copiar el valor del token.

> No guardar el token en archivos, comentarios, Pull Requests ni mensajes de Copilot Chat.

<br>

## Crear el secreto del repositorio

<br>

* [BROWSER] Repositorio del laboratorio
  * [BUTTON] Settings
  * [BUTTON] Secrets and variables
  * [BUTTON] Actions
  * [BUTTON] New repository secret
  * En `Name`, ingresar `COPILOT_GITHUB_TOKEN`.
  * En `Secret`, pegar el token generado.
  * [BUTTON] Add secret

<br>

## Configurar los permisos de GitHub Actions

<br>

* [BROWSER] Repositorio del laboratorio
  * [BUTTON] Settings
  * [BUTTON] Actions
  * [BUTTON] General
  * Buscar la sección `Workflow permissions`.
  * Seleccionar `Read and write permissions`.
  * Seleccionar `Allow GitHub Actions to create and approve pull requests`.
  * [BUTTON] Save

<br>

## Inicializar Agentic Workflows

<br>

* [CONSOLE]

```bash
gh aw init --create-pull-request --engine copilot --completions
```

* El comando crea un Pull Request con la configuración inicial.
  * `.github/skills/agentic-workflows/SKILL.md`
  * `.github/agents/agentic-workflows.md`
  * `.github/mcp.json`
  * `.github/workflows/copilot-setup-steps.yml`
  * `.gitattributes`

* [BROWSER] Repositorio del laboratorio
  * [BUTTON] Pull requests
  * Abrir el Pull Request de configuración.
  * Revisar los archivos generados.
  * [BUTTON] Merge pull request
  * [BUTTON] Confirm merge

<br>

## Crear la rama de trabajo

<br>

* [COPILOT TERMINAL] Presionar `Ctrl + I` en Windows o `Cmd + I` en macOS.

```text
Make sure I am on the latest main branch.
Create a new branch named create-mona-updater.
```

* [BUTTON] Run
* Esperar hasta que Copilot actualice `main` y cree la rama `create-mona-updater`.

<br>

## Configurar el agente de creación de workflows

<br>

* [COPILOT CHAT] Presionar `Ctrl + Alt + I` en Windows o `Ctrl + Cmd + I` en macOS.
  * Abrir el selector de agentes.
  * [BUTTON] agentic-workflows

* [FILE] `.github/agents/agentic-workflows.md`
  * Buscar la sección `Important Notes`.
  * Agregar la siguiente instrucción:

```text
When creating or editing agentic workflow files, do not compile them.
Only create or update the markdown workflow file.
```

  * Guardar el archivo.

<br>

## Crear el archivo Markdown del agente

<br>

* [COPILOT CHAT] Con el agente `agentic-workflows` seleccionado, ingresar:

```text
Create .github/workflows/update-github-info.md as an agentic workflow markdown file.

Requirements:

- Set the frontmatter name to exactly update-github-info.
- Run the workflow daily and on demand with workflow_dispatch.
- Give the workflow edit access through the tools configuration.
- Set network.allowed to include github.com and github.blog.
- Use safe-outputs with create-pull-request.
- Read notes/mona-notes.md.
- Fetch https://github.blog/latest/.
- Fetch https://github.blog/changelog/.
- Update site/content/github-info.md.
- Open a pull request for Mona to review.
- Validate the agentic workflow configuration.
- Use web-fetch for external public information.
- Use GitHub repository API tools for repository files.
- Do not use terminal, CLI, or sandbox commands to read repository files.
- Do not compile the workflow.
```

* [BUTTON] Allow, cuando Copilot solicite permisos.
* Esperar hasta que Copilot genere el archivo.

* [FILE] `.github/workflows/update-github-info.md`
  * Verificar que el archivo incluya:
    * El nombre `update-github-info`.
    * Ejecución manual con `workflow_dispatch`.
    * Ejecución programada con `schedule`.
    * La herramienta `edit`.
    * La herramienta `web-fetch`.
    * Acceso a `github.com`.
    * Acceso a `github.blog`.
    * Safe Output para crear Pull Requests.
    * Instrucciones para actualizar `site/content/github-info.md`.

<br>

## Verificar la definición del agente

<br>

* [FILE] `.github/workflows/update-github-info.md`
  * Verificar que el contenido sea similar al siguiente:

```markdown
---
name: update-github-info
description: Draft website updates for Mona's GitHub Info site from official GitHub sources.
on:
  workflow_dispatch:
  schedule:
    - cron: '17 9 * * *'
safe-outputs:
  create-pull-requ*st:
    title-prefix: "[mona] "
  * draft: true
    fallback-as-issue* false
tools:
  edit:
  web-fetch:*network:
  allowed:
    - github.c*m
    - github.blog
---

# Update *ona's GitHub Info website

Read `n*tes/mona-notes.md` before making c*anges.

Use these sources:

- `not*s/mona-notes.md`
- GitHub Blog: https://github.blog/latest/
- GitHub *hangelog: https://github.blog/chan*elog/

Update `site/content/github*info.md` with concise and practica* updates.

Include the source cont*xt when content comes from the Git*ub Blog or GitHub Changelog.

Open*a pull request for Mona to review.*
Do not write directly to `main`. *se `safe-outputs` with `create-pul*-request`.
```

<br>

## Compilar *l agente en un workflow YAML

<br>*
* [CONSOLE]

```bash
gh aw compil* .github/workflows/update-github-i*fo.md
```

* Verificar que la comp*lación finalice correctamente.
* V*rificar que se genere el siguiente*archivo:

```text
.github/workflow*/update-github-info.lock.yml
```

* El archivo `.md` contiene las ins*rucciones del agente en lenguaje n*tural.
* El archivo `.lock.yml` es*el workflow YAML que ejecutará Git*ub Actions.

<br>

## Publicar el *orkflow

<br>

* [COPILOT TERMINAL* Presionar `Ctrl + I` en Windows o*`Cmd + I` en macOS.

```text
Commi* all changed files, including the *gentic workflow markdown and the c*mpiled .lock.yml file.

Push the l*test commits to the existing creat*-mona-updater branch.

Open a pull*request into main if one does not *lready exist.

Use the pull reques* title "Create Mona website update* workflow".
```

* [BUTTON] Run
* *sperar hasta que Copilot complete *l commit, el push y la creación de* Pull Request.

* [BROWSER] Reposi*orio del laboratorio
  * [BUTTON] *ull requests
  * Abrir `Create Mon* website updater workflow`.
  * Re*isar los cambios.
  * [BUTTON] Mer*e pull request
  * [BUTTON] Confir* merge

<br>

## Agregar una nueva*fuente al agente

<br>

* [COPILOT*CHAT] Seleccionar el agente `agent*c-workflows`.

```text
Update .git*ub/workflows/update-github-info.md*

Keep the existing sources and ne*work access.

Add https://awesome-*opilot.github.com/ as a source.

A*d awesome-copilot.github.com to ne*work.allowed without removing gith*b.com or github.blog.

Do not comp*le the workflow.
```

* [BUTTON] A*low, si Copilot solicita permisos.** Esperar hasta que Copilot actual*ce el archivo.

* [FILE] `.github/*orkflows/update-github-info.md`
  * Verificar que se haya agregado `h*tps://awesome-copilot.github.com/`*
  * Verificar que `awesome-copilo*.github.com` aparezca en `network.*llowed`.
  * Verificar que no se h*yan eliminado los dominios anterio*es.

<br>

## Compilar nuevamente *l workflow

<br>

* [CONSOLE]

```*ash
gh aw compile .github/workflow*/update-github-info.md
```

* Veri*icar que se actualice el archivo:
*```text
.github/workflows/update-g*thub-info.lock.yml
```

<br>

## P*blicar la actualización

<br>

* [*OPILOT TERMINAL] Presionar `Ctrl +*I` en Windows o `Cmd + I` en macOS*

```text
Commit all changed files* including the agentic workflow ma*kdown and the updated compiled .lo*k.yml file.

Push the latest commi*s to the existing create-mona-upda*er branch.

Open a pull request in*o main if one does not already exi*t.

Use the pull request title "Up*ate Mona website updater workflow"*
```

* [BUTTON] Run
* Esperar has*a que Copilot complete las operaci*nes.

* [BROWSER] Repositorio del *aboratorio
  * [BUTTON] Pull reque*ts
  * Abrir `Update Mona website *pdater workflow`.
  * Revisar los *ambios.
  * [BUTTON] Merge pull re*uest
  * [BUTTON] Confirm merge

<*r>

## Ejecutar el agente

<br>

* [BROWSER] Repositorio del laboratorio
  * [BUTTON] Actions
  * Seleccionar el workflow `update-github-info`.
  * [BUTTON] Run workflow
  * Seleccionar la rama `main`.
  * [BUTTON] Run workflow
  * Esperar hasta que la ejecución finalice correctamente.

* Durante la ejecución, el agente:
  * Lee `notes/mona-notes.md`.
  * Consulta las fuentes públicas configuradas.
  * Analiza la información encontrada.
  * Propone cambios en `site/content/github-info.md`.
  * Crea un Pull Request sin modificar directamente la rama `main`.

<br>

## Revisar el Pull Request generado por el agente

<br>

* [BROWSER] Repositorio del laboratorio
  * [BUTTON] Pull requests
  * Abrir el Pull Request cuyo título comienza con `[mona]`.
  * [BUTTON] Files changed
  * Verificar los cambios realizados en `site/content/github-info.md`.
  * Verificar que el contenido indique las fuentes utilizadas.
  * Dejar el Pull Request abierto para revisión.
  * No realizar el merge.

<br>

# Conclusiones

<br>

En este laboratorio aprendimos a:

* Inicializar GitHub Agentic Workflows en un repositorio.
* Configurar el token de GitHub Copilot como secreto.
* Otorgar permisos de escritura a GitHub Actions.
* Definir las instrucciones de un agente en un archivo Markdown.
* Configurar herramientas, fuentes, acceso de red y Safe Outputs.
* Compilar el archivo Markdown en un workflow YAML.
* Ejecutar el agente mediante GitHub Actions.
* Leer información del repositorio y de fuentes externas.
* Modificar archivos de contenido del sitio.
* Generar automáticamente un Pull Request.
* Mantener la intervención humana antes de aplicar cambios en `main`.
