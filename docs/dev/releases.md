## Frecuencia de publicaciones

En Geoladris no tenemos una frecuencia de publicación constante, sino que publicamos en función de las necesidades de los usuarios.

El proceso de publicación se inicia simplemente con una conversación en la [lista de correo](contribute.md#lista-de-correo) de desarrollo en la que se acuerda la versión de la nueva release y la fecha aproximada de la misma.

¿Necesítas una nueva versión? [Escríbenos](contribute.md#lista-de-correo).

## Proceso

El proceso que se describe aquí incluye únicamente un repositorio (`core` **o** `plugins` **o** `apps`). Para publicar todos los repositorioes habrá que llevar este proceso en paralelo para todos ellos.

### Acordar una versión y una fecha

A través de la [lista de correo](contribute.md#lista-de-correo) se acuerda la siguiente versión que se va a publicar y una fecha aproximada de cuándo va a ocurrir.

Para decidir qué versión publicar se puede revisar el [changelog](../ref/changelog.md) donde se incluyen los cambios que se han ido añadiendo a cada una de las versiones sin publicar.

### Preparar milestone en el repositorio

Las versiones se gestionan a traves de milestones en el repositorio de Github. Una vez se ha acordado por la lista de correo, habrá que crear una milestone con la versión y la fecha acordadas.

### Preparar la rama de release

Una vez se ha decidido la versión, hay que preparar la rama en el repositorio. Lo hacemos de una manera similar a [GeoServer](http://docs.geoserver.org/stable/en/developer/release-guide/index.html#if-you-are-cutting-the-first-rc-of-a-series-create-the-stable-branch).

Únicamente debemos buscar la rama que se corresponde a la versión que queremos publicar. Por ejemplo, si queremos publicar la versión `7.1.0` deberemos buscar la rama que tenga la versión `7.1.0-SNAPSHOT` en el fichero `pom.xml`. Esta rama únicamente puede ser `master` o `7.1.x`.

En caso de que la rama con la versión a publicar sea `master` deberemos crear la rama `7.1.x`:

```bash
git checkout master
git checkout -b 7.1.x
git push -u origin 7.1.x
```

y dejar la rama `master` preparada para seguir desarrollando la `7.2.0-SNAPSHOT`:

```bash
git checkout master
mvn versions:set -DnewVersion=7.2.0-SNAPSHOT
yarn version --no-git-tag-version --new-version 7.2.0-SNAPSHOT
rm `git status | grep pom.xml.versionsBackup`
git add .
git commit -m "Bump version to 7.2.0-SNAPSHOT"
git push
```

### Preparar la rama de documentación

Para tener documentación de todas las versiones utilizamos ramas y tags como un repositorio de desarrollo que luego publicaremos en [ReadTheDocs](https://readthedocs.org/projects/geoladris/).

El proceso es idéntico al de la [rama de release](#preparar-la-rama-de-release) pero sin necesidad de actualizar versiones. Si hemos tenido que crear la rama de release a partir de `master` para el desarrollo, lo haremos aquí también:

```bash
git checkout master
git checkout -b 7.1.x
git push -u origin 7.1.x
```

### Correcciones de última hora

Una vez las ramas están listas para el despliegue se probará todo bien y se harán cambios de última hora.

Los únicos cambios que se pueden introducir son correcciones de errores.

Todos los cambios que afecten a la publicación de la versión deberán tener asociada una issue en GitHub. La issue deberá pertenecer siempre a la milestone de la versión.

Todos los cambios hay que documentarlos inmediatamente en el changelog.

### Desplegar

Cuando el software esté listo para el despliegue, actualizaremos la rama de release con la versión definitiva y la publicaremos:

```bash
# Bump
git checkout 7.1.x
mvn versions:set -DnewVersion=7.1.0
yarn version --no-git-tag-version --new-version 7.1.0
rm `git status | grep pom.xml.versionsBackup`

# Commit
git add .
git commit -m "Bump version to 7.1.0"

# Push
git tag 7.1.0
git push
git push --tags
```

El despliegue se debería realizar automáticamente con Travis. Deberemos esperar a que Travis envíe el correo de notificación de éxito y luego comprobar que los artefactos se han publicado correctamente, tanto con [Maven](nullisland.geomati.co:8082/repository/releases/org/geoladris) como con [yarn](https://www.npmjs.com/search?q=geoladris).

Para terminar, dejaremos la rama de release preparada con la siguiente versión (patch version++):

```bash
mvn versions:set -DnewVersion=7.1.1-SNAPSHOT
yarn version --no-git-tag-version --new-version 7.1.1-SNAPSHOT
rm `git status | grep pom.xml.versionsBackup`
git add .
git commit -m "Bump version to 7.1.1-SNAPSHOT"
git push
```

### Actualizar documentación

Una vez la versión está publicada, deberemos actualizar la documentación. Las tareas a realizar son:

* Actualizar el changelog con las nuevas versiones, incluyendo fecha de publicación.

* Crear un tag en el repositorio de documentación:

        :::bash
        git checkout 7.1.x
        git tag 7.1.0
        git push --tags


* Añadir una versión en ReadTheDocs con el nuevo tag.

* Mezclar cambios de la documentación en `master` (al menos el changelog).

### Enviar mensaje a la lista de correo

Por último, es necesario enviar un correo a la lista. Se puede utilizar la siguiente plantilla:

**Asunto**: Nueva versión `<versión>`

**Mensaje**:

Hola a todos,

Acabamos de publicar una nueva versión de Geoladris, la versión `<versión>`.

Podéis encontrar los cambios que trae esta nueva versión en https://geoladris.github.io/doc/ref/changelog/.

La documentación general la encontraréis en https://geoladris.github.io/doc/.

Y la documentación sobre versiones anteriores está disponible en ReadTheDocs: https://readthedocs.org/projects/geoladris/.

Saludos.
