This project follows [semantic versioning](http://semver.org).

## Core - Sin publicar (7.0.0)

### Modificado

* El proceso de empaquetado de aplicaciones ahora se gestiona con Maven y `yarn` conjuntamente (ver [aplicaciones](../dev/apps.md)).
* Las dependencias de los plugins cliente se gestionan ahora con `yarn`.
* Estructura de los plugins cliente (ver [plugins](../dev/plugins.md)).
* Los paquetes Java ahora se despliegan en el repo central de Maven.
* El idioma se cambia con la ruta `/setlang?<idioma>` en lugar de con el parámetro `?lang=<idioma>`.

### Eliminado

* Variable de entorno `GEOLADRIS_MINIFIED`. Los recursos en cliente se sirven minificados por defecto, con la posibilidad de servirlos sin minificar con el parámetro `debug=true`.

### Corregido

* Error al arrancar aplicaciones con Tomcat 8.0.x.

## Plugins - Sin publicar (7.0.0)

### Añadido

* Posibilidad de especificar API para Google Maps ([#38](https://github.com/geoladris/plugins/issues/38)).

### Modificado

* El nombre del recurso de base de datos en el fichero de Tomcat `context.xml` ahora debe ser `geoladris` en lugar de `unredd-portal`.
* El plugin `ol2Controls` ahora se llama `ol2controls` (sin mayúscula).
* Los paquetes Java ahora se despliegan en el repo central de Maven.

### Corregido

* Checkboxes de la lista de capas (`layer-list.js`) no respondían.
* `legend-panel.js` lanzaba excepciones.
* La barra de desplazamiento de la lista de capas no funcionaba.
* Multiples correcciones de la interfaz de usuario (CSS).
* El panel de transparencia mostraba capas que no estaban visibles.
* Corrección de varios *bugs* en el plugin `layers-editor`.
* El plugin `language-buttons` no cambiaba el idioma correctamente.
* El plugin `tour` no incluye ninguna configuración por defecto (ya que es específica de las aplicaciones).
* Los servicios de estadísticas no funcionaban en ciertos entornos con Tomcat (no tenía permisos para escribir el fichero de log).

## Apps - Sin publicar (7.0.0)

### Añadido

* Aplicación `develop`, para facilitar el desarrollo.
* Aplicación `essential`, que incluye solo el núcleo sin ningún plugin.
* Despliegue con Docker de la aplicación `demo`.
* Posibilidad de especificar la conexión a base de datos de las aplicaciones `demo` y `develop` mediante variables de entorno (`GEOLADRIS_DB_URL`, `GEOLADRIS_DB_USER`, `GEOLADRIS_DB_PASS`) en lugar de editando el fichero `context.xml`.

### Corregido

* Redirección en caso de no añadir `/` al final de la URL en el navegador.

## Core - 6.0.1 [30-08-2017]

### Corregido

* Error al arrancar aplicaciones con Tomcat 8.0.x.

## Plugins - 6.0.3 [30-08-2017]

### Añadido

* Posibilidad de especificar API para Google Maps ([#38](https://github.com/geoladris/plugins/issues/38)).

### Corregido

* Checkboxes de la lista de capas (`layer-list.js`) no respondían.
* `legend-panel.js` lanzaba excepciones .

## Core - 6.0.0 [2017-04-21]

### Modificado

* Los plugins empaquetados como `.jar` pasan a estar contenidos en un subdirectorio dentro de `geoladris` (en lugar de directamente en `geoladris`). Así, todos los plugins están en subdirectorios, independientemente de su empaquetado (ver [guía de migración](../dev/migrate.md)).
* Todos los plugins toman su nombre del subdirectorio en el que están contenidos.
* `installInRoot` por defecto a `false` para todos los plugins, independientemente del empaquetado; `core` se mantiene con `installInRoot : true` para que sus módulos se puedan referenciar de manera sencilla desde otros plugins.
* `ModuleConfigurationProvider` añadidos a una lista en `ServletContext` en lugar de a un objeto `Config` (ver [guía de migración](../dev/migrate.md#ModuleConfigurationProviders)).

### Corregido

* [Minificación no funciona para plugins con `installInRoot : false`](https://github.com/geoladris/geoladris/issues/24).
* [Peticiones a recursos cualificados devuelven recursos sin cualificar (y viceversa)](https://github.com/geoladris/geoladris/issues/26).
* `NullPointerException` cuando algún `ModuleConfigurationProvider` devuelve configuración para un plugin no existente.
* Bug con ficheros CSS llamados igual que el directorio que los contiene (`styles/styles.css`, `modules/modules.css`, `theme/theme.css`) en plugins con `installInRoot:false`.

### Añadido

* Script (`geoladris_build.sh`) para generar paquetes `.war` a partir de un descriptor `build.json` y un directorio de configuración.
* Posibilidad de obtener la configuración desde una base de datos, tanto configuración de plugins (`public-conf.json`) como propiedades (`portal.properties`) y mensajes (`messages/messages*.properties`).
* [Posibilidad de añadir módulos en subdirectorios](https://github.com/geoladris/core/issues/10).
* [Detectar cambios en los directorios de plugins](https://github.com/geoladris/core/issues/33).
* [Fichero descriptor de plugin (`<plugin>-conf.json`) opcional](https://github.com/geoladris/core/issues/36).
* Variable `GEOLADRIS_CACHE_TIMEOUT` (en segundos) para limpiar la caché de configuración. Únicamente se tiene en cuenta si la variable `GEOLADRIS_CONFIG_CACHE` es `true`.
* Método `PortalRequestConfiguration.getCurrentConfiguration()` para poder modificar desde un `ModuleConfigurationProvider` la configuración en construcción, obtenida de los `ModuleConfigurationProvider` previos.

## Core - 5.0.1 [2016-12-06]

### Corregido

* [Devolver `Content-type` correcto para imágenes SVG](https://github.com/geoladris/geoladris/issues/34).

## Core - 5.0.0 [2016-11-25]

### Corregido
* [Especificar _arrays_ JSON como configuración de los módulos](https://github.com/geoladris/geoladris/issues/2).
* [Soporte para Java 8](https://github.com/geoladris/geoladris/issues/20).
* [Enviar al cliente únicamente las propiedades bien conocidas](https://github.com/geoladris/geoladris/issues/8).
* [Posibilidad de tener un directorio de configuración distinto por aplicación](https://github.com/geoladris/core/issues/30).

### Añadido
* Nuevo descriptor de aplicación `public-conf.json`. Permite activar y desactivar plugins. Sustituye a `plugin-conf.json`, que se mantiene temporalmente por compatibilidad hacia atrás.
* Mezclar la configuración por defecto de los plugins y no sólo de sobreescribirla.
* Especificar [configuración específica de usuario](../user/config.md#configuración-específica-de-usuarios).
* Añadir plugins (sólo parte cliente) en el directorio de configuración.
* Cualificar los módulos con el nombre del plugin al que pertenecen ([`installInRoot:false`](../dev/plugins.md)).
* Directorio [`theme`](../dev/plugins.md) en los plugins. Contiene ficheros CSS con el estilo de la aplicación.
* Parámetro [`debug`](minify_js_css.md) que carga los módulos sin minificación.
* Especificar el título del documento HTML en el fichero [`portal.properties`](../user/config.md).

### Modificado
* El soporte para el fichero `layers.json` se ha movido al plugin `base`.
* El directorio `nfms` que contiene los recursos se ha renombrado a `geoladris`.

### Bugs conocidos
* [Minificación no funciona para plugins con installInRoot:false](https://github.com/geoladris/geoladris/issues/24).
* [Es posible cargar recursos instalados en la raíz como si pertenecieran a un plugin inexistente.](https://github.com/geoladris/geoladris/issues/26).
