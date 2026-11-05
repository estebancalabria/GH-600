# Clase Dos - 5 de Noviembre del 2026

# Repaso 

* Agentes en Github
  * Github Actions como Entorno de Ejecucion
  * El agente usa la IA
    * Necesitamos la subscripcion a copilot
    * Es agente cuando la automatizacion usa copilot
    * Hay que cargar un token en secretos para que el agente pueda usar copilot
  * Github Agentic Workflows
    * Era una libreria que me permite escribir el agente primero en markdown
      * Tiene un frontmatter en yaml que se pone siempre
    * El md se compila a yaml
        * gh aw compile
  * Github
    * (usuario) -> Settings -> Developer Settings -> PAT
    * (proyecto) -> Settings -> Secrets and variables
    * Codespaces para editar el proyecto
    * Actions para ejectuar workflows y ver logs
   
---

# Links y Notas

* https://www.examtopics.com/exams/microsoft/gh-600/
* Para la certificacion ver la sintaxis del YAML (aca en este repo hay material)

--

> [!NOTE]
> Antes que nada pueden ir iniciando los XtermeLabs para la segunda parte

# Conceptos General de Agentes

* Conceptos
    * Memoria
    * Human-In-The-Loop
        * Aprobacion
    * Estado del agente

## Laboratorio Human-in-The-Loop con Enviroments
 
* Crear un repositorio

* Crear el archivo .github/worflows/deploy.yml

```
name: Deploy application

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  validate:
    name: Validate application
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Validate application
        run: |
          test -f index.html
          echo "Application validation completed"

  deploy:
    name: Deploy to production
    needs: validate
    runs-on: ubuntu-latest
    environment:
      name: production

    steps:
      - name: Deploy application
        run: |
          echo "Starting production deployment"
          echo "Application deployed to production"

```

> [!NOTE]
> Prestarle atencion a la parte de Enviroment

* Voy a ir a (proyecto) -> Settings -> Enviroment
  * Crear un enviroment con aprobadores y poner al usuario actual
 
* Ejectuar el workflow y verificar el proceso de aprobacion

## Laboratorio Human-in-The-Loop con Pull Request
 
* La proteccion de un branch por pull request se hace en
 * (Proyecto) -> Settings -> Rule Set
   * New Ruleset -> NEw Branch RuleSet
     * Creamos un Rule Set para el banch mail
     * Require a pull request before merging

---
# Break
# Hasta y 30
--

# Multi agente


# Governance
