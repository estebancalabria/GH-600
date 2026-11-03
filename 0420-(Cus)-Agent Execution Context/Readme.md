# Dos formas de ejecutar agentes en GitHub

En GitHub podemos trabajar con agentes de IA de dos formas principales:

1. **Ejecución interactiva mediante GitHub Copilot CLI**
2. **Ejecución automatizada mediante GitHub Agentic Workflows y GitHub Actions**

No se trata necesariamente de dos tipos diferentes de agentes. La diferencia principal es **dónde se ejecuta el agente, cómo se inicia y cuánto participa una persona durante la ejecución**.

> **Idea central:** Copilot CLI se utiliza para trabajo interactivo iniciado por una persona. GitHub Agentic Workflows se utiliza para automatizaciones reproducibles ejecutadas por GitHub Actions.

---

## Comparación rápida

| Característica | GitHub Copilot CLI | GitHub Agentic Workflows |
|---|---|---|
| Modalidad | Interactiva | Automatizada |
| Entorno de ejecución | Terminal local o Codespace | GitHub Actions |
| Inicio | Prompt o instrucción de una persona | Trigger de GitHub |
| Participación humana | Durante la ejecución | Principalmente al definir, supervisar y revisar |
| Configuración principal | Sesión e instrucciones en la terminal | Archivo Markdown con frontmatter e instrucciones |
| Contexto | Directorio de trabajo y repositorio disponible | Repositorio y contexto del evento de GitHub |
| Resultado típico | Explicación, comandos, cambios o propuesta de implementación | Issue, actualización, comentario, branch, PR u otra salida permitida |
| Trazabilidad | Historial de terminal, commits y PR creados | Workflow run, logs, outputs, commits y PR |
| Caso de uso ideal | Exploración, desarrollo y resolución guiada | Tareas repetibles, programadas o disparadas por eventos |

---

# 1. Agente interactivo con GitHub Copilot CLI

## ¿Qué es?

GitHub Copilot CLI permite trabajar con un agente desde la terminal. La persona inicia la interacción, describe el objetivo y supervisa el trabajo mientras ocurre.

El agente puede analizar el repositorio, proponer un plan, explicar código, sugerir comandos y, según las herramientas y permisos disponibles, modificar archivos o ejecutar acciones.

## Flujo general

```text
Persona
   ↓
Prompt en Copilot CLI
   ↓
El agente analiza el repositorio
   ↓
Propone y ejecuta pasos
   ↓
La persona revisa el resultado
   ↓
Commit, Pull Request o siguiente instrucción
```

## Cuándo utilizarlo

Copilot CLI es apropiado cuando:

- Queremos explorar una base de código.
- Necesitamos investigar un error.
- Estamos desarrollando una funcionalidad de manera iterativa.
- Queremos pedir explicaciones antes de realizar cambios.
- Necesitamos ajustar el objetivo durante la ejecución.
- Queremos supervisar de cerca cada paso.

## Ejemplo de instrucción

```text
Analizá este repositorio y proponé un plan para actualizar la dependencia vulnerable.
No modifiques archivos todavía. Incluí alcance, riesgos, pruebas y estrategia de rollback.
```

Después de revisar el plan, se puede continuar con otra instrucción:

```text
Implementá el plan aprobado en una branch nueva.
Ejecutá las pruebas disponibles y prepará un Pull Request con el resumen y la evidencia.
```

## Ventajas

- Interacción inmediata con el agente.
- Permite refinar el objetivo durante el trabajo.
- Facilita la exploración y el aprendizaje.
- La persona puede revisar cada paso antes de continuar.
- Es conveniente para tareas no repetitivas o poco estructuradas.

## Consideraciones

- El resultado depende del contexto disponible en la sesión.
- La persona debe revisar comandos y cambios antes de aceptarlos.
- Los permisos del agente deben limitarse a lo necesario.
- Para cambios persistentes conviene trabajar en una branch y abrir un Pull Request.
- Una instrucción como “no modifiques producción” no reemplaza controles técnicos reales.

---

# 2. Agente automatizado con GitHub Agentic Workflows

## ¿Qué es?

GitHub Agentic Workflows permite definir workflows basados en agentes que se ejecutan dentro de GitHub Actions.

La definición se escribe en un archivo Markdown. El frontmatter establece elementos como el nombre, el trigger, los permisos y las salidas permitidas. El cuerpo contiene las instrucciones en lenguaje natural que seguirá el agente.

La extensión `gh-aw` compila la definición Markdown y genera un archivo `.lock.yml` ejecutable por GitHub Actions.

## Flujo general

```text
Archivo Markdown del agente
   ↓
gh aw compile
   ↓
Archivo .lock.yml
   ↓
Trigger de GitHub
   ↓
Ejecución en GitHub Actions
   ↓
Logs, evidencia y salida controlada
```

## Estructura del repositorio

```text
.github/
└── workflows/
    ├── hello-agent.md
    └── hello-agent.lock.yml
```

- `hello-agent.md`: definición fuente que se mantiene y edita.
- `hello-agent.lock.yml`: workflow generado para GitHub Actions.

## Ejemplo mínimo

```markdown
---
name: hello-agent
on:
  workflow_dispatch:
safe-outputs:
  create-issue:
---

Leé el README del repositorio.
Creá un Issue con:

1. Un resumen del proyecto.
2. Tres posibles mejoras.
3. Una recomendación priorizada.

No modifiques archivos del repositorio.
```

> El esquema exacto puede variar según la versión de `gh-aw`. Validá la definición con la versión instalada antes de incorporarla al repositorio.

## Preparación

Verificar GitHub CLI y la autenticación:

```bash
gh --version
gh auth status
```

Instalar la extensión:

```bash
gh extension install github/gh-aw
```

Comprobar la instalación:

```bash
gh aw version
```

Compilar el workflow:

```bash
gh aw compile .github/workflows/hello-agent.md
```

Después de compilar, se debe versionar la definición y el archivo generado:

```bash
git add .github/workflows/hello-agent.md
git add .github/workflows/hello-agent.lock.yml
git commit -m "Add hello agent workflow"
git push
```

## Ejecución manual

Si el workflow utiliza `workflow_dispatch`:

1. Abrir la pestaña **Actions** del repositorio.
2. Seleccionar el workflow.
3. Elegir **Run workflow**.
4. Revisar el estado y los logs de la ejecución.
5. Verificar el resultado producido por el agente.

También pueden utilizarse otros triggers, por ejemplo:

```yaml
on:
  schedule:
  issues:
  pull_request:
  workflow_dispatch:
```

Cada trigger entrega un contexto diferente. Los jobs deben validar que el evento recibido sea el esperado antes de realizar acciones sensibles.

## Credenciales y secretos

Las credenciales no deben incluirse en el archivo Markdown, en el código ni en el workflow como texto plano.

Deben almacenarse como secretos de GitHub Actions:

```text
Settings
└── Secrets and variables
    └── Actions
        └── Repository secrets
```

En el laboratorio puede utilizarse un secreto como:

```text
COPILOT_GITHUB_TOKEN
```

El token es la credencial. El secret es el mecanismo utilizado para almacenarla e inyectarla durante la ejecución.

## Safe Outputs

Los Safe Outputs limitan las acciones que puede producir el agente. Por ejemplo, si únicamente se habilita `create-issue`, el workflow debe canalizar el resultado mediante la creación controlada de un Issue.

```text
Agente
   ↓
Safe Output permitido
   ↓
Create Issue
```

Esto evita diseñar el workflow bajo la suposición de que el agente puede realizar cualquier operación.

## Cuándo utilizarlo

GitHub Agentic Workflows es apropiado cuando:

- La tarea debe repetirse con reglas consistentes.
- La ejecución se inicia por un evento de GitHub.
- Se necesita una programación periódica.
- Se requieren logs y evidencia centralizados.
- Se deben declarar permisos y salidas controladas.
- El resultado debe integrarse con Issues, Pull Requests o checks.

## Ventajas

- Ejecución reproducible en GitHub Actions.
- Integración con eventos del repositorio.
- Logs y estado visibles en cada workflow run.
- Permisos y secretos administrados por GitHub.
- Posibilidad de aplicar Safe Outputs y guardrails.
- Adecuado para automatizaciones repetibles y auditables.

## Consideraciones

- El workflow debe solicitar permisos mínimos.
- Las credenciales deben inyectarse en runtime.
- Los resultados deben quedar visibles y ser revisables.
- Los cambios de código deben realizarse en una branch y pasar por Pull Request.
- Los checks, reviews, rulesets y CODEOWNERS deben configurarse como controles técnicos cuando sean obligatorios.

---

# ¿Cuál conviene utilizar?

## Utilizá Copilot CLI cuando

- La tarea sea exploratoria.
- Necesites conversar con el agente.
- El objetivo pueda cambiar durante el trabajo.
- Quieras supervisar activamente la ejecución.
- Estés resolviendo un problema puntual.

## Utilizá GitHub Agentic Workflows cuando

- La tarea sea repetible.
- Deba iniciarse mediante un trigger.
- Necesites ejecución desatendida.
- Requieras logs, permisos y outputs declarados.
- Quieras integrar el agente con la automatización del repositorio.

## Se pueden combinar

Las dos modalidades son complementarias:

1. Usar **Copilot CLI** para explorar el problema y diseñar las instrucciones.
2. Probar el comportamiento en un repositorio controlado.
3. Convertir la tarea estable en un **GitHub Agentic Workflow**.
4. Ejecutarla mediante **GitHub Actions** con permisos mínimos y salidas controladas.

---

# Controles recomendados para ambas modalidades

Independientemente del entorno de ejecución:

1. **No trabajar directamente sobre `main`.**
2. **Usar una branch para los cambios.**
3. **Abrir un Pull Request antes del merge.**
4. **Mantener visible el objetivo y el plan.**
5. **Aplicar mínimo privilegio a tokens y herramientas.**
6. **Ejecutar tests, lint y análisis de seguridad.**
7. **Configurar required checks y required reviews cuando corresponda.**
8. **Usar CODEOWNERS para áreas sensibles.**
9. **Proteger secretos y deployments con Environments.**
10. **Mantener evidencia mediante commits, logs, checks, artifacts y reviews.**

El agente puede proponer y ejecutar dentro de límites definidos. Las personas y las políticas deciden qué cambios se aceptan.

---

# Resumen

GitHub ofrece dos formas prácticas de trabajar con agentes:

- **Copilot CLI:** ejecución interactiva, iniciada y supervisada desde la terminal.
- **GitHub Agentic Workflows:** ejecución automatizada, iniciada por triggers y procesada en GitHub Actions.

La elección no depende solamente de la capacidad del agente. Depende del nivel de interacción, repetibilidad, trazabilidad, riesgo y control que requiere la tarea.
