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

## LAboratorio uso memoria (con algo de copilot)

* Crear un secreto a nivel Usuario
   * (usuario) -> Settings -> Developer Settings -> PAT
     * Permisos
       * Account : Copilot Requests
       * Repository : Issues (No se si es necesesario) REad and Write

* Copiar el PAT en un lugar seguro se muestra una sola vez

* Crear secreo a nivel repositorio
  * (Repo) -> Settings -> Settings and Variables -> Actions
    * COPILOT_GITHUB_TOKEN
    * Ponerle el PAT que sacamos anteriomente
   
* en .github/workflows/memeory.yaml

```
name: Agent memory demo

on:
  workflow_dispatch:
    inputs:
      action:
        description: Memory operation
        required: true
        type: choice
        options:
          - remember
          - recall

permissions:
  contents: read
  issues: write

jobs:
  remember:
    name: Remember a phrase
    if: ${{ inputs.action == 'remember' }}
    runs-on: ubuntu-latest

    steps:
      - name: Install GitHub Copilot CLI
        run: |
          npm install -g @github/copilot

      - name: Generate phrase with Copilot
        id: phrase
        env:
          COPILOT_GITHUB_TOKEN: ${{ secrets.COPILOT_GITHUB_TOKEN }}
        run: |
          PHRASE=$(copilot -p \
            "Inventar una frase corta e inaginativa. Devovler la frase sin acotar nada mas")

          echo "Generated phrase:"
          echo "$PHRASE"

          {
            echo "phrase<<EOF"
            echo "$PHRASE"
            echo "EOF"
          } >> "$GITHUB_OUTPUT"

      - name: Find existing memory Issue
        id: memory
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          ISSUE_NUMBER=$(gh issue list \
            --label "agent-memory" \
            --state open \
            --json number \
            --jq '.[0].number // empty')

          echo "issue_number=$ISSUE_NUMBER" >> "$GITHUB_OUTPUT"

      - name: Create memory Issue
        if: ${{ steps.memory.outputs.issue_number == '' }}
        env:
          GH_TOKEN: ${{ github.token }}
          PHRASE: ${{ steps.phrase.outputs.phrase }}
        run: |
          gh issue create \
            --title "Agent Memory" \
            --label "agent-memory" \
            --body "$(cat <<EOF
          ## Stored memory

          $PHRASE

          ## Metadata

          Stored by the Agent Memory Lab.
          EOF
          )"

      - name: Update memory Issue
        if: ${{ steps.memory.outputs.issue_number != '' }}
        env:
          GH_TOKEN: ${{ github.token }}
          PHRASE: ${{ steps.phrase.outputs.phrase }}
          ISSUE_NUMBER: ${{ steps.memory.outputs.issue_number }}
        run: |
          gh issue edit "$ISSUE_NUMBER" \
            --body "$(cat <<EOF
          ## Stored memory

          $PHRASE

          ## Metadata

          Stored by the Agent Memory Lab.
          EOF
          )"

      - name: Confirm stored memory
        run: |
          echo "The phrase was stored in a GitHub Issue."

  recall:
    name: Recall the phrase
    if: ${{ inputs.action == 'recall' }}
    runs-on: ubuntu-latest

    steps:
      - name: Find memory Issue
        id: memory
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          ISSUE_NUMBER=$(gh issue list \
            --label "agent-memory" \
            --state open \
            --json number \
            --jq '.[0].number // empty')

          if [ -z "$ISSUE_NUMBER" ]; then
            echo "No memory Issue was found."
            exit 1
          fi

          echo "issue_number=$ISSUE_NUMBER" >> "$GITHUB_OUTPUT"

      - name: Read memory Issue
        env:
          GH_TOKEN: ${{ github.token }}
          ISSUE_NUMBER: ${{ steps.memory.outputs.issue_number }}
        run: |
          BODY=$(gh issue view "$ISSUE_NUMBER" \
            --json body \
            --jq '.body')

          PHRASE=$(printf '%s\n' "$BODY" |
            sed -n '/^## Stored memory$/,/^## Metadata$/p' |
            sed '1d;$d' |
            sed '/^[[:space:]]*$/d')

          echo "Memory recovered from Issue #$ISSUE_NUMBER"
          echo ""
          echo "Remembered phrase:"
          echo "$PHRASE"
```

* Vamos a (Repo)->Issues->LAbes
  * Vamos a crear el label "agent-memory"

# Multi agente


# Governance
