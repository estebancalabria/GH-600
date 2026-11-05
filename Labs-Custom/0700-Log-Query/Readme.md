# Laboratorio: Análisis de Tests con GitHub Agentic Workflows

# Objetivos

En este laboratorio aprenderás a crear un GitHub Agentic Workflow que ejecute tests sobre una aplicación, analice los resultados utilizando GitHub Copilot y genere automáticamente un informe.

* Crear un repositorio en GitHub.
* Abrir el repositorio utilizando GitHub Codespaces.
* Crear una aplicación mínima.
* Crear tests automatizados.
* Ejecutar los tests.
* Crear un Personal Access Token para utilizar GitHub Copilot desde GitHub Actions.
* Guardar el token como un Secret del repositorio.
* Instalar la extensión `gh-aw`.
* Crear un GitHub Agentic Workflow utilizando Markdown.
* Permitir que el agente ejecute los tests.
* Permitir que el agente analice el reporte y los logs generados.
* Generar un informe utilizando IA.
* Publicar el informe automáticamente como una Issue del repositorio.
* Compilar el Agentic Workflow.
* Ejecutar el agente desde GitHub Actions.
* Comprobar el resultado generado por el agente.

# Arquitectura del laboratorio

```text
GitHub Actions
      ↓
GitHub Agentic Workflow
      ↓
GitHub Copilot
      ↓
Execute Tests
      ↓
Analyze Test Results and Logs
      ↓
Generate Report
      ↓
Create GitHub Issue
```

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Actions.
* Acceso a GitHub Codespaces.
* Acceso a GitHub Copilot.
* Permisos para crear repositorios.
* Permisos para crear Secrets.
* GitHub CLI disponible.
* Tiempo estimado: 30 a 40 minutos.

# Crear un repositorio

* [BROWSER] GitHub
  * Iniciar sesión con una cuenta de GitHub.
  * [BUTTON] New
  * En `Repository name`, ingresar:

```text
ai-test-analysis-lab
```

  * Seleccionar `Public`.
  * Seleccionar `Add a README file`.
  * [BUTTON] Create repository

* Esperar hasta que se muestre el nuevo repositorio.

# Abrir el repositorio en GitHub Codespaces

* [BROWSER] Repositorio `ai-test-analysis-lab`
  * [BUTTON] Code
  * Seleccionar la pestaña `Codespaces`.
  * [BUTTON] Create codespace on main

* Esperar hasta que GitHub Codespaces termine de iniciar.
* Si aparece una solicitud de confianza:
  * [BUTTON] I Trust
* Verificar que la terminal esté disponible.

# Crear la aplicación

Crear el archivo:

```text
calculator.js
```

* [FILE] `calculator.js`

Agregar:

```javascript
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

function divide(a, b) {
  if (b === 0) {
    throw new Error("Division by zero is not allowed");
  }

  return a / b;
}

module.exports = {
  add,
  subtract,
  multiply,
  divide
};
```

* Guardar el archivo.

# Crear los tests

Crear la carpeta:

```text
tests
```

Dentro de la carpeta crear:

```text
calculator.test.js
```

La estructura debería quedar:

```text
ai-test-analysis-lab
│
├── tests
│   └── calculator.test.js
│
├── calculator.js
│
└── README.md
```

* [FILE] `tests/calculator.test.js`

Agregar:

```javascript
const test = require("node:test");
const assert = require("node:assert/strict");

const {
  add,
  subtract,
  multiply,
  divide
} = require("../calculator");

test("add returns the sum of two numbers", () => {
  assert.equal(add(2, 3), 5);
});

test("subtract returns the difference between two numbers", () => {
  assert.equal(subtract(10, 4), 6);
});

test("multiply returns the product of two numbers", () => {
  assert.equal(multiply(3, 4), 12);
});

test("divide returns the quotient of two numbers", () => {
  assert.equal(divide(10, 2), 5);
});

test("divide throws an error when divisor is zero", () => {
  assert.throws(
    () => divide(10, 0),
    /Division by zero is not allowed/
  );
});
```

* Guardar el archivo.

# Crear package.json

Crear:

```text
package.json
```

* [FILE] `package.json`

Agregar:

```json
{
  "name": "ai-test-analysis-lab",
  "version": "1.0.0",
  "description": "GitHub Agentic Workflow test analysis laboratory",
  "main": "calculator.js",
  "scripts": {
    "test": "node --test tests"
  },
  "license": "MIT"
}
```

* Guardar el archivo.

# Ejecutar los tests manualmente

Antes de utilizar IA, verificar que los tests funcionan.

* [CONSOLE]

```bash
npm test
```

El resultado debería indicar que los cinco tests fueron ejecutados correctamente.

Debería observarse un resultado similar a:

```text
tests 5
pass 5
fail 0
```

El resultado completo de esta ejecución incluye información que posteriormente podrá ser analizada por el agente.

# Crear el Personal Access Token

El agente utilizará GitHub Copilot como motor de IA.

Para este laboratorio utilizaremos un Fine-grained Personal Access Token.

* [BROWSER] GitHub
  * Seleccionar la imagen de perfil.
  * Seleccionar `Settings`.
  * Seleccionar `Developer settings`.
  * Seleccionar `Personal access tokens`.
  * Seleccionar `Fine-grained tokens`.
  * [BUTTON] Generate new token

En `Token name`, ingresar:

```text
Agentic Workflows Lab
```

Seleccionar una fecha de expiración apropiada para realizar el laboratorio.

En el acceso a repositorios seleccionar el repositorio:

```text
ai-test-analysis-lab
```

Configurar el permiso necesario para realizar solicitudes de GitHub Copilot.

Buscar:

```text
Copilot Requests
```

Configurar:

```text
Copilot Requests: Read and write
```

* [BUTTON] Generate token

Copiar el token generado.

> [!IMPORTANT]
> GitHub muestra el token una única vez.
>
> Copiarlo antes de abandonar la página.

# Crear el Secret

Ahora guardaremos el Personal Access Token como un Secret de GitHub Actions.

* [BROWSER] Repositorio `ai-test-analysis-lab`
  * Seleccionar `Settings`.
  * Seleccionar `Secrets and variables`.
  * Seleccionar `Actions`.
  * [BUTTON] New repository secret

En `Name`, escribir:

```text
COPILOT_GITHUB_TOKEN
```

En `Secret`, pegar el Personal Access Token generado anteriormente.

* [BUTTON] Add secret

Debería aparecer:

```text
COPILOT_GITHUB_TOKEN
```

> [!IMPORTANT]
> Nunca escribir el token directamente en el Agentic Workflow ni almacenarlo en el repositorio.

# Verificar GitHub CLI

Volver a GitHub Codespaces.

* [CONSOLE]

```bash
gh --version
```

GitHub CLI debería estar disponible.

Verificar la autenticación:

```bash
gh auth status
```

Si fuera necesario autenticarse:

```bash
gh auth login
```

# Instalar GitHub Agentic Workflows

Instalar la extensión `gh-aw`:

```bash
gh extension install github/gh-aw
```

Verificar la instalación:

```bash
gh aw --help
```

La terminal debería mostrar los comandos disponibles para trabajar con GitHub Agentic Workflows.

# Crear el GitHub Agentic Workflow

Crear las carpetas:

```text
.github
.github/workflows
```

Dentro de `.github/workflows`, crear:

```text
test-analyzer.md
```

La estructura debería quedar:

```text
ai-test-analysis-lab
│
├── .github
│   └── workflows
│       └── test-analyzer.md
│
├── tests
│   └── calculator.test.js
│
├── calculator.js
├── package.json
└── README.md
```

* [FILE] `.github/workflows/test-analyzer.md`

Agregar:

```markdown
---
name: AI Test Analyzer

on:
  workflow_dispatch:

safe-outputs:
  create-issue:
---

# AI Test Analyzer

Act as a software testing analyst.

Analyze the application contained in this repository.

## Tasks

1. Inspect `package.json` to determine how the automated tests are executed.

2. Execute the automated tests.

3. Capture and analyze the complete test output.

4. Determine:
   - How many tests were executed.
   - How many tests passed.
   - How many tests failed.
   - Which tests failed, if any.
   - The error messages produced by failed tests.
   - Possible causes of the failures.

5. Review the relevant source code when necessary to understand the test results.

6. Produce a concise technical assessment of the current state of the application.

7. Create a GitHub Issue containing the analysis.

## Report format

Use the following structure for the GitHub Issue:

# AI Test Analysis Report

## Test Summary

Include:

- Total tests
- Passed tests
- Failed tests

## Failed Tests

Describe any failed tests and their associated error messages.

If no tests failed, explicitly indicate that all tests passed.

## Analysis

Explain what the test results indicate about the current implementation.

## Recommendations

Provide concrete recommendations based only on the evidence obtained from the tests and repository.

## Conclusion

Indicate one of the following results:

- PASS
- FAIL
- REVIEW REQUIRED

Do not modify the application source code.

Do not modify the tests.

Do not invent tests, failures, errors, or metrics that are not present in the execution results.

Create a single GitHub Issue with the final report.
```

* Guardar el archivo.

# Comprender el Agentic Workflow

El archivo:

```text
.github/workflows/test-analyzer.md
```

no es un workflow tradicional de GitHub Actions.

Es la definición de un GitHub Agentic Workflow.

El bloque:

```yaml
on:
  workflow_dispatch:
```

permite ejecutarlo manualmente desde GitHub Actions.

La sección:

```yaml
safe-outputs:
  create-issue:
```

permite que el agente solicite la creación de una Issue como resultado de su trabajo.

El resto del archivo contiene las instrucciones en lenguaje natural que seguirá el agente.

En este caso deberá:

```text
Inspeccionar el repositorio
        ↓
Ejecutar npm test
        ↓
Leer stdout y stderr
        ↓
Analizar los tests
        ↓
Interpretar errores
        ↓
Generar un informe
        ↓
Crear una Issue
```

# Compilar el Agentic Workflow

Los GitHub Agentic Workflows escritos en Markdown deben compilarse para generar el workflow que podrá ejecutar GitHub Actions.

Desde la terminal ejecutar:

```bash
gh aw compile
```

Después de la compilación, dentro de:

```text
.github/workflows
```

debería encontrarse también el workflow compilado.

La estructura será similar a:

```text
.github
└── workflows
    ├── test-analyzer.md
    └── test-analyzer.lock.yml
```

El archivo:

```text
test-analyzer.md
```

es la definición que nosotros escribimos.

El archivo:

```text
test-analyzer.lock.yml
```

es el workflow generado para su ejecución mediante GitHub Actions.

> [!IMPORTANT]
> No editar manualmente `test-analyzer.lock.yml`.
>
> Las modificaciones deben realizarse sobre `test-analyzer.md` y luego volver a compilar el workflow.

# Publicar el laboratorio

Verificar los cambios:

```bash
git status
```

Agregar los archivos:

```bash
git add .
```

Crear el commit:

```bash
git commit -m "Add AI test analyzer agentic workflow"
```

Publicar:

```bash
git push
```

Esperar hasta que finalice el push.

# Ejecutar el Agentic Workflow

* [BROWSER] Repositorio `ai-test-analysis-lab`
  * Seleccionar `Actions`.

Buscar:

```text
AI Test Analyzer
```

Abrir el workflow.

* [BUTTON] Run workflow
* [BUTTON] Run workflow

Esperar hasta que aparezca la ejecución.

Abrir el workflow run.

# Observar la ejecución del agente

Durante la ejecución, GitHub Actions prepara el entorno necesario para ejecutar el Agentic Workflow.

El agente recibe las instrucciones definidas en:

```text
test-analyzer.md
```

El agente debería realizar tareas similares a:

```text
Inspect repository
        ↓
Read package.json
        ↓
Execute npm test
        ↓
Analyze output
        ↓
Inspect relevant files
        ↓
Generate report
        ↓
Create issue
```

Abrir los diferentes pasos del workflow para observar su ejecución.

Buscar en el log la ejecución de los tests.

Debería observarse información similar a:

```text
tests 5
pass 5
fail 0
```

# Verificar el análisis generado por IA

Una vez finalizado correctamente el workflow:

* [BROWSER] Repositorio
  * Seleccionar `Issues`.

Debería existir una nueva Issue creada por el Agentic Workflow.

Abrir la Issue.

El contenido debería seguir aproximadamente esta estructura:

```text
AI Test Analysis Report

Test Summary

Total tests: 5
Passed tests: 5
Failed tests: 0

Failed Tests

All tests passed.

Analysis

The automated tests completed successfully...

Recommendations

...

Conclusion

PASS
```

> [!NOTE]
> El texto exacto puede variar porque el informe es generado por un modelo de IA.

# Provocar un error

Ahora modificaremos uno de los tests para poder analizar un escenario con errores.

Abrir:

```text
tests/calculator.test.js
```

Buscar:

```javascript
test("add returns the sum of two numbers", () => {
  assert.equal(add(2, 3), 5);
});
```

Modificarlo por:

```javascript
test("add returns the sum of two numbers", () => {
  assert.equal(add(2, 3), 10);
});
```

Guardar el archivo.

# Publicar el cambio

* [CONSOLE]

```bash
git add .
```

```bash
git commit -m "Introduce failing test"
```

```bash
git push
```

# Ejecutar nuevamente el agente

* [BROWSER] Repositorio
  * Seleccionar `Actions`.
  * Seleccionar `AI Test Analyzer`.
  * [BUTTON] Run workflow
  * [BUTTON] Run workflow

Abrir la nueva ejecución.

Ahora el agente debería ejecutar nuevamente:

```text
npm test
```

Pero esta vez debería detectar al menos un test fallido.

El log debería contener información indicando que:

```text
2 + 3
```

produjo:

```text
5
```

mientras el test esperaba:

```text
10
```

# Revisar el nuevo informe

Cuando finalice el Agentic Workflow:

* [BROWSER] Repositorio
  * Seleccionar `Issues`.

Abrir la nueva Issue generada.

El agente debería haber identificado:

* La existencia de un test fallido.
* El test que produjo el error.
* El valor esperado.
* El valor obtenido.
* La posible causa del fallo.
* Una recomendación basada en la evidencia disponible.

La conclusión debería ser:

```text
FAIL
```

# Comparar las dos ejecuciones

Ahora deberían existir al menos dos informes generados automáticamente.

Primera ejecución:

```text
Tests
  ↓
5 passed
  ↓
AI Analysis
  ↓
PASS
```

Segunda ejecución:

```text
Tests
  ↓
4 passed
1 failed
  ↓
AI Analysis
  ↓
FAIL
```

La diferencia importante es que no programamos mediante JavaScript o Python la lógica necesaria para interpretar cada posible error.

Definimos el objetivo mediante instrucciones:

```text
Execute the tests.
Analyze the results.
Identify failures.
Explain possible causes.
Generate a report.
```

El agente utiliza el contexto disponible para decidir cómo realizar ese trabajo.

# Restaurar el test

Volver a:

```text
tests/calculator.test.js
```

Restaurar:

```javascript
test("add returns the sum of two numbers", () => {
  assert.equal(add(2, 3), 5);
});
```

Guardar y publicar:

```bash
git add .
git commit -m "Restore passing test"
git push
```

# Archivos finales del repositorio

Al finalizar el laboratorio, la estructura del repositorio debería ser similar a:

```text
ai-test-analysis-lab
│
├── .github
│   └── workflows
│       ├── test-analyzer.md
│       └── test-analyzer.lock.yml
│
├── tests
│   └── calculator.test.js
│
├── calculator.js
├── package.json
└── README.md
```

# Conclusiones

En este laboratorio aprendimos a:

* Crear un repositorio desde cero.
* Crear una aplicación mínima.
* Crear y ejecutar tests automatizados.
* Crear un Personal Access Token para GitHub Copilot.
* Almacenar el token utilizando `COPILOT_GITHUB_TOKEN`.
* Instalar la extensión `gh-aw`.
* Crear un GitHub Agentic Workflow.
* Escribir la definición del agente utilizando Markdown.
* Utilizar lenguaje natural para definir el trabajo del agente.
* Compilar el archivo Markdown a un workflow ejecutable por GitHub Actions.
* Ejecutar el agente desde GitHub Actions.
* Permitir que el agente ejecute tests.
* Permitir que la IA analice los resultados y logs de los tests.
* Detectar tests exitosos y tests fallidos.
* Generar automáticamente un informe técnico.
* Utilizar Safe Outputs para crear una Issue.
* Comprobar cómo cambia el análisis cuando cambia el resultado de los tests.

El funcionamiento completo del laboratorio es:

```text
Developer
    ↓
GitHub Repository
    ↓
GitHub Actions
    ↓
Agentic Workflow
    ↓
GitHub Copilot
    ↓
Execute Tests
    ↓
Read Test Output
    ↓
Reason About Results
    ↓
Generate Technical Report
    ↓
Safe Output
    ↓
GitHub Issue
```

Un GitHub Agentic Workflow combina la ejecución automatizada de GitHub Actions con la capacidad de razonamiento de un agente de IA.

En este laboratorio, GitHub Actions proporciona el entorno de ejecución mientras que el agente interpreta los resultados de los tests y genera un informe basado en la evidencia obtenida durante la ejecución.
