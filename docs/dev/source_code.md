## Tecnologías

> TODO
>
> * yarn
> * Maven (incluso para plugins cliente). mvn verify para probar todo.
> * RequireJS
> * ...

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

# Depurando JavaScript en el navegador

Por defecto, se utilizan recursos minificados en el cliente. Los recursos minificados forman parte del [empaquetado](apps.md).

También es posible utilizar recursos *no* minificados añadiendo el parámetro `debug=true` a la petición HTML. Por ejemplo: http://localhost:8080/demo/?debug=true.

## Git

> TODO
>
> * Repositorios.
> * Modelo de ramas. Revisar doc:
>     - [GeoServer](http://docs.geoserver.org/stable/en/developer/source.html)
>     - [gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows#gitflow-workflow)
>     - [cactus](https://barro.github.io/2016/02/a-succesful-git-branching-model-considered-harmful/)

