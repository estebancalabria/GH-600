# Memoria externa de un agente con GitHub Issues

# Objetivos

En este laboratorio aprenderás a implementar una forma simple de memoria externa utilizando GitHub Issues.

El laboratorio utilizará ejecuciones independientes de GitHub Actions.

En una primera ejecución, GitHub Copilot generará una frase inventada y el workflow la almacenará en un Issue.

En una segunda ejecución, el workflow localizará el Issue, recuperará la frase almacenada y la mostrará nuevamente.

* Crear un repositorio en GitHub.
* Crear un Fine-grained Personal Access Token para GitHub Copilot.
* Almacenar el token como un GitHub Actions Secret.
* Crear una label para identificar el Issue utilizado como memoria.
* Crear un workflow de GitHub Actions.
* Utilizar GitHub Copilot para generar una frase.
* Persistir la frase en un GitHub Issue.
* Finalizar completamente la primera ejecución.
* Iniciar una segunda ejecución independiente.
* Localizar el Issue utilizado como memoria.
* Recuperar la frase almacenada.
* Mostrar la frase en los logs.
* Comprobar que la información persiste entre ejecuciones independientes.
* Actualizar la memoria y recuperar posteriormente su nuevo valor.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Copilot.
* Acceso a GitHub Actions.
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

Para poder localizar posteriormente ese Issue, utilizaremos una label específica.

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

* [BUTTON] Commit changes

En el mensaje del commit, ingresar:

```text
Add agent memory workflow
```

* [BUTTON] Commit changes

## Comprender las dos operaciones

El workflow permite seleccionar entre dos operaciones:

```text
remember
recall
```

La operación:

```text
remember
```

genera una frase con GitHub Copilot y la almacena en un Issue.

La operación:

```text
recall
```

se ejecuta posteriormente, localiza ese Issue y recupera la información almacenada.

El flujo que vamos a comprobar es:

```text
remember
   ↓
Copilot genera una frase
   ↓
Issue
   ↓
workflow termina


recall
   ↓
nuevo workflow run
   ↓
Issue
   ↓
recupera la frase
```

## Primera ejecución: guardar una memoria

Primero ejecutaremos:

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
> La frase es generada durante la ejecución.
>
> El resultado puede ser diferente cada vez que se ejecute `remember`.

## Verificar el Issue utilizado como memoria

Después de finalizar el workflow:

* [BROWSER] Repositorio
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

La frase ahora se encuentra almacenada fuera del workflow run que la generó.

> [!IMPORTANT]
> El workflow que generó la frase ya terminó.
>
> Las variables y outputs de esa ejecución no serán utilizados por la siguiente ejecución.
>
> La información que queremos conservar está almacenada en un recurso persistente de GitHub.

## Observar la memoria persistente

El flujo realizado hasta este momento es:

```text
Workflow run #1
      ↓
GitHub Copilot
      ↓
genera una frase
      ↓
Issue: Agent Memory
      ↓
label: agent-memory
      ↓
workflow termina
```

El primer workflow run terminó.

Sin embargo, la información continúa existiendo en el Issue.

## Segunda ejecución: recuperar la memoria

Ahora iniciaremos una ejecución nueva.

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

## Localizar la memoria

Abrir el paso:

```text
Find memory Issue
```

El workflow busca un Issue abierto identificado mediante:

```text
agent-memory
```

La operación utilizada es equivalente a:

```bash
gh issue list \
  --label "agent-memory" \
  --state open \
  --json number
```

El workflow no depende del número del último Issue creado.

Busca explícitamente el recurso marcado como memoria del laboratorio.

## Recuperar la frase

Abrir:

```text
Read memory Issue
```

El workflow obtiene el contenido del Issue.

Luego recupera la sección:

```text
## Stored memory
```

Verificar que los logs muestren algo similar a:

```text
Memory recovered from Issue #1

Remembered phrase:
Los pingüinos programan mejor cuando nadie mira el teclado.
```

La frase debe ser la misma que fue almacenada durante la ejecución anterior.

## Comparar las dos ejecuciones

Volver a:

* [BROWSER] Repositorio
  * Seleccionar `Actions`.
  * Seleccionar:

```text
Agent memory demo
```

Deberían existir dos workflow runs diferentes.

La primera ejecución utilizó:

```text
remember
```

La segunda ejecución utilizó:

```text
recall
```

El flujo completo fue:

```text
Workflow run #1
      ↓
Copilot genera una frase
      ↓
Issue: Agent Memory
      ↓
workflow termina


Workflow run #2
      ↓
nuevo runner
      ↓
Find memory Issue
      ↓
Read memory Issue
      ↓
recupera la frase
```

> [!IMPORTANT]
> La segunda ejecución no comparte variables ni outputs con la primera.
>
> La información necesaria para continuar se encuentra en el Issue.

## Modificar la memoria

Ahora volveremos a ejecutar:

```text
remember
```

* [BROWSER] Actions
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

Copilot generará una nueva frase.

Por ejemplo:

```text
Las tortugas revisan los pull requests antes del desayuno.
```

Esta vez el workflow encontrará el Issue existente identificado mediante:

```text
agent-memory
```

En lugar de crear un nuevo Issue, actualizará el contenido del existente.

## Verificar la actualización de la memoria

* [BROWSER] Repositorio
  * Seleccionar `Issues`.
  * Abrir:

```text
Agent Memory
```

Comprobar que:

* Sigue existiendo un único Issue de memoria.
* El Issue continúa teniendo la label `agent-memory`.
* La frase almacenada cambió.

El contenido podría ser:

```text
## Stored memory

Las tortugas revisan los pull requests antes del desayuno.

## Metadata

Stored by the Agent Memory Lab.
```

La memoria tiene ahora un nuevo valor.

## Recuperar nuevamente la memoria

Ejecutar nuevamente:

```text
Agent memory demo
```

Seleccionar:

```text
recall
```

Abrir el paso:

```text
Read memory Issue
```

Verificar que ahora aparezca:

```text
Remembered phrase:
Las tortugas revisan los pull requests antes del desayuno.
```

La ejecución recuperó el valor actualmente almacenado en el Issue.

No recuperó la frase de la primera ejecución.

## Analizar qué ocurrió

Durante el laboratorio realizamos cuatro ejecuciones independientes:

```text
RUN #1
remember
   ↓
Copilot genera frase A
   ↓
Issue guarda frase A


RUN #2
recall
   ↓
Issue
   ↓
recupera frase A


RUN #3
remember
   ↓
Copilot genera frase B
   ↓
Issue actualiza la memoria
   ↓
guarda frase B


RUN #4
recall
   ↓
Issue
   ↓
recupera frase B
```

Los workflow runs son ejecuciones separadas.

El elemento que proporciona continuidad entre ellas es:

```text
Issue: Agent Memory
Label: agent-memory
```

## Diferenciar contexto y memoria

Durante `remember`, existen temporalmente datos dentro de la ejecución:

```text
PHRASE
steps.phrase.outputs.phrase
```

Esos datos pertenecen al contexto de esa ejecución.

Cuando la ejecución termina, el siguiente workflow run no depende de esas variables.

En cambio:

```text
Issue: Agent Memory
```

continúa existiendo en GitHub.

Por eso, en este laboratorio, el Issue funciona como memoria externa.

## Identificar la memoria

La label:

```text
agent-memory
```

cumple una función importante.

Permite que una ejecución futura pueda localizar el recurso que contiene la información.

El patrón utilizado es:

```text
guardar
   ↓
marcar
   ↓
terminar
   ↓
localizar
   ↓
recuperar
```

Esto evita depender de reglas frágiles como:

```text
recuperar el último Issue creado
```

La memoria tiene una identidad explícita dentro del repositorio.

# Conclusiones

En este laboratorio aprendimos a:

* Crear una credencial para utilizar GitHub Copilot desde GitHub Actions.
* Almacenar esa credencial como un GitHub Actions Secret.
* Utilizar GitHub Copilot para generar información.
* Utilizar un GitHub Issue como memoria externa.
* Utilizar una label para identificar dónde se encuentra esa memoria.
* Generar información en una ejecución.
* Persistir esa información fuera del workflow run.
* Finalizar completamente la ejecución que creó la información.
* Iniciar una nueva ejecución independiente.
* Localizar posteriormente la memoria.
* Recuperar la información almacenada.
* Actualizar la memoria existente.
* Recuperar el valor actualizado desde otra ejecución.
* Mantener un único Issue como fuente de verdad para la memoria del laboratorio.

El comportamiento final es:

```text
        WORKFLOW RUN #1
               ↓
           Copilot
               ↓
       genera una frase
               ↓
     Issue: Agent Memory
      label: agent-memory
               ↓
        ejecución termina


        WORKFLOW RUN #2
               ↓
        nuevo contexto
               ↓
    busca agent-memory
               ↓
     Issue: Agent Memory
               ↓
       recupera la frase
```

La idea principal del laboratorio es:

```text
El contexto de una ejecución es temporal.

La información que necesitamos conservar
se guarda fuera de ese contexto.

Una ejecución futura puede localizar
esa información y recuperarla.
```

En este ejemplo, GitHub Issue funciona como memoria externa persistente entre ejecuciones independientes.
