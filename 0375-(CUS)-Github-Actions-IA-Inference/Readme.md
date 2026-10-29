# Inferencia de IA en GitHub Actions

## Objetivo

En la guía anterior vimos cómo leer y escribir un workflow de GitHub Actions.

Aprendimos a reconocer elementos como:

```text
on
jobs
runs-on
steps
uses
run
with
env
secrets
```

Ahora vamos a agregar una pieza nueva:

```text
inferencia de IA
```

La pregunta que queremos responder es muy concreta:

> **¿Dónde se utiliza realmente la IA dentro de un workflow?**

Para entenderlo, primero vamos a utilizar un workflow pequeño donde la llamada al modelo sea visible.

Después podremos comprender workflows más complejos donde esa llamada está encapsulada dentro de una Action, una CLI o un runtime de agente.

---

# 1. Un workflow normal no tiene IA

Consideremos este workflow:

```yaml
name: Automatización

on:
  workflow_dispatch:

jobs:
  ejemplo:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Ejecutar script
        run: python script.py
```

Tenemos:

```text
GitHub Actions
      │
      ▼
    runner
      │
      ├── checkout
      │
      └── python script.py
```

GitHub Actions ejecuta pasos.

No hay ningún modelo de IA involucrado solamente porque el workflow se ejecute en GitHub Actions.

---

# 2. Agregar IA significa llamar a un modelo

Supongamos ahora que queremos enviarle una pregunta a un modelo de IA.

Conceptualmente necesitamos:

```text
prompt
   │
   ▼
modelo de IA
   │
   ▼
respuesta
```

Para hacerlo desde un workflow necesitamos algún software que se comunique con el servicio que ofrece ese modelo.

```text
GitHub Actions
      │
      ▼
    runner
      │
      ▼
programa cliente
      │
      ▼
 API de inferencia
      │
      ▼
modelo de IA
```

La IA **no está en el YAML**.

El YAML inicia un programa que solicita una inferencia a un modelo.

---

# 3. Primer ejemplo: la llamada al modelo es visible

El siguiente ejemplo es deliberadamente genérico. La URL, el nombre del modelo y el formato exacto dependen del proveedor utilizado.

```yaml
- name: Consultar modelo
  env:
    AI_TOKEN: ${{ secrets.AI_TOKEN }}

  run: |
    curl https://api.ejemplo.com/v1/responses \
      -H "Authorization: Bearer $AI_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "model": "modelo-x",
        "input": "Explica qué hace este repositorio"
      }'
```

En este step podemos ver directamente las piezas necesarias para solicitar una inferencia.

---

# 4. `env`: proporcionar la credencial

Tenemos:

```yaml
env:
  AI_TOKEN: ${{ secrets.AI_TOKEN }}
```

El valor sensible está almacenado como un Secret y se inyecta como variable de entorno.

```text
GitHub Secret
      │
      ▼
secrets.AI_TOKEN
      │
      ▼
AI_TOKEN
      │
      ▼
programa
```

El token **no es la IA**.

Solamente es una credencial que permite autenticarse contra el servicio.

---

# 5. `curl`: el programa cliente

A continuación aparece:

```yaml
run: |
  curl https://api.ejemplo.com/v1/responses
```

`curl` tampoco es IA.

Es un programa capaz de realizar una petición HTTP.

El runner ejecuta `curl` y `curl` se comunica con la API del proveedor.

---

# 6. El modelo

Dentro de la petición encontramos algo como:

```json
{
  "model": "modelo-x"
}
```

Esto indica qué modelo queremos utilizar.

El modelo normalmente **no se ejecuta dentro del runner de GitHub Actions**. El runner solicita una inferencia a un servicio externo.

---

# 7. El prompt

También enviamos:

```json
{
  "input": "Explica qué hace este repositorio"
}
```

Este texto es la instrucción que recibe el modelo.

```text
"Explica qué hace este repositorio"
              │
              ▼
         API de IA
              │
              ▼
           modelo
              │
              ▼
         inferencia
              │
              ▼
          respuesta
```

Ese procesamiento realizado por el modelo es la **inferencia**.

---

# 8. Qué significa inferencia

Un modelo fue entrenado previamente.

Cuando durante la ejecución le proporcionamos una entrada y obtenemos una respuesta, estamos realizando una inferencia.

```text
INPUT
"Explica qué hace este repositorio"

        │
        ▼

      MODELO

        │
        ▼

OUTPUT
"Este repositorio contiene..."
```

El entrenamiento del modelo no ocurre dentro del workflow.

El workflow utiliza un modelo que ya existe.

---

# 9. Workflow completo con inferencia visible

Este ejemplo lee el `README.md`, construye un JSON con `jq`, consulta una API de IA y guarda la respuesta como artifact.

> **Importante:** la URL y el contrato de la API son ilustrativos. Deben reemplazarse por los valores del proveedor real.

```yaml
name: AI inference example

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  analyze:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Preparar solicitud
        env:
          PROMPT: Analiza el README y propone una mejora concreta
        run: |
          jq -n \
            --arg model "modelo-x" \
            --arg input "$PROMPT\n\n$(cat README.md)" \
            '{model: $model, input: $input}' \
            > request.json

      - name: Solicitar inferencia al modelo
        env:
          AI_TOKEN: ${{ secrets.AI_TOKEN }}
        run: |
          curl --fail-with-body \
            --request POST \
            --url https://api.ejemplo.com/v1/responses \
            --header "Authorization: Bearer $AI_TOKEN" \
            --header "Content-Type: application/json" \
            --data @request.json \
            --output response.json

      - name: Conservar respuesta
        uses: actions/upload-artifact@v4
        with:
          name: ai-response
          path: response.json
          if-no-files-found: error
```

---

# 10. Qué pertenece a GitHub Actions

Estas partes siguen siendo GitHub Actions:

```yaml
on:
jobs:
runs-on:
steps:
env:
permissions:
```

GitHub Actions proporciona la infraestructura donde se ejecuta el programa.

---

# 11. Qué pertenece a la inferencia de IA

La inferencia aparece cuando el programa realiza algo conceptualmente equivalente a:

```text
enviar contexto + instrucciones
            │
            ▼
         MODELO
            │
            ▼
     recibir respuesta
```

Por eso conviene distinguir:

```text
GitHub Actions
      │
      │ ejecuta
      ▼
software cliente
      │
      │ solicita
      ▼
servicio de inferencia
      │
      │ ejecuta
      ▼
modelo
```

---

# 12. La IA puede no aparecer directamente en el YAML

No siempre encontraremos algo como:

```yaml
curl https://api.ejemplo.com/...
```

La llamada puede estar encapsulada dentro de una Action.

```yaml
- name: Analizar repositorio
  uses: ejemplo/ai-action@v1
  with:
    prompt: "Analiza este repositorio"
  env:
    AI_TOKEN: ${{ secrets.AI_TOKEN }}
```

Leyendo solamente el workflow vemos:

```yaml
uses: ejemplo/ai-action@v1
```

Pero internamente esa Action podría hacer:

```text
Action
  │
  ├── preparar prompt
  ├── recopilar contexto
  ├── llamar a la API de IA
  └── procesar la respuesta
```

La inferencia ocurre dentro del software ejecutado por la Action.

---

# 13. `uses` puede ocultar mucha implementación

Recordemos:

```yaml
- uses: actions/checkout@v4
```

significa:

> Ejecutá el software implementado por esa Action.

Exactamente lo mismo ocurre con una Action relacionada con IA:

```yaml
- uses: ejemplo/ai-action@v1
```

El step puede representar muchas líneas de código.

Por lo tanto, al investigar un workflow debemos preguntar:

> **¿Qué hace internamente esta Action?**

---

# 14. Lo mismo puede ocurrir con `run`

Tampoco debemos asumir que un comando sencillo implica una operación sencilla.

```yaml
- name: Ejecutar agente
  run: ai-agent run
```

El YAML solamente muestra:

```text
ai-agent run
```

Pero ese programa podría internamente hacer:

```text
leer instrucciones
       │
       ▼
recopilar contexto
       │
       ▼
construir prompt
       │
       ▼
llamar al modelo
       │
       ▼
recibir respuesta
       │
       ▼
ejecutar una herramienta
       │
       ▼
volver a consultar al modelo
```

`run` nos dice qué programa se ejecuta. No necesariamente nos dice qué hace internamente ese programa.

---

# 15. Automatización determinística e inferencia

## Automatización tradicional

```yaml
- name: Tests
  run: npm test
```

La operación fue especificada previamente:

```text
npm test
   │
   ▼
ejecutar tests
   │
   ▼
resultado
```

## Inferencia

```yaml
- name: Analizar código
  run: ai-client analyze
```

Internamente:

```text
contexto + instrucciones
          │
          ▼
        modelo
          │
          ▼
     respuesta generada
```

La salida depende de la inferencia realizada por el modelo.

---

# 16. Usar IA no significa necesariamente usar un agente

Supongamos este workflow:

```yaml
steps:
  - name: Consultar modelo
    run: ai-client ask "Resume README.md"

  - name: Guardar resumen
    run: ./save-summary.sh
```

Acá utilizamos IA.

Pero el workflow sigue definiendo explícitamente la secuencia:

```text
Paso 1
  ↓
Paso 2
```

Esto puede ser una **automatización que utiliza IA**, pero no necesariamente un agente.

---

# 17. Qué cambia con un agente

En una automatización tradicional programamos todos los pasos intermedios:

```text
Paso 1
↓
Paso 2
↓
Paso 3
```

En un sistema agéntico proporcionamos un objetivo y un conjunto acotado de herramientas.

```text
Objetivo
   │
   ▼
Modelo
   │
   ├── necesita leer un archivo
   │
   ▼
Herramienta
   │
   ▼
Resultado
   │
   ▼
Modelo
   │
   ├── necesita ejecutar otra operación
   │
   ▼
Herramienta
   │
   ▼
Resultado
   │
   ▼
Modelo
   │
   └── produce la salida final
```

El agente mantiene el objetivo, decide pasos intermedios, utiliza herramientas e itera a partir de los resultados.

---

# 18. El loop agéntico

Podemos representarlo así:

```text
        ┌──────────────┐
        │    Modelo    │
        └──────┬───────┘
               │
          decide acción
               │
               ▼
        ┌──────────────┐
        │ Herramienta  │
        └──────┬───────┘
               │
            resultado
               │
               └──────────► Modelo
```

Este ciclo puede repetirse.

El runtime puede:

1. cargar el objetivo y las instrucciones;
2. recopilar contexto;
3. consultar al modelo;
4. interpretar una respuesta o tool call;
5. ejecutar una herramienta permitida;
6. devolver el resultado al modelo;
7. continuar hasta producir una salida o alcanzar un límite.

---

# 19. ¿Dónde está entonces la IA?

Al leer un workflow podemos encontrar tres situaciones.

## Caso A: inferencia visible

```yaml
run: |
  curl https://api.ejemplo.com/v1/responses ...
```

Podemos ver directamente la petición al servicio de IA.

## Caso B: inferencia dentro de una Action

```yaml
uses: ejemplo/ai-action@v1
```

Tenemos que inspeccionar qué implementa esa Action.

## Caso C: inferencia dentro de una CLI o runtime

```yaml
run: ai-agent run
```

Tenemos que entender qué hace internamente ese programa.

La regla mental es:

```text
YAML
 │
 ▼
step
 │
 ├── uses → Action
 │
 └── run  → programa
             │
             ▼
          runtime
             │
             ▼
       llamada al modelo
```

---

# 20. El YAML es un orquestador

GitHub Actions no se convierte en IA.

GitHub Actions continúa haciendo lo que ya conocemos:

```text
recibir evento
↓
crear runner
↓
ejecutar jobs
↓
ejecutar steps
```

Uno de esos steps puede ejecutar software que utiliza un modelo.

Es ese software el que introduce la inferencia.

---

# 21. Relación con GitHub Agentic Workflows

En GitHub Agentic Workflows trabajamos con un archivo fuente como:

```text
hello-agent.md
```

que se compila con:

```bash
gh aw compile .github/workflows/hello-agent.md
```

y produce:

```text
hello-agent.lock.yml
```

La relación conceptual es:

```text
hello-agent.md
       │
       │ gh aw compile
       ▼
hello-agent.lock.yml
       │
       │ GitHub Actions
       ▼
runtime del Agentic Workflow
       │
       ▼
motor de IA
```

El archivo `.lock.yml` es el workflow ejecutable por GitHub Actions. Las instrucciones del Markdown son utilizadas por el runtime del agente.

---

# 22. La credencial de Copilot

En un Agentic Workflow que emplea GitHub Copilot puede aparecer el secreto:

```text
COPILOT_GITHUB_TOKEN
```

Podemos visualizar su recorrido así:

```text
GitHub Secret
      │
      ▼
COPILOT_GITHUB_TOKEN
      │
      ▼
runtime del agente
      │
      ▼
GitHub Copilot
      │
      ▼
modelo
```

El token no realiza la inferencia.

El token permite que el runtime se autentique contra el servicio que proporciona el motor de IA.

---

# 23. Dos niveles de ejecución

## Nivel 1: GitHub Actions

GitHub Actions entiende:

```text
jobs
steps
uses
run
env
permissions
```

y ejecuta el workflow.

## Nivel 2: runtime del agente

El runtime entiende:

```text
objetivo
instrucciones
contexto
modelo
herramientas
resultados
```

y controla la interacción agéntica.

```text
GitHub Actions
      │
      ▼
Agent Runtime
      │
      ▼
     LLM
      │
      ▼
 Tool calls
```

---

# 24. Del modelo a una acción

Supongamos que el objetivo es:

```text
Analiza el README y propone una mejora.
```

El agente podría realizar:

```text
1. Leer README
2. Enviar contenido + objetivo al modelo
3. Obtener una propuesta
4. Generar una salida
```

Pero debemos distinguir:

```text
MODELO
  │
  │ propone
  ▼
ACCIÓN
```

El hecho de que el modelo proponga una acción no significa que automáticamente pueda ejecutarla.

Los permisos, las herramientas disponibles y los controles técnicos determinan qué operaciones están realmente permitidas.

---

# 25. Instrucciones y controles técnicos

Una instrucción como:

```text
No modifiques archivos de producción.
```

orienta el comportamiento del modelo.

Pero no constituye por sí sola un bloqueo técnico.

Los controles técnicos incluyen, por ejemplo:

- permisos mínimos;
- listas de herramientas permitidas;
- branches protegidas;
- Pull Requests;
- required checks;
- required reviews;
- CODEOWNERS;
- Environments y approvals;
- límites de tiempo y reintentos;
- validación de outputs.

Regla mental:

```text
instrucciones
→ orientan al modelo

controles técnicos
→ limitan al sistema
```

---

# 26. Safe Outputs

En GitHub Agentic Workflows podemos declarar salidas controladas.

Por ejemplo:

```yaml
safe-outputs:
  create-issue:
```

Conceptualmente:

```text
Modelo
   │
   ▼
resultado propuesto
   │
   ▼
Safe Output
   │
   ▼
operación permitida
```

El modelo genera o propone el contenido. El mecanismo de salida controlada determina cómo ese resultado puede convertirse en una operación sobre GitHub.

---

# 27. Automatización, IA y agente

## Automatización

```text
input
 ↓
reglas programadas
 ↓
acciones predeterminadas
 ↓
output
```

Ejemplo:

```yaml
- run: npm test
```

## Automatización con IA

```text
input
 ↓
modelo
 ↓
respuesta
 ↓
siguiente step predeterminado
```

Ejemplo conceptual:

```yaml
- run: ai-client summarize README.md
```

## Agente

```text
objetivo
 ↓
modelo
 ↓
decisión
 ↓
herramienta
 ↓
resultado
 └─────► modelo
```

Los pasos intermedios pueden depender de los resultados anteriores.

---

# 28. Cómo investigar un YAML que utiliza IA

Cuando encuentres un workflow nuevo, además de las preguntas de la guía de GitHub Actions, agregá las siguientes.

## 1. ¿Dónde aparece el runtime?

Buscá:

```yaml
uses:
```

y:

```yaml
run:
```

## 2. ¿Qué programa ejecuta?

Por ejemplo:

```yaml
run: herramienta-agentica
```

Preguntate:

> ¿Qué implementa esa herramienta?

## 3. ¿Qué credenciales recibe?

Buscá referencias a:

```text
secrets.
```

Por ejemplo:

```yaml
env:
  AI_TOKEN: ${{ secrets.AI_TOKEN }}
```

## 4. ¿Dónde se configura el modelo?

Puede aparecer explícitamente:

```yaml
model: modelo-x
```

o estar configurado dentro del runtime.

## 5. ¿Dónde están las instrucciones?

Podrían estar en:

```yaml
with:
  prompt: ...
```

En un archivo:

```text
instructions.md
```

o ser cargadas por el runtime desde otro archivo fuente.

## 6. ¿Qué herramientas puede utilizar?

Revisá su configuración, sus permisos y las allowlists correspondientes.

## 7. ¿Qué acciones puede producir?

No confundir:

```text
lo que el modelo propone
```

con:

```text
lo que el sistema permite ejecutar
```

## 8. ¿Qué límites tiene?

Revisá:

```yaml
timeout-minutes:
permissions:
environment:
if:
```

También verificá si el runtime limita iteraciones, reintentos, tokens o herramientas.

## 9. ¿Qué evidencia conserva?

Buscá:

- logs;
- outputs;
- artifacts;
- Issues;
- Pull Requests;
- checks;
- comentarios;
- resultados de seguridad.

---

# 29. Mapa mental completo

Después de las dos guías podemos pensar el sistema completo así:

```text
EVENTO
  │
  │ on
  ▼
GITHUB ACTIONS
  │
  ▼
JOB
  │
  │ runs-on
  ▼
RUNNER
  │
  ▼
STEP
  │
  ├── run
  │
  └── uses
       │
       ▼
SOFTWARE / RUNTIME
       │
       ├── instrucciones
       ├── contexto
       ├── credenciales
       │
       ▼
SERVICIO DE IA
       │
       ▼
MODELO
       │
       ▼
INFERENCIA
       │
       ▼
RESPUESTA
```

Si además estamos frente a un agente:

```text
RESPUESTA
   │
   ▼
decisión / tool call
   │
   ▼
HERRAMIENTA
   │
   ▼
RESULTADO
   │
   └────────► MODELO
```

---

# 30. Chuleta rápida

## Step determinístico

```yaml
- name: Tests
  run: npm test
```

## Inferencia visible

```yaml
- name: Consultar modelo
  env:
    AI_TOKEN: ${{ secrets.AI_TOKEN }}
  run: curl https://api.ejemplo.com/v1/responses ...
```

## Inferencia encapsulada en una Action

```yaml
- name: Analizar repositorio
  uses: ejemplo/ai-action@v1
```

## Inferencia encapsulada en una CLI

```yaml
- name: Ejecutar runtime
  run: ai-agent run
```

## Cadena de ejecución

```text
YAML
↓
step
↓
Action / CLI / runtime
↓
servicio de IA
↓
modelo
↓
inferencia
```

## Cadena agéntica

```text
objetivo
↓
modelo
↓
herramienta
↓
resultado
└────► modelo
```

---

# 31. Idea fundamental

La transición más importante entre GitHub Actions y Agentic Workflows puede resumirse así:

```text
GitHub Actions
 ejecuta steps

        ↓

un step inicia
software agéntico

        ↓

el runtime interactúa
con un modelo

        ↓

el modelo decide
pasos intermedios

        ↓

el runtime ejecuta
acciones permitidas
```

> **La IA no reemplaza a GitHub Actions. La inferencia ocurre dentro de software ejecutado por GitHub Actions.**

Cuando ese software mantiene un objetivo, consulta al modelo, utiliza herramientas y evalúa sus resultados, comenzamos a hablar de un **runtime agéntico**.
