# Memoria, estado y evaluación de agentes en GitHub

A medida que los agentes realizan tareas más largas y complejas, necesitan mecanismos para **recordar información relevante, mantener el estado del trabajo, continuar tareas entre sesiones y evaluar si los resultados son correctos**.

En GitHub, esto puede gestionarse mediante elementos persistentes como **Issues, Pull Requests, commits, GitHub Actions, logs y artifacts**.

## 1. Memoria del agente

La memoria permite que un agente conserve la información necesaria para trabajar de manera consistente.

### Memoria de corto plazo

Es el contexto utilizado durante la tarea actual.

Puede incluir:

- Instrucciones recientes.
- Feedback recibido.
- Decisiones inmediatas.
- Pasos necesarios para continuar.

Esta información normalmente está asociada a la sesión actual y no necesariamente se conserva entre sesiones.

### Memoria de largo plazo

Contiene conocimiento seleccionado que puede reutilizarse en tareas futuras.

Por ejemplo:

- Decisiones importantes.
- Patrones.
- Conocimiento resumido.
- Información reutilizable.

### Memoria externa

La memoria externa se almacena fuera del contexto interno del agente, utilizando sistemas persistentes.

En GitHub puede incluir:

- Issues.
- Pull Requests.
- Documentación.
- Commits.
- Logs.
- Resultados de workflows.
- Artifacts.

Esta memoria proporciona información persistente que puede revisarse posteriormente.

## 2. Elegir dónde almacenar la información

Cada tipo de información debería tener una ubicación clara:

- **Requisitos y criterios de aceptación → Issue**
- **Decisiones y progreso → Pull Request**
- **Reglas de validación → Instrucciones del repositorio**
- **Resultados → Workflow runs, logs y artifacts**
- **Procesos repetibles → Instrucciones o skills reutilizables**

La memoria debería limitarse a la información que realmente afecta al resultado de la tarea.

Esto incluye principalmente:

- Requisitos.
- Restricciones.
- Decisiones.
- Métodos de validación.
- Criterios de aceptación.

Evitar información innecesaria o duplicada ayuda a reducir la confusión y el uso de contexto desactualizado.

## 3. Estado del agente

La **memoria** permite recordar qué información es importante.

El **estado** representa:

- Qué se hizo.
- Qué decisiones se tomaron.
- Qué falta realizar.
- Qué resultados se obtuvieron.

En GitHub, este estado puede representarse mediante:

- Issues.
- Pull Requests.
- Commits.
- Workflow runs.
- Logs.
- Artifacts.

Estos elementos permiten conservar el progreso y continuar una tarea en otra sesión sin comenzar nuevamente desde cero.

## 4. Pull Request como punto central del estado

El Pull Request puede funcionar como el lugar central para seguir el trabajo realizado por un agente.

Puede contener:

- Descripción de la tarea.
- Criterios de aceptación o referencia al Issue.
- Plan de implementación.
- Decisiones tomadas.
- Actualizaciones del progreso.
- Commits.
- Resultados de workflows.
- Discusiones y revisiones.

Cuando el trabajo se reanuda, el agente puede:

1. Abrir el Pull Request existente.
2. Revisar su descripción y el Issue relacionado.
3. Verificar los commits realizados.
4. Revisar los resultados de los workflows.
5. Continuar desde el último estado conocido.

## 5. Context drift

El **context drift** ocurre cuando las acciones del agente comienzan a desviarse del objetivo original o de decisiones tomadas anteriormente.

Puede detectarse comprobando:

- Si los cambios cumplen los criterios de aceptación.
- Si los commits contradicen decisiones anteriores.
- Si existen workflows fallidos.
- Si faltan validaciones esperadas.

Para corregirlo:

1. Revisar nuevamente el Issue o Pull Request.
2. Comparar los cambios con los criterios de aceptación.
3. Actualizar la descripción si las decisiones cambiaron.
4. Ejecutar nuevamente las validaciones necesarias.

## 6. Expiración, limpieza y reinicio de memoria

No toda la información debe conservarse indefinidamente.

### Expiración

Los logs y artifacts de GitHub Actions tienen un período de retención y posteriormente pueden eliminarse.

### Pruning

La información antigua o innecesaria debería eliminarse para evitar contexto obsoleto.

Por ejemplo:

- Eliminar artifacts innecesarios.
- Eliminar workflow runs obsoletos.
- Evitar almacenar información redundante.

### Resumen

Un historial extenso puede reducirse a información esencial.

Por ejemplo:

- Actualizar el Pull Request con las decisiones finales.
- Referenciar commits importantes.
- Vincular ejecuciones de workflows en lugar de duplicar sus logs.

### Reset

Puede ser necesario reiniciar parte del contexto cuando:

- Cambian significativamente los requisitos.
- Se selecciona un nuevo enfoque.
- Las suposiciones anteriores dejan de ser válidas.

## 7. Continuidad entre herramientas y entornos

Un agente puede comenzar una tarea en un IDE, continuar mediante CLI y finalizar en GitHub.

Para mantener la continuidad deberían utilizarse referencias persistentes como:

- Número del Pull Request.
- Nombre de la rama.
- SHA del commit.
- URL del Issue.
- URL del Pull Request.
- Referencia a un workflow run.

En lugar de depender exclusivamente del historial temporal de una sesión, cada entorno puede consultar el mismo estado persistente almacenado en GitHub.

GitHub funciona así como una **fuente de verdad compartida** entre diferentes herramientas y entornos.

## 8. Evitar contexto conflictivo

El contexto entra en conflicto cuando la misma información aparece en diferentes lugares con valores distintos.

Para evitarlo:

- Definir una única fuente de verdad para cada información.
- No duplicar requisitos innecesariamente.
- Actualizar la fuente original cuando una decisión cambia.
- Mantener sincronizados Issues y Pull Requests con el estado actual.

Por ejemplo, si cambian los criterios de aceptación, es preferible actualizar el Issue correspondiente en lugar de definir criterios diferentes en nuevos prompts.

## 9. Evitar contexto desactualizado

El agente puede utilizar información que alguna vez fue correcta pero ya no representa el estado actual.

Esto puede suceder cuando:

- Una rama quedó desactualizada respecto de la rama base.
- Los resultados de un workflow corresponden a una versión anterior.
- Cambiaron los requisitos.
- Existen commits más recientes.

Antes de continuar una tarea, el agente debería revisar:

- Commits recientes.
- Estado actual de la rama.
- Issue y Pull Request.
- Últimos workflow runs.
- Resultados de validación.

## 10. Evaluación del trabajo del agente

La evaluación determina si el trabajo realizado está completo y es correcto.

Todo comienza definiendo **criterios de éxito claros**.

Por ejemplo:

- La funcionalidad se comporta como se esperaba.
- Los tests finalizan correctamente.
- No se introducen nuevos problemas de seguridad.

Los criterios de aceptación pueden definirse en el Issue y referenciarse posteriormente desde el Pull Request.

## 11. GitHub Actions como mecanismo de evaluación

GitHub Actions permite automatizar parte de la evaluación.

Un workflow puede ejecutar tareas como:

1. Tests.
2. Linting.
3. Compilación.
4. Validaciones.
5. Controles de seguridad.

Los resultados aparecen asociados al Pull Request y pueden revisarse mediante los **status checks**, los workflow runs y la pestaña **Checks**.

Los workflows se almacenan normalmente en:

`.github/workflows/`

## 12. Quality gates

GitHub permite impedir que determinados cambios se integren hasta cumplir ciertas condiciones.

Por ejemplo:

- Tests obligatorios.
- Status checks requeridos.
- Revisión obligatoria.
- Aprobación del Pull Request.
- Rama actualizada antes del merge.
- Restricciones sobre pushes directos.

Estas políticas pueden implementarse utilizando **branch protection rules** y otros mecanismos de protección del repositorio.

Un flujo típico puede ser:

1. Un Issue define la tarea y los criterios de aceptación.
2. El agente crea una rama.
3. El agente abre un Pull Request.
4. GitHub Actions ejecuta los workflows.
5. Los status checks muestran los resultados.
6. Los checks obligatorios deben aprobarse.
7. Un revisor aprueba los cambios.
8. El Pull Request puede integrarse.

## 13. Seguridad como parte de la evaluación

La evaluación también puede incluir controles de seguridad.

GitHub dispone de mecanismos como:

- **Code scanning**.
- **Dependency review**.
- **Secret scanning**.
- **Push protection**.

Los resultados de seguridad pueden utilizarse como parte de los criterios necesarios antes de permitir un merge.

## 14. Análisis de fallos del agente

Los agentes no siempre completan correctamente una tarea en el primer intento.

Los fallos pueden analizarse utilizando:

- Logs de GitHub Actions.
- Cambios realizados en el Pull Request.
- Discusiones.
- Historial de commits.
- Resultados de workflows.
- Artifacts.

El análisis consiste en comparar la **intención original** con el **resultado producido**.

La intención puede estar definida en:

- **Issue → requisitos y criterios de aceptación**
- **Pull Request → plan y decisiones**

Mientras que los resultados aparecen en:

- Commits.
- Cambios de código.
- Workflow outputs.
- Logs.

## 15. Clasificación de fallos

Los fallos pueden agruparse en tres categorías principales.

### Errores de razonamiento

El agente interpreta o decide incorrectamente.

Ejemplos:

- Interpretar mal los requisitos.
- Implementar una lógica incorrecta.
- Ignorar criterios de aceptación.

### Uso incorrecto de herramientas

El agente utiliza incorrectamente los mecanismos disponibles.

Ejemplos:

- Workflow mal configurado.
- Comandos incorrectos.
- Workflow que no se ejecuta cuando debería.
- Permisos incorrectos.

### Problemas de contexto

El agente trabaja con información incorrecta, incompleta o desactualizada.

Ejemplos:

- Utilizar un Pull Request desactualizado.
- Perder decisiones anteriores.
- Tener información contradictoria entre artifacts.

## 16. Mejorar el comportamiento del agente

Una vez identificada la causa del problema, pueden realizarse mejoras en tres áreas.

### Prompts e instrucciones

- Aclarar los criterios de aceptación.
- Agregar restricciones.
- Mejorar las instrucciones del repositorio.

### Memoria y estado

- Actualizar Issues y Pull Requests.
- Eliminar información obsoleta.
- Evitar contexto contradictorio.
- Mantener una única fuente de verdad.

### Configuración de herramientas

- Corregir workflows.
- Revisar eventos como `push` y `pull_request`.
- Verificar permisos.
- Configurar los checks necesarios.

## 17. Ciclo de mejora continua

El comportamiento del agente puede mejorarse mediante un ciclo iterativo:

1. Ejecutar el agente.
2. Observar los resultados.
3. Analizar logs y artifacts.
4. Identificar la causa del fallo.
5. Corregir prompts, memoria, estado o herramientas.
6. Ejecutar nuevamente el workflow.
7. Evaluar nuevamente el resultado.

Este ciclo permite mejorar progresivamente la confiabilidad del sistema.

## Resumen conceptual

El funcionamiento confiable de un agente puede entenderse mediante cuatro elementos principales:

**Memoria → Estado → Evaluación → Mejora**

- **Memoria:** conserva la información relevante.
- **Estado:** registra qué ocurrió y qué falta hacer.
- **Evaluación:** determina si el resultado cumple los criterios establecidos.
- **Mejora:** analiza los fallos y ajusta instrucciones, memoria o herramientas.

En GitHub, los **Issues, Pull Requests, commits, GitHub Actions, logs y artifacts** proporcionan la infraestructura necesaria para mantener estos elementos de forma persistente y trazable.

La idea central es que un agente no debería depender únicamente de su contexto temporal. El repositorio y sus artifacts proporcionan una **fuente de verdad persistente** que permite mantener continuidad, validar resultados y mejorar el comportamiento del agente a lo largo del tiempo.
