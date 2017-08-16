## Tecnologías

En Geoladris utilizamos [Maven](https://maven.apache.org/) para construir nuestros proyectos, tanto el *core* como los plugins. Incluso los plugins que únicamente tienen parte cliente (CSS y JavaScript) se gestionan a través de Maven para poder tratarlos conjuntamente; para ello utilizamos el plugin [frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin).

Tanto el core como los plugins con parte servidora utilizan la [API Servlet 3.1](https://javaee.github.io/servlet-spec/downloads/servlet-3.1/Final/servlet-3_1-final.pdf). Utilizan ficheros `web-fragment.xml` para definir sus servlets, filters y application listeners.

En cuanto a los plugins cliente, todos tienen un fichero `package.json` con sus dependencias. Las dependencias JavaScript las manejamos con `yarn`. Como se explica arriba, `Maven` se encarga de hacer las llamadas correspondientes a `yarn` mediante el `frontend-maven-plugin`.

Por último, el código JavaScript se organiza en módulos de [RequireJS](http://requirejs.org/).

## Integración continua

Geoladris está configurado en [Travis](https://travis-ci.org/geoladris/) para su integración continua.

La configuración debe garantizar que se pasan todos los tests y que se hace un `mvn deploy` de todos los artefactos que componen Geoladris.

El deploy de Maven requiere autenticación. Para proporcionarla hay un fichero `deploy-settings.xml` en la raíz del repositorio que utiliza variables de entorno para las credenciales:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings
        xmlns="http://maven.apache.org/SETTINGS/1.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>geoladris-releases</id>
            <username>${env.GEOLADRIS_DEPLOY_USER}</username>
            <password>${env.GEOLADRIS_DEPLOY_PASS}</password>
        </server>
        <server>
            <id>geoladris-snapshots</id>
            <username>${env.GEOLADRIS_DEPLOY_USER}</username>
            <password>${env.GEOLADRIS_DEPLOY_PASS}</password>
        </server>
    </servers>
</settings>
```

Basta con configurar Maven en el fichero `.travis.yml` para que utilice este fichero:

```
after_success:
  - mvn deploy -s deploy-settings.xml
```

Por último, hay que configurar las variables de entorno en la interfaz web de Travis.

## Depurando JavaScript en el navegador

Por defecto, se utilizan recursos minificados en el cliente. Los recursos minificados forman parte del [empaquetado](apps.md).

También es posible utilizar recursos *no* minificados añadiendo el parámetro `debug=true` a la petición HTML. Por ejemplo: http://localhost:8080/demo/?debug=true.

> TODO Secuencia de inicio de la aplicación (customization, `modules-loaded`, `layers-loaded`, etc.).

## Git

### Repositorios

El código de Geoladris se organiza en tres repositorios:

* [core](https://github.com/geoladris/core). Proyecto Maven. Se encarga de empaquetar todos los plugins (en tiempo de compilación) y luego servir sus recursos (en tiempo de ejecución).
* [plugins](https://github.com/geoladris/plugins): Conjunto de plugins con funcionalidad aislada que pueden ser o no incluidos en las aplicaciones de manera independiente (ver [plugins](plugins.md)).
* [aplicaciones](https://github.com/geoladris/apps): Aplicaciones que definen qué plugins utilizan y cómo se configuran.

### Modelo de ramas

> * Modelo de ramas. Revisar doc:
>     - [GeoServer](http://docs.geoserver.org/stable/en/developer/source.html)
>     - [gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows#gitflow-workflow)
>     - [cactus](https://barro.github.io/2016/02/a-succesful-git-branching-model-considered-harmful/)
> * Actualizar versiones (semántico) cuando se integra una nueva feature. De esta forma al [publicar](releases.md) ya tenemos la versión que toca.
> * Actualizar changelog cuand se hace cualquier cambio. De esta forma al [publicar](releases.md) ya tenemos el changelog listo.
