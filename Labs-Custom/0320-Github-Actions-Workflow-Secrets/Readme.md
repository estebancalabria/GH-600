# Laboratorio 0: Mi primer workflow de GitHub Actions

## Objetivo

En este laboratorio vas a crear y ejecutar tu primer workflow de GitHub Actions.

No necesitás tener experiencia previa con GitHub Actions ni con archivos YAML.

Al finalizar vas a haber creado:

- Un repositorio de GitHub.
- Un Personal Access Token.
- Un Secret dentro del repositorio.
- Un workflow escrito manualmente en YAML.
- Una ejecución manual de GitHub Actions.
- Una comprobación desde los logs.

## Resultado esperado

Al terminar el laboratorio tendrás este recorrido funcionando:

```text
Repositorio
    ↓
Secret
    ↓
Workflow YAML
    ↓
GitHub Actions
    ↓
Logs
```

El workflow mostrará un mensaje y comprobará que puede acceder a un Secret sin revelar su contenido.

---

# Parte 1: Crear un repositorio

## Paso 1. Abrir GitHub

Ingresá a:

https://github.com

Iniciá sesión con tu cuenta.

## Paso 2. Crear un repositorio

En GitHub:

1. Seleccioná **New repository**.
2. En **Repository name**, escribí:

```text
mi-primer-workflow
```

3. Podés dejar el repositorio como **Public**.
4. Marcá **Add a README file**.
5. Seleccioná **Create repository**.

Cuando termine el proceso deberías estar viendo tu nuevo repositorio.

## ¿Qué acabamos de crear?

Un repositorio es el lugar donde GitHub almacena los archivos de nuestro proyecto y mantiene su historial de cambios.

Por ahora nuestro repositorio contiene solamente un archivo:

```text
README.md
```

---

# Parte 2: Crear una credencial

GitHub permite utilizar tokens como credenciales.

En este laboratorio vamos a crear un **Personal Access Token**, también llamado **PAT**.

> Importante: el token es una credencial. No debemos escribirlo directamente dentro de nuestro código o del archivo YAML.

## Paso 3. Crear un Personal Access Token

En GitHub:

1. Seleccioná tu foto de perfil.
2. Entrá en **Settings**.
3. Entrá en **Developer settings**.
4. Seleccioná **Personal access tokens**.
5. Seleccioná **Fine-grained tokens**.
6. Seleccioná **Generate new token**.

Usá un nombre como:

```text
laboratorio-github-actions
```

Elegí una expiración corta adecuada para el laboratorio.

Configurá el token con el alcance mínimo que necesites para este repositorio.

Generá el token.

## Paso 4. Copiar el token

Cuando GitHub muestre el token, copialo.

Guardalo temporalmente en un lugar seguro.

No lo pegues en:

- README.md
- Archivos YAML
- Código fuente
- Issues
- Commits

Vamos a almacenarlo como un **GitHub Actions Secret**.

---

# Parte 3: Crear un Secret

Un token y un Secret no son lo mismo.

```text
TOKEN = la credencial

SECRET = el lugar seguro donde almacenamos la credencial
```

El workflow no necesita conocer el valor directamente.

GitHub Actions obtiene el valor del Secret durante la ejecución.

## Paso 5. Abrir la configuración del repositorio

Volvé al repositorio:

```text
mi-primer-workflow
```

Seleccioná:

```text
Settings
```

Después:

```text
Secrets and variables
```

Luego:

```text
Actions
```

## Paso 6. Crear el Secret

Seleccioná:

```text
New repository secret
```

En **Name** escribí:

```text
MI_TOKEN
```

En **Secret** pegá el Personal Access Token que generaste anteriormente.

Seleccioná:

```text
Add secret
```

Deberías ver ahora:

```text
MI_TOKEN
```

en la lista de secrets del repositorio.

No vas a poder volver a consultar el valor almacenado.

Eso es normal.

---

# Parte 4: Crear nuestro primer workflow

Ahora vamos a crear manualmente un archivo de GitHub Actions.

Un workflow de GitHub Actions se define mediante un archivo YAML.

Los workflows se almacenan dentro del repositorio en:

```text
.github/workflows/
```

## Paso 7. Crear el archivo

Volvé a la pestaña:

```text
Code
```

Seleccioná:

```text
Add file
```

Luego:

```text
Create new file
```

Como nombre del archivo escribí exactamente:

```text
.github/workflows/mi-primer-workflow.yml
```

GitHub va a crear automáticamente las carpetas necesarias.

La estructura será:

```text
mi-primer-workflow
│
├── README.md
│
└── .github
    └── workflows
        └── mi-primer-workflow.yml
```

---

# Parte 5: Escribir el workflow

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

      - name: Comprobar que existe el secret
        env:
          MI_TOKEN: ${{ secrets.MI_TOKEN }}
        run: |
          if [ -n "$MI_TOKEN" ]; then
            echo "El secret está disponible."
          else
            echo "El secret NO está disponible."
            exit 1
          fi
```

Todavía no guardes el archivo.

Primero vamos a entender qué escribimos.

---

# Parte 6: Entender el YAML

No es necesario memorizar la sintaxis.

Vamos a leer el archivo de arriba hacia abajo.

## name

```yaml
name: Mi primer workflow
```

Define el nombre del workflow.

Ese es el nombre que veremos posteriormente en la pestaña **Actions**.

---

## on

```yaml
on:
  workflow_dispatch:
```

Define cuándo se puede ejecutar el workflow.

`workflow_dispatch` significa que podremos iniciarlo manualmente desde GitHub.

En este laboratorio no queremos que el workflow se ejecute automáticamente.

Queremos decidir nosotros cuándo probarlo.

---

## jobs

```yaml
jobs:
```

Un workflow puede contener uno o más trabajos.

Cada trabajo se denomina **job**.

Nuestro workflow tiene un solo job:

```yaml
hola:
```

---

## runs-on

```yaml
runs-on: ubuntu-latest
```

Le indica a GitHub qué tipo de máquina debe utilizar para ejecutar el job.

En este caso utilizaremos una máquina Linux proporcionada para la ejecución del workflow.

No necesitás instalar esa máquina.

GitHub prepara el entorno cuando ejecutamos el workflow.

---

## steps

```yaml
steps:
```

Un job está formado por uno o más pasos.

Nuestro primer paso es:

```yaml
- name: Mostrar mensaje
  run: echo "Hola desde GitHub Actions"
```

Ejecuta este comando:

```bash
echo "Hola desde GitHub Actions"
```

El resultado aparecerá en los logs de GitHub Actions.

---

# Parte 7: Usar el Secret

Ahora observemos el segundo paso:

```yaml
- name: Comprobar que existe el secret
  env:
    MI_TOKEN: ${{ secrets.MI_TOKEN }}
```

La expresión:

```text
${{ secrets.MI_TOKEN }}
```

le indica a GitHub Actions:

> Buscá el Secret llamado MI_TOKEN y hacé que esté disponible durante este paso.

Después ejecutamos:

```bash
if [ -n "$MI_TOKEN" ]; then
  echo "El secret está disponible."
else
  echo "El secret NO está disponible."
  exit 1
fi
```

El propósito del laboratorio no es mostrar el token.

Solamente queremos comprobar que GitHub Actions puede acceder al Secret.

## Regla importante

Nunca hagas esto:

```yaml
run: echo "${{ secrets.MI_TOKEN }}"
```

No necesitamos imprimir una credencial para comprobar que existe.

En este laboratorio solamente comprobaremos si su valor está disponible.

---

# Parte 8: Guardar el workflow

Ahora sí vamos a guardar el archivo.

Seleccioná:

```text
Commit changes...
```

Como mensaje del commit podés usar:

```text
Agregar mi primer workflow
```

Confirmá con:

```text
Commit changes
```

El archivo ya forma parte del repositorio.

---

# Parte 9: Ejecutar el workflow

Llegó el momento de probarlo.

## Paso 8. Abrir GitHub Actions

En la parte superior del repositorio seleccioná:

```text
Actions
```

En la lista de workflows debería aparecer:

```text
Mi primer workflow
```

Seleccioná el workflow.

## Paso 9. Ejecutarlo manualmente

Seleccioná:

```text
Run workflow
```

Luego volvé a seleccionar:

```text
Run workflow
```

Acabás de iniciar manualmente una ejecución.

---

# Parte 10: Observar la ejecución

Después de unos segundos debería aparecer una ejecución del workflow.

Seleccioná esa ejecución.

Vas a encontrar el job:

```text
hola
```

Abrilo.

Vas a ver los diferentes pasos realizados por GitHub Actions.

Buscá:

```text
Mostrar mensaje
```

Deberías encontrar en los logs:

```text
Hola desde GitHub Actions
```

Ahora abrí:

```text
Comprobar que existe el secret
```

Deberías encontrar:

```text
El secret está disponible.
```

Si ves ese mensaje, el experimento funcionó.

---

# checkpoint

Antes de continuar, verificá que puedas responder estas preguntas:

- [ ] ¿Dónde está guardado nuestro workflow?
- [ ] ¿Qué extensión tiene el archivo?
- [ ] ¿Qué significa `workflow_dispatch`?
- [ ] ¿Qué es un job?
- [ ] ¿Qué es un step?
- [ ] ¿Dónde almacenamos el token?
- [ ] ¿Qué nombre tiene nuestro Secret?
- [ ] ¿Dónde observamos el resultado de la ejecución?

Si podés responderlas, ya entendés las piezas fundamentales del laboratorio.

---

# Parte 11: Vamos a romperlo

Una parte importante del desarrollo de software es aprender a observar qué ocurre cuando algo falla.

Vamos a provocar un error intencionalmente.

## Paso 10. Editar el workflow

Abrí:

```text
.github/workflows/mi-primer-workflow.yml
```

Seleccioná el botón para editar el archivo.

Buscá:

```yaml
MI_TOKEN: ${{ secrets.MI_TOKEN }}
```

Cambialo por:

```yaml
MI_TOKEN: ${{ secrets.MI_TOKEN_QUE_NO_EXISTE }}
```

No vamos a crear ese Secret.

Por lo tanto, el workflow está intentando utilizar un nombre que no existe.

## Paso 11. Guardar el cambio

Hacé un nuevo commit.

Por ejemplo:

```text
Probar workflow con secret inexistente
```

---

# Parte 12: Ejecutarlo nuevamente

Entrá nuevamente en:

```text
Actions
```

Seleccioná:

```text
Mi primer workflow
```

Ejecutalo usando:

```text
Run workflow
```

Esperá que termine.

Esta vez el workflow debería indicar que hubo un error.

Abrí los logs del paso:

```text
Comprobar que existe el secret
```

Deberías encontrar:

```text
El secret NO está disponible.
```

El script ejecuta además:

```bash
exit 1
```

por lo que el step y el job terminan con error.

---

# ¿Qué acabamos de demostrar?

El token original no cambió.

El Secret original tampoco cambió.

Lo que modificamos fue:

```yaml
${{ secrets.MI_TOKEN }}
```

por:

```yaml
${{ secrets.MI_TOKEN_QUE_NO_EXISTE }}
```

El workflow solamente puede obtener el Secret cuyo nombre estamos referenciando.

Esta diferencia es fundamental:

```text
Credencial
    ↓
se almacena en
    ↓
Secret
    ↓
es referenciado por
    ↓
Workflow
    ↓
durante la ejecución de
    ↓
GitHub Actions
```

---

# Parte 13: Reparar el workflow

Volvé a editar:

```text
.github/workflows/mi-primer-workflow.yml
```

Restaurá:

```yaml
MI_TOKEN: ${{ secrets.MI_TOKEN }}
```

Hacé un nuevo commit:

```text
Corregir referencia al secret
```

Ejecutá nuevamente el workflow.

Ahora debería volver a finalizar correctamente.

---

# Resultado final

Tu repositorio contiene ahora:

```text
mi-primer-workflow
│
├── README.md
│
└── .github
    └── workflows
        └── mi-primer-workflow.yml
```

También tenés un Secret:

```text
MI_TOKEN
```

Y ejecutaste un workflow de GitHub Actions manualmente.

El recorrido completo fue:

```text
Repositorio
    ↓
.github/workflows
    ↓
Workflow YAML
    ↓
workflow_dispatch
    ↓
GitHub Actions
    ↓
Job
    ↓
Steps
    ↓
Secret
    ↓
Logs
```

---

# Lo más importante del laboratorio

No necesitás memorizar el YAML.

Necesitás entender las piezas.

## Repositorio

Contiene nuestro proyecto y sus archivos.

## Workflow

Describe una automatización.

## YAML

Es el formato que utilizamos para definir este workflow.

## Trigger

Define cuándo comienza la ejecución.

En nuestro laboratorio:

```yaml
workflow_dispatch:
```

permite iniciarla manualmente.

## Job

Es un trabajo que GitHub Actions debe ejecutar.

## Step

Es uno de los pasos dentro de un job.

## Token

Es una credencial.

## Secret

Permite almacenar de forma segura un valor sensible que el workflow necesita durante la ejecución.

## GitHub Actions

Es la infraestructura que ejecutó nuestro workflow.

## Logs

Son la evidencia que nos permite observar qué ocurrió durante la ejecución.

---

# Desafío opcional

Agregá un nuevo step al workflow:

```yaml
- name: Mostrar fecha
  run: date
```

Guardá el cambio y ejecutá nuevamente el workflow.

Buscá el resultado en los logs.

Después probá agregar:

```yaml
- name: Mostrar sistema operativo
  run: uname -a
```

Volvé a ejecutar el workflow.

El objetivo no es memorizar comandos.

El objetivo es comprobar que cada `step` representa una acción que GitHub Actions ejecuta.

---

# Preguntas de cierre

Antes de terminar, intentá responder sin mirar las respuestas anteriores.

### 1. ¿Qué inicia nuestro workflow?

```text
workflow_dispatch
```

### 2. ¿Dónde está definido?

```text
.github/workflows/mi-primer-workflow.yml
```

### 3. ¿Dónde se ejecutan los comandos?

Dentro de un job de GitHub Actions.

### 4. ¿Escribimos el token directamente en el YAML?

No.

### 5. ¿Dónde está almacenado?

En un GitHub Actions Secret.

### 6. ¿Cómo se llama nuestro Secret?

```text
MI_TOKEN
```

### 7. ¿Cómo comprobamos qué ocurrió?

Revisando la ejecución y sus logs en GitHub Actions.

---

# Siguiente laboratorio: del workflow al agente

Lo que construiste en este laboratorio es un **workflow tradicional de GitHub Actions**.

Todavía no construimos un agente.

Nuestro flujo actual es:

```text
Persona
    ↓
Run workflow
    ↓
GitHub Actions
    ↓
Comandos definidos previamente
```

El siguiente paso será transformar esta idea en un **Agentic Workflow**:

```text
Persona
    ↓
Objetivo
    ↓
Agente
    ↓
GitHub Actions
    ↓
Acción controlada
```

En lugar de escribir solamente acciones predeterminadas, vamos a darle al agente instrucciones expresadas en lenguaje natural.

Para hacerlo vamos a aprender:

- GitHub CLI (`gh`)
- GitHub Agentic Workflows (`gh-aw`)
- Agentic Workflow Markdown
- Frontmatter
- Compilación de `.md` a `.lock.yml`
- Safe Outputs
- Creación controlada de un GitHub Issue

Pero las piezas que aprendiste hoy siguen estando debajo:

```text
Repositorio
+
Credenciales
+
Secrets
+
Triggers
+
GitHub Actions
+
Logs
```

Por eso este laboratorio es el punto de partida.
