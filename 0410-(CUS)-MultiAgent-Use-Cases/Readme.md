# Casos de uso de sistemas agénticos en GitHub

Los agentes de IA pueden participar en distintas etapas del ciclo de vida del desarrollo de software. La clave no es solamente qué tareas pueden ejecutar, sino **qué nivel de autonomía reciben, qué controles se aplican y qué evidencia generan**.

## 1. Actualización automática de dependencias

Un agente puede analizar una dependencia vulnerable o desactualizada y preparar su actualización.

### Flujo

1. Detectar la dependencia afectada.
2. Analizar el alcance del cambio.
3. Crear una branch.
4. Actualizar la dependencia y el lockfile.
5. Ejecutar build, tests y análisis de seguridad.
6. Crear un Pull Request.
7. Analizar los resultados de CI.
8. Corregir el cambio si alguna validación falla.
9. Solicitar revisión humana antes del merge.

### Resultado esperado

Un Pull Request revisable que incluya:

- dependencia actualizada;
- cambios realizados;
- resultados de tests;
- comprobaciones de seguridad;
- evidencia de que la vulnerabilidad fue corregida.

---

## 2. Corrección de vulnerabilidades

Un agente puede recibir un Issue asociado a una vulnerabilidad y trabajar sobre una solución.

### Flujo

Issue → Plan → Branch → Implementación → Pull Request → Security Checks → Review → Merge

### Controles recomendados

- limitar qué archivos puede modificar;
- impedir cambios directos sobre `main`;
- ejecutar análisis de seguridad;
- requerir `CODEOWNERS` en componentes sensibles;
- exigir Required Checks;
- requerir aprobación humana antes del merge.

> Que CI termine correctamente no significa necesariamente que la vulnerabilidad haya sido eliminada. El criterio de éxito debe comprobar explícitamente el objetivo de seguridad.

---

## 3. Resolución de Issues

Un Issue puede convertirse en la entrada de una tarea para un agente.

El agente puede:

- interpretar el objetivo;
- analizar el repositorio;
- elaborar un plan;
- identificar los archivos afectados;
- implementar cambios;
- crear commits;
- abrir un Pull Request;
- responder al feedback;
- iterar hasta cumplir los criterios definidos.

### Patrón

Issue → Planning → Implementation → Validation → Pull Request → Human Review

El Issue representa el objetivo y el Pull Request convierte la solución propuesta en un artefacto visible y revisable.

---

## 4. Generación y mantenimiento de documentación

Los cambios sobre documentación suelen representar un riesgo menor que los cambios sobre código, infraestructura o producción.

Un agente puede:

- actualizar archivos `README`;
- generar documentación de nuevas funcionalidades;
- corregir enlaces;
- mantener ejemplos;
- actualizar guías de instalación;
- detectar documentación desactualizada.

### Autonomía posible

Para directorios como `docs/` puede permitirse mayor autonomía si existen validaciones automáticas.

Por ejemplo:

Issue → Agent → Branch → Docs Update → Validation → Pull Request → Automerge

---

## 5. Generación de Pull Requests

El agente puede transformar una tarea en una contribución completa.

El Pull Request puede contener:

### Plan

- objetivo;
- alcance;
- archivos afectados;
- pasos de implementación;
- riesgos;
- criterios de éxito;
- estrategia de rollback.

### Evidencia

- resultados de tests;
- workflow runs;
- análisis de seguridad;
- artifacts;
- validaciones realizadas.

### Revisión

- checks obligatorios;
- revisión de `CODEOWNERS`;
- aprobación humana cuando corresponda.

El agente puede preparar la propuesta, pero la aceptación del cambio permanece gobernada por las políticas del repositorio.

---

## 6. Análisis y corrección de fallos de CI

Un agente puede reaccionar ante fallos producidos durante la validación.

### Ciclo

Check falla
→ Agente analiza el error
→ Modifica la branch
→ Commit
→ CI vuelve a ejecutarse
→ Evaluate

Si el problema puede resolverse, el agente continúa iterando.

Si supera el número permitido de intentos o encuentra un problema que no puede resolver:

→ Escalación humana

### La escalación debería incluir

- qué falló;
- qué intentó el agente;
- qué archivos modificó;
- qué evidencia existe;
- qué recomienda hacer a continuación.

---

## 7. Revisión automática de código

Un agente especializado puede funcionar como `Review Agent`.

Puede analizar:

- cambios introducidos;
- resultados de tests;
- cumplimiento de políticas;
- posibles problemas de seguridad;
- alcance del Pull Request;
- evidencia generada durante CI.

Una arquitectura recomendable es mantener este agente en modo **read-only**.

Implementation Agent → Pull Request → Review Agent → Findings → Human Review

Esto separa la generación del cambio de su evaluación.

---

## 8. Preparación de despliegues

Un agente puede preparar un deployment sin tener autoridad para ejecutarlo directamente en producción.

Puede:

- preparar los artefactos;
- ejecutar tests;
- crear el Pull Request;
- validar configuraciones;
- preparar el deployment.

Para producción:

Agent → Prepare → Validate → Environment Gate → Human Approval → Deploy

Los `Environments` permiten introducir una frontera explícita antes de acceder a recursos o secretos protegidos.

---

## 9. Automatización basada en eventos

Los Agentic Workflows pueden iniciarse mediante distintos eventos de GitHub.

Ejemplos:

- ejecución manual con `workflow_dispatch`;
- creación o modificación de Issues;
- cambios en Pull Requests;
- ejecuciones programadas;
- otros eventos del repositorio.

Esto permite pasar de un agente utilizado bajo demanda a uno integrado directamente en el workflow de desarrollo.

---

## 10. Agentes especializados

En lugar de proporcionar todas las capacidades a un único agente, pueden separarse responsabilidades.

### Planning Agent

Permisos principalmente de lectura.

Responsabilidades:

- analizar;
- investigar;
- definir alcance;
- identificar riesgos;
- proponer un plan.

### Implementation Agent

Permisos de escritura controlados.

Responsabilidades:

- crear branches;
- modificar archivos;
- generar commits;
- crear o actualizar Pull Requests.

### Review Agent

Permisos principalmente de lectura.

Responsabilidades:

- analizar código;
- revisar tests;
- comprobar evidencia;
- verificar cumplimiento de políticas.

### Flujo

Planning Agent
→ Human Approval
→ Implementation Agent
→ Pull Request
→ Automated Checks
→ Review Agent
→ Human Review
→ Merge

---

# Principio general

La autonomía del agente debería ser proporcional al riesgo de la operación.

| Riesgo | Ejemplo | Autonomía sugerida |
|---|---|---|
| Bajo | Documentación, formato | Validaciones + posible automerge |
| Medio | Código y dependencias | PR + checks + review |
| Alto | Infraestructura y workflows | CODEOWNERS + múltiples reviews |
| Crítico | Producción y secretos | Aprobación humana obligatoria |

La arquitectura objetivo no consiste en permitir que el agente haga todo, sino en permitir que haga **todo lo necesario dentro de límites explícitos, verificables y auditables**.
