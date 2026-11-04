# GitHub Copilot CLI: guía técnica avanzada

Guía técnica para utilizar GitHub Copilot CLI como agente de ingeniería desde la terminal. Cubre instalación, contexto, instrucciones, `AGENTS.md`, agentes personalizados, Agent Skills, MCP, hooks, permisos, sandboxing, memoria, integración con GitHub, automatización, seguridad y comparación con Claude Code y Codex CLI.

> Esta documentación distingue GitHub Copilot CLI, GitHub CLI y GitHub Agentic Workflows. Son herramientas relacionadas, pero no equivalentes.

## Índice

- [1. Ecosistema de CLI de GitHub](#1-ecosistema-de-cli-de-github)
- [2. Qué es GitHub Copilot CLI](#2-qué-es-github-copilot-cli)
- [3. Instalación y autenticación](#3-instalación-y-autenticación)
- [4. Uso interactivo y programático](#4-uso-interactivo-y-programático)
- [5. Plan Mode](#5-plan-mode)
- [6. Contexto e instrucciones](#6-contexto-e-instrucciones)
- [7. AGENTS.md](#7-agentsmd)
- [8. Instrucciones específicas por ruta](#8-instrucciones-específicas-por-ruta)
- [9. Agentes personalizados](#9-agentes-personalizados)
- [10. Directorio .github/agents](#10-directorio-githubagents)
- [11. Agent Skills](#11-agent-skills)
- [12. Model Context Protocol](#12-model-context-protocol)
- [13. Hooks](#13-hooks)
- [14. Herramientas y permisos](#14-herramientas-y-permisos)
- [15. Sandboxing](#15-sandboxing)
- [16. Memoria, contexto y estado](#16-memoria-contexto-y-estado)
- [17. Integración con GitHub](#17-integración-con-github)
- [18. Automatización](#18-automatización)
- [19. GitHub Agentic Workflows](#19-github-agentic-workflows)
- [20. Seguridad y gobernanza](#20-seguridad-y-gobernanza)
- [21. Arquitectura multiagente](#21-arquitectura-multiagente)
- [22. Comparación con Claude Code y Codex CLI](#22-comparación-con-claude-code-y-codex-cli)
- [23. Estructura recomendada del repositorio](#23-estructura-recomendada-del-repositorio)
- [24. Ejemplos de configuración](#24-ejemplos-de-configuración)
- [25. Flujo de trabajo recomendado](#25-flujo-de-trabajo-recomendado)
- [26. Checklist de madurez](#26-checklist-de-madurez)
- [27. Referencias](#27-referencias)

## 1. Ecosistema de CLI de GitHub

Existen varias herramientas relacionadas que no deben confundirse.

### GitHub CLI

GitHub CLI utiliza el ejecutable:

```bash
gh
```

Se emplea para operar sobre GitHub desde la terminal:

```bash
gh auth status
gh repo view
gh issue list
gh pr list
gh pr create
gh workflow list
gh run list
```

### GitHub Copilot CLI

GitHub Copilot CLI utiliza el ejecutable:

```bash
copilot
```

Es un agente de IA que puede:

- comprender una tarea;
- inspeccionar un repositorio;
- elaborar un plan;
- leer y modificar archivos;
- ejecutar comandos;
- utilizar herramientas;
- realizar operaciones Git;
- consultar GitHub;
- trabajar con Issues y Pull Requests;
- utilizar MCP;
- cargar instrucciones persistentes;
- utilizar Agent Skills;
- ejecutar hooks;
- mantener una sesión iterativa.

### GitHub Agentic Workflows

GitHub Agentic Workflows se distribuye como una extensión de GitHub CLI:

```bash
gh extension install github/gh-aw
```

Sus comandos utilizan:

```bash
gh aw
```

Se emplea para definir workflows agénticos que se ejecutan mediante GitHub Actions.

### Resumen

| Herramienta | Ejecutable | Función principal |
|---|---|---|
| GitHub CLI | `gh` | Operar sobre GitHub |
| GitHub Copilot CLI | `copilot` | Trabajar con un agente desde la terminal |
| GitHub Agentic Workflows | `gh aw` | Ejecutar automatizaciones agénticas en GitHub Actions |

## 2. Qué es GitHub Copilot CLI

GitHub Copilot CLI es una interfaz de línea de comandos para trabajar con Copilot directamente desde una terminal.

Puede funcionar como asistente y como agente.

Un asistente responde una solicitud y devuelve el control al usuario.

Un agente mantiene un objetivo, decide pasos intermedios, utiliza herramientas, observa resultados e itera hasta terminar, bloquearse o requerir intervención humana.

El ciclo conceptual es:

```text
Objetivo
  ↓
Contexto e instrucciones
  ↓
Plan
  ↓
Selección de herramientas
  ↓
Acción
  ↓
Observación
  ↓
Evaluación
  ↓
Nueva acción, finalización o escalamiento
```

Un agente profesional no debe considerarse confiable solo porque genere una respuesta convincente. La confiabilidad depende de que sus acciones sean:

- limitadas;
- observables;
- verificables;
- auditables;
- reversibles;
- gobernadas por políticas.

## 3. Instalación y autenticación

### Instalación con npm

```bash
npm install -g @github/copilot
```

### Instalación con Homebrew

```bash
brew install --cask copilot-cli
```

### Instalación con WinGet

```powershell
winget install GitHub.Copilot
```

### Comprobar la instalación

```bash
copilot --version
```

### Iniciar una sesión

```bash
copilot
```

### Autenticación

La autenticación puede realizarse mediante el flujo interactivo de GitHub.

Cuando se utiliza un token, no debe almacenarse en archivos versionados. Las variables que pueden utilizarse incluyen:

```text
COPILOT_GITHUB_TOKEN
GH_TOKEN
GITHUB_TOKEN
```

Aplicar siempre mínimo privilegio al token.

No colocar secretos en:

```text
AGENTS.md
CLAUDE.md
.github/copilot-instructions.md
.github/agents/*.agent.md
SKILL.md*README.md
arch*vos MCP versionados
código fuente
*ork*lows en texto plano
```

## 4* Uso interactivo y programático

#*# Interfaz interactiva

```bash
co*ilot
```

Ejemplo de solicitud:

`*`text
Analyze the authentication m*dule.

Identify the main security*risks.

Do not modify any files.

*roduce an implementation plan with*validation and rollback steps.
```*
Después se puede*continuar:

```text
Implement the *irst approved step.

Run the relev*nt tests afterward.

Do not modify*workflows or infrastructure.
```

*a interfaz interactiva permite:

-*mantener una conversación;
- aprob*r o rechazar herramientas;
- corre*ir el rumbo;
- proporcionar feedba*k;
- dividir una tarea;
- inspecci*nar resultados;
- rean*dar trabajo.

### Interfaz program*tica

```bash
cop*lot -p "Summarize the last 10 comm*ts"
```

Para permitir únicamente *peraciones Git:

*``bash
copilot -p "*ummarize the last 10 commits and i*entify breaking changes" \
  --all*w-tool='shell(git)'
```

El*modo programático resulta adecuado*para:

- scripts;
- análisis repet*bles;
- mantenimiento;
- pipelines*
- auditorías;
- generación*de documentación;
- tareas program*das.

### Difer*ncia operativa

```text
Modo inter*ctivo
Usuario ↔ agente ↔ herramien*as

Modo programático
Script → age*te → herramientas → resultado
```
*## 5. Plan Mode

GitHub Copilot CL* dispone de un modo de planificaci*n.

Dentro de una sesión interacti*a puede alternarse entre modos con*

```text
Shift + Tab
```

En Plan*Mode, Copilot analiza la tarea y p*oduce un plan antes de modificar a*chivos.

Un plan útil debería incl*ir:

```text
Objective
Scope
Affec*ed files
Implementation steps
Depe*dencies
Risks
Success criteria
Val*dation
Rollback
Open questions
```*
Ejemplo de prompt:

```text
Creat* a plan for replacing the current *uthentication middleware.

Include*

- affected files;
- compatibilit* risks;
- required migrations;
- t*sts;
- security validation;
- roll*ack strategy.

Do not modify files*
```

### Cuándo utilizar Plan Mod*

Utilizarlo especialmente para:

* autenticación;
- autorización;
- *nfraestructura;
- workflows;
- seg*ridad;
- cambios en datos;
- migra*iones;
- dependencias críticas;
- *omponentes difíciles de revertir.
*Para cambios pequeños y reversible* puede utilizarse un enfoque combi*ado:

```text
Plan + implementació*
  ↓
Pull Request
  ↓
Checks
  ↓
R*view
```

## 6. Contexto e instruc*iones

Una tarea no debería depend*r de repetir toda la arquitectura *entro de cada prompt.

GitHub Copi*ot CLI puede descubrir instruccion*s en distintas ubicaciones.

Entre*las ubicaciones soportadas se encu*ntran:

```text
.github/copilot-in*tructions.md
.github/instructions/**/*.instructions.md
AGENTS.md
CLA*DE.md
.claude/CLAUDE.md
GEMINI.md
*``

También existen instrucciones *ersonales:

```text
~/.copilot/cop*lot-instructions.md
~/.copilot/ins*ructions/**/*.instructions.md
```
*Si se configura `COPILOT_HOME`, Co*ilot utiliza ese directorio en lug*r de `~/.copilot`.

Dentro de una *esión puede inspeccionarse el conj*nto de instrucciones descubierto:
*```text
/instructions
```

Las ins*rucciones aplicables se combinan. *or ese motivo deben evitarse regla* incompatibles.

Mal diseño:

```t*xt
AGENTS.md
- Never modify genera*ed files.

copilot-instructions.md*- Always regenerate and commit gen*rated files.
```

Mejor diseño:

`*`text
AGENTS.md
- Generated files *ust not be edited manually.
- Rege*erate them only with `npm*run generate`.
- Commit generated*output only*when the source definition changes*
```

## 7. AGENTS.md

`AGENTS.md`*es un formato portable para propor*ionar instrucciones a agentes de p*ogramación.

Puede utilizarse con *itHub Copilot CLI, Codex CLI y otr*s sistemas compatibles.

Ejemplo:
*```markdown
# AGENTS.md

## Projec*

This repository contains a TypeS*ript REST API.

## Architecture

-*`src/domain/` contains domain logi*.
- `src/http/` contains HTTP cont*ollers.
- `src/infrastructure/` co*tains external adapters.
- `tests/* contains automated tests.
- `infr*/` contains infrastructure definit*ons.

## Install

```bash
npm ci
`*`

## Build

```bash
npm run build*```

## Test

```bash
npm test
```*
## Lint

```bash
npm run lint
```*
## Validation

Before finishing a*code change:

*. Run the relevant*unit tests.
2. Run the full test s*ite when*shared behavior changes.
3. Run th* linter.
4. Run the build.
5. Revi*w the final diff.

## Architecture*rules

- Keep domain logic indepen*ent from HTTP controllers.
- Do no* import infrastructure adapters in*o the domain layer.
- Do not add d*pendencies without explaining the *eed.
- Add tests for behavioral ch*nges.
- Do not*edit generated files manually.

##*Security rules*
- Never commit secrets.
- Do not *og credentials, tokens, or persona* data.
- Do not modify*authentication or authorization wi*hout security tests.
- Do not modi*y `infra/` or `.github/workflows*` unless explicitly requested.

##*Git rules

- Never push directly t* `main`.
- Work on a dedicated bra*ch.
- Keep commits focused.
- Do n*t rewrite shared history.
- Do not*merge without required checks and *eview.

## Pull requests

Every PR*must include:

- objective;
- scop*;
- affected files;
- tests execut*d;
- risks;
- rollback strategy;
-*unresolved questions.
```

### Jer*rquía por directorios

Puede coloc*rse un `AGENTS.md` adicional en su*directorios para proporcionar regl*s más específicas.

Ejemplo:

```t*xt
repo/
├── AGENTS.md
├── fronten*/
│   └── AGENTS.md
└── infra/
   *└── AGENTS.md
```

El archivo raíz*contiene reglas generales y los ar*hivos internos describen restricci*nes del área correspondiente.

## *. Instrucciones específicas por ru*a

Copilot admite instrucciones mo*ulares en:

```text
.github/instru*tions/
```

Los archivos deben ter*inar en:

```text
.instructions.md*```

Ejemplo:

```text
.github/
└─* instructions/
    ├── typescript.*nstructions.md
    ├── tests.instr*ctions.md
    └── terraform.instru*tions.md
```

### TypeScript

```m*rkdown
---
applyTo: "**/*.ts,**/*.*sx"
---

# TypeScript instructions*
- Use strict TypeScript.
- Do not*introduce `any`.
- Prefer explicit*return types for public functions.*- Use existing project abstraction*.
- Add tests for behavioral chang*s.
- Do not suppress compiler erro*s without an explanation.
```

###*Tests

```markdown
---
applyTo: "t*sts/**/*.ts,**/*.test.ts,**/*.spec*ts"
---

# Test instructions

- Us* Vitest.
- Follow Arrange, Act, As*ert.
- Test public behavior rather*than private implementation.
- Inc*ude boundary and failure cases.
- *void time-dependent and network-de*endent tests.
```

### Terraform

*``markdown
---
applyTo: "infra/**/*.tf"
---

# Terraform instructions*
- Run `terraform fmt -check`.
- R*n `terraform validate`.
- Do not e*ecute `terraform apply`.
- Do not *odify production state.
- Do not e*pose secret values in outputs.
- E*plain the blast radius of every in*rastructure change.
```

## 9. Age*tes personalizados

Los agentes pe*sonalizados permiten crear version*s especializadas de Copilot para t*reas concretas.

Ejemplos:

```tex*
Planner
Implementer
Reviewer
Secu*ity Reviewer
Documentation Agent
C* Diagnostics Agent
Dependency Agen*
```

### Separación recomendada

*``text
Planner
- lectura;
- búsque*a;
- análisis;
- plan;
- sin escri*ura.

Implementer
- lectura;
- bús*ueda;
- edición;
- comandos limita*os;
- branch y Pull Request.

Revi*wer
- lectura;
- tests;
- análisis*de diff;
- findings;
- sin escritu*a.
```

Evitar un único agente con*todas las capacidades cuando sea p*sible.

Un agente que crea el camb*o, lo evalúa, decide que está corr*cto y lo publica sin una frontera *ndependiente concentra demasiado p*der y reduce la calidad de la revi*ión.

### Principio de diseño

```*ext
Responsabilidad concreta
+
her*amientas mínimas
+
alcance explíci*o
+
resultado verificable
```

## *0. Directorio .github/agents

Las *efiniciones de agentes personaliza*os asociadas al repositorio se org*nizan en:

```text
.github/agents/*```

Ejemplo:

```text
.github/
└─* agents/
    ├── planner.agent.md
*   ├── implementer.agent.md
    ├─* reviewer.agent.md
    └── security-reviewer.agent.md
```

Los nombres de archivo utilizan habitualmente el sufijo:

```text
.agent.md
```

### Responsabilidad de cada archivo

Un archivo de agente debería expresar:

- propósito;
- responsabilidad;
- límites;
- herramientas;
- entradas;
- procedimiento;
- criterios de éxito;
- formato de salida;
- condiciones de escalamiento.

### Importante sobre la sintaxis

No debe asumirse que los archivos de `.github/agents/` utilizan el mismo frontmatter que:

```text
.github/workflows/*.md
```

Los custom agents y los Agentic Workflows son conceptos distintos.

El esquema exacto admitido por `.agent.md` puede depender de la superficie y versión de Copilot. Antes de agregar propiedades al frontmatter, verificar la documentación oficial actual de custom agents.

### Cuerpo recomendado de un agente

Aunque el esquema de frontmatter debe verificarse, el cuerpo Markdown puede organizarse así:

```markdown
# Planner

## Purpose

Analyze the task and produce an implementation plan.

## Responsibilities

- Understand the objective.
- Inspect the relevant repository areas.
- Identify affected files.
- Identify dependencies and risks.
- Define success criteria.
- Define validation and rollback.

## Restrictions

- Do not modify files.
- Do not create commits.
- Do not push branches.
- Do not open or merge pull requests.
- Do not access production secrets.

## Required output

Return:

1. Objective
2. Scope
3. Affected files
4. Implementation steps
5. Risks
6. Success criteria
7. Validation
8. Rollback
9. Open questions

## Escalation

Stop and request clarification when:

- requirements conflict;
- scope is ambiguous;
- the task affects production;
- credentials are required;
- the proposed change is difficult to reverse.
```

### Ejemplo de implementador

```markdown
# Implementer

## Purpose

Implement an approved plan on a dedicated branch.

## Preconditions

- An approved plan exists.
- Scope and success criteria are explicit.
- Required credentials are available through approved mechanisms.

## Responsibilities

- Create or reuse the task branch.
- Modify only files included in scope.
- Add or update tests.
- Run required validation.
- Produce focused commits.
- Prepare a pull request.

## Restrictions

- Never push directly to `main`.
- Do not modify `.github/workflows/`, `infra/`, or security policies unless explicitly approved.
- Do not merge the pull request.
- Do not deploy to production.
- Do not expose secrets.
- Do not bypass failing checks.

## Required output

Return:

- branch;
- files changed;
- commits;
- tests executed;
- validation results;
- remaining risks;
- pull request URL or draft description.
```

### Ejemplo de revisor

```markdown
# Reviewer

## Purpose

Review a proposed change without modifying it.

## Responsibilities

- Compare the implementation against the stated objective.
- Validate scope.
- Inspect tests and checks.
- Identify correctness, security, reliability, and maintainability issues.
- Verify that required evidence exists.
- Report findings by severity.

## Restrictions

- Do not edit files.
- Do not create commits.
- Do not approve or merge automatically.
- Do not dismiss failing checks.
- Do not treat passing CI as proof that the objective was met.

## Finding format

For each finding include:

- severity;
- file and location;
- issue;
- impact;
- evidence;
- recommended correction.

## Final recommendation

Return one of:

- ready for human review;
- changes required;
- blocked by missing evidence;
- escalation required.
```

## 11. Agent Skills

Una Agent Skill encapsula un procedimiento reutilizable.

Puede contener:

```text
instructions
scripts
templates
references
examples
resources
```

Copilot puede cargar la skill cuando la tarea resulta relevante.

Ubicaciones de proyecto compatibles incluyen:

```text
.github/skills/
.claude/skills/
.agents/skills/
```

Ubicaciones personales:

```text
~/.copilot/skills/
~/.agents/skills/
```

### Ejemplo de estructura

```text
.github/
└── skills/
    └── release-notes/
        ├── SKILL.md
        ├── scripts/
        │   └── collect-changes.sh
        ├── templates/
        │   └── release-notes.md
        └── references/
            └── categories.md
```

### Diferencia entre instrucciones y skills

Una instrucción define una regla transversal:

```text
Always run tests after changing business logic.
```

Una skill define un procedimiento especializado:

```text
When preparing release notes:

1. Identify the previous release tag.
2. Collect merged pull requests.
3. Classify changes.
4. Identify breaking changes.
5. List migrations.
6. Generate notes using the repository template.
7. Validate issue and PR links.
```

### Ejemplo de SKILL.md

```markdown
# Release Notes

## Purpose

Generate accurate release notes from repository history.

## Inputs

- previous release tag;
- target release tag or branch;
- merged pull requests;
- repository release-note template.

## Procedure

1. Determine the comparison range.
2. Collect commits and merged pull requests.
3. Remove duplicates and internal-only changes.
4. Classify changes:
   - breaking;
   - features;
   - fixes;
   - security;
   - documentation;
   - maintenance.
5. Identify migrations and operational actions.
6. Generate the release notes.
7. Validate links and references.

## Output

Produce Markdown containing:

- summary;
- breaking changes;
- features;
- fixes;
- security notes;
- migrations;
- known issues;
- contributor acknowledgements.

## Restrictions

- Do not invent issue or pull request references.
- Do not include secrets.
- Do not publish a release.
```

## 12. Model Context Protocol

Model Context Protocol, o MCP, conecta agentes con herramientas y fuentes de datos externas.

Sin MCP:

```text
Agent
├── filesystem
├── shell
└── git
```

Con MCP:

```text
Agent
├── filesystem
├── shell
├── git
├── GitHub
├── Jira
├── documentation
├── databases
├── internal APIs
└── custom services
```

### Riesgo de MCP

Agregar un MCP server aumenta el radio de impacto del agente.

Antes de habilitarlo, responder:

```text
¿Qué datos puede leer?
¿Qué recursos puede modificar?
¿Qué identidad utiliza?
¿Qué secretos necesita?
¿Qué operaciones tienen efectos externos?
¿Qué registros genera?
¿Qué agente necesita realmente ese acceso?
```

### Buenas prácticas

- utilizar servidores confiables;
- preferir capacidades de lectura;
- no exponer herramientas innecesarias;
- utilizar credenciales específicas;
- separar entornos;
- limitar acceso a producción;
- auditar operaciones con efectos externos;
- probar el servidor en un sandbox;
- versionar configuración sin secretos;
- tratar una ampliación de MCP como un cambio de seguridad.

### Ejemplo conceptual

```json
{
  "servers": {
    "project-docs": {
      "command": "node",
      "args": [
        "./tools/docs-mcp/server.js"
      ],
      "env": {
        "DOCS_ROOT": "./docs"
      }
    }
  }
}
```

No colocar valores secretos directamente en el archivo.

## 13. Hooks

Los hooks ejecutan comandos determinísticos en momentos específicos de la sesión o del uso de herramientas.

Los hooks de repositorio se almacenan en:

```text
.github/hooks/
```

Los hooks personales se almacenan en:

```text
~/.copilot/hooks/
```

Si existe `COPILOT_HOME`, se utiliza:

```text
$COPILOT_HOME/hooks/
```

### Eventos

Entre los eventos disponibles se encuentran:

```text
sessionStart
sessionEnd
userPromptSubmitted
preToolUse
postToolUse
errorOccurred
agentStop
```

### Plantilla

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [],
    "sessionEnd": [],
    "userPromptSubmitted": [],
    "preToolUse": [],
    "postToolUse": [],
    "errorOccurred": []
  }
}
```

### Hook de inicio de sesión

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [
      {
        "type": "command",
        "bash": "mkdir -p logs && echo \"Session started: $(date -Iseconds)\" >> logs/copilot-session.log",
        "powershell": "New-Item -ItemType Directory -Force logs | Out-Null; Add-Content -Path logs/copilot-session.log -Value \"Session started: $(Get-Date -Format o)\"",
        "cwd": ".",
        "timeoutSec": 10
      }
    ]
  }
}
```

### Hook mediante script

```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "bash": "./scripts/validate-tool.sh",
        "powershell": "./scripts/validate-tool.ps1",
        "cwd": ".",
        "timeoutSec": 10
      }
    ]
  }
}
```

### Casos de uso

`preToolUse` puede:

- bloquear comandos peligrosos;
- validar argumentos;
- restringir directorios;
- impedir acceso a secretos;
- impedir operaciones de producción.

`postToolUse` puede:

- ejecutar formato;
- registrar auditoría;
- capturar resultados;
- validar archivos modificados.

`errorOccurred` puede:

- registrar fallos;
- agregar contexto diagnóstico;
- generar alertas;
- iniciar un escalamiento.

### Instrucción frente a control

```text
Instrucción:
"Never deploy to production."

Control:
hook, permisos o Environment que impide el deployment.
```

Las instrucciones orientan al modelo.

Los controles técnicos imponen límites independientemente del razonamiento del modelo.

## 14. Herramientas y permisos

Copilot CLI solicita aprobación antes de utilizar herramientas no autorizadas.

En general, el usuario puede:

1. aprobar una ejecución;
2. aprobar la herramienta durante la sesión;
3. rechazarla e indicar otro enfoque.

Aprobar una herramienta para toda la sesión puede permitir usos más amplios que el comando originalmente mostrado.

Si se autoriza `rm` para toda la sesión, no se está autorizando únicamente un archivo. Se está autorizando la herramienta para usos posteriores dentro de esa sesión.

### Allowlist

Ejemplo limitado a Git:

```bash
copilot -p "Summarize repository history" \
  --allow-tool='shell(git)'
```

### Evitar autorización total

```bash
copilot --allow-all-tools
```

No debe ser la configuración habitual.

Con autorización general, el agente puede obtener un alcance comparable al del usuario sobre:

- filesystem;
- shell;
- Git;
- credenciales disponibles;
- red;
- herramientas instaladas;
- repositorios accesibles.

### Estrategia recomendada

```text
Default deny
  ↓
Read and search
  ↓
Specific shell tools
  ↓
Specific write operations
  ↓
High-risk operations only with explicit approval
```

## 15. Sandboxing

El sandbox reduce el impacto de una acción inesperada.

### Sandbox local

Dentro de una sesión:

```text
/sandbox enable
```

Puede restringir:

- filesystem;
- red;
- capacidades del sistema.

### Sandbox cloud

```bash
copilot --cloud
```

Una sesión cloud se ejecuta en un entorno aislado administrado por GitHub.

### Defensa en profundidad

```text
Sandbox
+
Tool allowlist
+
Dedicated credentials
+
Branch protection
+
Required checks
+
Human approval
```

Ninguna capa sustituye completamente a las demás.

## 16. Memoria, contexto y estado

No debe confundirse el contexto conversacional con el estado persistente del proyecto.

### Contexto de sesión

Incluye:

- prompts recientes;
- feedback;
- decisiones inmediatas;
- archivos leídos;
- resultados recientes.

Comandos útiles:

```text
/context
/compact
```

`/context` muestra cómo se utiliza la ventana de contexto.

`/compact` resume el historial cuando resulta necesario.

### Estado persistente

Las decisiones importantes deben almacenarse en artefactos del repositorio:

```text
Issue
Pull Request
branch
commit
workflow run
artifact
ADR
documentation
```

### Fuente de verdad

Recomendación:

```text
Requirements            → Issue
Plan and decisions      → Pull Request
Code changes            → Branch and commits
Validation              → Checks and workflow runs
Large reports           → Artifacts
Architecture decisions  → ADRs
Reusable procedures     → Skills
Repository rules        → Instructions
```

Evitar mantener criterios diferentes en prompts sucesivos.

Si cambian los criterios de aceptación, actualizar la fuente original.

## 17. Integración con GitHub

Copilot CLI puede interactuar con GitHub.com.

Ejemplos:

```text
List my open pull requests.
```

```text
List open issues assigned to me in OWNER/REPO.
```

```text
Start working on issue #123 in a dedicated branch.
```

```text
Review pull request #456 and report serious problems.
```

```text
Create a pull request containing the current changes.
```

```text
Use the GitHub MCP server to find beginner-friendly issues in OWNER/REPO.
```

El resultado profesional de una tarea no deberíadenciales personales amplias;
- separar análisis y aplicación.

## 19. GitHub Agentic Workflows

GitHub Agentic Workflows permite ejecutar workflows con razonamiento agéntico dentro de GitHub Actions.

Instalación:

```bash
gh extension install github/gh-aw
```

Comprobar versión:

```bash
gh aw version
```

Comandos relevantes:

```bash
gh aw init
gh aw new
gh aw add
gh aw add-wizard
gh aw list
gh aw compile
gh aw run
gh aw status
gh aw logs
gh aw doctor
```

### Estructura conceptual

```text
.github/
└── workflows/
    ├── repository-analysis.md
    └── repository-analysis.lock.yml
```

El Markdown es el archivo fuente.

El archivo `.lock.yml` es generado por la compilación y representa el workflow ejecutable.

Ejemplo:

```bash
gh aw compile .github/workflows/repository-analysis.md
```

### Diferencia con `.github/agents`

```text
.github/agents/
    define agentes especializados

.github/workflows/*.md
    define workflows agénticos

.github/workflows/*.lock.yml
    workflow generado para GitHub Actions
```

No intercambiar sus esquemas de frontmatter.

## 20. Seguridad y gobernanza

La autonomía debe ser proporcional al riesgo.

| Riesgo | Ejemplos | Autonomía recomendada |
|---|---|---|
| Bajo | documentación, formato | validación automática |
| Medio | código, dependencias | branch, PR, checks y review |
| Alto | infraestructura, workflows | CODEOWNERS, checks y múltiples aprobaciones |
| Crítico | producción, secretos | Environment y aprobación humana obligatoria |

### Áreas sensibles

```text
.github/workflows/
infra/
security/
authentication/
authorization/
production/
secrets/
```

### Principios

1. No trabajar directamente sobre `main`.
2. Hacer visible el plan.
3. Separar planificación, ejecución y evaluación.
4. No confundir instrucciones con controles.
5. Definir criterios de éxito verificables.
6. Conservar evidencia.
7. Aplicar mínimo privilegio.
8. Diseñar suponiendo que ocurrirán fallos.
9. Utilizar reintentos limitados.
10. Mantener decisiones críticas bajo control humano.

### Secretos

Los secretos deben:

- inyectarse en runtime;
- tener alcance mínimo;
- estar disponibles solo para el proceso necesario;
- limitarse mediante Environments cuando corresponda;
- rotarse;
- quedar excluidos de logs y artifacts.

## 21. Arquitectura multiagente

Una arquitectura recomendada separa responsabilidades.

```text
Task
  ↓
Planner
  ↓
Plan
  ↓
Human approval when required
  ↓
Implementer
  ↓
Branch
  ↓
Pull Request
  ↓
Automated checks
  ↓
Reviewer
  ↓
Human review
  ↓
Merge
```

### Planner

```text
Read
Search
Analyze
Plan
No write
```

### Implementer

```text
Read
Search
Edit
Limited shell
Branch
Commits
Pull Request
```

### Reviewer

```text
Read
Search
Tests
Policy validation
Findings
No write
```

### Handoffs

Cada transferencia debería incluir:

```text
objective
current state
decisions
evidence
remaining work
risks
next owner
```

### Concurrencia

Los agentes paralelos deben trabajar en ámbitos aislados.

Ejemplo:

```text
agent/dependency/issue-123
agent/documentation/issue-124
agent/security/issue-125
```

Evitar que varios agentes modifiquen simultáneamente los mismos archivos.

## 22. Comparación con Claude Code y Codex CLI

Los tres productos permiten trabajar con agentes de ingeniería desde una terminal, pero se integran con ecosistemas diferentes.

| Área | GitHub Copilot CLI | Claude Code | Codex CLI |
|---|---|---|---|
| Terminal interactiva | Sí | Sí | Sí |
| Ejecución programática | `copilot -p` | `claude -p` | `codex exec` |
| Edición de archivos | Sí | Sí | Sí |
| Shell | Sí | Sí | Sí |
| Git | Sí | Sí | Sí |
| Planificación | Plan Mode | Planificación integrada | Planificación integrada |
| Instrucciones principales | `copilot-instructions.md`, `AGENTS.md` | `CLAUDE.md`, `AGENTS.md` | `AGENTS.md` |
| Instrucciones por ruta | `.instructions.md` | reglas y memoria por proyecto | jerarquía de `AGENTS.md` |
| Skills | Sí | Sí | Sí |
| MCP | Sí | Sí | Sí |
| Hooks | Sí | Sí | mecanismos propios |
| Agentes especializados | Custom agents | Subagents | Subagents |
| Cloud execution | Sí | Sí | Sí |
| GitHub integration | Muy profunda | Mediante Git, GitHub y Actions | Mediante Git, GitHub, plugins y cloud |
| Sandboxing | Local y cloud | Permisos y aislamiento | Sandbox y permisos |

### GitHub Copilot CLI

Resulta especialmente adecuado cuando GitHub es el sistema central de trabajo:

```text
Issues
Pull Requests
Actions
CODEOWNERS
Rulesets
Environments
Copilot
GitHub MCP
```

Ventajas principales:

- integración con GitHub.com;
- custom instructions;
- `AGENTS.md`;
- custom agents;
- Agent Skills;
- MCP;
- hooks;
- sandbox local y cloud;
- continuidad con otras superficies de Copilot.

### Claude Code

Claude Code utiliza principalmente:

```text
CLAUDE.md
```

También puede trabajar con:

```text
AGENTS.md
skills
hooks
MCP
subagents
parallel agents
```

Su ecosistema es especialmente fuerte para:

- flujos de terminal;
- automatización Unix;
- subagentes;
- hooks;
- habilidades reutilizables;
- integración con herramientas externas mediante MCP.

Ejemplo programático:

```bash
claude -p "Review the current diff for security issues"
```

### Codex CLI

Codex CLI utiliza `AGENTS.md` como archivo central de instrucciones.

Comandos interactivos relevantes:

```text
/init
/status
/permissions
/model
/review
```

Automatización:

```bash
codex exec "Run the tests and explain any failures"
```

También soporta:

```text
skills
plugins
subagents
MCP
web search
cloud execution
```

### Portabilidad

Una estrategia portable es:

```text
AGENTS.md
    reglas compartidas

.github/copilot-instructions.md
    comportamiento específico de Copilot

CLAUDE.md
    comportamiento específico de Claude Code

skills/
    procedimientos reutilizables

tool-specific configuration
    permisos, hooks y MCP por plataforma
```

### Criterio de elección

No elegir únicamente por la calidad percibida del modelo.

Evaluar:

- integración con el repositorio;
- identidad y autenticación;
- herramientas disponibles;
- capacidad de automatización;
- controles de permisos;
- sandbox;
- hooks;
- skills;
- MCP;
- trazabilidad;
- ejecución local o cloud;
- políticas organizacionales;
- costos y límites vigentes.

## 23. Estructura recomendada del repositorio

```text
repo/
├── AGENTS.md
├── README.md
├── CLAUDE.md
├── docs/
├── src/
├── tests/
├── scripts/
│   ├── validate-tool.sh
│   └── validate-tool.ps1
└── .github/
    ├── copilot-instructions.md
    ├── instructions/
    │   ├── backend.instructions.md
    │   ├── frontend.instructions.md
    │   ├── tests.instructions.md
    │   └── terraform.instructions.md
    ├── agents/
    │   ├── planner.agent.md
    │   ├── implementer.agent.md
    │   ├── reviewer.agent.md
    │   └── security-reviewer.agent.md
    ├── skills/
    │   ├── release-notes/
    │   │   ├── SKILL.md
    │   │   ├── scripts/
    │   │   └── templates/
    │   └── security-review/
    │       ├── SKILL.md
    │       └── references/
    ├── hooks/
    │   └── security.json
    ├── workflows/
    │   ├── ci.yml
    │   ├── security.yml
    │   └── deploy.yml
    ├── pull_request_template.md
    └── CODEOWNERS
```

### Responsabilidad de cada componente

```text
AGENTS.md
    contrato portable para agentes

copilot-instructions.md
    instrucciones globales específicas de Copilot

instructions/
    instrucciones específicas por ruta

agents/
    agentes especializados

skills/
    procedimientos reutilizables

hooks/
    controles determinísticos

MCP
    acceso a herramientas y datos externos

workflows/
    validación automatizada

CODEOWNERS
    asignación de revisión según rutas

Rulesets
    políticas obligatorias del repositorio

Environments
    control sobre deployments y secretos
```

## 24. Ejemplos de configuración

### `.github/copilot-instructions.md`

```markdown
# Repository instructions

## Working method

1. Understand the requested outcome.
2. Inspect the relevant files.
3. Produce a plan for non-trivial changes.
4. Modify only files within the approved scope.
5. Run the required validation.
6. Review the final diff.
7. Report evidence and remaining risks.

## Restrictions

- Never push directly to `main`.
- Never merge pull requests.
- Never deploy to production.
- Never expose secrets.
- Do not modify `.github/workflows/` or `infra/` unless explicitly requested.
- Do not bypass tests, linting, security checks, or branch policies.
- Do not claim success without executing the relevant validation.

## Output

When finishing a task, report:

- summary;
- files changed;
- commands executed;
- test results;
- unresolved risks;
- recommended next step.
```

### `.github/pull_request_template.md`

```markdown
## Objective

Describe the problem and intended outcome.

## Scope

List what is included and excluded.

## Plan

1.
2.
3.

## Files affected

-

## Validation

- [ ] Build
- [ ] Unit tests
- [ ] Integration tests
- [ ] Lint
- [ ] Security checks
- [ ] Manual validation

## Evidence

Add workflow runs, test reports, screenshots or artifacts.

## Risks

-

## Rollback

Explain how to revert the change safely.

## Review

- [ ] Scope matches the plan
- [ ] Required owners reviewed sensitive areas
- [ ] Required checks passed
- [ ] No secrets or sensitive data were added
```

### `.github/CODEOWNERS`

```text
/security/          @security-team
/infra/             @platform-team
/.github/workflows/ @platform-team
/.github/agents/    @platform-team
/.github/hooks/     @security-team
*                   @core-team
```

Tener `CODEOWNERS` no vuelve obligatoria la revisión por sí solo. Debe combinarse con required reviews mediante Rulesets o Branch Protection.

### Workflow de CI

```yaml
name: CI

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

### Environment protegido

```yaml
name: Deploy

on:
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: production
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest

    environment:
      name: production

    steps:
      - uses: actions/checkout@v4
      - run: ./scripts/deploy.sh
```

La aprobación debe configurarse en el Environment de GitHub.

## 25. Flujo de trabajo recomendado

### Paso 1: definir el objetivo

Utilizar un Issue o tarea con:

```text
objetivo
alcance
restricciones
criterios de aceptación
riesgo
```

### Paso 2: planificar

```text
Analyze issue #123.

Produce a plan containing:

- objective;
- affected files;
- implementation steps;
- risks;
- success criteria;
- validation;
- rollback.

Do not modify files.
```

### Paso 3: revisar el plan

Confirmar:

- que el alcance sea correcto;
- que no falten sistemas afectados;
- que los criterios sean verificables;
- que exista rollback;
- que los permisos sean adecuados.

### Paso 4: implementar en una branch

```text
Implement the approved plan on a dedicated branch.

Do not modify files outside the approved scope.

Run the required tests and report the results.
```

### Paso 5: inspeccionar el diff

```bash
git status
git diff --stat
git diff
```

### Paso 6: validar

```bash
npm run lint
npm test
npm run build
```

Agregar las validaciones correspondientes al lenguaje y proyecto.

### Paso 7: preparar el Pull Request

El PR debería contener:

- objetivo;
- plan;
- cambios;
- evidencia;
- riesgos;
- rollback;
- cuestiones abiertas.

### Paso 8: aplicar políticas

- required checks;
- CODEOWNERS;
- required reviews;
- secret scanning;
- dependency review;
- code scanning;
- Environment approvals.

### Paso 9: decisión humana

El agente puede proponer y ejecutar dentro de límites.

La aceptación, el merge y los deployments críticos deben permanecer bajo políticas exigibles y responsables identificados.

## 26. Checklist de madurez

### Contexto

- [ ] Existe `AGENTS.md`.
- [ ] Las instrucciones son breves y no contradictorias.
- [ ] Build, test y lint están documentados.
- [ ] Las restricciones arquitectónicas son explícitas.
- [ ] Existen instrucciones específicas por ruta cuando corresponde.

### Agentes

- [ ] Cada agente tiene una responsabilidad concreta.
- [ ] Planning, implementation y review están diferenciados.
- [ ] Los agentes de revisión son de solo lectura.
- [ ] No existe un agente con permisos indiscriminados.
- [ ] Los handoffs tienen un formato definido.

### Skills

- [ ] Los procesos repetibles se encapsulan como skills.
- [ ] Cada skill incluye propósito, entradas, procedimiento y salida.
- [ ] Los scripts están separados de las instrucciones generales.
- [ ] Las skills no contienen secretos.

### MCP

- [ ] Cada MCP server tiene una justificación.
- [ ] Las herramientas expuestas siguen mínimo privilegio.
- [ ] Las credenciales tienen alcance limitado.
- [ ] Las operaciones con efectos externos quedan auditadas.
- [ ] Los servidores se prueban en un entorno aislado.

### Hooks

- [ ] Los hooks sensibles fallan de forma segura.
- [ ] Las operaciones peligrosas se validan antes de ejecutarse.
- [ ] Las acciones relevantes se registran.
- [ ] Los scripts de hooks se prueban independientemente.
- [ ] La configuración utiliza JSON válido.

### Ejecución

- [ ] Los cambios ocurren en branches.
- [ ] No existen pushes directos a `main`.
- [ ] Los cambios llegan mediante Pull Request.
- [ ] Se ejecutan tests.
- [ ] Se ejecutan linters y builds.
- [ ] Se ejecutan security checks.
- [ ] Los reintentos tienen límite.

### GitHub

- [ ] `main` está protegida.
- [ ] Required checks están configurados.
- [ ] Required reviews están configuradas.
- [ ] CODEOWNERS cubre áreas sensibles.
- [ ] Rulesets o Branch Protection hacen obligatorias las políticas.
- [ ] Los deployments críticos utilizan Environments.

### Seguridad

- [ ] No existen secretos en instrucciones.
- [ ] No se utiliza `--allow-all-tools` habitualmente.
- [ ] El agente se ejecuta desde directorios confiables.
- [ ] Las herramientas siguen mínimo privilegio.
- [ ] Las sesiones críticas utilizan sandbox.
- [ ] Las credenciales de producción no están disponibles durante tareas normales.

### Evidencia

Para cada cambio importante debe poder reconstruirse:

```text
Goal
  ↓
Plan
  ↓
Changeset
  ↓
Validation
  ↓
Evidence
  ↓
Human judgment
  ↓
Outcome
```

Deben poder responderse estas preguntas:

```text
¿Qué intentó hacer el agente?
¿Qué modificó?
¿Qué herramientas utilizó?
¿Qué comandos ejecutó?
¿Qué validaciones pasaron?
¿Qué validaciones fallaron?
¿Qué evidencia quedó?
¿Quién revisó?
¿Quién aprobó?
¿Cuál fue el resultado final?
¿Cómo se revierte?
```

## 27. Referencias

- [GitHub Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-copilot-cli)
- [Agregar instrucciones personalizadas a GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions)
- [Hooks para GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-hooks)
- [Agent Skills en GitHub Copilot](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Codex CLI](https://developers.openai.com/codex/cli/)
- https://github.com/agentsmd/agents.md
- https://github.com/agentskills/agentskills
- [GitHub Agentic Workflows](https://github.github.com/gh-aw/)

## Regla de oro

Un agente confiable no es el que recibe más permisos ni el que puede hacer más cosas.

Es el que puede completar el trabajo necesario dentro de límites:

```text
explícitos
+
mínimos
+
verificables
+
observables
+
reversibles
+
auditables
```

El patrón profesional no es:

```text
Prompt
  ↓
Agente
  ↓
Producción
```

El patrón profesional es:

```text
Goal
  ↓
Instructions
  ↓
Plan
  ↓
Agent
  ↓
Tools, Skills and MCP
  ↓
Controlled execution
  ↓
Tests, security and policies
  ↓
Pull Request
  ↓
Human or policy decision
  ↓
Merge or deployment
```

Ese es el salto entre utilizar un coding agent y diseñar un entorno profesional de ingeniería agéntica.
