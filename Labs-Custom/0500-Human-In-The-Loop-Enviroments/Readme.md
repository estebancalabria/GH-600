# Human in the Loop con GitHub Actions y Environments

# Objetivos

En este laboratorio aprenderás a implementar un punto de aprobación humana dentro de un workflow de GitHub Actions utilizando GitHub Environments.

* Crear un repositorio en GitHub.
* Abrir el repositorio utilizando GitHub Codespaces.
* Crear una aplicación mínima.
* Crear un workflow de GitHub Actions.
* Ejecutar un deployment sin aprobación humana.
* Crear un Environment denominado `production`.
* Configurar una aprobación humana para el Environment.
* Ejecutar nuevamente el workflow.
* Verificar que el deployment se detenga antes de ejecutarse.
* Rechazar un deployment.
* Comprobar que el deployment no se ejecute.
* Ejecutar nuevamente el workflow.
* Aprobar manualmente el deployment.
* Comprobar que el workflow continúe únicamente después de la aprobación.

# Requisitos

* Cuenta en GitHub.
* Acceso a GitHub Actions.
* Acceso a GitHub Codespaces.
* Permisos para administrar la configuración del repositorio.
* Para realizar la aprobación con una persona diferente, trabajar con un segundo usuario o en parejas.
* Tiempo estimado: 30 a 40 minutos.

# Pasos

## Crear un repositorio

* [BROWSER] GitHub
  * Iniciar sesión con una cuenta de GitHub.
  * [BUTTON] New
  * En `Repository name`, ingresar:

```text
human-in-the-loop-lab
```

  * Seleccionar `Public`.
  * Seleccionar `Add a README file`.
  * [BUTTON] Create repository

* Esperar hasta que se muestre el nuevo repositorio.

## Abrir el repositorio en GitHub Codespaces

* [BROWSER] Repositorio `human-in-the-loop-lab`
  * [BUTTON] Code
  * Seleccionar la pestaña `Codespaces`.
  * [BUTTON] Create codespace on main

* Esperar hasta que GitHub Codespaces termine de iniciar.
* Si aparece una solicitud de confianza:
  * [BUTTON] I Trust
* Verificar que la terminal esté disponible.

## Crear la aplicación

Crear el archivo:

```text
index.html
```

* [FILE] `index.html`

Agregar:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Human in the Loop</title>
</head>
<body>
  <h1>Human in the Loop Demo</h1>
  <p>Application ready for deployment.</p>
</body>
</html>
```

* Guardar el archivo.

## Crear el workflow de deployment

Crear las carpetas:

```text
.github
.github/workflows
```

Dentro de `.github/workflows`, crear:

```text
deploy.yml
```

La estructura debería quedar:

```text
human-in-the-loop-lab
│
├── .github
│   └── workflows
│       └── deploy.yml
│
├── index.html
│
└── README.md
```

* [FILE] `.github/workflows/deploy.yml`

Agregar:

```yaml
name: Deploy application

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  validate:
    name: Validate application
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Validate application
        run: |
          test -f index.html
          echo "Application validation completed"

  deploy:
    name: Deploy to production
    needs: validate
    runs-on: ubuntu-latest
    environment:
      name: production

    steps:
      - name: Deploy application
        run: |
          echo "Starting production deployment"
          echo "Application deployed to production"
```

* Guardar el archivo.

El workflow contiene dos jobs:

```text
validate
deploy
```

El segundo job utiliza:

```yaml
environment:
  name: production
```

Todavía no configuraremos ninguna protección para `production`.

## Publicar el workflow

* [CONSOLE]

```bash
git add .
```

* [CONSOLE]

```bash
git commit -m "Add deployment workflow"
```

* [CONSOLE]

```bash
git push
```

* Esperar hasta que finalice el push.

## Ejecutar el workflow sin aprobación humana

Primero ejecutaremos el workflow sin configurar ninguna aprobación.

* [BROWSER] Repositorio `human-in-the-loop-lab`
  * Seleccionar `Actions`.
  * Seleccionar:

```text
Deploy application
```

  * [BUTTON] Run workflow
  * [BUTTON] Run workflow

* Esperar hasta que aparezca la nueva ejecución.
* Abrir la ejecución.

Verificar que se ejecuten:

```text
Validate application
Deploy to production
```

El resultado esperado es:

```text
Validate application     Success
Deploy to production     Success
```

Abrir:

```text
Deploy to production
```

Abrir el paso:

```text
Deploy application
```

Verificar que aparezca:

```text
Starting production deployment
Application deployed to production
```

En este momento el workflow puede llegar hasta el deployment sin solicitar una decisión humana.

## Crear el Environment de producción

Ahora configuraremos un control sobre el mismo Environment utilizado por el workflow.

* [BROWSER] Repositorio `human-in-the-loop-lab`
  * [BUTTON] Settings
  * Seleccionar `Environments`.

Debería aparecer:

```text
production
```

Si no aparece:

* [BUTTON] New environment
* En `Environment name`, ingresar:

```text
production
```

* [BUTTON] Configure environment

## Configurar la aprobación humana

Dentro del Environment:

```text
production
```

Buscar la sección:

```text
Deployment protection rules
```

Habilitar:

```text
Required reviewers
```

Agregar el usuario que deberá autorizar el deployment.

Guardar la configuración.

> [!IMPORTANT]
> El workflow no fue modificado.
>
> El archivo `.github/workflows/deploy.yml` continúa siendo exactamente el mismo.
>
> El cambio realizado se encuentra en la configuración del Environment `production`.

## Ejecutar nuevamente el mismo workflow

Volver a:

* [BROWSER] Repositorio
  * Seleccionar `Actions`.
  * Seleccionar:

```text
Deploy application
```

  * [BUTTON] Run workflow
  * [BUTTON] Run workflow

Abrir la nueva ejecución.

Esperar hasta que termine:

```text
Validate application
```

Verificar que el deployment no continúe inmediatamente.

La ejecución debería quedar esperando autorización antes de ejecutar:

```text
Deploy to production
```

> [!IMPORTANT]
> No aprobar todavía el deployment.
>
> Observar que el workflow llegó hasta el punto protegido y no puede continuar automáticamente.

## Verificar que el deployment todavía no ocurrió

Mientras el workflow espera aprobación, comprobar que el paso:

```text
Deploy application
```

todavía no se ejecutó.

Por lo tanto, todavía no debería existir una ejecución que muestre:

```text
Application deployed to production
```

El workflow permanece detenido en el punto de aprobación.

## Rechazar el deployment

Ahora utilizaremos la decisión humana para impedir que continúe.

* [BROWSER] Workflow run
  * Revisar el deployment pendiente.
  * [BUTTON] Review deployments
  * Seleccionar:

```text
production
```

  * Seleccionar la opción para rechazar el deployment.
  * Ingresar como comentario:

```text
Deployment rejected for demonstration.
```

  * Confirmar el rechazo.

Esperar hasta que GitHub actualice el workflow.

## Verificar el resultado del rechazo

Comprobar que:

```text
Validate application
```

se completó correctamente.

Comprobar que:

```text
Deploy to production
```

no llegó a ejecutar el deployment.

El paso:

```text
Deploy application
```

no debería haber producido:

```text
Application deployed to production
```

La decisión humana impidió que la automatización continuara hasta producción.

## Ejecutar nuevamente el workflow

Ahora repetiremos exactamente la misma ejecución.

* [BROWSER] Actions
  * Seleccionar:

```text
Deploy application
```

  * [BUTTON] Run workflow
  * [BUTTON] Run workflow

Abrir la ejecución.

Esperar hasta que:

```text
Validate application
```

termine correctamente.

Volver a observar que:

```text
Deploy to production
```

queda esperando autorización.

## Aprobar el deployment

Esta vez autorizaremos la continuación.

* [BROWSER] Workflow run
  * [BUTTON] Review deployments
  * Seleccionar:

```text
production
```

  * Seleccionar la opción para aprobar el deployment.
  * Ingresar como comentario:

```text
Deployment approved.
```

  * Confirmar la aprobación.

## Observar la continuación del workflow

Después de aprobar, esperar unos segundos.

Verificar que el workflow continúe.

Abrir:

```text
Deploy to production
```

Abrir:

```text
Deploy application
```

Verificar que ahora sí aparezca:

```text
Starting production deployment
Application deployed to production
```

La ejecución debería finalizar correctamente.

## Comparar las ejecuciones

Ahora deberían existir tres ejecuciones del mismo workflow.

### Ejecución 1

Sin protección del Environment:

```text
Validate
   ↓
Deploy
```

El deployment continuó automáticamente.

### Ejecución 2

Con el Environment protegido:

```text
Validate
   ↓
Waiting for approval
   ↓
Reject
   ↓
Deploy NO se ejecuta
```

La decisión humana impidió continuar.

### Ejecución 3

Con el mismo Environment protegido:

```text
Validate
   ↓
Waiting for approval
   ↓
Approve
   ↓
Deploy
```

La decisión humana permitió continuar.

## Verificar la diferencia entre una instrucción y un control

Observar nuevamente el workflow:

```yaml
deploy:
  needs: validate
  runs-on: ubuntu-latest
  environment:
    name: production
```

El workflow no contiene una instrucción como:

```text
Ask a human before deploying.
```

El control está configurado en GitHub mediante:

```text
Environment
    ↓
production
    ↓
Required reviewers
```

La ejecución llega al Environment protegido y GitHub espera la decisión requerida antes de continuar con el job.

## ¿Qué ocurrió?

Durante el laboratorio:

1. Creamos un workflow con validación y deployment.
2. Ejecutamos el workflow sin protección.
3. El deployment continuó automáticamente.
4. Configuramos el Environment `production`.
5. Agregamos un reviewer requerido.
6. Ejecutamos exactamente el mismo workflow.
7. La validación terminó correctamente.
8. El deployment quedó esperando una decisión humana.
9. Rechazamos el deployment.
10. El paso de deployment no se ejecutó.
11. Ejecutamos nuevamente el workflow.
12. El deployment volvió a quedar esperando.
13. Una persona aprobó la continuación.
14. Solamente después de la aprobación se ejecutó el deployment.

# Conclusiones

En este laboratorio aprendimos a:

* Crear un punto explícito de intervención humana dentro de un workflow.
* Utilizar GitHub Environments para proteger una operación.
* Configurar `production` como Environment protegido.
* Requerir una decisión humana antes de continuar.
* Observar un workflow detenido esperando autorización.
* Rechazar una ejecución y evitar que llegue al deployment.
* Aprobar una ejecución y permitir que continúe.
* Separar automatización de autorización.
* Comprobar que una política configurada en GitHub puede controlar técnicamente la ejecución.

El comportamiento final es:

```text
GitHub Actions
      ↓
   Validate
      ↓
Environment: production
      ↓
Waiting for approval
      ↓
      Human
     ↙     ↘
 Reject   Approve
   ↓        ↓
 Stop     Deploy
```

La automatización puede preparar la operación y llegar hasta el punto protegido.

La decisión de continuar permanece en manos de una persona.
