# Desarrollo en sistemas de IA agéntica

## 1. Evolución del desarrollo asistido por IA

El desarrollo asistido por inteligencia artificial está evolucionando desde herramientas que simplemente **sugieren código** hacia sistemas capaces de **realizar acciones dentro del ciclo de vida del desarrollo de software (SDLC)**.

En GitHub, esta evolución puede observarse en experiencias como el **cloud agent de Copilot**, que puede:

- Explorar un repositorio.
- Proponer un plan de trabajo.
- Realizar cambios en una rama.
- Abrir un **Pull Request (PR)** para que un desarrollador lo revise.

Esto modifica el rol del desarrollador. Ya no se trata solamente de escribir código, sino también de:

- **Guiar** sistemas de IA.
- **Supervisar** sus acciones.
- **Validar** los resultados generados.
- Controlar sistemas capaces de **planificar, actuar y mejorar** dentro de los workflows de desarrollo.

---

## 2. ¿Qué es la IA agéntica en el SDLC?

Muchos desarrolladores ya utilizan IA siguiendo un modelo de **asistente**:

**Prompt → Respuesta → Control vuelve al usuario**

Un **agente** va más allá. Puede:

- Interpretar un objetivo.
- Mantener ese objetivo durante múltiples pasos.
- Determinar acciones intermedias.
- Utilizar herramientas.
- Ejecutar acciones dentro de un workflow.
- Crear o modificar artefactos persistentes.
- Utilizar feedback para continuar o modificar su trabajo.

La diferencia es importante porque la IA pasa de simplemente **ayudar con el desarrollo** a **participar activamente en el proceso de desarrollo**.

---

## 3. Asistente vs. agente

### Asistente

Los sistemas basados en asistentes son principalmente **reactivos**:

- Generan sugerencias o explicaciones.
- Pueden sugerir código, explicar resultados o resumir cambios.
- Dependen del usuario para decidir qué hacer a continuación.
- No realizan acciones sobre el repositorio.
- El usuario debe ejecutar manualmente los distintos pasos.
- No hacen avanzar independientemente el trabajo dentro del repositorio.

### Agente

Los sistemas basados en agentes están **orientados a objetivos**:

- Interpretan una tarea.
- Elaboran un enfoque para resolverla.
- Mantienen un objetivo durante varios pasos.
- Deciden acciones intermedias.
- Utilizan herramientas.
- Ejecutan acciones dentro del workflow.
- Generan resultados persistentes como **branches, commits y Pull Requests**.
- Iteran a partir de feedback proveniente de checks, reviews o análisis.

La diferencia fundamental puede resumirse así:

> **Un asistente propone. Un agente puede proponer y actuar dentro del workflow.**

---

## 4. Comportamiento agéntico dentro de GitHub

En GitHub, el comportamiento del agente aparece mediante las mismas estructuras que ya utilizan los desarrolladores:

- **Branches y commits:** muestran qué cambió.
- **Pull Requests:** muestran qué se propone, por qué y permiten revisarlo.
- **Workflows y checks:** proporcionan evidencia sobre el cambio.
- **Review comments y approvals:** registran qué aceptaron o rechazaron las personas.

El agente, por lo tanto, **no reemplaza el workflow**.

> **El agente ingresa al workflow como un participante.**

Un modelo frecuente está orientado al Pull Request:

**Tarea → Branch → Cambios → Commits → Pull Request → Checks/Reviews → Validación → Merge**

---

## 5. Ejemplo: agente vs. asistente

### Comportamiento de un agente

Se genera una alerta de seguridad. El agente:

1. Crea una branch, por ejemplo `agent/bump-dep-2026-04-03`.
2. Actualiza una dependencia y el lockfile.
3. Abre un Pull Request con un resumen y un plan.
4. Espera los checks de CI y el feedback de revisión.
5. Realiza modificaciones adicionales si son necesarias.

### Comportamiento de un asistente

El usuario pregunta:

> "¿Cómo actualizo esta dependencia de manera segura?"

El asistente proporciona:

- Comandos recomendados.
- Un checklist de riesgos.
- Cambios de código sugeridos.

Pero es el usuario quien debe crear la branch, realizar los cambios y abrir el Pull Request.

---

## 6. Ciclo de vida de un agente

Los sistemas agénticos no toman una única decisión y terminan. Funcionan mediante ciclos.

El modelo fundamental es:

**Plan → Act → Evaluate**

No es una secuencia que ocurre una sola vez, sino un **loop**:

**Plan → Act → Evaluate → Plan → Act → Evaluate → ...**

El agente continúa iterando hasta que:

- Se cumplen los criterios de éxito definidos, o
- El trabajo se escala a una persona.

### 6.1 Plan

Durante la planificación, el agente:

- Interpreta el objetivo.
- Determina qué pasos necesita ejecutar.
- Define cómo alcanzar el resultado esperado.

En sistemas de calidad, los planes no deberían quedar como estados internos ocultos. Deben convertirse en **artefactos estructurados y revisables**.

Ejemplos dentro de GitHub:

- Un plan estructurado en la descripción del Pull Request.
- Un Issue relacionado.
- Un checklist que especifique alcance y criterios de éxito.

Un buen plan debería definir:

- **Scope:** qué va a cambiar.
- **Success criteria:** cómo determinar que funcionó.
- **Rollback o escalation path:** qué hacer si algo falla o no puede resolverse.

### 6.2 Act

Durante la fase de acción, el agente ejecuta el plan dentro del repositorio.

Puede:

- Crear una branch.
- Modificar archivos.
- Crear y enviar commits.
- Abrir o actualizar un Pull Request.
- Responder al feedback de una revisión realizando nuevos cambios.

Esto permite mantener la ejecución **acotada**:

- A un repositorio determinado.
- A una branch.
- Al workflow de Pull Requests.

En lugar de permitir cambios directos y no controlados sobre la branch por defecto.

### 6.3 Evaluate

En esta etapa, el agente y las personas que lo supervisan utilizan señales del sistema para determinar si el resultado es aceptable.

En GitHub pueden utilizarse:

#### Checks técnicos

- Builds.
- Tests.
- Linting.
- Workflow runs.
- Status checks.

#### Code Review

- Requested changes.
- Review comments.
- Approvals.

#### Señales de seguridad

- Code scanning.
- Resultados SARIF.
- Secret scanning.
- Dependency alerts.
- Push protection.

La idea fundamental es:

> **La evaluación debe basarse en señales del sistema, no en la confianza que el agente tenga en su propio resultado.**

Si los checks fallan, permanecen riesgos o no se cumplen los requisitos, el trabajo todavía no terminó.

El ciclo continúa:

**Evaluate → nuevo Plan → nueva Action → nueva Evaluation**

---

## 7. Ejemplo completo del ciclo Plan → Act → Evaluate

Supongamos que un agente debe resolver una vulnerabilidad actualizando una dependencia.

### Plan

Define:

- Qué paquete debe actualizarse.
- Qué archivos deberían modificarse.
- Qué resultado se espera obtener.

### Act

El agente:

- Crea una branch.
- Actualiza la dependencia.
- Modifica los archivos correspondientes.
- Genera commits.
- Abre un Pull Request.

### Evaluate

GitHub ejecuta:

- CI.
- Tests.
- Checks.
- Señales de seguridad.

Además, las personas pueden revisar el Pull Request.

Si un workflow falla o la vulnerabilidad continúa:

**el trabajo no está terminado.**

El agente debe:

- Revisar el plan.
- Modificar el cambio.
- Volver a evaluar.
- O escalar el problema a una persona.

Un sistema agéntico de calidad hace visibles las tres etapas:

- **Plan:** inspeccionable.
- **Act:** limitado a workflows controlados.
- **Evaluate:** basado en señales objetivas.

---

## 8. GitHub como System of Record

Los sistemas agénticos necesitan un entorno capaz de:

- Capturar intención.
- Registrar acciones.
- Aplicar políticas.
- Ejecutar validaciones.

En este modelo, **GitHub funciona como System of Record** porque almacena los artefactos mediante los cuales el trabajo se propone, ejecuta y evalúa.

Entre ellos:

- Repositories.
- Branches.
- Commits.
- Pull Requests.
- Issues.
- Discussions.
- Workflow runs.
- Workflow artifacts.
- Review history.

Estos elementos cumplen una doble función:

1. Permiten realizar el desarrollo.
2. Permiten inspeccionar posteriormente qué hizo el agente.

---

## 9. GitHub como Control Plane

GitHub también funciona como **Control Plane** porque puede proporcionar los puntos de control que determinan lo que las contribuciones de un agente pueden y no pueden hacer.

| Control de GitHub | Qué controla | Importancia para agentes |
|---|---|---|
| Pull Requests | Los cambios se proponen antes del merge | Permiten revisar y discutir el trabajo del agente |
| Required Reviews | Revisión antes del merge | Evitan cambios sin supervisión |
| Required Status Checks | Evidencia de CI antes del merge | Convierten la evaluación en una política |
| CODEOWNERS | Enrutamiento de reviews según archivos | Involucra especialistas en áreas sensibles |
| Rulesets / Branch Protection | Políticas sobre branches | Evitan merges inseguros |
| Environments | Aprobaciones para deployments/secrets | Controlan ejecuciones y accesos sensibles |

Estos mecanismos dependen de su **configuración y de los permisos establecidos**.

Por ejemplo, los required checks o rulesets deben estar configurados para transformarse efectivamente en mecanismos obligatorios.

---

## 10. GitHub Actions dentro del Control Plane

GitHub Actions también forma parte del control porque los workflows constituyen uno de los lugares donde se valida la ejecución.

Sin embargo, los permisos son tan importantes como los checks.

Debe aplicarse el principio de **least privilege**:

- Definir permisos conservadores para los workflow tokens.
- Utilizar permisos read-only cuando sea posible.
- Otorgar permisos superiores únicamente a los jobs que realmente los necesiten.
- Utilizar Environments y approvals para controlar secrets y deployments sensibles.

En sistemas agénticos:

> **Lo que puede hacer el agente está estrechamente relacionado con lo que pueden hacer sus workflow tokens y las credenciales de las herramientas que utiliza.**

---

## 11. Ejemplos de mecanismos de control

### Ejecución de workflows aprobada por humanos

En determinados workflows de PR, una persona puede tener que aprobar explícitamente la ejecución mediante acciones como:

**Approve and run workflows**

Esto impide que cambios no confiables ejecuten automáticamente workflows con privilegios.

### Environments

Si un job utiliza un Environment que requiere reviewers:

1. El workflow llega al Environment.
2. La ejecución queda en espera.
3. Una persona debe aprobarla.
4. Solo entonces puede continuar.

Esto puede impedir acceso automático a:

- Secrets protegidos.
- Deployments sensibles.

### CODEOWNERS

Si un agente modifica rutas sensibles, por ejemplo:

- `.github/workflows/`
- `infra/`

CODEOWNERS puede solicitar automáticamente la revisión de los responsables correspondientes.

Combinado con required reviews, ayuda a garantizar que las modificaciones de alto impacto sean supervisadas por las personas adecuadas.

---

## 12. Control de una contribución de un agente

Si un agente abre un Pull Request con un security fix, GitHub puede:

1. Hacer visible el cambio mediante el PR.
2. Solicitar los reviewers apropiados mediante CODEOWNERS.
3. Evaluarlo mediante workflows y required checks.
4. Bloquear el merge hasta que se cumplan las políticas.
5. Impedir acceso a secrets protegidos de un Environment hasta obtener las aprobaciones correspondientes.

Por lo tanto:

> **GitHub no es solamente donde queda almacenado el trabajo del agente. También es donde ese trabajo puede ser supervisado, validado y gobernado.**

---

## 13. Responsabilidad en sistemas agénticos

Que un agente pueda ejecutar trabajo **no significa que asuma la responsabilidad sobre el resultado**.

Las personas y equipos continúan siendo responsables de:

- Definir la tarea.
- Establecer permisos.
- Elegir y configurar los controles.
- Revisar los resultados.
- Aprobar los cambios.

Por lo tanto:

> **La ejecución puede delegarse al agente; la responsabilidad permanece en las personas y equipos.**

El modelo de Pull Request lo hace explícito:

**El sistema puede proponer. Las personas deciden qué se acepta.**

---

## 14. Riesgos y anti-patterns

Los sistemas agénticos pueden presentar varios patrones de riesgo.

### Planless execution

El agente comienza a modificar código sin disponer de un plan claro e inspeccionable.

**Ejemplo:** un PR contiene un diff pero no explica el plan ni el motivo.

**Mitigación:**

- PR Template con una sección obligatoria de planificación.
- Review antes del merge.

### Over-permissioned agents

El agente, workflow token o credenciales utilizadas tienen más privilegios de los necesarios.

**Mitigación:**

- Least-privilege `GITHUB_TOKEN`.
- Environments con required reviewers.
- Restricciones sobre quién puede ejecutar workflows.

### Hidden reasoning

Solamente se expone el resultado final, por ejemplo el diff, sin mostrar:

- Plan.
- Alcance.
- Supuestos.
- Decisiones.
- Contexto de ejecución.

**Mitigación:**

- Registrar el plan.
- Relacionar workflow runs.
- Registrar decisiones dentro del Pull Request.

### Blind trust in automation

Se considera que un cambio es correcto simplemente porque CI pasó.

Pero:

> **Un build exitoso no significa automáticamente que el cambio sea completo, apropiado o de bajo riesgo.**

Los checks solamente validan aquello para lo cual fueron diseñados.

**Mitigación:**

Combinar:

- Automated checks.
- CODEOWNERS.
- Required Reviews.
- Aprobaciones basadas en riesgo.

---

## 15. Trazabilidad y observabilidad

Para supervisar correctamente un agente no alcanza con observar el diff final.

Debe existir un **trail** que permita reconstruir lo sucedido.

En GitHub puede incluir:

- Pull Requests.
- Commit history.
- Review comments.
- Approvals.
- Workflow runs.
- Test reports.
- Logs.
- Workflow artifacts.
- Code scanning alerts.
- Secret scanning alerts.
- Push protection events.
- Organization audit logs, dependiendo de la configuración disponible.

El objetivo no es solamente compliance.

También permite responder ante un problema:

- ¿Qué cambió?
- ¿Por qué cambió?
- ¿Quién lo aprobó?
- ¿Qué evidencias existían?
- ¿Qué sucedió posteriormente?

---

## 16. Audit trail mínimo para contribuciones de agentes

Una contribución debería disponer como mínimo de:

1. **Goal:** objetivo declarado mediante Issue o descripción del PR.
2. **Plan:** plan inspeccionable.
3. **Bounded changeset:** branch y commits claramente delimitados.
4. **Automated evidence:** workflow runs y artifacts.
5. **Human judgment:** review y aprobación.
6. **Clear outcome:** merge, revert o escalamiento.

Por ejemplo, si un agente corrige una vulnerabilidad, CI pasa, pero posteriormente aparece una regresión, debería poder determinarse:

- Si existía un plan y scope visibles.
- Si los reviewers adecuados participaron.
- Si los checks correspondían al riesgo del cambio.
- Si existe información suficiente para reconstruir lo sucedido.

---

## 17. Contributor Model

Una forma de evaluar el trabajo producido por agentes es dejar de considerarlo como algo completamente diferente del trabajo humano.

En su lugar:

> **El agente se trata como un contributor y su trabajo se evalúa como una contribución.**

En GitHub, el **Pull Request es la unidad natural de contribución**.

Independientemente de si el autor es una persona o un agente, deben hacerse las mismas preguntas:

- ¿El cambio resuelve el problema?
- ¿El scope es apropiado y está explicado?
- ¿Los checks requeridos pasan?
- ¿Los responsables adecuados revisaron las áreas afectadas?
- ¿El cambio respeta estándares, arquitectura y políticas?

---

## 18. Evitar los dos extremos

El Contributor Model evita dos errores opuestos.

### Excessive suspicion

Rechazar un cambio simplemente porque:

> "Lo escribió una IA."

### Excessive trust

Aceptar automáticamente un cambio porque:

> "Lo produjo una automatización."

El principio correcto es:

> **Evaluar el trabajo utilizando los estándares del workflow, no según quién o qué lo produjo.**

---

## 19. Checklist para revisar un Pull Request generado por un agente

### Intent

- ¿Existe un objetivo claro?
- ¿Existe un plan visible?

### Scope

- ¿Los archivos modificados corresponden con el plan?

### Evidence

- ¿Pasaron los required checks?
- ¿Existen logs o artifacts cuando son necesarios?

### Ownership

- ¿Los CODEOWNERS correspondientes revisaron las áreas sensibles?

### Policy

- ¿El cambio cumple con rulesets?
- ¿Respeta branch rules?
- ¿Respeta las políticas de Environments?

### Fallback

- ¿Existe un mecanismo claro de rollback?
- ¿Está establecido cuándo debe escalarse el cambio a una persona?

---

## 20. Características de una buena contribución agéntica

Una contribución de un agente correctamente supervisada debería ser:

### Understandable

Tiene un objetivo y un plan claros.

### Bounded

El alcance está limitado y se aplica least privilege.

### Reviewable

Existen evidencias y participan los owners correspondientes.

### Policy-compliant

Respeta:

- Rulesets.
- Branch rules.
- Environments.
- Controles definidos para el repositorio.

### Reconstructable

Existe suficiente trazabilidad como para realizar posteriormente un análisis de lo sucedido.

Estos criterios **no constituyen un estándar especial para IA**.

Son los principios de un workflow de ingeniería saludable, aplicados de manera consistente también al trabajo generado por agentes.

---

## 21. Modelo conceptual completo

El funcionamiento puede sintetizarse de la siguiente manera:

**Objetivo**

↓

**Agente interpreta la tarea**

↓

**Plan**

- Scope.
- Success criteria.
- Rollback / escalation.

↓

**Act**

- Branch.
- Cambios.
- Commits.
- Pull Request.

↓

**Evaluate**

- CI.
- Tests.
- Status Checks.
- Security Signals.
- Reviews.
- Approvals.

↓

**¿Cumple los criterios?**

- **No:** revisar → modificar → volver a evaluar.
- **Sí:** puede continuar hacia el merge según las políticas configuradas.
- **No puede resolverlo:** escalar a una persona.

Todo este proceso queda contenido dentro de los mecanismos de **registro, control, revisión y trazabilidad de GitHub**.

---

## 22. Cambio en el rol del desarrollador

El desarrollo agéntico cambia quién puede realizar parte del trabajo, pero **no elimina la responsabilidad humana sobre el resultado**.

El rol evoluciona desde:

**Desarrollador → escribe y modifica código**

hacia:

**Desarrollador → define objetivos + guía agentes + configura controles + supervisa acciones + revisa evidencias + valida resultados**

El agente puede participar activamente en el SDLC, pero las personas siguen siendo responsables de establecer los límites y decidir finalmente qué cambios se aceptan.

---

## 23. Ideas fundamentales del módulo

- La **IA agéntica** puede participar activamente en el SDLC.
- Un **asistente** principalmente responde y recomienda; un **agente** puede mantener objetivos, decidir pasos, utilizar herramientas y actuar.
- Los agentes trabajan utilizando un ciclo iterativo de **Plan → Act → Evaluate**.
- El resultado de **Evaluate** alimenta una nueva iteración cuando el resultado todavía no es aceptable.
- La evaluación debe basarse en **señales objetivas del sistema**, no en la confianza del propio agente.
- En GitHub, el agente participa mediante mecanismos ya existentes como **branches, commits, Pull Requests, workflows y reviews**.
- GitHub funciona como **System of Record** porque conserva los artefactos, decisiones y evidencias del desarrollo.
- GitHub funciona como **Control Plane** porque puede aplicar políticas y limitar lo que una contribución puede hacer.
- **Pull Requests, Required Reviews, Required Status Checks, CODEOWNERS, Rulesets, Branch Protection y Environments** permiten supervisar y restringir la actividad.
- Los workflows y credenciales deben aplicar **least privilege**.
- La ejecución puede ser realizada por agentes, pero **la responsabilidad sobre los resultados sigue siendo humana**.
- Deben evitarse anti-patterns como **planless execution, over-permissioned agents, hidden reasoning y blind trust in automation**.
- La **trazabilidad y observabilidad** permiten reconstruir qué ocurrió y por qué.
- Mediante el **Contributor Model**, el trabajo del agente se evalúa como cualquier otra contribución.
- No debe confiarse ni desconfiarse de un cambio únicamente porque fue generado por IA.
- Un buen sistema agéntico mantiene el trabajo **understandable, bounded, reviewable, policy-compliant y reconstructable**.
- El agente **no reemplaza el workflow de desarrollo**: se incorpora a él como un participante sujeto a sus controles.

---

## 24. Módulos relacionados

El contenido forma parte del recorrido sobre **Developing in Agentic AI Systems**:

- [Developing Agent Architecture and SDLC Integration](https://learn.microsoft.com/en-us/training/modules/design-agent-architecture-integration/)
- [Tooling, MCP, and Agent Execution Environments](https://learn.microsoft.com/en-us/training/modules/agent-tooling-mcp-execution-environments/)
