# Gobernanza de sistemas agénticos en GitHub

Los sistemas agénticos modernos pueden analizar repositorios, proponer soluciones, modificar archivos, crear ramas y Pull Requests, ejecutar validaciones e iterar a partir de sus resultados.

La cuestión fundamental ya no es únicamente **qué puede hacer un agente**, sino **qué debería estar autorizado a hacer**.

El principio central es:

> **A mayor riesgo, menor autonomía. A menor riesgo y mayor reversibilidad, mayor posibilidad de automatización.**

Un sistema agéntico confiable debe operar dentro de límites explícitos, verificables y auditables.

---

## 1. Autonomía basada en riesgo

No todas las acciones tienen las mismas consecuencias.

Modificar documentación es relativamente sencillo de revertir. En cambio, alterar infraestructura, workflows, secretos o producción puede tener consecuencias sobre todo el sistema.

Una clasificación práctica es:

| Riesgo | Ejemplos | Control recomendado |
|---|---|---|
| 🟢 Bajo | Documentación, formato | Automatización |
| 🟡 Medio | Dependencias, refactors seguros | PR + checks |
| 🔴 Alto | Infraestructura, workflows | CODEOWNERS + approvals + checks |
| ⚫ Crítico | Producción, secretos | Environment + aprobación humana + evidencia |

Áreas como:

- `.github/workflows/`
- `infra/`
- `security/`

deben considerarse **de alto riesgo por defecto**.

Son ejemplos de cambios con **diff pequeño pero consecuencias potencialmente grandes**.

---

# 2. Niveles de autonomía

Una forma práctica de diseñar agentes es definir distintos niveles.

### Solo lectura

El agente puede:

- inspeccionar
- analizar
- clasificar
- resumir
- recomendar

pero no modificar recursos.

### Solo propuesta

El agente puede:

- crear branches
- realizar cambios
- abrir Pull Requests

pero no puede hacer merge ni desplegar.

### Ejecución con guardrails

Puede ejecutar operaciones previamente aprobadas, por ejemplo:

- tests
- builds
- análisis de seguridad
- deployments a staging

### Ejecución autorizada por humanos

Las acciones críticas requieren aprobación explícita:

- deployments a producción
- modificación de workflows
- cambios de infraestructura
- acceso a secretos

---

# 3. Pipeline: ejecutar según el riesgo

El nivel de riesgo debería ser un **dato estructurado y procesable por el workflow**, no solamente una explicación escrita por el agente.

Por ejemplo, un `plan.json` puede contener el nivel de riesgo y GitHub Actions puede decidir qué camino seguir.

```yaml
name: agent-plan-apply

on:
  workflow_dispatch:

jobs:
  plan:
    runs-on: ubuntu-latest

    outputs:
      risk: ${{ steps.read.outputs.risk }}

    steps:
      - uses: actions/checkout@v4

      - name: Download plan artifact
        uses: actions/download-artifact@v4
        with:
          name: plan
          path: out

      - id: read
        name: Read risk from plan.json
        run: |
          echo "risk=$(jq -r .risk out/plan.json)" >> "$GITHUB_OUTPUT"

  apply:
    needs: plan
    runs-on: ubuntu-latest

    environment: approval-required

    if: ${{ needs.plan.outputs.risk != 'low' }}

    steps:
      - name: Apply agent plan
        run: ./scripts/apply.sh out/plan.json

  apply_auto:
    needs: plan
    runs-on: ubuntu-latest

    if: ${{ needs.plan.outputs.risk == 'low' }}

    steps:
      - name: Apply agent plan automatically
        run: ./scripts/apply.sh out/plan.json
```

La idea es simple:

```text
Plan
  ↓
Clasificación de riesgo
  ↓
┌──────────────┬─────────────────┐
│ Bajo         │ Medio / Alto    │
├──────────────┼─────────────────┤
│ Automático   │ Approval        │
└──────────────┴─────────────────┘
```

---

# 4. Pull Requests como frontera de control

Un patrón especialmente importante consiste en evitar que el agente trabaje directamente sobre `main`.

El recorrido recomendado es:

```text
Agente
  ↓
Branch
  ↓
Pull Request
  ↓
Checks + Reviews + Policies
  ↓
Merge
```

El agente puede preparar y ejecutar trabajo, pero **proponer y ejecutar no significa decidir**.

El Pull Request proporciona un punto donde pueden aplicarse:

- required checks
- required reviews
- CODEOWNERS
- rulesets
- branch protection
- análisis de seguridad
- aprobación humana

---

# 5. Protección de `main`

Para una rama protegida como `main`, una configuración razonable debería exigir:

- Pull Request para integrar cambios
- required checks exitosos
- approving reviews
- CODEOWNERS para áreas sensibles
- restricción del push directo
- bloqueo de force pushes
- bloqueo de eliminación de la rama

Un workflow de validación puede activarse ante cada PR:

```yaml
on:
  pull_request:
    branches: [main]
```

Los checks deberían estar relacionados con riesgos concretos:

- build
- unit tests
- lint
- security scanning
- validaciones de políticas
- controles especiales para `infra/` y `.github/workflows/`

---

# 6. CODEOWNERS

`CODEOWNERS` permite asignar responsables según las áreas modificadas.

```text
/security/          @security-team
/infra/             @platform-team
/.github/workflows/ @platform-team
*                   @core-team
```

De esta forma, un cambio de infraestructura o de workflows puede involucrar automáticamente a los equipos responsables.

Importante: tener un archivo `CODEOWNERS` no es suficiente. La configuración del repositorio debe **exigir las reviews correspondientes**.

---

# 7. Human in the loop

Los humanos no necesitan intervenir en cada operación.

Deberían hacerlo principalmente en **puntos de decisión de alto impacto**:

- merge
- deployment
- acceso a secretos
- cambios de infraestructura
- modificación de controles de seguridad

El flujo típico es:

```text
Agente crea PR
      ↓
Checks automáticos
      ↓
CODEOWNERS / reviewers
      ↓
Aprobación
      ↓
Merge
```

Esto permite conservar velocidad sin renunciar al control.

---

# 8. Pipeline: aprobación para producción

Los GitHub Environments permiten introducir una aprobación antes de ejecutar un deployment sensible.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest

    environment:
      name: production

    steps:
      - run: ./deploy.sh
```

El agente puede preparar el deployment y ejecutar las validaciones previas, mientras que la autorización final permanece en manos de los responsables del entorno.

---

# 9. Evitar deployments simultáneos

Los deployments sobre producción no deberían ejecutarse de forma descontrolada en paralelo.

```yaml
concurrency:
  group: production
  cancel-in-progress: true
```

La serialización ayuda a evitar:

- race conditions
- releases incompatibles
- estados inconsistentes
- ejecuciones difíciles de auditar

---

# 10. Defense in depth

Los controles no deberían depender de una sola barrera.

Un modelo robusto combina:

1. **Pre-action constraints**
   - permisos
   - tool allowlists
   - hooks

2. **In-action validation**
   - tests
   - security scans
   - policy checks

3. **Post-action traceability**
   - logs
   - artifacts
   - approvals
   - historial del PR

> Una instrucción al agente no equivale a un control técnico.

Decir `"no modifiques producción"` orienta al modelo. Un Environment protegido, permissions o un hook pueden impedir técnicamente la operación.

---

# 11. Pipeline de guardrails

Un segundo workflow puede comprobar requisitos adicionales antes del merge.

```yaml
name: agent-guardrails

on:
  pull_request:
    branches: ["main"]

permissions:
  contents: read

jobs:
  enforce-human-review:
    runs-on: ubuntu-latest

    steps:
      - name: Fail if PR is authored by Copilot agent
        if: ${{ github.event.pull_request.user.login == 'github-copilot[bot]' }}
        run: |
          echo "PRs authored by github-copilot[bot] must be opened/owned by a human."
          exit 1

      - name: Require at least one human approval
        uses: actions/github-script@v7
        with:
          script: |
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const pull_number = context.payload.pull_request.number;

            const reviews =
              await github.rest.pulls.listReviews({
                owner,
                repo,
                pull_number
              });

            const approvedByHuman = reviews.data.some(r =>
              r.state === "APPROVED" &&
              r.user &&
              r.user.type === "User" &&
              r.user.login !== "github-copilot[bot]"
            );

            if (!approvedByHuman) {
              core.setFailed(
                "Human approval is required before merge."
              );
            }
```

Este workflow es **defensa en profundidad**. No reemplaza branch protection, rulesets o CODEOWNERS.

---

# 12. Least privilege

Los permisos determinan la capacidad real del agente.

La regla práctica es:

> **Read-only por defecto. Elevar privilegios solamente cuando una operación realmente los necesita.**

Ejemplo:

```yaml
permissions:
  contents: read
  pull-requests: write
```

Una opción más segura es mantener el workflow en lectura y elevar permisos únicamente en el job correspondiente.

```yaml
permissions:
  contents: read

jobs:

  analyze:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v5

      - run: echo "Read-only analysis"

  update_artifacts:
    runs-on: ubuntu-latest

    permissions:
      contents: write
      pull-requests: write

    steps:
      - uses: actions/checkout@v5

      - run: echo "Write operations happen here"
```

Esto reduce el **blast radius** de un fallo o una acción inesperada.

---

# 13. Secretos

Los secretos nunca deberían escribirse directamente en:

- código fuente
- instrucciones del agente
- archivos versionados
- YAML del workflow en texto plano

Deben:

- inyectarse durante runtime
- existir únicamente donde sean necesarios
- estar limitados por Environment cuando corresponda
- seguir el principio de mínimo privilegio

El acceso a secretos de producción debe considerarse una operación crítica.

---

# 14. Observabilidad y evidencia

Un sistema agéntico confiable debe producir evidencia suficiente para reconstruir lo ocurrido.

Como mínimo debemos poder responder:

1. ¿Qué intentó hacer el agente?
2. ¿Qué modificó?
3. ¿Qué validaciones se ejecutaron?
4. ¿Cuál fue el resultado?
5. ¿Quién aprobó el cambio?

GitHub proporciona múltiples fuentes de evidencia:

- Pull Requests
- commits
- workflow runs
- job logs
- required checks
- security scans
- reviews
- approvals
- artifacts

---

# 15. Pipeline: conservar evidencia

Los resultados relevantes pueden almacenarse como artifacts.

```yaml
- name: Upload execution report
  uses: actions/upload-artifact@v4
  with:
    name: execution-report
    path: report.json
```

Un workflow crítico debería producir evidencia como:

```text
plan.json
test-results/
security-report/
execution-report.json
```

La regla operativa puede resumirse así:

> **Sin evidencia, no hay validación suficiente.**

Si un cambio no puede auditarse posteriormente, no debería considerarse una ejecución confiable.

---

# 16. Tool access para agentes

La autonomía también se controla por las herramientas disponibles.

Un agente que solamente necesita analizar el historial Git no debería recibir permisos generales sobre la terminal.

Ejemplo con Copilot CLI:

```bash
copilot -p "Summarize the last 10 commits and highlight breaking changes." \
  --allow-tool 'shell(git)'
```

Para un agente que necesite leer y buscar:

```bash
copilot agent run CodeAgent --allow-tool 'read,search'
```

Principio:

> **Las capacidades disponibles deben corresponder a las consecuencias admitidas.**

---

# 17. Autonomía progresiva

La autonomía debería crecer a medida que el comportamiento del sistema demuestra ser confiable.

Un rollout posible:

1. **Fase 1:** el agente puede abrir PRs pero no hacer merge.
2. **Fase 2:** puede integrar cambios de bajo riesgo después de los checks.
3. **Fase 3:** puede desplegar automáticamente a staging.
4. **Fase 4:** puede preparar producción, pero el Environment exige aprobación humana.

No es necesario comenzar con autonomía máxima.

---

# 18. Gobernanza continua

La gobernanza no termina después de configurar el repositorio.

Los controles pueden degradarse con el tiempo:

- checks eliminados o renombrados
- permisos ampliados
- CODEOWNERS desactualizado
- approvals relajadas
- secretos movidos a scopes más amplios
- nuevos workflows sin los guardrails originales

Una cadencia útil es:

### Semanalmente

- revisar workflows fallidos
- analizar violaciones frecuentes de políticas

### Mensualmente

- revisar permisos
- revisar scopes de secretos

### Trimestralmente

- auditar Rulesets
- CODEOWNERS
- Environment reviewers
- políticas de evidencia y retención

---

# 19. Resiliencia y recuperación

Los sistemas agénticos deben diseñarse suponiendo que habrá fallos.

Buenas prácticas:

- retries acotados para errores transitorios
- escalamiento después de fallos repetidos
- Pull Requests pequeños
- mecanismos simples de revert
- rollback explícito para operaciones de alto impacto
- investigar fallos de seguridad en lugar de reintentarlos indefinidamente

Flujo:

```text
Check falla
   ↓
Agente analiza
   ↓
Corrige la branch
   ↓
Check nuevamente
   ↓
¿Sigue fallando?
   ↓
Escalamiento humano
```

---

# 20. Arquitectura recomendada

```text
Tarea
  ↓
Planning Agent
  ↓
Plan + clasificación de riesgo
  ↓
Aprobación cuando corresponda
  ↓
Implementation Agent
  ↓
Branch
  ↓
Pull Request
  ↓
Tests + Security + CODEOWNERS
  ↓
Required Checks
  ↓
Human Review
  ↓
Merge
  ↓
Environment Gate
  ↓
Deploy
```

La arquitectura separa tres preocupaciones:

### Planning

¿Qué debe hacerse y por qué?

### Execution

¿Qué cambios concretos se realizaron?

### Validation

¿Qué evidencia demuestra que el resultado es correcto y seguro?

---

# Principios finales

1. No trabajar directamente sobre `main`.
2. Hacer visible el plan del agente.
3. Separar Planning, Execution y Validation.
4. Recordar que una instrucción no es un control técnico.
5. Definir criterios de éxito verificables.
6. Registrar evidencia de cada ejecución.
7. Ajustar la autonomía al nivel de riesgo.
8. Diseñar suponiendo que habrá fallos.
9. Aplicar **least privilege** en agentes, workflows y herramientas.
10. Mantener decisiones críticas bajo controles humanos y políticas exigibles.

## Idea fundamental

> **Un agente confiable no es el que tiene más autonomía, sino el que puede actuar dentro de límites explícitos, verificables, auditables y proporcionales al riesgo.**

GitHub permite construir estos límites combinando **Pull Requests, GitHub Actions, Required Checks, Rulesets, Branch Protection, CODEOWNERS, Environments, Secrets, Artifacts y Hooks**.

El objetivo no es eliminar la autonomía, sino hacerla **segura, observable, reversible y gobernable**.
