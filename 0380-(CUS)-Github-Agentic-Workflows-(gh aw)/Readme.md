# GitHub Agentic Workflows (`gh aw`)

Guía introductoria para crear, compilar y ejecutar **GitHub Agentic Workflows** utilizando `gh aw`.

> **Idea principal:** escribimos la definición del agente principalmente en Markdown, `gh aw` la compila y **GitHub Actions proporciona la infraestructura donde se ejecuta el workflow**.

---

## ¿Qué es GitHub Agentic Workflows?

**GitHub Agentic Workflows (`gh-aw`)** permite definir workflows basados en agentes de IA que se ejecutan dentro de GitHub Actions.

En lugar de escribir manualmente un workflow completo de GitHub Actions en YAML, podemos definir el Agentic Workflow utilizando un archivo Markdown.

El archivo Markdown combina:

- configuración del workflow mediante **frontmatter**
- instrucciones para el agente en **lenguaje natural**

`gh aw` compila esa definición y genera el workflow que será ejecutado por GitHub Actions.

## Documentación oficial

https://github.github.com/gh-aw/

---

# Arquitectura básica

El modelo mental puede resumirse así:

```text
Agentic Workflow
      │
      ▼
archivo .md
      │
      │ gh aw compile
      ▼
archivo .lock.yml
      │
      ▼
GitHub Actions
      │
      ▼
Agente de IA
      │
      ▼
Herramientas / Repositorio
      │
      ▼
Resultado
Issue / PR / etc.
```

Esto significa que:

- `.md` es la **fuente** del Agentic Workflow.
- `.lock.yml` es el **workflow generado**.
- GitHub Actions es la **infraestructura de ejecución**.

Por lo tanto:

> `gh-aw` simplifica cómo definimos un workflow agéntico, pero no elimina GitHub Actions.

---

# Relación entre `gh`, `gh aw` y GitHub Actions

Es importante distinguir las tres piezas.

### GitHub CLI

`gh` es la CLI de GitHub.

Por ejemplo:

```bash
gh --version
```

### GitHub Agentic Workflows

`gh aw` permite trabajar con GitHub Agentic Workflows desde GitHub CLI.

Por ejemplo:

```bash
gh aw version
```

### GitHub Actions

GitHub Actions proporciona la infraestructura donde se ejecutan los workflows.

En otras palabras:

```text
gh
└── gh aw
      │
      ├── crear
      ├── compilar
      ├── ejecutar
      └── inspeccionar
             │
             ▼
       GitHub Actions
```

---

# Requisitos

Antes de comenzar necesitás:

- un repositorio de GitHub
- GitHub CLI (`gh`)
- autenticación contra GitHub
- `gh-aw`
- GitHub Actions habilitado en el repositorio
- las credenciales requeridas por el motor de IA que utilice el workflow

---

# Verificar GitHub CLI

```bash
gh --version
```

Comprobar la autenticación:

```bash
gh auth status
```

---

# Instalar `gh-aw`

`gh-aw` se instala utilizando GitHub CLI:

```bash
gh extension install github/gh-aw
```

Verificar la instalación:

```bash
gh aw version
```

También podemos consultar la ayuda:

```bash
gh aw --help
```

---

# Estructura básica

Los Agentic Workflows se almacenan bajo `.github/workflows/`.

Por ejemplo:

```text
.github/
└── workflows/
    ├── hello-agent.md
    └── hello-agent.lock.yml
```

Tenemos dos archivos diferentes.

### `hello-agent.md`

Es el archivo fuente.

Es el archivo que editamos y mantenemos.

### `hello-agent.lock.yml`

Es generado por `gh aw compile`.

Contiene el workflow de GitHub Actions que permite ejecutar el Agentic Workflow.

> No es necesario escribir manualmente todo el YAML generado. El YAML sigue existiendo porque GitHub Actions lo necesita para ejecutar el workflow.

---

# El archivo Markdown

Un Agentic Workflow puede tener una estructura similar a esta:

```markdown
---
name: hello-agent

on:
  workflow_dispatch:

safe-outputs:
  create-issue:
---

Analiza el README de este repositorio.

Genera un resumen de los principales objetivos del proyecto.

Finalmente crea un Issue con el resultado.
```

El documento tiene dos partes principales:

```text
┌─────────────────────────────┐
│ Frontmatter                 │
│                             │
│ Configuración del workflow  │
└─────────────────────────────┘
              +
┌─────────────────────────────┐
│ Markdown                    │
│                             │
│ Instrucciones del agente    │
└─────────────────────────────┘
```

---

# Frontmatter

El **frontmatter** contiene la configuración del Agentic Workflow.

Por ejemplo:

```yaml
---
name: hello-agent

on:
  workflow_dispatch:

safe-outputs:
  create-issue:
---
```

Allí podemos definir aspectos como:

- nombre
- triggers
- permisos
- capacidades
- safe outputs

Después del frontmatter aparecen las instrucciones en lenguaje natural para el agente.

---

# Triggers

Como ocurre con otros workflows, necesitamos definir **cuándo se ejecutará**.

Por ejemplo:

```yaml
on:
  workflow_dispatch:
```

`workflow_dispatch` permite iniciar manualmente el workflow.

Otros escenarios pueden utilizar eventos como:

```yaml
schedule:
```

```yaml
issues:
```

```yaml
pull_request:
```

El trigger determina **qué evento inicia el Agentic Workflow**.

---

# Compilar un Agentic Workflow

Uno de los comandos fundamentales de `gh aw` es:

```bash
gh aw compile .github/workflows/hello-agent.md
```

Conceptualmente:

```text
hello-agent.md

      │
      │ gh aw compile
      ▼

hello-agent.lock.yml
```

El archivo `.md` es la fuente que mantenemos.

El archivo `.lock.yml` contiene la configuración ejecutable por GitHub Actions.

Una distinción importante es que la compilación genera la configuración ejecutable, mientras que las instrucciones del Markdown pueden cargarse durante la ejecución.

---

# Ejecutar el agente

Una vez generado el workflow, la ejecución ocurre mediante GitHub Actions.

Con un trigger:

```yaml
on:
  workflow_dispatch:
```

podemos ejecutarlo manualmente desde:

```text
Repositorio
    ↓
Actions
    ↓
Workflow
    ↓
Run workflow
    ↓
GitHub Actions
    ↓
Agentic Workflow
```

GitHub Actions ejecutará el workflow y permitirá consultar posteriormente su estado y sus logs.

---

# Safe Outputs

Una característica importante es la posibilidad de definir **salidas controladas**.

Por ejemplo:

```yaml
safe-outputs:
  create-issue:
```

Esto permite expresar qué tipo de operación queremos habilitar como resultado del agente.

Conceptualmente:

```text
Agente
   ↓
Safe Output
   ↓
Create Issue
   ↓
GitHub Issue
```

La idea no es simplemente darle al agente permisos ilimitados.

El objetivo es que el agente actúe dentro de capacidades y límites explícitamente definidos.

---

# Credenciales y Secrets

Los agentes pueden necesitar credenciales para acceder a determinados servicios.

Estas credenciales no deberían escribirse directamente dentro del workflow o las instrucciones.

GitHub Actions permite almacenarlas utilizando **Secrets**.

Por ejemplo:

```text
Settings
    ↓
Secrets and variables
    ↓
Actions
    ↓
Repository secrets
```

En un escenario que utilice GitHub Copilot puede existir, por ejemplo:

```text
COPILOT_GITHUB_TOKEN
```

Es importante distinguir:

```text
PAT
│
│ credencial
▼
Secret
│
│ almacenamiento seguro
▼
GitHub Actions
│
│ utiliza la credencial
▼
Agentic Workflow
```

Nunca debemos colocar tokens o secretos directamente dentro del repositorio.

---

# Comandos de `gh aw`

## Consultar versión

```bash
gh aw version
```

Muestra la versión instalada.

---

## Inicializar

```bash
gh aw init
```

Inicializa la configuración necesaria para trabajar con Agentic Workflows.

---

## Crear un workflow

```bash
gh aw new
```

Permite comenzar la creación de un nuevo Agentic Workflow.

---

## Agregar workflows

```bash
gh aw add
```

Permite agregar un Agentic Workflow.

---

## Agregar mediante un asistente

```bash
gh aw add-wizard
```

Proporciona un flujo guiado para agregar/configurar un workflow.

---

## Compilar

```bash
gh aw compile
```

Compila la definición del Agentic Workflow.

Ejemplo:

```bash
gh aw compile .github/workflows/hello-agent.md
```

Conceptualmente:

```text
.md → gh aw compile → .lock.yml
```

---

## Listar workflows

```bash
gh aw list
```

Lista los Agentic Workflows disponibles.

---

## Ejecutar

```bash
gh aw run
```

Permite iniciar la ejecución de un Agentic Workflow.

---

## Consultar estado

```bash
gh aw status
```

Permite inspeccionar el estado de ejecución.

---

## Consultar logs

```bash
gh aw logs
```

Permite consultar los logs asociados a las ejecuciones.

---

## Diagnóstico

```bash
gh aw doctor
```

Ayuda a diagnosticar problemas de configuración o del entorno.

---

# Ayuda de los comandos

Para conocer los comandos disponibles en la versión instalada:

```bash
gh aw --help
```

También podemos inspeccionar un comando específico:

```bash
gh aw compile --help
```

```bash
gh aw run --help
```

```bash
gh aw logs --help
```

Esto es especialmente útil porque `gh-aw` continúa evolucionando.

---

# Flujo de trabajo completo

Un flujo típico sería:

### 1. Verificar GitHub CLI

```bash
gh --version
```

### 2. Verificar autenticación

```bash
gh auth status
```

### 3. Instalar `gh-aw`

```bash
gh extension install github/gh-aw
```

### 4. Verificar la instalación

```bash
gh aw version
```

### 5. Crear el Agentic Workflow

```text
.github/workflows/hello-agent.md
```

### 6. Compilar

```bash
gh aw compile .github/workflows/hello-agent.md
```

Esto genera:

```text
.github/workflows/hello-agent.lock.yml
```

### 7. Commit y push

```bash
git add .
git commit -m "Add hello agent workflow"
git push
```

### 8. Ejecutar

Desde GitHub:

```text
Actions
→ hello-agent
→ Run workflow
```

### 9. Observar la ejecución

Revisar:

- workflow run
- jobs
- logs
- errores
- outputs producidos por el agente

### 10. Verificar el resultado

Dependiendo del Agentic Workflow, el resultado podría ser un Issue, un Pull Request u otro artefacto permitido.

---

# Modelo mental

La forma más sencilla de recordar toda la arquitectura es:

```text
Markdown
   │
   │ definición del agente
   ▼
gh aw
   │
   │ compilación
   ▼
.lock.yml
   │
   │ workflow generado
   ▼
GitHub Actions
   │
   │ infraestructura de ejecución
   ▼
Agente de IA
   │
   │ herramientas + contexto
   ▼
Safe Outputs
   │
   ▼
GitHub
```

Por lo tanto:

> **Markdown es la fuente, `gh aw` es la herramienta y GitHub Actions es la infraestructura de ejecución.**

---

# Buenas prácticas

Cuando un Agentic Workflow puede modificar un repositorio o producir acciones persistentes:

- utilizar **least privilege**
- limitar los permisos del workflow
- evitar acceso innecesario a secretos
- utilizar branches y Pull Requests para cambios de código
- utilizar checks para validar resultados
- conservar logs y evidencia de ejecución
- utilizar Safe Outputs cuando corresponda
- incorporar revisión humana para operaciones sensibles

Una instrucción escrita para el agente no reemplaza un control técnico.

Por ejemplo:

```text
"No modifiques producción"
```

es solamente una instrucción.

Los límites reales deben implementarse mediante mecanismos como:

```text
Permissions
Rulesets
Branch Protection
Required Checks
CODEOWNERS
Environments
Approvals
Hooks
Safe Outputs
```

---

# Resumen

| Componente | Responsabilidad |
|---|---|
| `gh` | CLI de GitHub |
| `gh aw` | Gestión de Agentic Workflows |
| `.md` | Fuente del Agentic Workflow |
| Frontmatter | Configuración |
| Markdown | Instrucciones para el agente |
| `gh aw compile` | Compilación |
| `.lock.yml` | Workflow generado |
| GitHub Actions | Infraestructura de ejecución |
| Trigger | Determina cuándo comienza |
| Safe Outputs | Define salidas controladas |
| Secrets | Almacena credenciales de manera segura |

---

## Comandos rápidos

```bash
# GitHub CLI
gh --version
gh auth status

# Instalar gh-aw
gh extension install github/gh-aw

# Verificar
gh aw version

# Ayuda
gh aw --help

# Inicializar
gh aw init

# Crear
gh aw new

# Agregar
gh aw add
gh aw add-wizard

# Compilar
gh aw compile .github/workflows/hello-agent.md

# Listar
gh aw list

# Ejecutar
gh aw run

# Estado
gh aw status

# Logs
gh aw logs

# Diagnóstico
gh aw doctor
```

---

## Referencias

- [GitHub Agentic Workflows](https://github.github.com/gh-aw/)
- [GitHub CLI](https://cli.github.com/)
- [GitHub Actions](https://docs.github.com/actions)

---

## Idea final

**`gh-aw` permite expresar workflows agénticos de una forma más cercana al lenguaje natural sin eliminar la infraestructura subyacente de GitHub Actions.**

```text
.md → gh aw compile → .lock.yml → GitHub Actions → Agent
```

El objetivo no es simplemente dar autonomía a un agente, sino permitir que actúe dentro de **límites explícitos, observables y controlados**.
