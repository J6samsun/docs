---
title: Crear una acción de JavaScript
intro: 'En esta guía, aprenderás como desarrollar una acción de JavaScript usando el kit de herramientas de acciones.'
redirect_from:
  - /articles/creating-a-javascript-action
  - /github/automating-your-workflow-with-github-actions/creating-a-javascript-action
  - /actions/automating-your-workflow-with-github-actions/creating-a-javascript-action
  - /actions/building-actions/creating-a-javascript-action
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
topics:
  - Action development
  - JavaScript
shortTitle: Acción de JavaScript
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}

## Introducción

En esta guía, aprenderás acerca de los componentes básicos necesarios para crear y usar una acción de JavaScript empaquetada. Para centrar esta guía en los componentes necesarios para empaquetar la acción, la funcionalidad del código de la acción es mínima. La acción imprime "Hello World" en los registros o "Hello [who-to-greet]"si proporcionas un nombre personalizado.

Esta guía usa el módulo Node.js del kit de herramientas {% data variables.product.prodname_actions %} para acelerar el desarrollo. Para obtener más información, consulta el repositorio [actions/toolkit](https://github.com/actions/toolkit).

Una vez que completes este proyecto, deberías comprender cómo crear tu propia acción de JavaScript y probarla en un flujo de trabajo.

{% data reusables.actions.pure-javascript %}

{% data reusables.actions.context-injection-warning %}

## Prerrequisitos

Antes de que comiences, necesitarás descargar Node.js y crear un repositorio público de {% data variables.product.prodname_dotcom %}.

1. Descarga e instala Node.js {% ifversion fpt or ghes > 3.3 or ghae-issue-5504 or ghec %}16.x{% else %}12.x{% endif %}, el cual incluye a npm.

  {% ifversion fpt or ghes > 3.3 or ghae-issue-5504 or ghec %}https://nodejs.org/en/download/{% else %}https://nodejs.org/en/download/releases/{% endif %}

1. Crea un repositorio público nuevo en {% data variables.product.product_location %} y llámalo "hello-world-javascript-action". Para obtener más información, consulta "[Crear un repositorio nuevo](/articles/creating-a-new-repository)".

1. Clona el repositorio en tu computadora. Para obtener más información, consulta "[Clonar un repositorio](/articles/cloning-a-repository)".

1. Desde tu terminal, cambia los directorios en el repositorio nuevo.

  ```shell{:copy}
  cd hello-world-javascript-action
  ```

1. Desde tu terminal, inicializa el directorio con npm para generar un archivo `package.json`.

  ```shell{:copy}
  npm init -y
  ```

## Crear un archivo de metadatos de una acción

Crea un archivo nuevo que se llame `action.yml` en el directorio `hello-world-javascript-action` con el siguiente código de ejemplo. Para obtener más información, consulta la sección "[Sintaxis de metadatos para {% data variables.product.prodname_actions %}](/actions/creating-actions/metadata-syntax-for-github-actions)".

```yaml{:copy}
name: 'Hello World'
description: 'Greet someone and record the time'
inputs:
  who-to-greet:  # id of input
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  time: # id of output
    description: 'The time we greeted you'
runs:
  using: {% ifversion fpt or ghes > 3.3 or ghae-issue-5504 or ghec %}'node16'{% else %}'node12'{% endif %}
  main: 'index.js'
```

Este archivo define la entrada `who-to-greet` y la salida `time`. También informa al ejecutador de la acción cómo empezar a ejecutar esta acción de JavaScript.

## Agregar paquetes de kit de herramientas de las acciones

El kit de herramientas de acciones es una recopilación de los paquetes Node.js que te permiten desarrollar rápidamente acciones de JavaScript con más consistencia.

El paquete del kit de herramientas [`@actions/Core`](https://github.com/actions/toolkit/tree/main/packages/core) proporciona una interfaz a los comandos del flujo de trabajo, a las variables de entrada y de salida, a los estados de salida y a los mensajes de depuración.

El kit de herramientas también ofrece un paquete [`@actions/github`](https://github.com/actions/toolkit/tree/main/packages/github) que devuelve un cliente Octokit REST autenticado así como el acceso a los contextos de las GitHub Actions.

El kit de herramientas ofrece más de un paquete `core` y `github`. Para obtener más información, consulta el repositorio [actions/toolkit](https://github.com/actions/toolkit).

En tu terminal, instala los paquetes `core` y `github` del kit de herramientas de acciones.

```shell{:copy}
npm install @actions/core
npm install @actions/github
```

Ahora deberías ver un directorio `node_modules` con los módulos que acabas de instalar y un archivo `package-lock.json` con las dependencias del módulo instalado y las versiones de cada módulo instalado.

## Escribir el código de la acción

Esta acción usa el kit de herramientas para obtener la variable de entrada `who-to-greet` requerida en el archivo de metadatos de la acción e imprime "Hello [who-to-greet]" en un mensaje de depuración del registro. A continuación, el script obtiene la hora actual y la establece como una variable de salida que pueden usar las acciones que se ejecutan posteriormente en unt rabajo.

Las Acciones de GitHub proporcionan información de contexto sobre el evento de webhooks, las referencias de Git, el flujo de trabajo, la acción y la persona que activó el flujo de trabajo. Para acceder a la información de contexto, puedes usar el paquete `github`. La acción que escribirás imprimirá el evento de webhook que carga el registro.

Agrega un archivo nuevo denominado `index.js`, con el siguiente código.

{% raw %}
```javascript{:copy}
const core = require('@actions/core');
const github = require('@actions/github');

try {
  // `who-to-greet` input defined in action metadata file
  const nameToGreet = core.getInput('who-to-greet');
  console.log(`Hello ${nameToGreet}!`);
  const time = (new Date()).toTimeString();
  core.setOutput("time", time);
  // Get the JSON webhook payload for the event that triggered the workflow
  const payload = JSON.stringify(github.context.payload, undefined, 2)
  console.log(`The event payload: ${payload}`);
} catch (error) {
  core.setFailed(error.message);
}
```
{% endraw %}

Si en el ejemplo anterior de `index.js` ocurre un error, entonces `core.setFailed(error.message);` utilizará el paquete [`@actions/core`](https://github.com/actions/toolkit/tree/main/packages/core) del kit de herramientas de las acciones para registrar un mensaje y configurar un código de salida defectuoso. Para obtener más información, consulta la sección "[Configurar los códigos de salida para las acciones](/actions/creating-actions/setting-exit-codes-for-actions)".

## Crear un README

Puedes crear un archivo README para que las personas sepan cómo usar tu acción. Un archivo README resulta más útil cuando planificas el intercambio de tu acción públicamente, pero también es una buena manera de recordarle a tu equipo cómo usar la acción.

En tu directorio `hello-world-javascript-action`, crea un archivo `README.md` que especifique la siguiente información:

- Una descripción detallada de lo que hace la acción.
- Argumentos necesarios de entrada y salida.
- Argumentos opcionales de entrada y salida.
- Secretos que utiliza la acción.
- Variables de entorno que utiliza la acción.
- Un ejemplo de cómo usar tu acción en un flujo de trabajo.

```markdown
# Hello world docker action

Esta acción imprime "Hello World" o "Hello" + el nombre de una persona a quien saludar en el registro.

## Inputs

## `who-to-greet`

**Required** The name of the person to greet. Default `"World"`.

## Outputs

## `time`

The time we greeted you.

## Example usage

uses: actions/hello-world-javascript-action@v1.1
with:
  who-to-greet: 'Mona the Octocat'
```

## Confirma, etiqueta y sube tu acción a GitHub

{% data variables.product.product_name %} descarga cada acción ejecutada en un flujo de trabajo durante el tiempo de ejecución y la ejecuta como un paquete completo de código antes de que puedas usar comandos de flujo de trabajo como `run` para interactuar con la máquina del ejecutor. Eso significa que debes incluir cualquier dependencia del paquete requerida para ejecutar el código de JavaScript. Necesitarás verificar los paquetes `core` y `github` del kit de herramientas para el repositorio de tu acción.

Desde tu terminal, confirma tus archivos `action.yml`, `index.js`, `node_modules`, `package.json`, `package-lock.json` y `README.md`. Si agregaste un archivo `.gitignore` que enumera `node_modules`, deberás eliminar esa línea para confirmar el directorio `node_modules`.

También se recomienda agregarles una etiqueta de versión a los lanzamientos de tu acción. Para obtener más información sobre el control de versiones de tu acción, consulta la sección "[Acerca de las acciones](/actions/automating-your-workflow-with-github-actions/about-actions#using-release-management-for-actions)".

```shell{:copy}
git add action.yml index.js node_modules/* package.json package-lock.json README.md
git commit -m "My first action is ready"
git tag -a -m "My first action release" v1.1
git push --follow-tags
```

Ingresar tu directorio de `node_modules` puede causar problemas. Como alternativa, puedes utilizar una herramienta que se llama [`@vercel/ncc`](https://github.com/vercel/ncc) para compilar tu código y módulos en un archivo que se utilice para distribución.

1. Instala `vercel/ncc` ejecutando este comando en tu terminal. `npm i -g @vercel/ncc`

1. Compila tu archivo `index.js`. `ncc build index.js --license licenses.txt`

  Verás un nuevo archivo `dist/index.js` con tu código y los módulos compilados. También verás un archivo asociado de `dist/licenses.txt` que contiene todas las licencias de los `node_modules` que estás utilizando.

1. Cambia la palabra clave `main` en tu archivo `action.yml` para usar el nuevo archivo `dist/index.js`. `main: 'dist/index.js'`

1. Si ya has comprobado tu directorio `node_modules`, eliminínalo. `rm -rf node_modules/*`

1. Desde tu terminal, confirma las actualizaciones para tu `action.yml`, `dist/index.js` y `node_modules`.
```shell
git add action.yml dist/index.js node_modules/*
git commit -m "Use vercel/ncc"
git tag -a -m "My first action release" v1.1
git push --follow-tags
```

## Probar tu acción en un flujo de trabajo

Ahora estás listo para probar tu acción en un flujo de trabajo. Cuando una acción esté en un repositorio privado, la acción solo puede usarse en flujos de trabajo en el mismo repositorio. Las acciones públicas pueden ser usadas por flujos de trabajo en cualquier repositorio.

{% data reusables.actions.enterprise-marketplace-actions %}

### Ejemplo usando una acción pública

Este ejemplo demuestra cómo se puede ejecutar tu acción nueva y pública desde dentro de un repositorio externo.

Copia el siguiente YAML en un archivo nuevo en `.github/workflows/main.yml` y actualiza la línea `uses: octocat/hello-world-javascript-action@v1.1` con tu nombre de usuario y con el nombre del repositorio público que creaste anteriormente. También puedes reemplazar la entrada `who-to-greet` con tu nombre.

{% raw %}
```yaml{:copy}
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      - name: Hello world action step
        id: hello
        uses: octocat/hello-world-javascript-action@v1.1
        with:
          who-to-greet: 'Mona the Octocat'
      # Use the output from the `hello` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```
{% endraw %}

Cuando se activa este flujo de trabajo, el ejecutor descargará la acción `hello-world-javascript-action` desde tu repositorio público y luego lo ejecutará.

### Ejemplo usando una acción privada

Copia el siguiente ejemplo de código de flujo de trabajo en un archivo `.github/workflows/main.yml` en tu repositorio de acción. También puedes reemplazar la entrada `who-to-greet` con tu nombre.

{% raw %}
**.github/workflows/main.yml**
```yaml{:copy}
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      # To use this repository's private action,
      # you must check out the repository
      - name: Checkout
        uses: actions/checkout@v2
      - name: Hello world action step
        uses: ./ # Uses an action in the root directory
        id: hello
        with:
          who-to-greet: 'Mona the Octocat'
      # Use the output from the `hello` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```
{% endraw %}

Desde tu repositorio, da clic en la pestaña de **Acciones** y selecciona la última ejecución de flujo de trabajo. Under **Jobs** or in the visualization graph, click **A job to say hello**. Deberías ver "Hello Mona the Octocat" o el nombre que usaste para la entrada `who-to-greet` y la marcación de hora impresa en el registro.

![Captura de pantalla del uso de tu acción en un flujo de trabajo](/assets/images/help/repository/javascript-action-workflow-run-updated-2.png)
