# Laboratorio: Inroducción a CODEOWNERS

## Objetivo

En este laboratorio vas a crearcion archivo `CODEOWNERS` y comprobar su funcionamiento mediante un Pull Request.

`CODEOWNERS` permite de**nir qué usuarios o equipos son re**onsables de determinadas partes d**un repositorio. GitHub utiliza es**s reglas para solicitar revisione**según los archivos modificados. 

## 1. Crear una rama

Desde la terminal del repositorio ejecuta:

```bash
git checkout -b **d-codeowners
```

## 2. Crear el **chivo CODEOWNERS

Crea la carpeta**.github` si todavía no existe:

`**bash
mkdir -p .github
```

Crea e**archivo:

```text
.github/CODEOWN**S
```

Agrega la**iguiente regla:

```**xt
* @TU-US**RIO
```

Reemplaza `TU-USUARIO` p** tu nombre de usuario**e**itHub.

Por ejemplo:

```text
* @**tocat
```

El carácter `** indica que la regla aplica a tod** los archivos del**epositorio.

Por lo tanto:

```te**
* @octocat
```

indica que `@oct**at` es Code Owner de**os archivos del repositorio.

** 3. Guardar los cambios

Ejecuta:**```bash
git add .github/CODEOWNER**git commit -m "Add CODEOWNERS****
