# Human in the Loop con GitHub Copilot CLI y Pull Requests

# Objetivos

En este laboratorio aprenderás a incorporar revisión humana dentro del trabajo realizado por un agente de GitHub Copilot CLI.

* Crear un repositorio en GitHub.
* Abrir el repositorio utilizando GitHub Codespaces.
* Crear una aplicación JavaScript sencilla.
* Crear pruebas automatizadas.
* Configurar GitHub Actions para validar Pull Requests.
* Configurar una regla para proteger la branch `main`.
* Requerir un Pull Request para integrar cambios.
* Requerir validaciones automáticas antes del merge.
* Requerir aprobación humana antes del merge.
* Crear un agente de implementación con GitHub Copilot CLI.
* Permitir que el agente implemente un cambio y cree un Pull Request.
* Revisar manualmente el Pull Request.
* Solicitar cambios al agente.
* Permitir que el agente responda al feedback humano.
* Aprobar el Pull Request después de verificar la corrección.
* Realizar el merge solamente después de completar la revisión humana.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Copilot.
* Acceso a GitHub Codespaces.
* Acceso a GitHub Actions.
* Permisos para configurar reglas del repositorio.
* Un segundo usuario de GitHub para realizar la revisión del Pull Request.
* Conocimientos básicos de JavaScript.
* Conocimientos básicos de Git y GitHub.
* Tiempo estimado: 45 a 60 minutos.

> [!IMPORTANT]
> Para demostrar una aprobación humana real, este laboratorio debe realizarse en parejas o utilizando dos cuentas de GitHub.
>
> El autor del Pull Request no debe ser quien realice la aprobación requerida.

# Pasos

## Crear un repositorio

* [BROWSER] GitHub
  * Iniciar sesión con una cuenta de GitHub.
  * [BUTTON] New
  * En `Repository name`, ingresar:

```text
human-review-lab
```

  * Seleccionar `Public`.
  * Seleccionar `Add a README file`.
  * [BUTTON] Create repository

* Esperar hasta que se muestre el nuevo repositorio.

## Agregar un colaborador

Para que otra persona pueda revisar el Pull Request:

* [BROWSER] Repositorio `human-review-lab`
  * [BUTTON] Settings
  * Seleccionar `Collaborators`.
  * [BUTTON] Add people
  * Agregar al usuario que realizará la revisión.

El segundo usuario debe aceptar la invitación antes de continuar.

## Abrir el repositorio en GitHub Codespaces

* [BROWSER] Repositorio `human-review-lab`
  * [BUTTON] Code
  * Seleccionar `Codespaces`.
  * [BUTTON] Create codespace on main

* Esperar hasta que GitHub Codespaces termine de iniciar.
* Si aparece una solicitud de confianza:
  * [BUTTON] I Trust
* Verificar que la terminal esté disponible.

## Crear la aplicación inicial

Crear las carpetas:

```text
src
test
.github/workflows
```

* [CONSOLE]

```bash
mkdir -p src test .github/workflows
```

Crear:

```text
src/profile.js
```

* [FILE] `src/profile.js`

Agregar:

```javascript
function createProfile(name, email) {
  return {
    name,
    email
  }
}

module.exports = { createProfile }
```

Crear:

```text
test/profile.test.js
```

* [FILE] `test/profile.test.js`

Agregar:

```javascript
const test = require("node:test")
const assert = require("node:assert")
const { createProfile } = require("../src/profile")

test("creates a profile", () => {
  const profile = createProfile("Mona", "mona@example.com")

  assert.equal(profile.name, "Mona")
  assert.equal(profile.email, "mona@example.com")
})
```

## Ejecutar las pruebas

* [CONSOLE]

```bash
node --test
```

Verificar que la prueba finalice correctamente.

## Crear la validación automática

Crear:

```text
.github/workflows/test.yml
```

* [FILE] `.github/workflows/test.yml`

Agregar:

```yaml
name: Profile tests

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

## Guardar la aplicación inicial

* [CONSOLE]

```bash
git add .
```

* [CONSOLE]

```bash
git commit -m "Create initial profile application"
```

* [CONSOLE]

```bash
git push
```

Esperar hasta que finalice el push.

## Crear una branch temporal para registrar el check

Antes de convertir `Run automated tests` en un required check, haremos que GitHub lo ejecute al menos una vez.

* [CONSOLE]

```bash
git switch -c setup/verify-ci
```

* [CONSOLE]

```bash
echo "" >> README.md
git add README.md
git commit -m "Verify pull request validation"
git push -u origin setup/verify-ci
```

Crear un Pull Request hacia:

```text
main
```

Esperar hasta que aparezca:

```text
Run automated tests
```

Verificar que finalice correctamente.

Realizar el merge de este Pull Request.

Volver al Codespace.

* [CONSOLE]

```bash
git switch main
git pull
```

## Proteger la branch main

Ahora configuraremos las condiciones que debe cumplir un cambio antes de poder integrarse.

* [BROWSER] Repositorio `human-review-lab`
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

Configurar:

```text
Required approvals: 1
```

Habilitar:

```text
Require status checks to pass
```

Agregar como required check:

```text
Run automated tests
```

Guardar y activar el ruleset.

> [!IMPORTANT]
> A partir de este momento, que el código exista y que los tests pasen no es suficiente.
>
> El Pull Request también deberá cumplir la condición de revisión configurada en el repositorio.

## Crear el agente Implementation Agent

Crear:

```text
.github/agents/implementation-agent.agent.md
```

* [FILE] `.github/agents/implementation-agent.agent.md`

Agregar:

```markdown
---
name: Implementation Agent
description: Implements requested changes and submits them through Pull Requests.
---

# Role

You are an implementation agent.

Your responsibility is to implement requested code changes.

# Workflow

1. Inspect the repository.
2. Create a dedicated branch.
3. Implement the requested change.
4. Add or update tests.
5. Run the tests.
6. Commit the changes.
7. Push the branch.
8. Create a Pull Request.
9. Respond to review feedback when requested.

# Restrictions

Do not work directly on main.

Do not merge Pull Requests.

Do not modify files under `.github/workflows/`.

The final decision to accept the change belongs to a human reviewer.
```

Guardar el archivo.

## Publicar el agente

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

## Solicitar un cambio al agente

Con `Implementation Agent` seleccionado, ingresar:

```text
Update src/profile.js so createProfile validates its input.

Requirements:

- name cannot be empty;
- email cannot be empty;
- email must contain @;
- invalid input must throw an Error;
- existing valid behavior must continue working.

Add automated tests for the new validation.

Create a branch named feature/profile-validation.

Run the tests.

Commit and push the changes.

Create a Pull Request against main.

Do not modify .github/workflows/.

Do not merge the Pull Request.
```

Permitir que el agente realice las operaciones necesarias.

El agente debería:

* crear una branch;
* modificar `src/profile.js`;
* agregar pruebas;
* ejecutar `node --test`;
* crear un commit;
* realizar push;
* abrir un Pull Request.

## Abrir el Pull Request

* [BROWSER] Repositorio `human-review-lab`
  * Seleccionar `Pull requests`.
  * Abrir el Pull Request creado por el agente.

Verificar los archivos modificados.

Deberían incluir:

```text
src/profile.js
test/profile.test.js
```

No debería aparecer:

```text
.github/workflows/test.yml
```

## Esperar la validación automática

En el Pull Request, esperar la ejecución de:

```text
Run automated tests
```

Verificar que el check termine correctamente.

El resultado esperado es:

```text
Run automated tests ✅
```

## Comprobar que todavía falta una persona

Aunque los tests hayan finalizado correctamente, verificar el estado del Pull Request.

La aprobación humana requerida todavía no existe.

No realizar todavía el merge.

## Realizar la revisión humana

Ahora utilizar el segundo usuario.

* [BROWSER] Pull Request
  * Seleccionar `Files changed`.

Revisar:

```text
src/profile.js
```

y:

```text
test/profile.test.js
```

La persona revisora no aprobará el cambio todavía.

## Solicitar un cambio

El reviewer debe agregar el siguiente comentario:

```text
The validation works, but the error messages should identify
which field is invalid.

Please update the implementation so validation errors explicitly
mention "name" or "email", and add tests for the error messages.
```

Después:

* [BUTTON] Review changes
* Seleccionar:

```text
Request changes
```

* [BUTTON] Submit review

## Verificar el resultado de la revisión

Volver a la página principal del Pull Request.

Verificar que aparezca la revisión:

```text
Changes requested
```

El código existe.

Los tests pasaron.

Pero la persona responsable todavía no acepta el cambio.

## Volver al agente

Volver al Codespace y a GitHub Copilot CLI.

Con `Implementation Agent` seleccionado, ingresar:

```text
Inspect the review feedback on the current Pull Request.

Apply the requested changes.

Update the validation error messages so they identify
the invalid field.

Add tests that verify the error messages.

Run all tests.

Commit the changes and push them to the existing branch.

Do not create a new Pull Request.

Do not merge the Pull Request.
```

Esperar hasta que el agente termine.

El agente debería:

* consultar el feedback del Pull Request;
* modificar la implementación;
* agregar o modificar pruebas;
* ejecutar las pruebas;
* crear un nuevo commit;
* actualizar la misma branch;
* actualizar el Pull Request existente.

## Verificar la nueva versión

Volver al Pull Request.

Abrir:

```text
Files changed
```

Comprobar que los mensajes de error identifiquen:

```text
name
```

o:

```text
email
```

según corresponda.

Revisar las nuevas pruebas.

## Esperar nuevamente GitHub Actions

El nuevo push debería iniciar nuevamente:

```text
Run automated tests
```

Esperar hasta que finalice.

Verificar:

```text
Run automated tests ✅
```

## Realizar la segunda revisión humana

El segundo usuario debe volver a revisar el Pull Request.

Comprobar:

* que se respondió al feedback;
* que los mensajes identifican el campo incorrecto;
* que existen pruebas para esa condición;
* que las pruebas automáticas están aprobadas;
* que no se modificaron archivos fuera del alcance.

Después:

* [BUTTON] Review changes
* Seleccionar:

```text
Approve
```

* [BUTTON] Submit review

## Verificar que se cumplan las condiciones

Volver a la página principal del Pull Request.

Verificar:

```text
Automated tests    Passed
Human review       Approved
```

El cambio pasó por:

```text
Implementación
Validación automática
Feedback humano
Corrección
Nueva validación
Aprobación humana
```

## Realizar el merge

El merge debe realizarlo una persona.

* [BROWSER] Pull Request
  * [BUTTON] Merge pull request
  * Confirmar el merge.

Esperar hasta que aparezca:

```text
Merged
```

## Revisar el historial del Pull Request

Revisar la conversación del Pull Request.

Deberían quedar registrados:

* el Pull Request inicial creado por el agente;
* la primera ejecución de los checks;
* el `Request changes`;
* el comentario de la persona revisora;
* el nuevo commit del agente;
* la nueva ejecución de los checks;
* la aprobación humana;
* el merge.

# ¿Qué ocurrió?

Durante el laboratorio:

1. Configuramos `main` para exigir un Pull Request.
2. Configuramos una validación automática.
3. Configuramos una aprobación humana requerida.
4. Creamos un agente de implementación.
5. El agente creó una branch.
6. El agente implementó el cambio.
7. El agente agregó pruebas.
8. El agente abrió un Pull Request.
9. GitHub Actions validó automáticamente el cambio.
10. Una persona revisó el Pull Request.
11. La persona solicitó modificaciones.
12. El agente recibió el feedback.
13. El agente corrigió el cambio.
14. GitHub Actions volvió a validar el Pull Request.
15. La persona revisó nuevamente el resultado.
16. La persona aprobó el cambio.
17. El Pull Request se integró después de completar las condiciones requeridas.

# Conclusiones

En este laboratorio aprendimos a:

* Utilizar un Pull Request como punto de control entre un agente y una persona.
* Permitir que un agente implemente un cambio sin darle autoridad para aceptarlo.
* Utilizar GitHub Actions para validar automáticamente el código.
* Requerir una revisión humana antes del merge.
* Solicitar cambios sobre una contribución generada por un agente.
* Permitir que el agente responda al feedback humano.
* Ejecutar nuevamente las validaciones después de cada modificación.
* Aprobar el cambio solamente cuando el resultado sea aceptable.
* Mantener la decisión final de integración en manos de una persona.

El flujo implementado fue:

```text
Implementation Agent
        ↓
     Branch
        ↓
  Pull Request
        ↓
GitHub Actions
        ↓
      Tests
        ↓
  Human Review
        ↓
Request Changes
        ↓
Implementation Agent
        ↓
    Correction
        ↓
GitHub Actions
        ↓
      Tests
        ↓
  Human Review
        ↓
     Approve
        ↓
      Merge
```

El agente participa en el desarrollo y responde al feedback.

La persona mantiene la decisión de aceptar o rechazar el cambio.
