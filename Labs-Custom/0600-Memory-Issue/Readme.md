# Memoria externa de un agente con GitHub Issues

# Objetivos

En este laboratorio aprenderás a implementar una forma simple de memoria externa para un agente utilizando GitHub Issues.

El laboratorio utilizará ejecuciones independientes de GitHub Actions.

En la primera ejecución, GitHub Copilot generará una frase inventada y el workflow la almacenará en un Issue.

En una segunda ejecución, el workflow localizará ese Issue, recuperará la frase almacenada y la mostrará nuevamente.

* Crear un repositorio en GitHub.
* Abrir el repositorio utilizando GitHub Codespaces.
* Crear un Fine-grained Personal Access Token.
* Almacenar el token como un GitHub Actions Secret.
* Crear una label para identificar la memoria del agente.
* Crear un workflow de GitHub Actions.
* Utilizar GitHub Copilot para generar una frase.
* Persistir la frase en un GitHub Issue.
* Finalizar completamente la primera ejecución.
* Iniciar una segunda ejecución independiente.
* Localizar el Issue utilizado como memoria.
* Recuperar la frase almacenada.
* Mostrar la frase en los logs.
* Comprobar que la información puede persistir entre ejecuciones independientes.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Copilot.
* Acceso a GitHub Actions.
* Acceso a GitHub Codespaces.
* Permisos para crear un Fine-grained Personal Access Token.
* Permisos para administrar secrets del repositorio.
* Permisos para crear y modificar Issues.
* Tiempo estimado: 30 a 40 minutos.

# Pasos

## Crear un repositorio

* [BROWSER] GitHub
  * Iniciar sesión con una cuenta de GitHub.
  * [BUTTON] New
  * En `Repository name`, ingresar:

```text
agent-memory-lab
```

  * Seleccionar `Public`.
  * Seleccionar `Add a README file`.
  * [BUTTON] Create repository

* Esperar hasta que se muestre el nuevo repositorio.

## Abrir el repositorio en GitHub Codespaces

* [BROWSER] Repositorio `agent-memory-lab`
  * [BUTTON] Code
  * Seleccionar la pestaña `Codespaces`.
  * [BUTTON] Create codespace on main

* Esperar hasta que GitHub Codespaces termine de iniciar.

* Si aparece una solicitud de confianza:
  * [BUTTON] I Trust

* Verificar que la terminal esté disponible.

## Verificar GitHub CLI

* [CONSOLE]

```bash
gh --version
```

Verificar también el estado de autenticación:

* [CONSOLE]

```bash
gh auth status
```

La terminal debería mostrar que GitHub CLI se encuentra autenticada.

## Crear un Fine-grained Personal Access Token

Ahora crearemos la credencial que utilizará GitHub Copilot durante la ejecución del workflow.

* [BROWSER] GitHub
  * Seleccionar la imagen de perfil.
  * Seleccionar `Settings`.
  * Seleccionar `Developer settings`.
  * Seleccionar `Personal access tokens`.
  * Seleccionar `Fine-grained tokens`.
  * [BUTTON] Generate new token

En `Token name`, ingresar:

```text
agent-memory-lab
```

Configurar el acceso al repositorio utilizado en el laboratorio.

Configurar los permisos necesarios para utilizar GitHub Copilot desde el workflow.

* [BUTTON] Generate token

Copiar el token generado.

> [!IMPORTANT]
> GitHub mostrará el valor del token una única vez.
>
> No almacenar el token dentro del repositorio.
>
> No escribir el token directamente dentro del archivo YAML.

## Crear el secret para GitHub Copilot

Ahora almacenaremos el token como un GitHub Actions Secret.

* [BROWSER] Repositorio `agent-memory-lab`
  * [BUTTON] Settings
  * Seleccionar `Secrets and variables`.
  * Seleccionar `Actions`.
  * [BUTTON] New repository secret

En `Name`, ingresar:

```text
COPILOT_GITHUB_TOKEN
```

En `Secret`, pegar el Fine-grained Personal Access Token creado anteriormente.

* [BUTTON] Add secret

Verificar que aparezca:

```text
COPILOT_GITHUB_TOKEN
```

El workflow podrá utilizar esta credencial durante la ejecución sin almacenar su valor en el repositorio.

> [!IMPORTANT]
> El Personal Access Token es la credencial.
>
> `COPILOT_GITHUB_TOKEN` es el secret utilizado para almacenarla de forma segura y ponerla a disposición del workflow durante la ejecución.

## Crear una label para identificar la memoria

El repositorio utilizará un Issue como almacenamiento persistente.

Para poder localizar ese Issue posteriormente utilizaremos una label específica.

* [BROWSER] Repositorio `agent-memory-lab`
  * Seleccionar `Issues`.
  * Seleccionar `Labels`.
  * [BUTTON] New label

En `Label name`, ingresar:

```text
agent-memory
```

En `Description`, ingresar:

```text
Persistent memory used by the agent memory lab
```

* [BUTTON] Create label

La label:

```text
agent-memory
```

identificará el Issue utilizado como memoria externa.

> [!IMPORTANT]
> No recuperaremos un Issue simplemente porque sea el último Issue creado.
>
> El workflow buscará explícitamente un Issue identificado con la label `agent-memory`.

## Crear el workflow

Crear las carpetas:

```text
.github
.github/workflows
```

Dentro de `.github/workflows`, crear:

```text
memory.yml
```

La estructura debería quedar:

```text
agent-memory-lab
│
├── .github
│   └── workflows
│       └── memory.yml
│
└── README.md
```

* [FILE] `.github/workflows/memory.yml`

Agregar:

```yaml
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
            "Invent one short and original phrase. Return only the phrase, without explanation.")

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
          echo "This workflow run can now finish."

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

* Guardar el archivo.

El workflow contiene dos operaciones:

```text
remember
recall
```

`remember` genera y almacena una memoria.

`recall` recupera esa memoria durante una ejecución posterior.

## Publicar el workflow

* [CONSOLE]

```bash
git add .
```

* [CONSOLE]

```bash
git commit -m "Add agent memory workflow"
```

* [CONSOLE]

```bash
git push
```

* Esperar hasta que finalice el push.

## Primera ejecución: guardar una memoria

Primero ejecutaremos el workflow en modo:

```text
remember
```

* [BROWSER] Repositorio `agent-memory-lab`
  * Seleccionar `Actions`.
  * Seleccionar:

```text
Agent memory demo
```

  * [BUTTON] Run workflow

En `Memory operation`, seleccionar:

```text
remember
```

* [BUTTON] Run workflow

Esperar hasta que aparezca la nueva ejecución.

Abrir la ejecución.

Verificar que se ejecute:

```text
Remember a phrase
```

## Observar la frase generada

Abrir el paso:

```text
Generate phrase with Copilot
```

Copilot generará una frase inventada.

Por ejemplo:

```text
Los pingüinos programan mejor cuando nadie mira el teclado.
```

> [!NOTE]
> La frase es generada por Copilot durante la ejecución.
>
> El resultado puede ser diferente cada vez que se ejecute `remember`.

## Verificar el Issue de memoria

Después de finalizar el workflow:

* [BROWSER] Repositorio
  * Seleccionar `Issues`.

Localizar el Issue:

```text
Agent Memory
```

Verificar que tenga la label:

```text
agent-memory
```

Abrir el Issue.

El contenido debería tener una estructura similar a:

```text
## Stored memory

Los pingüinos programan mejor cuando nadie mira el teclado.

## Metadata

Stored by the Agent Memory Lab.
```

La frase ahora existe fuera del workflow run que la generó.

> [!IMPORTANT]
> El workflow que generó la frase ya terminó.
>
> Las variables utilizadas por esa ejecución ya no son necesarias.
>
> La información se encuentra persistida en un recurso del repositorio.

## Observar dónde quedó almacenada la información

El flujo realizado hasta este momento es:

```text
Workflow run #1
      ↓
GitHub Copilot
      ↓
genera una frase
      ↓
GitHub Issue
      ↓
label: agent-memory
      ↓
workflow termina
```

El primer workflow run puede finalizar completamente.

No es necesario mantener abierto el runner.

La información permanece almacenada en el Issue.

## Segunda ejecución: recuperar la memoria

Ahora iniciaremos una ejecución completamente nueva.

* [BROWSER] Repositorio
  * Seleccionar `Actions`.
  * Seleccionar:

```text
Agent memory demo
```

  * [BUTTON] Run workflow

En `Memory operation`, seleccionar:

```text
recall
```

* [BUTTON] Run workflow

Abrir la nueva ejecución.

Verificar que se ejecute:

```text
Recall the phrase
```

Esta ejecución es independiente de la ejecución que generó la frase.

## Localizar la 
