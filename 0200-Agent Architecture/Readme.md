# Arquitectura de sistemas agénticos en GitHub

## Introducción

Los sistemas agénticos están cambiando la forma en que se desarrolla y mantiene el software.

En lugar de depender exclusivamente de desarrolladores que escriben y modifican código, los equipos pueden utilizar agentes capaces de:

- Interpretar objetivos.
- Analizar el estado de un repositorio.
- Proponer soluciones.
- Crear ramas.
- Modificar archivos.
- Crear Pull Requests.
- Ejecutar procesos de validación.
- Iterar sobre los cambios a partir de pruebas, análisis de seguridad y revisiones de código.

Sin embargo, **darle más capacidades a un agente no lo convierte automáticamente en un agente confiable**.

Un sistema agéntico debe diseñarse de manera que sus acciones sean:

- Limitadas.
- Observables.
- Auditables.
- Validables.
- Reversibles.
- Gobernadas por políticas.

La idea central es:

> **Los agentes proponen y ejecutan dentro de límites definidos. Las personas y las políticas deciden qué se acepta.**

GitHub proporciona varios mecanismos para implementar esta arquitectura:

- Issues
- Branches
- Pull Requests
- GitHub Actions
- Rulesets y Branch Protection
- CODEOWNERS
- Environments
- Secrets
- Artifacts
- Hooks
- MCP

---

## 1. Conceptos fundamentales

### Plan

Un **plan** es un artefacto que describe qué cambios se pretende realizar.

Normalmente puede incluir:

- Objetivo.
- Alcance.
- Archivos que se modificarán.
- Pasos de implementación.
- Riesgos.
- Criterios de éxito.
- Estrategia de rollback.

El plan puede aparecer, por ejemplo, en la descripción de un Pull Request.

### Planning

**Planning** es la capacidad del agente para generar o refinar ese plan a partir de una tarea, Issue o instrucción.

Por lo tanto:

- **Planning** es una capacidad del agente.
- **Plan** es el artefacto producido.

---

## 2. Relacionar las responsabilidades del agente con el SDLC

Un agente no debería tener acceso irrestricto a todo el ciclo de vida del software.

Una arquitectura
