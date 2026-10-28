# Guía de sintaxis de GitHub Actions

## Objetivo

Esta guía es una referencia práctica para leer y escribir workflows de GitHub Actions.

Se asume que ya conocés la sintaxis básica de YAML.

Acá vamos a concentrarnos exclusivamente en la estructura y los conceptos de GitHub Actions:

- `name`
- `on`
- `permissions`
- `env`
- `defaults`
- `jobs`
- `runs-on`
- `needs`
- `if`
- `strategy`
- `matrix`
- `steps`
- `name`
- `uses`
- `run`
- `with`
- `env`
- `secrets`
- `vars`
- contextos
- expresiones
- outputs
- artifacts
- environments

---

# 1. Dónde se guarda un workflow

Los workflows de GitHub Actions se guardan dentro de:

```text
.github/workflows/
```

Por ejemplo:

```text
.github/
└── workflows/
    ├── ci.yml
    ├── tests.yml
    └── deploy.yml
```

Los archivos utilizan extensión:

```text
.yml
```

o:

```text
.yaml
```

---

# 2. Estructura mínima

Un workflow mínimo puede tener esta forma:

```yaml
name: Mi workflow

on:
  workflow_dispatch:

jobs:
  ejemplo:
    runs-on: ubuntu-latest

    steps:
      - name: Saludar
        run: echo "Hola desde GitHub Actions"
```

La estructura principal es:

```text
workflow
├── name
├── on
└── jobs
    └── job
        ├── runs-on
        └── steps
            ├── step
            ├── step
            └── step
```

Pensalo de esta forma:

```text
on
↓
¿Cuándo se ejecuta?

jobs
↓
¿Qué trabajos se ejecutan?

steps
↓
¿Qué pasos realiza cada trabajo?
```

---

# 3. `name`

`name` define el nombre visible del workflow.

```yaml
name: CI
```

Otro ejemplo:

```yaml
name: Build and Test
```

Este nombre aparece en la pestaña **Actions** de GitHub.

Si no especificás `name`, GitHub utiliza otra información del archivo para identificar el workflow.

---

# 4. `on`

`on` define qué evento dispara el workflow.

```yaml
on:
  workflow_dispatch:
```

En este caso el workflow puede iniciarse manualmente.

También podemos usar eventos del repositorio.

Por ejemplo:

```yaml
on:
  push:
```

Esto ejecuta el workflow cuando ocurre un `push`.

---

# 5. Eventos frecuentes

## Ejecución manual

```yaml
on:
  workflow_dispatch:
```

Útil para:

- laboratorios
- tareas manuales
- operaciones administrativas
- workflows que queremos iniciar bajo demanda

---

## Push

```yaml
on:
  push:
```

Ejecuta el workflow cuando se realiza un push.

También podemos limitarlo a determinadas branches:

```yaml
on:
  push:
    branches:
      - main
```

O varias:

```yaml
on:
  push:
    branches:
      - main
      - develop
```

---

## Pull Request

```yaml
on:
  pull_request:
```

Por ejemplo, para ejecutar solamente cuando el Pull Request apunta a `main`:

```yaml
on:
  pull_request:
    branches:
      - main
```

Este evento es especialmente útil para ejecutar:

- tests
- linting
- builds
- análisis de seguridad
- validaciones antes del merge

---

## Schedule

También podemos ejecutar workflows mediante una programación.

```yaml
on:
  schedule:
    - cron: "0 8 * * 1"
```

Los schedules utilizan expresiones cron.

---

# 6. Utilizar varios triggers

Un workflow puede responder a más de un evento.

```yaml
on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:
```

Ahora el mismo workflow puede ejecutarse:

```text
push a main
        │
        ├──→ workflow
        │
Pull Request a main
        │
        ├──→ workflow
        │
ejecución manual
        │
        └──→ workflow
```

---

# 7. Inputs en `workflow_dispatch`

Un workflow manual puede pedir parámetros al usuario.

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Entorno"
        required: true
        default: "dev"
        type: choice
        options:
          - dev
          - staging
          - production
```

Cuando alguien seleccione **Run workflow**, GitHub mostrará ese parámetro.

Podemos obtener el valor mediante:

```text
${{ inputs.environment }}
```

Por ejemplo:

```yaml
steps:
  - name: Mostrar entorno
    run: echo "Entorno seleccionado: ${{ inputs.environment }}"
```

---

# 8. `jobs`

`jobs` contiene los trabajos que ejecutará el workflow.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Ejecutando tests"
```

`test` es el identificador interno del job.

También podríamos tener:

```yaml
jobs:
  build:
    ...

  test:
    ...

  deploy:
    ...
```

El identificador puede utilizarse posteriormente para relacionar jobs.

---

# 9. Nombre interno y nombre visible de un job

Podemos escribir:

```yaml
jobs:
  test:
    name: Ejecutar pruebas

    runs-on: ubuntu-latest

    steps:
      - run: npm test
```

Acá:

```text
test
```

es el identificador interno.

Mientras que:

```text
Ejecutar pruebas
```

es el nombre visible.

La diferencia importa cuando otros jobs necesitan referenciarlo.

---

# 10. `runs-on`

`runs-on` especifica el runner donde se ejecutará el job.

Ejemplo:

```yaml
runs-on: ubuntu-latest
```

Otros ejemplos habituales son:

```yaml
runs-on: windows-latest
```

```yaml
runs-on: macos-latest
```

Cada job se ejecuta en un runner.

---

# 11. `steps`

Los jobs contienen pasos.

```yaml
jobs:
  ejemplo:
    runs-on: ubuntu-latest

    steps:
      - name: Paso 1
        run: echo "uno"

      - name: Paso 2
        run: echo "dos"

      - name: Paso 3
        run: echo "tres"
```

Dentro de un job, los steps se ejecutan secuencialmente.

```text
Job
 │
 ├── Step 1
 │
 ├── Step 2
 │
 └── Step 3
```

---

# 12. `run`

`run` ejecuta comandos dentro del runner.

Ejemplo:

```yaml
- name: Mostrar mensaje
  run: echo "Hola"
```

También puede ejecutar varios comandos:

```yaml
- name: Ejecutar varios comandos
  run: |
    echo "Comenzando"
    npm install
    npm test
    echo "Finalizado"
```

Los comandos disponibles dependen del sistema operativo y de las herramientas disponibles en el runner.

---

# 13. Directorio de trabajo

Podemos ejecutar un step desde un directorio determinado:

```yaml
- name: Ejecutar tests
  working-directory: ./app
  run: npm test
```

Esto resulta útil en repositorios con varias aplicaciones o proyectos.

Por ejemplo:

```text
repo/
├── frontend/
├── backend/
└── infrastructure/
```

Podríamos ejecutar:

```yaml
- name: Test frontend
  working-directory: ./frontend
  run: npm test
```

---

# 14. `uses`

No todos los steps ejecutan directamente comandos.

Un step también puede utilizar una **Action** existente.

```yaml
- uses: actions/checkout@v4
```

La forma general es:

```text
owner/repository@version
```

Por ejemplo:

```yaml
uses: actions/checkout@v4
```

significa que estamos utilizando la action `checkout` publicada en el repositorio `actions/checkout`, versión `v4`.

---

# 15. `run` versus `uses`

Esta diferencia aparece constantemente en GitHub Actions.

## Ejecutar un comando

```yaml
- name: Ejecutar tests
  run: npm test
```

## Utilizar una Action

```yaml
- name: Obtener código
  uses: actions/checkout@v4
```

Regla mental:

```text
run
↓
ejecutar un comando

uses
↓
ejecutar una Action
```

---

# 16. `actions/checkout`

Una de las Actions más frecuentes es:

```yaml
- uses: actions/checkout@v4
```

Permite que el job trabaje con el contenido del repositorio.

Un workflow típico puede comenzar:

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4

  - name: Ejecutar tests
    run: npm test
```

---

# 17. `with`

Muchas Actions aceptan parámetros.

Se especifican utilizando:

```yaml
with:
```

Ejemplo:

```yaml
- name: Configurar Node.js
  uses: actions/setup-node@v4
  with:
    node-version: "22"
```

En este caso:

```text
actions/setup-node
```

es la Action.

Y:

```text
node-version
```

es un input de esa Action.

Cada Action define sus propios inputs.

---

# 18. Variables de entorno con `env`

Podemos definir variables de entorno con:

```yaml
env:
```

## A nivel de workflow

```yaml
name: Ejemplo

on:
  workflow_dispatch:

env:
  APP_ENV: development

jobs:
  ejemplo:
    runs-on: ubuntu-latest

    steps:
      - run: echo "$APP_ENV"
```

La variable estará disponible para los jobs del workflow.

---

# 19. `env` a nivel de job

También podemos limitarla a un job.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    env:
      NODE_ENV: test

    steps:
      - run: echo "$NODE_ENV"
```

---

# 20. `env` a nivel de step

También podemos limitarla a un único step.

```yaml
steps:
  - name: Ejemplo
    env:
      NODE_ENV: test

    run: echo "$NODE_ENV"
```

La ubicación define el alcance:

```text
workflow env
    ↓
todos los jobs

job env
    ↓
steps de ese job

step env
    ↓
solo ese step
```

---

# 21. Expresiones de GitHub Actions

GitHub Actions permite evaluar expresiones utilizando:

```text
${{ ... }}
```

Por ejemplo:

```yaml
${{ github.event_name }}
```

Otro ejemplo:

```yaml
${{ secrets.API_TOKEN }}
```

Otro:

```yaml
${{ vars.ENVIRONMENT }}
```

Otro:

```yaml
${{ env.NODE_ENV }}
```

Las expresiones permiten consultar información disponible durante la ejecución.

---

# 22. Contextos

Dentro de `${{ }}` podemos consultar distintos contextos.

Algunos de los más importantes son:

```text
github
env
vars
secrets
inputs
steps
needs
runner
job
matrix
```

Ejemplo:

```yaml
run: echo "${{ github.event_name }}"
```

---

# 23. Contexto `github`

`github` contiene información sobre el workflow y el evento que inició la ejecución.

Ejemplos:

```text
${{ github.event_name }}
${{ github.repository }}
${{ github.ref }}
${{ github.sha }}
${{ github.actor }}
```

Podemos utilizarlos:

```yaml
steps:
  - name: Mostrar contexto
    run: |
      echo "Evento: ${{ github.event_name }}"
      echo "Repositorio: ${{ github.repository }}"
      echo "Ref: ${{ github.ref }}"
      echo "Commit: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
```

---

# 24. `vars`

GitHub Actions puede utilizar variables de configuración.

Se referencian mediante:

```text
${{ vars.NOMBRE }}
```

Por ejemplo:

```yaml
env:
  ENVIRONMENT: ${{ vars.DEFAULT_ENVIRONMENT }}
```

Las variables son adecuadas para configuración que no necesita ser secreta.

---

# 25. `secrets`

Los valores sensibles se referencian mediante el contexto:

```text
secrets
```

Por ejemplo:

```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

Después el comando puede utilizar la variable:

```yaml
run: ./deploy.sh
```

No se recomienda almacenar credenciales directamente en el workflow.

Evitar:

```yaml
env:
  API_TOKEN: ghp_xxxxxxxxxxxxxxxxx
```

Preferir:

```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

---

# 26. `GITHUB_TOKEN`

GitHub Actions puede proporcionar un token asociado a la ejecución:

```text
${{ secrets.GITHUB_TOKEN }}
```

Los permisos efectivos de ese token deben controlarse mediante `permissions`.

Ejemplo:

```yaml
permissions:
  contents: read
```

La idea es otorgar únicamente los permisos requeridos por el workflow.

---

# 27. `permissions`

`permissions` controla los permisos disponibles para `GITHUB_TOKEN`.

Ejemplo:

```yaml
permissions:
  contents: read
```

Otro ejemplo:

```yaml
permissions:
  contents: read
  issues: write
```

Otro:

```yaml
permissions:
  contents: read
  pull-requests: write
```

Podemos aplicar permisos a nivel del workflow:

```yaml
name: Ejemplo

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  ejemplo:
    ...
```

O definir permisos específicos en un job:

```yaml
jobs:
  crear-issue:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      issues: write
```

La práctica recomendada es otorgar solamente lo que cada operación necesita.

---

# 28. `if`

`if` permite ejecutar condicionalmente un job o step.

Ejemplo:

```yaml
- name: Solo en main
  if: github.ref == 'refs/heads/main'
  run: echo "Estamos en main"
```

También puede aplicarse a un job:

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
```

---

# 29. Condiciones según el evento

Un patrón muy útil:

```yaml
if: github.event_name == 'pull_request'
```

Por ejemplo:

```yaml
jobs:
  validate-pr:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
```

Aunque el workflow tenga varios triggers:

```yaml
on:
  pull_request:
  workflow_dispatch:
```

podemos hacer que ciertos jobs solamente se ejecuten para uno de esos eventos.

---

# 30. Operadores en expresiones

Las condiciones pueden utilizar operadores como:

```text
==
!=
&&
||
!
```

Ejemplo:

```yaml
if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```

Otro:

```yaml
if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'
```

---

# 31. Funciones de estado

GitHub Actions dispone de funciones relacionadas con el estado de ejecución.

Entre las más utilizadas:

```text
success()
failure()
always()
cancelled()
```

Ejemplo:

```yaml
- name: Ejecutar solamente si algo falló
  if: failure()
  run: echo "Ocurrió un error"
```

Para ejecutar independientemente del resultado anterior:

```yaml
- name: Recolectar información
  if: always()
  run: echo "Este paso intenta ejecutarse siempre"
```

---

# 32. Múltiples jobs

Un workflow puede contener muchos jobs.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build"

  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Test"

  security:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Security"
```

Si no definimos dependencias entre ellos, los jobs pueden ejecutarse independientemente.

---

# 33. `needs`

`needs` establece dependencias entre jobs.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build"

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Test"
```

Ahora:

```text
build
  ↓
test
```

`test` depende de `build`.

---

# 34. Dependencia de varios jobs

Un job puede depender de varios:

```yaml
jobs:
  unit-tests:
    ...

  integration-tests:
    ...

  deploy:
    needs:
      - unit-tests
      - integration-tests
```

Queda:

```text
unit-tests ──────────┐
                    ├──→ deploy
integration-tests ──┘
```

---

# 35. Outputs de un step

Un step puede producir información para los siguientes steps.

Primero necesita un `id`:

```yaml
- name: Obtener versión
  id: version
  run: echo "value=1.2.3" >> "$GITHUB_OUTPUT"
```

Después podemos acceder al resultado:

```text
${{ steps.version.outputs.value }}
```

Por ejemplo:

```yaml
- name: Mostrar versión
  run: echo "Versión: ${{ steps.version.outputs.value }}"
```

El `id` es fundamental:

```yaml
id: version
```

porque permite referenciar el step programáticamente.

---

# 36. Outputs entre jobs

También podemos exponer un output del job.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.version.outputs.value }}

    steps:
      - id: version
        run: echo "value=1.2.3" >> "$GITHUB_OUTPUT"

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - run: echo "Versión ${{ needs.build.outputs.version }}"
```

La cadena es:

```text
Step Output
    ↓
Job Output
    ↓
needs
    ↓
Otro Job
```

---

# 37. `strategy`

`strategy` permite ejecutar un job con distintas configuraciones.

El patrón más conocido utiliza:

```yaml
strategy:
  matrix:
```

Por ejemplo:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version:
          - 20
          - 22

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - run: npm test
```

GitHub generará ejecuciones para las diferentes versiones definidas en la matriz.

---

# 38. Matrices con varias dimensiones

También podemos combinar valores:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest

    node:
      - 20
      - 22
```

Y utilizar:

```yaml
runs-on: ${{ matrix.os }}
```

junto con:

```yaml
node-version: ${{ matrix.node }}
```

Esto permite probar múltiples combinaciones.

---

# 39. `timeout-minutes`

Podemos limitar cuánto tiempo puede ejecutar un job:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
```

También puede utilizarse para un step:

```yaml
- name: Ejecutar proceso
  timeout-minutes: 5
  run: ./script.sh
```

Esto evita ejecuciones que permanezcan activas indefinidamente.

---

# 40. `continue-on-error`

Podemos permitir que un step falle sin detener automáticamente el job:

```yaml
- name: Comprobación opcional
  continue-on-error: true
  run: ./optional-check.sh
```

Debe utilizarse deliberadamente.

No conviene ocultar un error que debería bloquear el workflow.

---

# 41. Artifacts

Un artifact permite conservar archivos producidos durante la ejecución.

Por ejemplo:

```yaml
- name: Upload test results
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: results/
```

Esto resulta útil para conservar:

- resultados de tests
- reportes
- logs
- builds
- archivos generados

Los artifacts quedan asociados al workflow run.

---

# 42. Descargar artifacts

Otro job puede descargar un artifact.

```yaml
- name: Descargar resultados
  uses: actions/download-artifact@v4
  with:
    name: test-results
```

Esto permite transportar archivos entre jobs.

---

# 43. `environment`

Un job puede asociarse a un GitHub Environment.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest

    environment:
      name: production

    steps:
      - run: ./deploy.sh
```

Los Environments pueden utilizarse para controlar deployments, protecciones, aprobaciones y acceso a secretos específicos del entorno.

---

# 44. `defaults`

Podemos definir valores predeterminados para comandos `run`.

Por ejemplo:

```yaml
defaults:
  run:
    shell: bash
    working-directory: ./app
```

Entonces los steps que utilicen `run` heredarán esa configuración.

---

# 45. `shell`

Podemos indicar qué shell utilizar.

```yaml
- name: Ejecutar script
  shell: bash
  run: |
    echo "Ejecutando con Bash"
```

La disponibilidad y el comportamiento del shell dependen del runner.

---

# 46. Reutilizar valores con `env`, `vars` y `secrets`

Es importante diferenciar estos tres conceptos.

## `env`

Variable disponible durante la ejecución:

```text
${{ env.APP_ENV }}
```

## `vars`

Configuración almacenada en GitHub:

```text
${{ vars.APP_ENV }}
```

## `secrets`

Información sensible almacenada como Secret:

```text
${{ secrets.API_TOKEN }}
```

Regla mental:

```text
env
→ entorno de ejecución

vars
→ configuración

secrets
→ valores sensibles
```

---

# 47. Ejemplo completo: CI para un Pull Request

```yaml
name: CI

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read

jobs:
  test:
    name: Tests

    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "22"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

Leído de arriba hacia abajo:

```text
name
↓
CI

on
↓
Pull Request hacia main

permissions
↓
solo lectura del contenido

jobs
↓
test

runs-on
↓
Ubuntu

steps
↓
checkout
setup Node
instalar dependencias
ejecutar tests
```

---

# 48. Ejemplo completo: varios jobs

```yaml
name: Validation

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: npm ci

      - run: npm test

  lint:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: npm ci

      - run: npm run lint

  summary:
    needs:
      - test
      - lint

    runs-on: ubuntu-latest

    steps:
      - run: echo "Validaciones completadas"
```

Tenemos:

```text
           ┌── test ──┐
trigger ───┤          ├── summary
           └── lint ──┘
```

---

# 49. Ejemplo completo: ejecución manual con input

```yaml
name: Manual deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Environment"
        required: true
        type: choice
        options:
          - dev
          - staging

permissions:
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Mostrar destino
        run: echo "Deploy a ${{ inputs.environment }}"
```

---

# 50. Ejemplo completo: condición por evento

```yaml
name: Validation

on:
  pull_request:
    branches:
      - main

  workflow_dispatch:

permissions:
  contents: read

jobs:
  information:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Evento ${{ github.event_name }}"

  validate-pr:
    if: github.event_name == 'pull_request'

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: echo "Validando Pull Request"
```

El workflow puede iniciarse de dos maneras.

Pero `validate-pr` tiene un control adicional:

```yaml
if: github.event_name == 'pull_request'
```

Por lo tanto, ese job solamente se ejecuta cuando el evento fue un Pull Request.

---

# 51. Ejemplo completo con Secret

```yaml
name: Secret example

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  example:
    runs-on: ubuntu-latest

    steps:
      - name: Ejecutar herramienta
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
        run: ./tool.sh
```

El workflow conoce:

```text
secrets.API_TOKEN
```

pero el comando recibe:

```text
API_TOKEN
```

como variable de entorno.

La credencial no está escrita directamente en el archivo.

---

# 52. Cómo leer un workflow que no conocemos

Cuando encuentres un workflow grande, no intentes entender todo al mismo tiempo.

Leelo en este orden.

## 1. ¿Cómo se llama?

Buscá:

```yaml
name:
```

## 2. ¿Qué lo dispara?

Buscá:

```yaml
on:
```

## 3. ¿Qué permisos tiene?

Buscá:

```yaml
permissions:
```

## 4. ¿Qué jobs existen?

Buscá:

```yaml
jobs:
```

## 5. ¿Qué runner utiliza cada job?

Buscá:

```yaml
runs-on:
```

## 6. ¿Hay dependencias?

Buscá:

```yaml
needs:
```

## 7. ¿Hay condiciones?

Buscá:

```yaml
if:
```

## 8. ¿Qué hace cada step?

Buscá:

```yaml
run:
```

y:

```yaml
uses:
```

## 9. ¿Recibe parámetros?

Buscá:

```yaml
with:
```

## 10. ¿Utiliza credenciales?

Buscá referencias a:

```text
secrets.
```

## 11. ¿Produce outputs?

Buscá:

```yaml
outputs:
```

y:

```text
$GITHUB_OUTPUT
```

---

# 53. Mapa mental de GitHub Actions

Un workflow puede pensarse como:

```text
EVENTO
  │
  │ on
  ▼
WORKFLOW
  │
  ├── permissions
  ├── env
  │
  ▼
JOBS
  │
  ├── Job A
  │    ├── runs-on
  │    ├── permissions
  │    ├── env
  │    ├── if
  │    │
  │    └── steps
  │          ├── uses
  │          ├── run
  │          ├── with
  │          └── env
  │
  └── Job B
       ├── needs: Job A
       └── steps
```

---

# 54. Chuleta rápida

## Workflow

```yaml
name: Mi workflow
```

## Trigger manual

```yaml
on:
  workflow_dispatch:
```

## Push

```yaml
on:
  push:
    branches:
      - main
```

## Pull Request

```yaml
on:
  pull_request:
    branches:
      - main
```

## Permisos

```yaml
permissions:
  contents: read
```

## Job

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

## Step con comando

```yaml
- name: Test
  run: npm test
```

## Step con Action

```yaml
- uses: actions/checkout@v4
```

## Action con parámetros

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: "22"
```

## Variable de entorno

```yaml
env:
  NODE_ENV: test
```

## Secret

```yaml
env:
  TOKEN: ${{ secrets.API_TOKEN }}
```

## Variable de configuración

```text
${{ vars.APP_ENV }}
```

## Contexto GitHub

```text
${{ github.event_name }}
```

## Condición

```yaml
if: github.ref == 'refs/heads/main'
```

## Dependencia entre jobs

```yaml
needs: build
```

## Output de step

```yaml
- id: version
  run: echo "value=1.2.3" >> "$GITHUB_OUTPUT"
```

## Leer output

```text
${{ steps.version.outputs.value }}
```

## Artifact

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: results
    path: results/
```

## Environment

```yaml
environment:
  name: production
```

---

# 55. Plantilla base para empezar

Cuando necesites escribir un workflow desde cero, podés partir de esta estructura:

```yaml
name: Nombre del workflow

on:
  workflow_dispatch:

permissions:
  contents: read

env:
  VARIABLE_GLOBAL: valor

jobs:
  ejemplo:
    name: Nombre del job

    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Ejecutar comando
        run: echo "Hola"

      - name: Ejecutar con variable
        env:
          MI_VARIABLE: valor
        run: echo "$MI_VARIABLE"
```

Y agregar solamente lo que necesites.

---

# 56. De GitHub Actions a Agentic Workflows

Entender esta estructura es importante porque un Agentic Workflow sigue ejecutándose sobre GitHub Actions.

Conceptos como:

```yaml
on:
  workflow_dispatch:
```

siguen definiendo cuándo comienza la ejecución.

GitHub Actions continúa proporcionando la infraestructura de ejecución, y los workflows generados siguen utilizando conceptos como permisos, jobs, steps, contextos y secrets.

Por eso, antes de trabajar con un workflow agéntico, deberías poder reconocer inmediatamente:

```text
on
permissions
jobs
runs-on
steps
uses
run
with
env
secrets
if
needs
outputs
```

No necesitás memorizar todas sus posibilidades.

Sí necesitás poder abrir un workflow y responder:

1. ¿Qué lo dispara?
2. ¿Qué permisos tiene?
3. ¿Qué jobs ejecuta?
4. ¿Dónde se ejecutan?
5. ¿En qué orden?
6. ¿Qué Actions utiliza?
7. ¿Qué comandos ejecuta?
8. ¿Qué secrets consume?
9. ¿Qué condiciones controlan la ejecución?
10. ¿Qué outputs o artifacts produce?

Si podés responder esas preguntas leyendo un archivo `.yml`, ya tenés la base necesaria para comprender cómo GitHub Actions ejecuta automatizaciones y cómo esa infraestructura se utiliza posteriormente para ejecutar workflows agénticos.
