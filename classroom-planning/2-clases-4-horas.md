# Planificación - Clase 1

**Duración:** 4 horas

## 1. Introducción y presentación del curso - 15 min

- Presentación de la clase y objetivos.
- Conocer la experiencia previa de los alumnos.
- Preguntar qué conocimientos tienen sobre GitHub, IA y agentes.
- Conversar sobre qué los trajo al curso y qué esperan aprender.

## 2. ¿Qué es un agente y qué puede hacer? - 15 min

- Diferencia entre asistente y agente.
- Ver ejemplos de agentes de GH-600.
- Qué puede hacer un agente dentro de GitHub.
- Introducir la idea de que un agente puede planificar, actuar sobre un repositorio e iterar a partir del feedback.

## 3. GitHub Actions y primer workflow - 20 min

- Qué es GitHub Actions.
- Qué es un workflow.
- Conceptos básicos: trigger, job y step.
- Crear un workflow simple.
- Ejecutarlo y observar el resultado.

## 4. Repaso de Issues, Branches y Pull Requests - 15 min

- Qué es un Issue.
- Qué es una Branch.
- Qué es un Pull Request.
- Relación entre Issue, Branch, cambios, Pull Request, checks, review y merge.
- Conectar estos conceptos con el trabajo que posteriormente realizará un agente.

## 5. Break - 15 min

## 6. Kahoot de repaso - 10 min

Preguntas sobre:

- Agentes.
- GitHub Actions.
- Workflows.
- Issues.
- Pull Requests.

## 7. Teoría: construcción de sistemas agénticos - 25 min

- Plan, Act y Evaluate.
- Planning, Execution y Validation.
- Human in the loop.
- Ejecución dentro de un entorno controlado.
- Alcance del agente.
- Permisos.
- Relación entre autonomía y riesgo.
- Importancia de los controles técnicos.

## 8. CODEOWNERS y ejercicio con Pull Request - 20 min

- Qué es CODEOWNERS.
- Ownership y revisión de cambios.
- Relación entre CODEOWNERS, Pull Requests y revisión humana.
- Crear o modificar un archivo CODEOWNERS.
- Realizar un cambio.
- Crear un Pull Request.
- Observar cómo CODEOWNERS participa del proceso de revisión.

## 9. Conceptos previos para crear agentes - 20 min

- GitHub Codespaces.
- GitHub CLI (`gh`).
- GitHub Agentic Workflows (`gh-aw`).
- Fine-grained Personal Access Token.
- GitHub Actions Secrets.
- Carpeta `.github/workflows`.
- Agentic Workflow en Markdown.
- Frontmatter.
- Trigger `workflow_dispatch`.
- Permisos.
- Compilación de `.md` a `.lock.yml`.
- Safe Outputs.

## 10. Laboratorio: Hola Mundo con agentes - 30 min

- Crear un Agentic Workflow mínimo.
- Definir el archivo Markdown.
- Configurar el frontmatter.
- Escribir las instrucciones del agente.
- Compilar el workflow.
- Ejecutarlo desde GitHub Actions.
- Observar la ejecución y los logs.
- Ver el resultado producido por el agente.
- Relacionar la ejecución con Safe Outputs.

## 11. Break - 15 min

## 12. Kahoot de repaso - 10 min

Preguntas sobre:

- Agentic Workflows.
- Frontmatter.
- Triggers.
- Permisos.
- Compilación.
- GitHub Actions.
- Safe Outputs.

## 13. Laboratorio oficial de Mona - 35 min

- Realizar el laboratorio oficial.
- Aplicar los conceptos vistos durante la clase.
- Analizar qué tarea recibe el agente.
- Identificar cómo se ejecuta.
- Observar los permisos y límites.
- Revisar los resultados producidos por el agente.

## 14. Cierre y recap - 10 min

Repasar el recorrido completo de la clase:

1. Objetivo.
2. Agente.
3. Plan.
4. Workflow.
5. Permisos y límites.
6. Ejecución.
7. Evidencia.
8. Revisión humana.

## Distribución total

- Contenido, demostraciones y laboratorios: 190 min
- Kahoots: 20 min
- Breaks: 30 min
- Total: 240 min / 4 horas

---

# Planificación - Clase 2

**Duración:** 4 horas

## 1. Repaso de la clase anterior - 15 min

- Repasar el concepto de agente y su diferencia con un asistente.
- Recordar el ciclo Plan, Act y Evaluate.
- Repasar Agentic Workflows, triggers, permisos y Safe Outputs.
- Recordar la relación entre Issues, Branches, Pull Requests, checks y revisión humana.
- Conectar estos conceptos con memoria, estado y evaluación.

## 2. Teoría: memoria, estado y evaluación de agentes - 30 min

- Qué significa memoria en un sistema agéntico.
- Memoria de corto plazo.
- Memoria de largo plazo.
- Memoria externa y persistente.
- Diferencia entre memoria y estado.
- GitHub como fuente de verdad persistente.
- Uso de Issues, Pull Requests, commits, workflow runs, logs y artifacts para conservar contexto.
- Context drift y pérdida de contexto.
- Evaluación de resultados mediante criterios de éxito.
- Tests, linting, builds, validaciones y controles de seguridad.
- Relación entre memoria, estado, evaluación y mejora continua.

## 3. Laboratorio: Human in the Loop con GitHub Environments - 30 min

- Crear un GitHub Environment.
- Configurar un Environment protegido.
- Configurar una aprobación manual.
- Crear un workflow que utilice el Environment.
- Ejecutar el workflow.
- Observar cómo la ejecución se detiene esperando aprobación.
- Aprobar manualmente la continuación del workflow.
- Analizar qué parte realizó automáticamente el agente y qué decisión quedó en manos de una persona.
- Relacionar el ejercicio con autonomía, riesgo y controles técnicos.

## 4. Break - 15 min

## 5. Kahoot de repaso - 10 min

Preguntas sobre:

- Memoria.
- Estado.
- Evaluación.
- Context drift.
- GitHub Environments.
- Human in the Loop.
- Aprobaciones.
- Logs y artifacts.

## 6. Laboratorio: memoria y continuidad de un agente - 35 min

- Ejecutar una tarea inicial con un agente.
- Registrar el objetivo y los criterios de aceptación.
- Guardar información relevante en artefactos persistentes de GitHub.
- Utilizar un Issue o Pull Request como fuente de contexto.
- Registrar decisiones y progreso.
- Finalizar una primera ejecución.
- Retomar la tarea utilizando el estado persistente.
- Observar qué información puede recuperarse entre ejecuciones.
- Comparar contexto temporal con memoria persistente.
- Identificar información que debería conservarse y qué información debería descartarse.
- Analizar cómo evitar contexto duplicado, contradictorio o desactualizado.

## 7. GitHub Copilot CLI y agentes desde la terminal - 20 min

- Introducción a GitHub Copilot CLI.
- Ejecutar un agente desde la terminal.
- Dar contexto del repositorio al agente.
- Analizar archivos y cambios desde CLI.
- Limitar las herramientas disponibles para el agente.
- Relacionar tool access con mínimo privilegio.
- Comparar el trabajo del agente en CLI con Agentic Workflows.

## 8. Laboratorio: Code Review con GitHub Copilot CLI - 25 min

- Preparar un cambio en una branch.
- Analizar el diff desde GitHub Copilot CLI.
- Pedir al agente que revise el cambio.
- Identificar posibles errores, riesgos y mejoras.
- Revisar el alcance del cambio.
- Comparar los findings del agente con los checks automáticos.
- Mantener el agente de revisión en modo de solo lectura.
- Analizar la separación entre quien genera el cambio y quien lo evalúa.
- Realizar la revisión humana final.

## 9. Break - 15 min

## 10. Kahoot de repaso - 10 min

Preguntas sobre:

- GitHub Copilot CLI.
- Tool access.
- Least privilege.
- Code Review.
- Review Agents.
- Evaluación.
- Human in the Loop.
- Memoria y estado.

## 11. Introducción a sistemas multiagente - 15 min

- Por qué utilizar varios agentes especializados.
- Diferencia entre un agente general y múltiples agentes especializados.
- Separación de responsabilidades.
- Planning Agent.
- Implementation Agent.
- Review Agent.
- Handoffs entre agentes.
- Orquestación secuencial y paralela.
- Aislamiento mediante branches y permisos.
- Evidencia y trazabilidad entre agentes.
- Riesgos de conflictos, trabajo duplicado y ciclos sin convergencia.

## 12. Laboratorio oficial: sistema multiagente - 30 min

- Realizar el laboratorio oficial de multiagentes.
- Identificar los agentes involucrados.
- Analizar la responsabilidad de cada agente.
- Observar cómo se distribuye el trabajo.
- Identificar los handoffs entre agentes.
- Revisar los permisos y el alcance de cada agente.
- Observar cómo se intercambian resultados y evidencia.
- Analizar cómo se coordinan las distintas etapas.
- Identificar los puntos de intervención humana.
- Revisar el resultado final y su trazabilidad.

## 13. Cierre y recap - 10 min

Repasar el recorrido completo de la clase:

1. Memoria.
2. Estado.
3. Evaluación.
4. Human in the Loop.
5. Environments.
6. Persistencia del contexto.
7. GitHub Copilot CLI.
8. Code Review.
9. Agentes especializados.
10. Multiagentes.

## Distribución total

- Contenido, demostraciones y laboratorios: 190 min
- Kahoots: 20 min
- Breaks: 30 min
- Total: 240 min / 4 horas

