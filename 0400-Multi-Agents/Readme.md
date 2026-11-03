# Sistemas multiagente en GitHub

## Introducción

Los sistemas multiagente son cada vez más comunes en el desarrollo moderno de software. En lugar de utilizar un único agente para realizar tareas aisladas, se pueden utilizar varios agentes especializados en diferentes responsabilidades, por ejemplo:

- Actualización de dependencias.
- Corrección de vulnerabilidades.
- Refactorización de código.
- Mejora de documentación.
- Generación de reportes del repositorio.
- Validación de cambios.

Cuando varios agentes trabajan sobre un mismo repositorio, el principal desafío deja de ser simplemente **si un agente puede realizar una tarea**. El problema pasa a ser **cómo coordinar múltiples agentes de manera segura**.

Sin una estrategia de coordinación clara, los agentes pueden:

- Modificar los mismos archivos.
- Crear Pull Requests que se superponen.
- Duplicar trabajo.
- Ejecutar workflows repetidamente.
- Generar conflictos entre cambios.
- Entrar en ciclos de ejecución sin convergencia.

GitHub permite coordinar estos sistemas mediante elementos nativos como **branches, Pull Requests, GitHub Actions, required checks, reviews, CODEOWNERS, environments, artifacts y políticas del repositorio**.

La idea central es:

> **Los agentes proponen cambios. Las políticas y las personas determinan si esos cambios son aceptados.**

---

# 1. Responsabilidades de los agentes

Un sistema multiagente funciona correctamente cuando cada agente posee una responsabilidad claramente definida.

Si varios agentes tienen responsabilidades demasiado amplias, pueden modificar los mismos componentes y producir cambios incompatibles.

Por ejemplo:

| Agente | Responsabilidad |
|---|---|
| Dependency Agent | Actualizar dependencias y lockfiles |
| Refactoring Agent | Modificar código de `src/` |
| Security Agent | Analizar vulnerabilidades y validar resultados |
| Documentation Agent | Actualizar documentación |

Una práctica importante consiste en definir **qué puede modificar cada agente y qué debe evitar modificar**.

Por ejemplo:

- El agente de dependencias puede modificar manifiestos y lockfiles.
- El agente de refactorización puede modificar `src/`, pero no dependencias o workflows.
- El agente de seguridad puede generar análisis, reportes o correcciones relacionadas con vulnerabilidades.

Esto reduce la superposición entre agentes y facilita la revisión de los cambios.

## Relación con el SDLC

Las responsabilidades también pueden asociarse con diferentes etapas del ciclo de vida del desarrollo de software.

| Etapa del SDLC | Responsabilidad | Elemento de GitHub |
|---|---|---|
| Planning | Definir objetivo, alcance y riesgos | PR / `PLAN.md` |
| Implementation | Realizar cambios | Branch + commits + PR |
| Validation | Generar evidencia | Actions + checks + artifacts |
| Acceptance | Revisar y aprobar | CODEOWNERS + reviews |
| Deployment | Controlar despliegues | Environments + approvals |

---

# 2. Orquestación de agentes con GitHub Actions

Una vez definidas las responsabilidades, es necesario determinar **cuándo se ejecuta cada agente y cómo se relacionan sus tareas**.

En GitHub, la orquestación puede implementarse mediante **GitHub Actions y eventos del repositorio**.

Algunos disparadores habituales son:

- Ejecuciones programadas mediante `schedule`.
- Eventos relacionados con Pull Requests.
- Finalización de otros workflows.
- Ejecución manual mediante `workflow_dispatch`.

Existen dos modelos principales de orquestación.

## Orquestación secuencial

Se utiliza cuando una tarea depende del resultado de otra.

Ejemplo:

1. Un agente actualiza una dependencia y crea un Pull Request.
2. El workflow de CI ejecuta las pruebas.
3. Se ejecuta una validación de seguridad.
4. Una persona revisa el cambio.
5. El Pull Request puede fusionarse cuando se cumplen todas las condiciones.

El siguiente paso solo comienza cuando el anterior ha terminado.

## Orquestación paralela

Se utiliza cuando varios agentes pueden trabajar simultáneamente sin interferirse.

Por ejemplo:

- Un agente actualiza documentación.
- Otro agente refactoriza código.

Ambos pueden trabajar en paralelo siempre que sus ámbitos estén correctamente aislados.

## Patrón Fan-Out / Fan-In

Un patrón habitual consiste en ejecutar varios trabajos en paralelo y posteriormente combinar sus resultados.

```text
                ┌─ Spec Analyzer ──┐
Inicio ─────────┤                  ├── Plan Merger
                └─ Risk Reviewer ──┘
```

Los primeros trabajos representan el **fan-out**.

El trabajo que espera sus resultados representa el **fan-in**.

En GitHub Actions, la dependencia entre jobs puede definirse mediante `needs`.

---

# 3. Coordinación mediante artifacts

En sistemas multiagente es conveniente evitar dependencias ocultas entre agentes.

Una estrategia más segura consiste en utilizar **artifacts compartidos y revisables**.

Un agente puede:

1. Ejecutar una tarea.
2. Generar un resultado estructurado.
3. Publicarlo como artifact.
4. Permitir que otro workflow o proceso utilice ese resultado.

Ejemplos de artifacts:

- Reportes.
- Resultados de análisis.
- Planes.
- Archivos JSON.
- Resultados de validaciones.

Esto crea una separación entre:

**razonamiento → evidencia → ejecución controlada**

Además, los artifacts permiten conservar evidencia sobre las acciones realizadas por los agentes.

---

# 4. Aislamiento de ejecución

El aislamiento evita que varios agentes interfieran entre sí.

Existen diferentes niveles de aislamiento.

## Branches

Cada agente debería trabajar desde una branch independiente.

Ejemplos:

```text
agent/dependency/ticket-123
agent/refactor/ticket-456
agent/security/ticket-789
```

Esto permite identificar fácilmente qué agente realizó cada cambio.

## Workflows

Los agentes pueden utilizar workflows independientes o jobs separados con permisos específicos.

Esto permite controlar individualmente:

- Triggers.
- Permisos.
- Outputs.
- Validaciones.

## Permisos

Los workflows deben utilizar el principio de **mínimo privilegio**.

Por ejemplo, un workflow que solamente necesita leer el repositorio y modificar información de un Pull Request podría utilizar:

```yaml
permissions:
  contents: read
  pull-requests: write
```

El agente obtiene únicamente los permisos necesarios para realizar su tarea.

---

# 5. Control de concurrencia

Cuando varios workflows se ejecutan simultáneamente pueden producirse ejecuciones superpuestas.

GitHub Actions permite controlar este comportamiento mediante `concurrency`.

Un patrón habitual es:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Esto hace que la concurrencia quede aislada por **workflow y branch**.

Si comienza una nueva ejecución del mismo workflow sobre la misma branch, la ejecución anterior puede cancelarse automáticamente.

Esto resulta especialmente útil cuando un Pull Request recibe actualizaciones frecuentes.

---

# 6. Detección y resolución de conflictos

Incluso cuando existe aislamiento, pueden aparecer conflictos.

Los principales tipos son:

### Conflictos de merge

Dos agentes modifican los mismos archivos o líneas.

### Conflictos semánticos

Los cambios pueden fusionarse técnicamente, pero el resultado combinado rompe el comportamiento de la aplicación.

### Conflictos de políticas

Algunos archivos pueden necesitar revisiones o aprobaciones especiales.

### Trabajo duplicado

Dos agentes intentan resolver el mismo problema utilizando soluciones diferentes.

GitHub permite detectar y controlar estos escenarios mediante:

- Pull Requests.
- Required checks.
- Branch protection.
- CODEOWNERS.
- Reviews.
- GitHub Actions.

## CODEOWNERS

`CODEOWNERS` permite determinar qué personas o equipos deben revisar cambios según las rutas modificadas.

Ejemplo:

```text
/security/ @security-team
/.github/workflows/ @platform-team
/infra/ @platform-team
* @core-team
```

De esta forma, los cambios sensibles se dirigen automáticamente hacia los responsables correspondientes.

---

# 7. Escalamiento a personas

La automatización no debería intentar resolver un problema indefinidamente.

Es conveniente definir límites de reintentos.

Por ejemplo, puede escalarse una situación cuando:

- Un conflicto continúa después de dos intentos.
- Un required check falla dos veces por la misma causa.
- Dos agentes proponen soluciones incompatibles.
- Una operación requiere una decisión de alto riesgo.

El escalamiento debería incluir información sobre:

- Qué ocurrió.
- Qué intentó hacer el agente.
- Qué resultado obtuvo.
- Qué opciones existen.

La intervención humana forma parte del diseño del sistema multiagente.

---

# 8. Observabilidad

Un sistema multiagente debe permitir entender **qué hizo cada agente, cuándo lo hizo y por qué**.

GitHub proporciona distintos elementos para mantener esta trazabilidad:

- Pull Requests.
- Workflow runs.
- Checks.
- Logs.
- Artifacts.
- Commits.
- Issues.
- Reviews.

Una acción importante debería dejar una evidencia persistente dentro del repositorio.

Por ejemplo, pueden utilizarse convenciones para identificar los Pull Requests:

```text
[agent: dependency] Update package
[agent: security] Fix vulnerability
[agent: refactor] Refactor authentication module
```

También pueden utilizarse labels:

```text
agent: dependency
agent: security
agent: refactor
```

Esto permite identificar rápidamente qué agente produjo cada cambio.

---

# 9. Evidencia y handoffs

Los Pull Requests pueden funcionar como punto de transferencia de información entre agentes y personas.

Una estructura útil puede contener:

```markdown
## Objective

## Plan

## Evidence

## Decisions and handoffs

## Risks and rollback
```

De esta forma quedan registrados:

- El objetivo.
- El plan.
- La evidencia.
- Las decisiones.
- Los responsables.
- Los riesgos.
- La estrategia de recuperación.

Los **handoffs** permiten transferir una tarea desde un agente hacia otro agente o hacia una persona.

---

# 10. Diagnóstico de fallos

Los sistemas multiagente pueden fallar incluso cuando cada agente individual funciona correctamente.

Algunos problemas frecuentes son:

### Ejecución parcial

El agente crea un Pull Request pero no genera la evidencia o validación esperada.

### Workflow bloqueado

Los checks fallan repetidamente o nunca llega una aprobación.

### Outputs incompatibles

Dos Pull Requests intentan introducir cambios que no pueden coexistir.

### Flapping

Los workflows se ejecutan repetidamente sin llegar a un estado estable.

GitHub permite investigar estos problemas mediante:

- Timeline del Pull Request.
- Required checks.
- Historial de GitHub Actions.
- Logs.
- Artifacts generados.

---

# 11. Recuperación ante errores

Un sistema multiagente confiable debe asumir que los errores van a ocurrir.

Las principales estrategias de recuperación son:

## Reintentos limitados

Un agente puede intentar corregir un problema, pero debe existir un número máximo de intentos.

Esto evita ciclos infinitos.

## Rollback

Los cambios deberían ser pequeños y estar correctamente aislados para permitir una reversión sencilla.

## Escalamiento

Si la automatización no consigue resolver el problema después de los intentos permitidos, el proceso debe detenerse y solicitar intervención humana.

La idea es:

> **Retry → límite → escalamiento humano**

---

# 12. Control de operaciones de alto riesgo

Las operaciones sensibles, como los despliegues en producción, pueden protegerse mediante **GitHub Environments**.

Por ejemplo, un environment denominado:

```text
production
```

puede requerir aprobación antes de permitir que el workflow continúe.

Esto introduce una barrera adicional entre la automatización y las operaciones críticas.

---

# 13. Subagentes y handoffs

Un agente puede delegar determinadas tareas en otros agentes especializados.

Por ejemplo:

```text
Planner
   │
   ├── Implementer
   │
   └── Code Review
```

El agente principal actúa como orquestador y los subagentes realizan tareas específicas.

Los **handoffs** permiten definir cómo una tarea pasa de un agente a otro.

Esto permite construir sistemas donde diferentes agentes participan en distintas etapas del proceso:

```text
Planning → Implementation → Validation → Review
```

---

# Arquitectura conceptual

Un sistema multiagente sobre GitHub puede representarse de forma simplificada como:

```text
Agentes especializados
        ↓
Branches aisladas
        ↓
Pull Requests
        ↓
GitHub Actions
        ↓
Checks + Artifacts + Logs
        ↓
CODEOWNERS + Reviews + Policies
        ↓
Merge / Deployment
```

GitHub actúa como **plataforma de coordinación, control y trazabilidad** entre los diferentes agentes.

---

# Conceptos clave

Un sistema multiagente confiable en GitHub se apoya en seis principios principales:

1. **Responsabilidad:** cada agente tiene una función y un alcance claramente definidos.
2. **Orquestación:** GitHub Actions coordina cuándo y cómo se ejecutan los agentes.
3. **Aislamiento:** branches, workflows, permisos y concurrency evitan interferencias.
4. **Arbitraje:** checks, CODEOWNERS y reviews permiten resolver conflictos.
5. **Observabilidad:** Pull Requests, logs y artifacts permiten rastrear cada acción.
6. **Recuperación:** retries limitados, rollback y escalamiento humano permiten responder ante fallos.

## Idea principal

Los sistemas multiagente no consisten simplemente en ejecutar varios agentes al mismo tiempo.

El objetivo es crear un sistema donde **cada agente tenga una responsabilidad clara, trabaje de forma aislada, produzca evidencia observable y entregue sus resultados mediante mecanismos controlados de GitHub**.

La confiabilidad no depende únicamente de la capacidad individual de los agentes, sino de cómo **GitHub coordina, valida, controla y registra la interacción entre ellos**.
