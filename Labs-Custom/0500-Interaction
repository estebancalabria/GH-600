# GitHub Copilot CLI: estado persistente, validación automática y Human in the Loop

# Objetivos

En este laboratorio aprenderás a utilizar GitHub Copilot CLI junto con GitHub Actions y los controles de GitHub para implementar una tarea mediante un agente, conservar el estado del trabajo entre sesiones y requerir validaciones antes de integrar el cambio.

* Crear un repositorio en GitHub.
* Abrir el repositorio utilizando GitHub Codespaces.
* Crear una pequeña aplicación JavaScript.
* Crear pruebas automatizadas para la aplicación.
* Configurar un workflow de GitHub Actions para ejecutar las pruebas.
* Crear un Issue con una tarea y criterios de aceptación.
* Crear un agente personalizado de implementación.
* Utilizar GitHub Copilot CLI para implementar la tarea.
* Trabajar sobre una branch separada de `main`.
* Crear un Pull Request desde GitHub Copilot CLI.
* Utilizar GitHub Actions para validar automáticamente el Pull Request.
* Observar el estado del trabajo mediante el Issue, la branch, los commits, el Pull Request y los checks.
* Cerrar la sesión de GitHub Copilot CLI.
* Iniciar una nueva sesión y reconstruir el contexto utilizando el estado almacenado en GitHub.
* Configurar una regla para proteger la branch `main`.
* Requerir un status check antes del merge.
* Comprobar que el Pull Request queda sujeto a los controles configurados.
* Integrar el cambio solamente después de cumplir las condiciones definidas.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Copilot.
* Acceso a GitHub Codespaces.
* GitHub Copilot CLI disponible en el Codespace.
* Conocimientos básicos de JavaScript.
* Conocimientos básicos de Git y GitHub.
* Conocimientos básicos de Markdown.
* Tiempo estimado: 60 a 75 minutos.

> [!NOTE]
> Este laboratorio utiliza dos mecanismos diferentes:
>
> * GitHub Copilot CLI para ejecutar el trabajo del agente.
> * GitHub Actions para validar automáticamente el cambio.
>
> No utilizaremos GitHub Agentic Workflows (`gh-aw`) en este laboratorio.

# Pasos

## Crear un repositorio

* [BROWSER] GitHub
  * Iniciar sesión con una cuenta de GitHub.
  * [BUTTON] New
  * En `Repository name`, ingresar:

```text
agent-state-lab
```

  * Seleccionar `Public`.
  * Seleccionar `Add a README file`.
  * [BUTTON] Create repository

* Esperar hasta que se muestre el nuevo repositorio.

## Abrir el repositorio en GitHub Codespaces

* [BROWSER] Repositorio `agent-state-lab`
  * [BUTTON] Code
  * Seleccionar la pestaña `Codespaces`.
  * [BUTTON] Create codespace on main

* Esperar hasta que GitHub Codespaces termine de iniciar.
* Si aparece una solicitud de confianza:
  * [BUTTON] I Trust
* Verificar que la terminal esté disponible.

## Crear la aplicación inicial

En la terminal crear las carpetas necesarias:

* [CONSOLE]

```bash
mkdir -p src test .github/workflows
```

Crear:

```text
src/registration.js
```

* [FILE] `src/registration.js`

```javascript
function registerUser(username, email) {
  return {
    username,
    email,
    registered: true
  }
}

module.exports = { registerUser }
```

Crear:

```text
test/registration.test.js
```

* [FILE] `test/registration.test.js`

```javascript
const test = require("node:test")
const assert = require("node:assert")
const { registerUser } = require("../src/registration")

test("registers a valid user", () => {
  const user = registerUser("mona", "mona@example.com")

  assert.equal(user.username, "mona")
  assert.equal(user.email, "mona@example.com")
  assert.equal(user.registered, true)
})
```

La estructura debería quedar:

```text
agent-state-lab
│
├── src
│   └── registration.js
│
├── test
│   └── registration.test.js
│
└── README.md
```

## Ejecutar las pruebas

* [CONSOLE]

```bash
node --test
```

* Verificar que la prueba finalice correctamente.
* El resultado debería indicar que existe una prueba aprobada.

## Crear el workflow de validación

Antes de permitir que el agente realice cambios, crearemos el mecanismo que validará los Pull Requests.

Crear:

```text
.github/workflows/test.yml
```

* [FILE] `.github/workflows/test.yml`

```yaml
name: Registration tests

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read

jobs:
  test:
    name: Run automated tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22"

      - name: Run tests
        run: node --test
```

> [!NOTE]
> El workflow se ejecutará cuando exista un Pull Request dirigido a `main`.
>
> La validación todavía no se ejecuta porque aún no existe ningún Pull Request.

## Guardar el estado inicial del repositorio

* [CONSOLE]

```bash
git add .
```

* [CONSOLE]

```bash
git commit -m "Create initial registration application and CI"
```

* [CONSOLE]

```bash
git push
```

* Esperar hasta que finalice el push.

Ahora el workflow de validación ya forma parte de `main`.

> [!IMPORTANT]
> El agente que utilizaremos posteriormente no necesitará modificar `.github/workflows/`.
>
> El mecanismo de validación ya existe antes de comenzar el trabajo del agente.

## Crear una tarea mediante un Issue

Ahora crearemos una tarea que deberá resolver el agente.

* [BROWSER] Repositorio `agent-state-lab`
  * Seleccionar `Issues`.
  * [BUTTON] New issue

En el título ingresar:

```text
Add validation to user registration
```

En la descripción ingresar:

```markdown
## Objective

Add input validation to the user registration function.

## Acceptance criteria

- Username cannot be empty.
- Email cannot be empty.
- Email must contain `@`.
- Invalid input must throw an Error with a clear message.
- Existing valid registration behavior must continue working.
- Automated tests must cover the new validation rules.
- All automated tests must pass.

## Scope

The implementation may modify:

- src/
- test/

Do not modify:

- .github/workflows/

## Delivery

Create the implementation in a separate branch.

Submit the change using a Pull Request.

Do not merge the Pull Request.
```

* [BUTTON] Create

* Anotar el número del Issue.

Por ejemplo:

```text
#1
```

> [!IMPORTANT]
> A partir de este momento, el Issue contiene el objetivo, el alcance y los criterios de aceptación de la tarea.
>
> No volveremos a copiar todos esos requisitos dentro del prompt del agente.

## Verificar GitHub CLI

Desde el Codespace:

* [CONSOLE]

```bash
gh --version
```

* [CONSOLE]

```bash
gh auth status
```

Verificar que GitHub CLI esté autenticado.

## Verificar GitHub Copilot CLI

* [CONSOLE]

```bash
copilot --version
```

Verificar que GitHub Copilot CLI se encuentre disponible.

## Crear el agente Implementation Agent

Crear la carpeta:

```text
.github/agents
```

* [CONSOLE]

```bash
mkdir -p .github/agents
```

Crear:

```text
.github/agents/implementation-agent.agent.md
```

* [FILE] `.github/agents/implementation-agent.agent.md`

Agregar:

```markdown
---
name: Implementation Agent
description: Implements GitHub Issues using branches, tests, commits, and Pull Requests.
---

# Role

You are an implementation agent.

Your responsibility is to implement changes requested in GitHub Issues.

# Workflow

When implementing an Issue:

1. Read the Issue and its acceptance criteria.
2. Inspect the repository before making changes.
3. Create a dedicated branch for the task.
4. Modify only files required by the Issue.
5. Add or update automated tests.
6. Execute the tests locally.
7. Review the resulting git diff.
8. Commit the implementation.
9. Push the branch.
10. Create a Pull Request.
11. Reference the original Issue in the Pull Request.
12. Report the resulting Pull Request.

# Pull Request

The Pull Request must contain:

## Objective

Describe the objective from the Issue.

## Changes

Describe the implemented changes.

## Validation

Describe which tests were executed.

## Evidence

Identify the relevant commits and test results.

## Remaining work

Describe anything that must still happen before merge.

# Restrictions

Do not work directly on main.

Do not merge Pull Requests.

Do not modify files under `.github/workflows/`.

Do not change repository protections.

Do not remove or disable tests.

The final decision to merge belongs to a human.
```

* Guardar el archivo.

## Publicar la definición del agente

El agente debe existir en el repositorio antes de utilizarlo.

* [CONSOLE]

```bash
git add .github/agents/implementation-agent.agent.md
```

* [CONSOLE]

```bash
git commit -m "Add implementation agent"
```

* [CONSOLE]

```bash
git push
```

## Iniciar GitHub Copilot CLI

* [CONSOLE]

```bash
copilot
```

Esperar hasta que aparezca la interfaz interactiva.

Ingresar:

```text
/agent
```

Seleccionar:

```text
Implementation Agent
```

Verificar que el agente esté activo.

## Solicitar al agente que implemente el Issue

Reemplazar `#1` si el Issue tiene otro número.

* [COPILOT CLI]

```text
Implement GitHub Issue #1.

Follow the acceptance criteria and scope defined in the Issue.

Create a branch, implement the change, add the required tests,
run the tests, commit the changes, push the branch,
and create a Pull Request.

Do not merge the Pull Request.
```

* Permitir que el agente inspeccione el repositorio.
* Permitir las operaciones necesarias para implementar la tarea.
* Revisar las operaciones que solicita antes de autorizarlas.
* Esperar hasta que termine.

El agente debería:

* leer el Issue;
* crear una branch;
* modificar `src/registration.js`;
* actualizar o agregar pruebas;
* ejecutar `node --test`;
* crear un commit;
* publicar la branch;
* crear un Pull Request;
* relacionar el Pull Request con el Issue.

> [!IMPORTANT]
> El agente puede implementar y proponer el cambio.
>
> No debe realizar el merge.

## Verificar la branch creada

Salir temporalmente de Copilot CLI o abrir una terminal adicional.

* [CONSOLE]

```bash
git branch --show-current
```

Verificar que no se encuentre trabajando directamente sobre:

```text
main
```

## Verificar los cambios

* [CONSOLE]

```bash
git log --oneline --decorate -5
```

Verificar que exista un commit correspondiente a la implementación.

Revisar también:

```bash
git status --short
```

## Abrir el Pull Request

* [BROWSER] Repositorio `agent-state-lab`
  * Seleccionar `Pull requests`.
  * Abrir el Pull Request creado por el agente.

Verificar que el Pull Request incluya:

* el objetivo;
* los cambios realizados;
* la validación;
* evidencia;
* trabajo pendiente antes del merge;
* referencia al Issue.

## Observar la ejecución de GitHub Actions

Al abrir el Pull Request, GitHub Actions debería ejecutar automáticamente:

```text
Registration tests
```

* [BROWSER] Pull Request
  * Revisar la sección de checks.
  * Esperar hasta que termine la ejecución.

También puede revisarse:

* [BROWSER] Repositorio
  * Seleccionar `Actions`.
  * Seleccionar el workflow `Registration tests`.
  * Abrir la ejecución correspondiente al Pull Request.
  * Abrir el job:

```text
Run automated tests
```

* Revisar sus pasos.
* Abrir:

```text
Run tests
```

Verificar que `node --test` haya finalizado correctamente.

El Pull Request debería mostrar el check como aprobado.

## Identificar el estado persistente del trabajo

Antes de continuar, revisar los elementos que ya existen en GitHub.

* [BROWSER] Issue
  * Objetivo.
  * Alcance.
  * Criterios de aceptación.

* [BROWSER] Pull Request
  * Cambios.
  * Commits.
  * Referencia al Issue.
  * Estado de los checks.
  * Conversación.

* [BROWSER] Actions
  * Workflow run.
  * Job.
  * Logs.
  * Resultado de las pruebas.

En este momento el trabajo ya posee referencias persistentes:

```text
Issue
Branch
Commit
Pull Request
Workflow run
Status check
```

## Finalizar la sesión de Copilot CLI

Volver a GitHub Copilot CLI.

Finalizar completamente la sesión actual.

El objetivo de este paso es eliminar el historial temporal de esa sesión antes de continuar.

## Iniciar una nueva sesión

Desde la terminal:

```bash
copilot
```

Seleccionar nuevamente:

```text
Implementation Agent
```

No explicar manualmente qué ocurrió en la sesión anterior.

Ingresar solamente:

```text
Inspect the current repository state.

Find the open Pull Request related to the most recent implementation task.

Inspect:
- the related Issue;
- the Pull Request;
- the branch;
- the commits;
- the changed files;
- the current GitHub Actions checks.

Summarize:

1. What is the original objective?
2. What has already been completed?
3. Which acceptance criteria are satisfied?
4. What validation has been executed?
5. Did the validation pass or fail?
6. What remains before this change can be merged?

Do not modify files.
```

* Esperar la respuesta.

> [!IMPORTANT]
> El nuevo agente no dispone del historial conversacional de la sesión anterior.
>
> La información necesaria para reconstruir el trabajo debe encontrarse en el repositorio y en los elementos asociados al Pull Request.

## Comparar la reconstrucción con GitHub

Comprobar manualmente que la respuesta coincida con:

* el Issue;
* la branch;
* los commits;
* el Pull Request;
* los archivos modificados;
* el workflow run;
* el resultado del check.

No aceptar información que solamente aparezca en la respuesta del agente.

Verificarla contra los elementos existentes en GitHub.

## Configurar una regla para proteger main

Ahora agregaremos un control técnico al repositorio.

* [BROWSER] Repositorio `agent-state-lab`
  * [BUTTON] Settings
  * Seleccionar `Rules`.
  * Seleccionar `Rulesets`.
  * [BUTTON] New ruleset
  * Seleccionar `New branch ruleset`.

En el nombre ingresar:

```text
Protect main
```

Configurar el ruleset para que afecte a:

```text
main
```

Habilitar:

```text
Require a pull request before merging
```

Habilitar:

```text
Require status checks to pass
```

Agregar como required check el check producido por el workflow del laboratorio.

El nombre esperado será:

```text
Run automated tests
```

> [!NOTE]
> GitHub puede requerir que el check se haya ejecutado al menos una vez antes de poder seleccionarlo como required check.
>
> Por eso configuramos esta regla después de ejecutar el primer Pull Request.

* Guardar y activar el ruleset.

## Comprobar el control técnico

Volver al Pull Request.

Verificar que GitHub muestre que existen condiciones que deben cumplirse antes de integrar el cambio.

El Pull Request debe haber pasado:

```text
Run automated tests
```

Si el check no está aprobado, el cambio no debería cumplir las condiciones requeridas para la integración.

> [!IMPORTANT]
> La instrucción:
>
> ```text
> Do not merge the Pull Request.
> ```
>
> forma parte del comportamiento esperado del agente.
>
> El ruleset, en cambio, es un control configurado en GitHub que establece condiciones para integrar cambios en `main`.

## Realizar la revisión humana

Abrir:

```text
Files changed
```

Revisar los cambios realizados por el agente.

Comprobar:

* que `src/registration.js` contiene las validaciones solicitadas;
* que existen pruebas para username vacío;
* que existen pruebas para email vacío;
* que existen pruebas para email sin `@`;
* que la funcionalidad válida continúa funcionando;
* que `.github/workflows/test.yml` no fue modificado.

Comparar los cambios con los criterios de aceptación definidos originalmente en el Issue.

## Aprobar el cambio

> [!NOTE]
> Si el repositorio pertenece a una única cuenta y no existe un segundo usuario colaborador, GitHub puede limitar la demostración de una aprobación independiente del autor.
>
> Para demostrar una review humana completa, utilizar un segundo usuario o trabajar en parejas.

Con un segundo usuario:

* [BROWSER] Pull Request
  * Seleccionar `Files changed`.
  * [BUTTON] Review changes
  * Seleccionar:

```text
Approve
```

  * [BUTTON] Submit review

Volver a la página principal del Pull Request.

Verificar:

* check automático aprobado;
* revisión humana registrada;
* conversación y commits disponibles;
* cambio listo para integrarse según las reglas configuradas.

## Realizar el merge

El merge debe realizarlo una persona.

* [BROWSER] Pull Request
  * [BUTTON] Merge pull request
  * Confirmar el merge.

Esperar hasta que el Pull Request aparezca como:

```text
Merged
```

## Verificar el Issue

* [BROWSER] Issues

Si el Pull Request utilizó una referencia de cierre como:

```text
Closes #1
```

verificar que el Issue haya quedado cerrado al realizar el merge.

Si el agente solamente creó una referencia sin semántica de cierre, cerrar el Issue manualmente una vez validado el resultado.

## Verificar el resultado final en main

* [BROWSER] Repositorio
  * Seleccionar la branch:

```text
main
```

Verificar que contenga:

```text
.github/
├── agents/
│   └── implementation-agent.agent.md
└── workflows/
    └── test.yml

src/
└── registration.js

test/
└── registration.test.js

README.md
```

Abrir:

```text
src/registration.js
```

Verificar que las validaciones implementadas formen parte de `main`.

## Reconstruir toda la ejecución

Antes de finalizar el laboratorio, verificar que sea posible reconstruir qué ocurrió utilizando solamente GitHub.

Localizar:

```text
Issue
```

Contiene el objetivo y los criterios de aceptación.

Localizar:

```text
Branch
```

Representa dónde se realizó el trabajo.

Localizar:

```text
Commits
```

Muestran qué cambios realizó el agente.

Localizar:

```text
Pull Request
```

Concentra la propuesta, el diff, la conversación, los commits y la revisión.

Localizar:

```text
GitHub Actions workflow run
```

Registra la ejecución de la validación.

Localizar:

```text
Status check
```

Indica el resultado de la validación automatizada.

Localizar:

```text
Review
```

Registra la intervención humana.

Localizar:

```text
Merge
```

Registra la aceptación final del cambio.

La secuencia completa debería poder reconstruirse como:

```text
Issue
  ↓
Implementation Agent
  ↓
Branch
  ↓
Cambios + tests
  ↓
Commit
  ↓
Pull Request
  ↓
GitHub Actions
  ↓
Status Check
  ↓
Human Review
  ↓
Merge
```

# ¿Qué ocurrió?

Durante el laboratorio:

1. Creamos un repositorio desde cero.
2. Creamos una aplicación JavaScript y una prueba automatizada.
3. Configuramos un workflow de GitHub Actions antes de ejecutar al agente.
4. Creamos un Issue con un objetivo, alcance y criterios de aceptación.
5. Creamos un agente `Implementation Agent`.
6. El agente leyó la tarea desde el Issue.
7. El agente creó una branch separada de `main`.
8. El agente implementó la funcionalidad.
9. El agente agregó pruebas.
10. El agente ejecutó las pruebas localmente.
11. El agente creó un commit y publicó la branch.
12. El agente creó un Pull Request.
13. GitHub Actions ejecutó las pruebas independientemente de la evaluación del agente.
14. El resultado quedó registrado como un check del Pull Request.
15. Finalizamos la sesión de Copilot CLI.
16. Abrimos una sesión nueva.
17. El agente reconstruyó el trabajo utilizando el estado persistente almacenado en GitHub.
18. Configuramos un ruleset para proteger `main`.
19. Utilizamos el status check como condición de integración.
20. Una persona revisó el cambio.
21. El cambio se integró después de comprobar los requisitos establecidos.

El Issue conserva:

```text
Objetivo
Alcance
Criterios de aceptación
```

La branch y los commits conservan:

```text
Cambios realizados
Historial de implementación
```

El Pull Request conserva:

```text
Propuesta
Cambios
Commits
Checks
Conversación
Review
```

GitHub Actions conserva:

```text
Ejecución
Resultado
Logs
Status check
```

La revisión conserva:

```text
Intervención humana
Decisión sobre el cambio
```

# Conclusiones

En este laboratorio aprendimos a:

* Utilizar un Issue como entrada persistente para una tarea.
* Definir criterios de aceptación antes de ejecutar un agente.
* Utilizar un agente personalizado de GitHub Copilot CLI para implementar cambios.
* Evitar que el agente trabaje directamente sobre `main`.
* Utilizar branches y commits para conservar el progreso.
* Utilizar un Pull Request como punto central del estado de una contribución.
* Ejecutar validaciones independientes mediante GitHub Actions.
* Convertir los resultados de las pruebas en un status check visible.
* Finalizar una sesión de GitHub Copilot CLI y continuar desde una sesión nueva.
* Reconstruir el contexto utilizando Issue, Pull Request, commits y resultados de workflows.
* Utilizar un ruleset para establecer condiciones de integración.
* Separar las instrucciones proporcionadas al agente de los controles aplicados por GitHub.
* Mantener la decisión final de integración bajo control humano.
* Conservar evidencia suficiente para reconstruir el trabajo realizado.

El agente realiza:

```text
Issue
  ↓
Interpretar tarea
  ↓
Crear branch
  ↓
Implementar
  ↓
Agregar tests
  ↓
Commit
  ↓
Crear Pull Request
```

GitHub Actions realiza:

```text
Pull Request
  ↓
Ejecutar tests
  ↓
Generar evidencia
  ↓
Status Check
```

GitHub conserva el estado mediante:

```text
Issue
+
Branch
+
Commits
+
Pull Request
+
Workflow runs
+
Checks
+
Reviews
```

La decisión final permanece fuera del agente:

```text
Agente propone el cambio
        ↓
GitHub Actions lo valida
        ↓
GitHub aplica las reglas
        ↓
Una persona revisa
        ↓
Una persona decide el merge
```
