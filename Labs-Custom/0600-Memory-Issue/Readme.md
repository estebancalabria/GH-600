# Memoria externa de un agente con GitHub Issues

# Objetivos

En este laboratorio aprenderás a implementar una forma simple de memoria externa utilizando GitHub Issues.

El laboratorio utilizará dos ejecuciones independientes de GitHub Actions.

En la primera ejecución, GitHub Copilot generará una frase inventada y el workflow la almacenará en un Issue.

Cuando esa ejecución termine, iniciaremos manualmente una segunda ejecución. Esta localizará el Issue, recuperará la frase almacenada y la mostrará nuevamente.

* Crear un repositorio en GitHub.
* Crear un Fine-grained Personal Access Token para GitHub Copilot.
* Almacenar el token como un GitHub Actions Secret.
* Crear una label para identificar el Issue utilizado como memoria.
* Crear un workflow de GitHub Actions.
* Utilizar GitHub Copilot para generar una frase.
* Persistir la frase en un GitHub Issue.
* Finalizar completamente la primera ejecución.
* Iniciar manualmente una segunda ejecución independiente.
* Localizar el Issue utilizado como memoria.
* Recuperar la frase almacenada.
* Comprobar que la información persiste entre ejecuciones independientes.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Copilot.
* Acceso a GitHub Actions.
* Permisos para crear un Fine-grained Personal Access Token.
* Permisos para administrar secrets del repositorio.
* Permisos para crear y modificar Issues.
* Tiempo estimado: 25 a 35 minutos.

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

## Crear un Fine-grained Personal Access Token

Crearemos una credencial que utilizará GitHub Copilot durante la ejecución del workflow.

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

El workflow utilizará esta credencial durante la ejecución sin almacenar su valor directamente en el archivo YAML.

> [!IMPORTANT]
> El Personal Access Token es la credencial.
>
> `COPILOT_GITHUB_TOKEN` es el secret que utilizamos para almacenarla y ponerla a disposición del workflow.

## Crear una label para identificar la memoria

El laboratorio utilizará un GitHub Issue como almacenamiento persistente.

Para poder localizar ese Issue desde otra ejecución utilizaremos una label específica.

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
> No recuperaremos un Issue porque sea el último Issue creado.
>
> El workflow buscará explícitamente el Issue identificado mediante la label `agent-memory`.

## Crear el workflow

Crearemos el workflow directamente desde la interfaz web de GitHub.

* [BROWSER] Repositorio `agent-memory-lab`
  * Seleccionar `Actions`.
  * Seleccionar `New workflow`.
  * Seleccionar `set up a workflow yourself`.

En el nombre del archivo, ingresar:

```text
memory.yml
```

El archivo deberá quedar almacenado en:

```text
.github/workflows/memory.yml
```

Reemplazar el contenido por:

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

* [BUTTON] Commit changes

En el mensaje del commit, ingresar:

```text
Add agent memory workflow
```

* [BUTTON] Commit changes

## Comprender las dos operaciones

El workflow ofrece dos operaciones:

```text
remember
recall
```

La operación:

```text
remember
```

utiliza GitHub Copilot para generar una frase y almacenarla en un Issue.

La operación:

```text
recall
```

localiza posteriormente ese Issue y recupera la frase almacenada.

> [!IMPORTANT]
> `remember` y `recall` no se ejecutan automáticamente uno después del otro.
>
> Ejecutaremos manualmente el workflow dos veces.
>
> Primero seleccionaremos `remember`.
>
> Cuando esa ejecución haya terminado, volveremos a ejecutar manualmente el workflow seleccionando `recall`.

## Primera ejecución: guardar una memoria

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

El job:

```text
Recall the phrase
```

debería aparecer como omitido porque en esta ejecución seleccionamos:

```text
remember
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
> La frase se genera durante la ejecución.
>
> El contenido puede ser diferente en cada ejecución.

## Verificar el Issue utilizado como memoria

Esperar hasta que finalice el workflow.

Luego:

* [BROWSER] Repositorio `agent-memory-lab`
  * Seleccionar `Issues`.

Localizar:

```text
Agent Memory
```

Verificar que el Issue tenga la label:

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

La frase se encuentra ahora almacenada en un recurso persistente del repositorio.

> [!IMPORTANT]
> La primera ejecución ya terminó.
>
> La siguiente ejecución no utilizará directamente las variables ni los outputs de este workflow run.
>
> La información que queremos recordar permanece almacenada en el Issue.

## Segunda ejecución: recuperar la memoria

Ahora iniciaremos manualmente una nueva ejecución.

* [BROWSER] Repositorio `agent-memory-lab`
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

Esperar hasta que aparezca la nueva ejecución.

Abrir la ejecución.

Verificar que esta vez se ejecute:

```text
Recall the phrase
```

El job:

```text
Remember a phrase
```

debería aparecer como omitido.

Esta segunda ejecución es independiente de la ejecución que generó la frase.

## Localizar la memoria

Abrir el paso:

```text
Find memory Issue
```

El workflow busca un Issue abierto identificado mediante la label:

```text
agent-memory
```

La operación utilizada es:

```bash
gh issue list \
  --label "agent-memory" \
  --state open \
  --json number
```

El workflow no busca simplemente el último Issue creado.

Busca explícitamente el recurso marcado como memoria del laboratorio.

## Recuperar la frase

Abrir el paso:

```text
Read memory Issue
```

El workflow obtiene el contenido del Issue y recupera la sección:

```text
## Stored memory
```

Verificar que los logs muestren algo similar a:

```text
Memory recovered from Issue #1

Remembered phrase:
Los pingüinos programan mejor cuando nadie mira el teclado.
```

La frase debe coincidir con la que fue almacenada durante la primera ejecución.

## Comprobar la memoria entre ejecuciones

Volver a:

* [BROWSER] Repositorio `agent-memory-lab`
  * Seleccionar `Actions`.
  * Seleccionar:

```text
Agent memory demo
```

Verificar que existen dos ejecuciones diferentes.

La primera ejecución utilizó:

```text
remember
```

y almacenó la frase en:

```text
Issue: Agent Memory
```

La segunda ejecución utilizó:

```text
recall
```

y recuperó la frase desde ese mismo Issue.

> [!IMPORTANT]
> La segunda ejecución no comparte variables ni outputs con la primera.
>
> La información sobrevivió porque fue almacenada en un recurso persistente de GitHub.

El comportamiento observado fue:

```text
RUN #1
remember
   ↓
Copilot genera una frase
   ↓
Issue: Agent Memory
   ↓
ejecución termina


RUN #2
recall
   ↓
nuevo workflow run
   ↓
localiza Agent Memory
   ↓
recupera la frase
```

El Issue funciona como memoria externa porque conserva información fuera del contexto temporal de una ejecución.

La label:

```text
agent-memory
```

permite que una ejecución posterior localice el recurso que contiene esa memoria.

# Conclusiones

En este laboratorio aprendimos a:

* Crear una credencial para utilizar GitHub Copilot desde GitHub Actions.
* Almacenar la credencial como un GitHub Actions Secret.
* Generar información utilizando GitHub Copilot.
* Persistir esa información en un GitHub Issue.
* Utilizar una label para identificar el Issue que contiene la memoria.
* Finalizar completamente la ejecución que generó la información.
* Iniciar manualmente una segunda ejecución independiente.
* Localizar desde esa ejecución la memoria almacenada.
* Recuperar la información generada anteriormente.
* Utilizar GitHub como fuente persistente de información entre ejecuciones.

La idea principal del laboratorio es:

```text
El contexto de una ejecución es temporal.

La memoria se almacena fuera de ese contexto.

Una ejecución futura puede localizarla
y recuperar la información.
```

En este ejemplo, el GitHub Issue funciona como memoria externa persistente entre dos ejecuciones independientes.
