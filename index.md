title: beerjs #48
author:
  name: Leonardo Gatica
  twitter: lgaticaq
  url: https://github.com/lgaticaq
  email: lgatica@pm.me
output: index.html
theme: juanbrujo/cleaver-beerjs
style: style.css
controls: true

--

# Release automáticos con semantic-release

--

# ¿Que es SemVer?

[**SemVer**](https://semver.org/) o *Semantic Versioning* es la forma en la que comúnmente se versiona el software.

--

![](https://cdn-images-1.medium.com/max/1200/1*7h56wnp4mqlOqRm4aF9cTQ.png)

--

# path

Cuando se corrigen bugs que **no** cambian el comportamiento actual debe incrementarse el **path** de la version. Es decir si la version actual es `1.0.0`, debe quedar en `1.0.1`.

--

# minor

Cuando se agregan nuevas funcionalidades que **no** cambian el comportamiento actual debe incrementarse el **minor** de la version. Es decir si la version actual es `1.0.1`, debe quedar en `1.1.0`.

--

# major

Cuando se agregan nuevas funcionalidades y/o corrigen bugs que **si** cambian el comportamiento actual debe incrementarse el **major** de la version. Es decir si la version actual es `1.1.0`, debe quedar en `2.0.0`.

--

# ¿Por que utilizar esto?

Es realmente util cuando se desarrollan librerías que serán usadas como dependencias. Para los devs que usan dicha lib puede saber cuando es seguro actualizar la dependencia sin miedo a que cambie el funcionamiento de esta.

--

# CHANGELOG / Releases

Una buena practica es mantener un [**CHANGELOG.md**](https://keepachangelog.com/en/1.0.0/) en el cual se registre el historial de cambios de versiones con el detalle de que cosas se corrigieron, agregaron o dejaron de funcionar.
Ayuda a la hora de decidir cambiar de una version a otra, sobre todo cuando el incremento es del tipo el *major*.

Otra forma de hacer esto es usar la sección de [**Releases**](https://docs.gitlab.com/ee/workflow/releases.html) disponible en la mayoría de sitios web de control de versiones como Gitlab.

--

# Commit Semánticos

De la mano con lo anterior existen varias convenciones de escribir el mensaje de commits. Semantic-release por defecto, y la mayoría de libs, usa la de [Angular](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines).

--

# Estructura de mensajes

```
<type>(<scope>): <subject>

<body>

<footer>
```

--

# Ejemplo

```
refactor(users): se agrega método para autentificar usuarios

Debido a que se estaba usando la misma función en varios
controladores se creo un static method en el modelo User

close #69
```

--

# ¿Que es semantic-release?

Es una lib que simplifica el proceso de release de un proyecto. Para esto depende de que los commits a su vez sean semánticos. La premisa es usar esta lib en el CI que se este usando.

--

# Pasos del release

| Paso | Paso Hook | Descripción |
| ---- | --------- | ----------- |
| Verificar condiciones | verifyConditions | Verifica todas las condiciones para proceder con el lanzamiento. |
| Conseguir el ultimo release | N/A | Obtiene el commit correspondiente al ultimo release, analizando los tags de Git. |
| Analizar commits | N/A | Determina el tipo de versión en función de los commits agregados desde el ultimo release. |
| Verificar release | verifyRelease | Verifica la conformidad del release. |
| Generar notas | generateNotes | Genera notas de la versión para los commits agregados desde el ultimo release. |
| Crear tag de Git | N/A | Crea un tag Git correspondiente al nuevo release. |
| Preparar | prepare | Prepara el release. |
| Publicar | publish | Publica el release. |
| Notificar | success, failure | Notifica el nuevo release o posibles errores. |

--

# Instalación en el proyecto

```bash
npm i -D -E semantic-release
```

--

# Plugins por defecto

Semantic-release viene con los siguientes plugins por defecto:

```json
"@semantic-release/commit-analyzer"
"@semantic-release/error"
"@semantic-release/github"
"@semantic-release/npm"
"@semantic-release/release-notes-generator"
```

--

# Plugins adicionales

```json
"@semantic-release/gitlab"
"@semantic-release/git"
"@semantic-release/changelog"
"@semantic-release/exec"
"semantic-release-slack-bot"
"semantic-release-docker"
"semantic-release-gcr"
"semantic-release-chrome"
"semantic-release-firefox"
```

--

# Configuración

Para establecer la configuración es posible usar alguna de las siguientes opciones:

- .releaserc (.yaml, .yml, .json o .js).
- release.config.js.
- La key "release" en el package.json del proyecto.

```js
{
  "branch": "next"
}
```

```json
{
  "release": {
    "branch": "next"
  }
}
```

--

# Configuración de plugins

```js
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/npm",
    "@semantic-release/git"
  ]
}
```

--

# Ejemplo de librería npm

```js
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/git",
    "@semantic-release/github"
  ]
}
```

--

# Ejemplo de app con docker y rancher

```js
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/git",
    "@semantic-release/gitlab",
    "@semantic-release-gitlab-registry",
    "@semantic-release-rancher",
  ]
}
```

--

# Ejemplo de .travis.yml

```yml
language: node_js
cache:
  directories:
    - ~/.npm
node_js:
  - "12.7.0"
notifications:
  email: false
stages:
  - lint
  - name: deploy
    if: branch = master
jobs:
  include:
    - stage: lint
      name: eslint
      script: npx eslint .
    - stage: lint
      name: commitlint
      before_script:
        - npm i -g @commitlint/travis-cli
      script: commitlint-travis
    - stage: deploy
      script: npx semantic-release
```

--

# Ejemplo de .gitlab-ci.yml

```yml
stages:
  - dependencies
  - lint
  - test
  - deploy

release:
  image: node:12.7.0-alpine
  stage: deploy
  services:
    - docker:dind
  variables:
    DOCKER_HOST: tcp://docker:2375
  before_script:
    - apk add --no-cache docker git
    - docker build
      -t $CI_REGISTRY_IMAGE
      --build-arg NODE_ENV=production
      --build-arg KNOWN_HOST=gitlab.com
      --build-arg SSH_PRIVATE_KEY="$SSH_PRIVATE_KEY" .
  script:
    - npx semantic-release
  only:
    - master
  environment:
    name: production
    url: https://api.eclass.com/
```
