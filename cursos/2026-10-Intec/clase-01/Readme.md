# Clase uno - 29 de Octubre del 2026

# Roadmap

* Agentes
 * Agente vs Asistente
 * Arquitectura de agentes
* Programacion de Agentes en Github
 * Github
   * Issues
   * Pull Request
   * Actions
   * Settings Github
   * Code Spaces
 * Agentes
* Tooling
 * MCP
 * Comunicacion con herramientas
* Multi Agente

# Links

* https://github.com/estebancalabria/gh-600
* Microsoft Learn
 * https://learn.microsoft.com/en-us/training/courses/gh-600t00
* Contacto del Profe
  * https://www.linkedin.com/in/esteban-calabria-7a44401a/
  * https://www.instagram.com/mct.esteban.calabria/

# Github

* Github te da cierta capacida de computo (sujeto tambien a subscripcion de github copilot)

## Github Actions

* Aqui se programan automatizaciones
  * Ejemplo. Todos los dias a las 8:00 ejecutar los unit test sobre el proyecto
  * Estas automatizaciones en general entran dentro de Devops y no necesariamente usan IA
* La plataforma de Github Actions da soporte y un entorno para la ejecucion de Agentes
  * Me da el computo para ejecutar mis agentes
  * Funcionar como orquestador de mis agentes

* Ir a Github y crear un Repo

* Crear un PAT (A nivel Usuario)
  * A Nivel Usuario
  * Me da permiso para usar la API de GIT
  * Aplico siempre la politica "Least Priviledge access"
  * Conviene restringit solo al repo al que voy a a trabajar
  * Una vez que muestra el codigo no se vuele a mostrar
  * Guardar el codigo en un lugar seguro, nunca en los fuetnes o archivos del REpositorio
 
* Crear un secreto del repositorio para guardar el PAT
  * (Settings del Repo) -> "Secrets and Variables" -> Actions -> New Repository Secret
    * Copiar el PAT
    * Add Secret
   
* Creammos el workflow / pipeline / script
  * .github/workflows/mi-primer-workflow.yml

```
name : Mi primero Workflow

on:
  workflow_dispatch: 

jobs:
  hola:
    runs-on: ubuntu-latest

    steps:
      - name: Mostrar Mensaje
        run: echo "Bienvenidos al Primero Workflow"
```

> [!NOTE]
> Formato YML : https://en-wikipedia-org.translate.goog/wiki/YAML?_x_tr_sl=en&_x_tr_tl=es&_x_tr_hl=es&_x_tr_pto=tc

> [!NOTE]
> Mientras escribimos me sugiera errores, autocompletado, snippets a la derecha de uso comun

* Dentro de la carpeta .github podemos encontrar esta estructura
  
.github/
├── workflows/
│   ├── ci.yml
│   ├── deploy.yml
│   └── security.yml
│
├── ISSUE_TEMPLATE/
│   ├── bug_report.md
│   ├── feature_request.md
│   └── config.yml
│
├── PULL_REQUEST_TEMPLATE/
│   ├── feature.md
│   └── bugfix.md
│
├── CODEOWNERS
├── dependabot.yml
├── CONTRIBUTING.md
├── SECURITY.md
└── FUNDING.yml

* Ejecutamos la accion en la parte actions del repo

<img width="939" height="401" alt="image" src="https://github.com/user-attachments/assets/5409621f-08a2-48f7-959a-136e3b48edf1" />

---
# BREAK
15 minutos hasta menos 45
---

* Abrir un codespace

* Probar el comando gh
  * Se llama Github CLI
  * https://cli.github.com/
  * Para usar github desde el CLI
    * Crear repos / borrar repos : gh repo create mi-repositorio
  * Se le pueden instalar extensiones

* Instalar y verificar instalacion de la extension github agentic workflow
  * https://github.github.com/gh-aw/

* Para ver la version (me sugiere instalar)
```
gh aw version
```

* Si la quiero instalar
```
gh extension install github/gh-aw
```

* Escribir el archivo .github/workflows/hello-agent.md

```
---
name: hello-agent

on:
  workflow_dispatch:

safe-outputs:
  create-issue:
---

# Agente Hello

Leer el Readme.md del repositorio
Creara una issue con el titulo
"Ejemplo de Issue generada por Agente"

En el cuerpo de la Issue poner:
- Un resumen del repositorio
- Una lista de bullets describiendo el reposiorio
- Finalizar con la oracion "Generado por Github Agentic Worflow"
```

* Compilar el Workflow

```
gh aw compile .github/workflows/hello-agent.md
```

* Esto crea un archivo hello-agent.lock.yaml
  * Vamos a mirar por arriba y pasarlo a copilot para entenderlo
  * Este archivo usa copilot para escribir el texto issue y usa github para guardar el issue en el repo
 
* Verificar los permisos del PAT
  * Vamos a los settings de usuario y vemos el PAT que creamos al principio de la clase
    * Permiso Repository
      * Issues
        * Read-and-write
    * Permiso Account
      * Copilot Requests
     
* En los settings del proyecto crear el secreto COPILOT_GITHUB_TOKEN
  * Verificar que ese secreto se usa en el archivo .lock.yml


* En las settings del repo vamos a darle permiso a la parte de actions para que escriba ISSUES
   * Actions -> General -> Workflow Permissions -> "Read and write permissions"
 
* Subo los cambios del agente a github

```

@estebancalabria ➜ /workspaces/curso-gh-600 (main) $ git add .
@estebancalabria ➜ /workspaces/curso-gh-600 (main) $ git commit -m "Primer Agente"
[main fa91eff] Primer Agente
 5 files changed, 2096 insertions(+), 2 deletions(-)
 create mode 100644 .gitattributes
 create mode 100644 .github/aw/actions-lock.json
 create mode 100644 .github/workflows/hello-agent.lock.yml
 create mode 100644 .github/workflows/hello-agent.md
@estebancalabria ➜ /workspaces/curso-gh-600 (main) $ git push
Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 2 threads
Compressing objects: 100% (9/9), done.
Writing objects: 100% (10/10), 25.54 KiB | 6.39 MiB/s, done.
Total 10 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To https://github.com/estebancalabria/curso-gh-600
   b34b52f..fa91eff  main -> main
@estebancalabria ➜ /workspaces/curso-gh-600 (main) $ 
```
