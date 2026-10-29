# Laboratorio 0: Mi primer workflow de GitHub Actions

## Objetivo

En este laboratorio vas a crear y ejecutar un workflow básico de GitHub Actions.

El workflow ejecutará un comando `echo` y mostrará el resultado en los logs.

Al finalizar vas a haber creado:

- Un repositorio de GitHub.
- Un workflow escrito en YAML.
- Una ejecución manual de GitHub Actions.
- Una comprobación del resultado mediante los logs.

---

# Parte 1: Crear el repositorio

## Paso 1. Crear un repositorio

Ingresá a:

https://github.com

Iniciá sesión con tu cuenta.

Seleccioná:

1. **New repository**
2. En **Repository name**, escribí:

```text
mi-primer-workflow
```

3. Podés dejar el repositorio como **Public**.
4. Marcá **Add a README file**.
5. Seleccioná **Create repository**.

El repositorio tendrá inicialmente esta estructura:

```text
mi-primer-workflow
└── README.md
```

---

# Parte 2: Crear el workflow

## Paso 2. Crear el archivo

Dentro del repositorio, seleccioná:

1. **Add file**
2. **Create new file**

Como nombre del archivo escribí:

```text
.github/workflows/mi-primer-workflow.yml
```

GitHub creará automáticamente las carpetas necesarias.

La estructura del repositorio será:

```text
mi-primer-workflow
├── README.md
└── .github
    └── workflows
        └── mi-primer-workflow.yml
```

---

# Parte 3: Escribir el workflow

Copiá el siguiente contenido dentro del archivo:

```yaml
name: Mi primer workflow

on:
  workflow_dispatch:

jobs:
  hola:
    runs-on: ubuntu-latest

    steps:
      - name: Mostrar mensaje
        run: echo "Hola desde GitHub Actions"
```

---

# Parte 4: Entender el workflow

## Nombre

```yaml
name: Mi primer workflow
```

Define el nombre que aparecerá en la pestaña **Actions**.

## Trigger

```yaml
on:
  workflow_dispatch:
```

`workflow_dispatch` permite ejecutar el workflow manualmente.

## Job

```yaml
jobs:
  hola:
```

El workflow contiene un job llamado `hola`.

## Entorno de ejecución

```yaml
runs-on: ubuntu-latest
```

GitHub Actions utilizará una máquina Linux para ejecutar el job.

## Step

```yaml
steps:
  - name: Mostrar mensaje
    run: echo "Hola desde GitHub Actions"
```

El step ejecuta el comando:

```bash
echo "Hola desde GitHub Actions"
```

El resultado aparecerá en los logs.

---

# Parte 5: Guardar el workflow

Seleccioná:

```text
Commit changes...
```

Como mensaje del commit escribí:

```text
Agregar mi primer workflow
```

Confirmá seleccionando:

```text
Commit changes
```

---

# Parte 6: Ejecutar el workflow

## Paso 3. Abrir GitHub Actions

En la parte superior del repositorio, seleccioná:

```text
Actions
```

En la lista de workflows debería aparecer:

```text
Mi primer workflow
```

Seleccioná el workflow.

## Paso 4. Ejecutar el workflow

Seleccioná:

```text
Run workflow
```

Volvé a seleccionar:

```text
Run workflow
```

GitHub Actions iniciará la ejecución.

---

# Parte 7: Revisar los logs

Después de unos segundos aparecerá la ejecución del workflow.

1. Seleccioná la ejecución.
2. Abrí el job:

```text
hola
```

3. Abrí el step:

```text
Mostrar mensaje
```

En los logs deberías encontrar:

```text
Hola desde GitHub Actions
```

---

# Resultado final

El recorrido realizado fue:

```text
Repositorio
    ↓
Archivo YAML
    ↓
GitHub Actions
    ↓
Job
    ↓
Step
    ↓
Comando echo
    ↓
Logs
```

---

# Conceptos principales

## Repositorio

Contiene los archivos del proyecto y su historial de cambios.

## Workflow

Define una automatización de GitHub Actions.

## YAML

Es el formato utilizado para definir el workflow.

## Trigger

Define cuándo comienza la ejecución.

En este laboratorio utilizamos:

```yaml
workflow_dispatch:
```

para iniciar el workflow manualmente.

## Job

Es un trabajo que GitHub Actions debe ejecutar.

## Step

Es un paso dentro de un job.

## Logs

Permiten observar los comandos ejecutados y sus resultados.

---

# Checkpoint

Verificá que puedas responder estas preguntas:

- [ ] ¿Dónde está guardado el workflow?
- [ ] ¿Qué extensión tiene el archivo?
- [ ] ¿Qué significa `workflow_dispatch`?
- [ ] ¿Cómo se llama el job?
- [ ] ¿Qué comando ejecuta el workflow?
- [ ] ¿Dónde observamos el resultado?

---

# Desafío opcional

Agregá un segundo step debajo de **Mostrar mensaje**:

```yaml
- name: Mostrar fecha
  run: date
```

El workflow completo quedará así:

```yaml
name: Mi primer workflow

on:
  workflow_dispatch:

jobs:
  hola:
    runs-on: ubuntu-latest

    steps:
      - name: Mostrar mensaje
        run: echo "Hola desde GitHub Actions"

      - name: Mostrar fecha
        run: date
```

Guardá el cambio, ejecutá nuevamente el workflow y revisá los logs.

---

# Conclusión

En este laboratorio creaste un workflow tradicional de GitHub Actions.

El flujo utilizado fue:

```text
Persona
    ↓
Run workflow
    ↓
GitHub Actions
    ↓
Comando definido en YAML
    ↓
Logs
```

Este workflow todavía no utiliza agentes ni inteligencia artificial.

Es una automatización básica que sirve como punto de partida para comprender cómo GitHub Actions ejecuta jobs y steps.
