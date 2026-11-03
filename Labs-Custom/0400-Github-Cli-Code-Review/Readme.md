# Hello World con GitHub Copilot CLI: crear un agente revisor de código

# Objetivos

En este laboratorio aprenderás a crear y utilizar un agente personalizado de revisión de código con GitHub Copilot CLI.

* Crear un repositorio en GitHub.
* Abrir el repositorio utilizando GitHub Codespaces.
* Crear un pequeño archivo JavaScript con problemas intencionales.
* Ejecutar el código y observar su comportamiento.
* Iniciar GitHub Copilot CLI desde la terminal.
* Utilizar GitHub Copilot CLI para realizar una primera revisión del código.
* Crear un agente personalizado en `.github/agents/`.
* Definir el rol, alcance y formato de respuesta del agente.
* Seleccionar el agente desde GitHub Copilot CLI.
* Utilizar el agente para revisar `src/calculator.js`.
* Generar un informe de revisión en `docs/code-review.md`.
* Verificar que el agente no modifique el archivo que está revisando.
* Guardar el agente y el informe en el repositorio.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Copilot.
* Acceso a GitHub Codespaces.
* Conocimientos básicos de JavaScript.
* Conocimientos básicos de Markdown.
* Tiempo estimado: 25 a 35 minutos.

# Pasos

## Crear un repositorio

* [BROWSER] GitHub
  * Iniciar sesión con una cuenta de GitHub.
  * [BUTTON] New
  * En `Repository name`, ingresar:

```text
hello-copilot-cli
```

  * Seleccionar `Public`.
  * Seleccionar `Add a README file`.
  * [BUTTON] Create repository

* Esperar hasta que se muestre el nuevo repositorio.

## Abrir el repositorio en GitHub Codespaces

* [BROWSER] Repositorio `hello-copilot-cli`
  * [BUTTON] Code
  * Seleccionar la pestaña `Codespaces`.
  * [BUTTON] Create codespace on main

* Esperar hasta que GitHub Codespaces termine de iniciar.
* Se abrirá Visual Studio Code directamente en el navegador.
* Si aparece una solicitud de confianza:
  * [BUTTON] I Trust
* Verificar que en el panel izquierdo aparezca:

```text
README.md
```

* Verificar que la terminal esté disponible en la parte inferior de Visual Studio Code.

> [!NOTE]
> Los comandos de este laboratorio se ejecutarán desde la terminal integrada de GitHub Codespaces.

## Crear la aplicación de ejemplo

Crear la carpeta:

```text
src
```

Dentro de `src`, crear el archivo:

```text
calculator.js
```

La estructura inicial debería quedar de la siguiente manera:

```text
hello-copilot-cli
│
├── src
│   └── calculator.js
│
└── README.md
```

* [FILE] `src/calculator.js`
* Agregar el siguiente código:

```javascript
function calculate(a, b, operation) {
  if (operation == "add") return a + b
  if (operation == "subtract") return a - b
  if (operation == "multiply") return a * b
  if (operation == "divide") return a / b
}

console.log(calculate("10", 2, "add"))
console.log(calculate(10, 0, "divide"))
console.log(calculate(10, 2, "subtract"))
console.log(calculate(10, 2, "multiply"))
console.log(calculate(10, 2, "unknown"))
```

* Guardar el archivo.

> [!IMPORTANT]
> El código contiene problemas intencionales.
>
> No corregirlos todavía.
>
> El objetivo del laboratorio es utilizar un agente especializado para revisar el código.

## Ejecutar la aplicación

Antes de utilizar GitHub Copilot, ejecutar el código.

* [CONSOLE]

```bash
node src/calculator.js
```

* Observar el resultado.

Deberían aparecer valores similares a:

```text
102
Infinity
8
20
undefined
```

El resultado `102` ocurre porque JavaScript concatena el valor `"10"` con `2`.

El resultado `Infinity` aparece al realizar una división por cero.

El valor `undefined` corresponde a una operación que la función no reconoce.

Estos comportamientos serán parte del código que deberá analizar el agente.

## Identificar los problemas del ejemplo

Antes de utilizar el agente, observar el archivo `src/calculator.js`.

El código contiene varios problemas intencionales:

* utiliza `==` en lugar de `===`;
* no valida los tipos de entrada;
* acepta un `string` para una operación numérica;
* no controla la división por cero;
* no informa un error cuando recibe una operación desconocida;
* no posee pruebas automatizadas;
* no proporciona mensajes de error.

No corregir estos problemas.

El objetivo será solicitar a un agente que los identifique y produzca una revisión.

## Verificar GitHub Copilot CLI

* [CONSOLE]

```bash
copilot --version
```

* Verificar que GitHub Copilot CLI se encuentre disponible.

Si GitHub Copilot CLI requiere autenticación, completar el proceso solicitado utilizando la cuenta de GitHub asociada a GitHub Copilot.

## Iniciar GitHub Copilot CLI

* [CONSOLE]

```bash
copilot
```

* Esperar hasta que aparezca la interfaz interactiva de GitHub Copilot CLI.

A partir de este momento los siguientes prompts se ingresarán dentro de GitHub Copilot CLI.

## Realizar una primera revisión sin un agente personalizado

En GitHub Copilot CLI, ingresar:

```text
Review src/calculator.js and identify correctness, reliability,
and maintainability issues.

Do not modify any files.
```

* Esperar hasta que GitHub Copilot analice el archivo.
* Leer los problemas identificados.

La revisión debería permitir detectar problemas relacionados con aspectos como:

* comparación mediante `==`;
* ausencia de validación de tipos;
* concatenación al sumar `"10"` y `2`;
* división por cero;
* operaciones desconocidas;
* ausencia de errores explícitos;
* falta de pruebas automatizadas.

> [!NOTE]
> En este paso estamos utilizando GitHub Copilot CLI directamente.
>
> Todavía no existe un agente personalizado para realizar la revisión.

## Crear la carpeta de agentes

Ahora vamos a definir un agente especializado para realizar revisiones de código.

Desde Visual Studio Code, crear las carpetas:

```text
.github
```

y dentro:

```text
agents
```

La estructura debería quedar:

```text
hello-copilot-cli
│
├── .github
│   └── agents
│
├── src
│   └── calculator.js
│
└── README.md
```

## Crear el agente Code Reviewer

Dentro de `.github/agents`, crear el archivo:

```text
code-reviewer.agent.md
```

La estructura del repositorio será:

```text
hello-copilot-cli
│
├── .github
│   └── agents
│       └── code-reviewer.agent.md
│
├── src
│   └── calculator.js
│
└── README.md
```

* [FILE] `.github/agents/code-reviewer.agent.md`
* Agregar:

```markdown
---
name: Code Reviewer
description: Reviews JavaScript source code and produces a structured code review.
---

# Role

You are a code review agent.

Your responsibility is to inspect JavaScript source code and identify
correctness, reliability, maintainability, and input-handling problems.

# Scope

Review files under:

src/

Do not modify files under `src/`.

# Review rules

For every relevant finding:

1. Identify the problem.
2. Assign a severity:
   - High
   - Medium
   - Low
3. Explain the impact.
4. Identify the affected code.
5. Recommend a concrete correction.
6. Suggest a validation case.

# Output

When asked to produce a review document, write the review to:

docs/code-review.md

Use the following structure:

# Code Review

## Summary

Short summary of the review.

## Findings

For each finding include:

### Finding

**Severity:** High | Medium | Low

**Problem:**  
Describe the problem.

**Impact:**  
Explain why it matters.

**Recommendation:**  
Describe the recommended correction.

**Validation:**  
Describe how the correction could be tested.

## Suggested test cases

List the most important test cases that should be implemented.

# Restrictions

Do not modify source code.

Do not fix the problems.

Do not rewrite `src/calculator.js`.

Your responsibility is review and recommendations only.
```

* Guardar el archivo.

## Comprender la definición del agente

El archivo:

```text
.github/agents/code-reviewer.agent.md
```

define un agente especializado.

La sección:

```markdown
---
name: Code Reviewer
description: Reviews JavaScript source code and produces a structured code review.
---
```

define el nombre y la descripción del agente.

La sección:

```markdown
# Role
```

define su responsabilidad principal.

La sección:

```markdown
# Scope
```

define qué parte del repositorio debe analizar.

La sección:

```markdown
# Review rules
```

establece un proceso repetible para cada revisión.

La sección:

```markdown
# Output
```

define la estructura que debe tener el resultado.

Finalmente:

```markdown
# Restrictions
```

establece explícitamente que el agente no debe corregir ni modificar `src/calculator.js`.

> [!NOTE]
> Las instrucciones del agente definen el comportamiento esperado. Una instrucción como `Do not modify source code` no constituye por sí sola un control técnico de permisos.

## Seleccionar el agente personalizado

Volver a la sesión de GitHub Copilot CLI.

Ingresar:

```text
/agent
```

* Revisar los agentes disponibles.
* Seleccionar:

```text
Code Reviewer
```

* Verificar que `Code Reviewer` sea el agente activo.

## Ejecutar el agente Code Reviewer

Con `Code Reviewer` seleccionado, ingresar:

```text
Review src/calculator.js.

Write the review to docs/code-review.md.
```

Notar que esta instrucción es considerablemente más breve que la utilizada en la primera revisión.

No necesitamos volver a especificar:

* los tipos de problemas que debe buscar;
* las severidades;
* la estructura del informe;
* las restricciones sobre `src/`;
* los datos que debe incluir cada hallazgo;
* los casos de validación.

Estas instrucciones ya forman parte de la definición del agente.

* Esperar hasta que el agente termine.
* Autorizar únicamente las operaciones necesarias para realizar la revisión y generar el documento solicitado.

> [!IMPORTANT]
> El agente debe analizar `src/calculator.js`, pero no debe corregirlo.
>
> El resultado esperado de la revisión es:
>
> `docs/code-review.md`

## Revisar el informe generado

En el explorador de Visual Studio Code, verificar que exista:

```text
docs/code-review.md
```

Abrir el archivo.

* [FILE] `docs/code-review.md`

Verificar que contenga:

* un resumen;
* los problemas encontrados;
* una severidad para cada problema;
* una explicación del impacto;
* una recomendación;
* casos de validación sugeridos.

Una estructura posible será:

```text
# Code Review

## Summary

...

## Findings

### Finding

Severity: High

Problem:
...

Impact:
...

Recommendation:
...

Validation:
...

## Suggested test cases

...
```

## Comparar las dos revisiones

Durante el laboratorio realizamos dos revisiones sobre el mismo archivo.

Primero utilizamos GitHub Copilot CLI directamente:

```text
Review src/calculator.js and identify correctness, reliability,
and maintainability issues.

Do not modify any files.
```

Después seleccionamos:

```text
Code Reviewer
```

y solicitamos solamente:

```text
Review src/calculator.js.

Write the review to docs/code-review.md.
```

El agente personalizado ya contiene instrucciones persistentes relacionadas con:

* su rol;
* su alcance;
* sus restricciones;
* los criterios de revisión;
* la clasificación por severidad;
* la información que debe incluir;
* el formato del resultado.

La tarea ingresada en la terminal puede concentrarse entonces en el objetivo concreto de esa ejecución.

## Verificar el alcance del agente

Ahora debemos comprobar qué archivos fueron modificados.

Salir de GitHub Copilot CLI o abrir una nueva terminal.

* [CONSOLE]

```bash
git status --short
```

Revisar los archivos creados y modificados.

La estructura resultante debería incluir:

```text
.github/agents/code-reviewer.agent.md
docs/code-review.md
src/calculator.js
```

Ahora revisar específicamente `src/calculator.js`.

* [CONSOLE]

```bash
git diff -- src/calculator.js
```

Verificar que el agente no haya realizado correcciones adicionales sobre el código fuente.

Por ejemplo, el agente no debería haber reemplazado:

```javascript
if (operation == "add")
```

por:

```javascript
if (operation === "add")
```

Tampoco debería haber agregado:

* validaciones de tipos;
* control de división por cero;
* excepciones;
* nuevas funciones;
* pruebas automatizadas.

El agente debía **recomendar** esas modificaciones en `docs/code-review.md`, no implementarlas.

> [!IMPORTANT]
> El criterio central del laboratorio es comprobar que el agente produjo una revisión sin corregir el código que estaba evaluando.

## Revisar el artefacto producido

El agente no se limita a responder en la terminal.

La revisión queda almacenada en:

```text
docs/code-review.md
```

Esto permite conservar el resultado de la tarea como un artefacto del repositorio.

Tenemos entonces tres elementos diferentes:

```text
.github/agents/code-reviewer.agent.md
```

Define el comportamiento del agente.

```text
src/calculator.js
```

Es el código que el agente analiza.

```text
docs/code-review.md
```

Es el resultado producido por el agente.

El flujo puede representarse como:

```text
code-reviewer.agent.md
         ↓
    Code Reviewer
         ↓
 src/calculator.js
         ↓
      análisis
         ↓
 docs/code-review.md
```

## Guardar los cambios en Git

* [CONSOLE]

```bash
git add .
```

* [CONSOLE]

```bash
git commit -m "Create custom code review agent"
```

* [CONSOLE]

```bash
git push
```

* Esperar hasta que finalice el envío de los archivos al repositorio de GitHub.

## Verificar el repositorio

* [BROWSER] Repositorio `hello-copilot-cli`
  * Actualizar la página.

Verificar que exista la estructura:

```text
hello-copilot-cli
│
├── .github
│   └── agents
│       └── code-reviewer.agent.md
│
├── docs
│   └── code-review.md
│
├── src
│   └── calculator.js
│
└── README.md
```

## ¿Qué ocurrió?

Durante el laboratorio:

1. Creamos una pequeña aplicación JavaScript con problemas intencionales.
2. Ejecutamos el código para observar su comportamiento.
3. Iniciamos GitHub Copilot CLI.
4. Realizamos una revisión directamente desde Copilot CLI.
5. Creamos un agente especializado llamado `Code Reviewer`.
6. Definimos instrucciones persistentes para ese agente.
7. Establecimos su alcance sobre `src/`.
8. Establecimos que no debía modificar el código fuente.
9. Definimos una estructura de revisión repetible.
10. Seleccionamos el agente mediante `/agent`.
11. Solicitamos una nueva revisión utilizando una instrucción más breve.
12. El agente revisó `src/calculator.js`.
13. El resultado quedó almacenado en `docs/code-review.md`.
14. Verificamos los archivos creados y modificados con Git.

El archivo:

```text
.github/agents/code-reviewer.agent.md
```

define **cómo debe comportarse el agente**.

El archivo:

```text
src/calculator.js
```

es **el objeto de la revisión**.

El archivo:

```text
docs/code-review.md
```

es **el artefacto producido por el agente**.

El agente tiene una responsabilidad específica:

```text
REVISAR
```

No tiene la responsabilidad de:

```text
IMPLEMENTAR
```

# Conclusiones

En este laboratorio aprendimos a:

* Crear un repositorio para trabajar con GitHub Copilot CLI.
* Ejecutar una aplicación JavaScript sencilla.
* Iniciar GitHub Copilot CLI desde GitHub Codespaces.
* Utilizar Copilot CLI para analizar código.
* Crear un agente personalizado bajo `.github/agents/`.
* Definir el rol de un agente.
* Definir su alcance.
* Definir restricciones sobre sus acciones.
* Establecer criterios de revisión repetibles.
* Establecer un formato de salida.
* Seleccionar un agente mediante `/agent`.
* Utilizar un agente especializado para revisar código.
* Generar un informe persistente de la revisión.
* Verificar los archivos afectados por la ejecución.

Un agente personalizado permite encapsular un comportamiento especializado:

```text
Rol
 ↓
Alcance
 ↓
Reglas
 ↓
Restricciones
 ↓
Formato de salida
```

En este laboratorio, `Code Reviewer` tiene una sola responsabilidad:

```text
Analizar código
       ↓
Identificar problemas
       ↓
Clasificar hallazgos
       ↓
Explicar impacto
       ↓
Recomendar correcciones
       ↓
Generar code-review.md
```

El código permanece sin corregir.

Las correcciones quedan para otro agente con una responsabilidad diferente.
