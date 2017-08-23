## Tecnologías

En Geoladris utilizamos [Maven](https://maven.apache.org/) para construir nuestros proyectos, tanto el *core* como los plugins. Incluso los plugins que únicamente tienen parte cliente (plugins [cliente](plugins.md#cliente) con CSS y JavaScript) se gestionan a través de Maven para poder tratarlos conjuntamente; para ello utilizamos el plugin [frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin).

Tanto el core como los plugins con parte servidora utilizan la [API Servlet 3.1](https://javaee.github.io/servlet-spec/downloads/servlet-3.1/Final/servlet-3_1-final.pdf). Utilizan ficheros `web-fragment.xml` para definir sus servlets, filters y application listeners.

En cuanto a los plugins cliente, todos tienen un fichero `package.json` con sus dependencias. Las dependencias JavaScript las manejamos con `yarn`. Como se explica arriba, `Maven` se encarga de hacer las llamadas correspondientes a `yarn` mediante el `frontend-maven-plugin`.

Por último, el código JavaScript se organiza en módulos de [RequireJS](http://requirejs.org/).

### Patrón de diseño `message-bus`

En Geoladris hacemos un uso extensivo del patrón de diseño `message-bus`.

El patrón de diseño `message-bus` permite desacoplar los componentes que forman una aplicación. En una aplicación modular, los distintos componentes necesitan interactuar entre sí. Si el acoplamiento es directo, la aplicación deja de ser modular ya que aparecen dependencias, con frecuencia recíprocas, entre los distintos módulos y no es posible realizar cambios a un módulo sin que otros se vean afectados.

En cambio, si los objetos se acoplan a través de un objeto intermediario (`message-bus`), casi todas las dependencias desaparecen, dejando sólo aquellas que hay entre el `message-bus` y los distintos módulos.

Para visualizar el patrón, propondremos un ejemplo, una aplicación modular que consta de tres componentes con representación gráfica y que están dispuestos de la siguiente manera:

* Map: En la parte central habrá un mapa.
* LayerList: En la parte izquierda habrá una lista de capas.
* NewLayer: En la parte superior existirá un control que permite añadir capas a los otros dos componentes.

Un posible diseño de dicha página consistiría en un módulo `layout` que maqueta la página HTML y que inicializa los otros tres objetos. En respuesta a la acción del usuario, el objeto `NewLayer` mandaría un mensaje a `LayerList` y `Map` para añadir el tema en ambos componentes. De la misma manera, `LayerList` podría mandar un mensaje a `Map` en caso de que se permitiera la eliminación de capas desde aquél. El siguiente grafo muestra los mensajes que se pasarían los distintos objetos:

![](../_images/eventbus/eventbus.png)

Es posible observar como en el caso de que se quisiera quitar el módulo `LayerList`, sería necesario modificar el objeto Layout así como el objeto `NewLayer`, ya que están directamente acoplados. Sin embargo, con el uso del `message-bus`, sería posible hacer que los distintos objetos no se referenciaran entre sí directamente sino a través del `message-bus`:

![](../_images/eventbus/eventbus2.png)

Así, el módulo `NewLayer` mandaría un mensaje al `message-bus` con los datos de la nueva capa y `Map` y `LayerList` símplemente escucharían el mensaje y reaccionarían convenientemente. Sería trivial quitar de la página `LayerList` ya que no hay ninguna referencia directa al mismo (salvo tal vez en `Layout`).

Y al contrario: sería posible incluir un nuevo módulo, por ejemplo un mapa adicional, y que ambos escuchasen el evento `add-layer` de forma que se añadirían los temas a ambos mapas.

De esta manera la aplicación es totalmente modular: es posible reemplazar módulos sin que los otros módulos se vean afectados, se pueden realizar contribuciones bien definidas que sólo deben entender los mensajes existentes para poder integrarse en la aplicación, etc.

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

También es posible utilizar recursos *no* minificados añadiendo el parámetro `debug=true` a la petición HTML. Por ejemplo: [http://localhost:8080/demo/?debug=true](http://localhost:8080/demo/?debug=true).

Cuando se depura JavaScript es importante conocer la secuencia de inicio de la aplicación:

1. Se carga `require.js`.
2. `require.js` carga el módulo `main`.
3. `main` carga el módulo `customization`.
4. `customization` carga dinámicamente los módulos especificados en su configuración (`config.js`).
5. Cuando todos los módulos se han cargado se lanza el evento `modules-loaded`.
6. Cuando todas las capas se han añadido al mapa se lanza el evento `layers-loaded`.

## Git

### Repositorios

A continuación se detalla la estructura del proyecto.

Si uno revisa el repositorio de código de [Geoladris](https://github.com/geoladris) encontrará 4 repositorios:

* [doc](https://github.com/geoladris/doc), el repositorio donde se encuentra la documentación oficial del proyecto
* [core](https://github.com/geoladris/core). Proyecto Maven. Se encarga de empaquetar todos los plugins (en tiempo de compilación) y luego servir sus recursos (en tiempo de ejecución).
* [plugins](https://github.com/geoladris/plugins): Conjunto de plugins con funcionalidad aislada que pueden ser o no incluidos en las aplicaciones de manera independiente (ver [plugins](plugins.md)).
* [apps](https://github.com/geoladris/apps): Aplicaciones que definen qué plugins utilizan y cómo se configuran.

### Modelo de ramas

En Geoladris gastamos un modelo de ramas muy parecido al de [GeoServer](http://docs.geoserver.org/stable/en/developer/source.html#repository-structure) y al modelo [cactus](https://barro.github.io/2016/02/a-succesful-git-branching-model-considered-harmful/).

Existen las siguientes ramas:

* **`master`**: Rama principal sobre la que se realizan los últimos desarrollos.
* **Ramas de _release_**: Ramas de desarrollo de una versión determinada. Tienen como nombre `<major>.<minor>.x` ([versionado semántico](http://semver.org)). Por ejemplo: `6.0.x`.

    Estas ramas nacen de `master` o de otra rama de release; por ejemplo, si queremos sacar la `6.1.x` y en `master` existen cambios que nos llevan a la `7.0.x`, sacaremos la `6.1.x` a partir de la `6.0.x`.

    Nunca se vuelven a mezclar en `master`. Si se quieren incluir los cambios en diferentes ramas, se utiliza [git cherry-pick](https://git-scm.com/docs/git-cherry-pick) (igual que [GeoServer](http://docs.geoserver.org/stable/en/developer/source.html#porting-changes-between-primary-branches)).

* **Ramas de _feature_**: Ramas para desarrollos específicos. Pueden salir tanto de `master` como de una rama de *release* y siempre se mezclan sobre la rama de la que salieron.

    Se recomienda no mantener estas ramas durante mucho tiempo para evitar dificultades al mezclar.


**IMPORTANTE**: Cada vez que introducimos un cambio (en cualquiera de las ramas), actualizamos tanto el changelog como las versiones (en los ficheros `pom.xml` y `package.json`).

Por ejemplo, si tenemos `6.1.0-SNAPSHOT` en `master` e introducimos un cambio no compatible hacia atrás, lo documentaremos en el changelog y cambiaremos la versión a `7.0.0-SNAPSHOT`, de forma que sepamos en todo momento qué version se corresponde con cada rama.
