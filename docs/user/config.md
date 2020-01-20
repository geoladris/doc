## Directorio de configuración

Las aplicaciones Geoladris se configuran mediante un directorio con ficheros de configuración.

Este directorio deberá contener un subdirectorio por cada aplicación desplegada. Por ejemplo, si se han desplegado las aplicaciones `demo.war` y `bosques.war`, y `GEOLADRIS_CONFIG_DIR` se ha establecido a `/var/geoladris`, se utilizarán los directorios `/var/geoladris/demo` y `/var/geoladris/bosques` para las aplicaciones `demo.war` y `bosques.war` respectivamente.

Si alguno de esos directorios no existe o si `GEOLADRIS_CONFIG_DIR` no se ha configurado correctamente, se utilizará el directorio por defecto para esa aplicación concreta: `$CATALINA_BASE/webapps/<app>/WEB-INF/default_config`.

Para configurar este directorio en Tomcat hay diferentes formas de hacerlo:

* Variable de entorno: `export GEOLADRIS_CONFIG_DIR=/var/geoladris`.
* Propiedad Java: `-DGEOLADRIS_CONFIG_DIR=/var/geoladris`.
* Parámetro en `web.xml`:

```xml
	<context-param>
		<param-name>GEOLADRIS_CONFIG_DIR</param-name>
		<param-value>/var/geoladris</param-value>
	</context-param>
```

A partir de ahora utilizaremos `<config_dir>` como el directorio de configuración de una aplicación.

## Configuración global

La configuración global de la aplicación se realiza con el fichero `<config_dir>/public-conf.json`. Es un fichero de texto que contiene un objeto JSON donde cada propiedad del objeto es el nombre del plugin a configurar y el valor es la configuración del plugin.

La configuración del plugin es a su vez otro objeto JSON. Cada propiedad es el nombre del módulo RequireJS a configurar y el valor es la configuración del módulo, tal y como se puede obtener con ``module.config()``.

Aparte existen las siguientes propiedades o psedo-módulos:

* `_enabled`: Activa (`true`) o desactiva (`false`) el plugin. Por defecto es `true`.
* `_override`: Sobreescribe (`true`) o mezcla (`false`) la configuración por defecto del plugin. Por defecto es `false`.

Ejemplo:

```json
{
  "base" : {
    "banner" : {
      "hide" : true,
      "show-flag" : false,
      "show-logos" : false
    }
  },
  "footnote": {
    "footnote": {
      "text": "footnote.text",
      "link": "http://example.com",
      "align": "center"
    }
  },
  "feedback": {
    "_enabled" : false
  }
}
```

De esta forma es posible no solo cambiar la configuración de un plugin, sino también añadir o eliminar plugins para un usuario concreto con la propiedad `_enabled`.

## Autenticación y configuración específica de usuario

Además de la configuración global, se puede adaptadar la aplicación en función del usuario que accede a ella. Basta con añadir un fichero `<config_dir>/role_conf/<role>.json` por cada rol con su configuración específica. El formato de estos ficheros es el mismo que el de `public-conf.json`.

Para mayor flexibilidad, la autenticación es gestionada por los plugins. Actualmente en Geoladris existe un plugin `auth` encargado de la autenticación. Para utilizarlo es necesario incluirlo en el momento de [crear](create.md) la aplicación.

La autenticación de este plugin se apoya en la autenticación del contenedor de servlets. Es decir, si se utiliza Tomcat, los usuarios de Geoladris serán aquellos que están configurados en Tomcat.

Para configurar la autenticación de Tomcat, lo más fácil es editar el fichero `$CATALINA_BASE/conf/tomcat-users.xml` y añadir los usuarios y roles que necesitemos. Por ejemplo:

```xml
<tomcat-users>
  <role rolename="viewer"/>
  <user username="user" password="pass" roles="viewer"/>
</tomcat-users>
```

Aunque hay otras opciones de configuración que se pueden encontrar en la [documentación oficial](https://tomcat.apache.org/tomcat-8.5-doc/realm-howto.html).

Una vez se han configurado los usuarios, lo siguiente es restringir qué roles pueden acceder al visor. Para ello, basta con añadir la propiedad `auth.roles` al fichero `portal.properties` del directorio de configuración:

```
auth.roles=viewer
```

Esta propiedad es una lista de roles separada por comas. Los roles deben de coincidir con los roles definidos en Tomcat.

Por último, para configurar el visor de manera específica para cada rol deberemos añadir un fichero `<role>.json` a `<config_dir>/role_conf` por cada rol definido en `auth.roles`.

Por ejemplo, para habilitar los plugins `layer-order` y `layers-editor` en el rol `viewer`, habrá que añadir un fichero `role_conf/viewer.json` con el siguiente contenido:

```json
{
	"layer-order" : {
		"_enabled" : true
	},
	"layers-editor" : {
		"_enabled" : true
	}
}
```

## Traducciones

Con Geoladris se pueden configurar las cadenas de texto que aparecen en el visor e incluso escribirlas en diferentes idiomas.

Para ello, basta con crear un directorio `<config_dir>/messages` que contenga ficheros `messages_<lang>.properties`. `<lang>` es el código del idioma según la nomenclatura [ISO 639-1](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) de dos letras.

Por ejemplo, para tener traducciones en inglés y español habría que crear los ficheros `messages_en.properties` y `messages_es.properties`. Además, existe un fichero `messages.properties` (sin el sufijo del idioma) con las traducciones por defecto, que puede ser una copia de los anteriores.

Un ejemplo de fichero de traducciones:

```
...
base_layers=Capas Base
cancel=Cancelar
chart=Gr\u00e1fico
data_source=Fuente de datos
...
```

Geoladris reconoce las siguientes traducciones:

* `title`: Título de la página HTML.

Aparte, cada plugin define sus propias traducciones. Para ver qué traducciones hay disponibles, ver [referencia](../ref/plugins.md).

Para añadir la posibilidad de cambiar de un idioma a otro en el visor, existe el plugin `language-buttons`.

## Añadir plugins después de desplegar

Una de las funcionalidades más interesantes de Geoladris es el despliegue de funcionalidad en caliente: es posible incluir nuevos plugins sin tener que parar y reiniciar el servidor.

Para esto, basta con copiar el plugin que se desea incluir dentro del directorio `<config_dir>/plugins`, [configurarlo](#configuracion-global) en caso necesario y recargar el navegador.

La única limitación es que los plugins han de ser plugins [cliente](../dev/plugins.md#cliente); es decir, plugins con código JavaScript exclusivamente. No es posible desplegar en caliente código Java con nuevos servicios.

Existen multitud de plugins. Se podrá encontrar información sobre ellos en la referencia a los [plugins](../ref/plugins.md)

## Recursos estáticos

En ocasiones es útil poder servir recursos estáticos directamente desde la aplicación web. Para eso existe el directorio `<config_dir>/static`. Cualquier fichero que se incluya en este directorio se servirá automáticamente a través de la URL `http://<host>/<app>/static/*`. Por ejemplo, se podría acceder a un fichero `<config_dir>/static/header.png` a través de `http://localhost:8080/demo/static/header.png`.

Geoladris reconoce algunos fichero estáticos específicos:

* `static/img/favicon.png`: Favicon de la página HTML.


## `<config_dir>/portal.properties`

Fichero de propiedades. Cada plugin define sus propiedades (ver [referencia](../ref/portal-properties.md)). Estas propiedades son privadas y nunca abandonan el servidor, por lo que es un buen sitio para almacenar contraseñas y otra información sensible.

## Configurar caché

La configuración del portal se obtiene a partir de varios ficheros (configuración global y específica de usuario) e incluso puede obtenerse de una base de datos o de un servicio externo para [desarrollos personalizados](../dev/plugins.md#configurar-la-aplicacion-mediante-programacion). En cualquier caso, obtenerla requiere un tiempo que puede hacer que el visor se ralentice cada vez.

Para ello, Geoladris proporciona una caché que se puede configurar mediante dos variables:

* `GEOLADRIS_CONFIG_CACHE`: `true`/`false`. Determina si se utiliza la caché o no. Por defecto es `false`.
* `GEOLADRIS_CACHE_TIMEOUT`: Número máximo de segundos que se mantiene la caché. Si no se especifica, la caché se mantiene siempre y es necesario reiniciar el servidor para releer la configuración.

De la misma manera que el directorio de configuración, estas variables pueden establecerse de varias maneras:

* Variable de entorno: `export GEOLADRIS_CONFIG_CACHE=true`.
* Propiedad Java: `-DGEOLADRIS_CONFIG_CACHE=true`.
* Parámetro en `web.xml`:

```xml
	<context-param>
		<param-name>GEOLADRIS_CONFIG_CACHE</param-name>
		<param-value>true</param-value>
	</context-param>
```

## Base de datos

Las aplicaciones `demo` y `develop` que proporciona Geoladris traen la posibilidad de conectar automáticamente a la base de datos que utilizarán algunos de los plugins (como el plugin `feedback`, por ejemplo).

Para configurar esta base de datos es necesario configurar las siguientes propiedades Java:

- `GEOLADRIS_DB_URL`: URL JDBC de la base de datos. Por ejemplo: `jdbc:postgresql://localhost:5432/geoladris`.
- `GEOLADRIS_DB_USER`: Usuario para la autenticación con la base de datos.
- `GEOLADRIS_DB_PASS`: Contraseña para la autenticación con la base de datos.

Por ejemplo, para configurarlo en Tomcat bastaría con establecer `JAVA_OPTS`:

```bash
JAVA_OPTS=-DGEOLADRIS_CONFIG_DIR=/var/geoladris -DGEOLADRIS_DB_URL=jdbc:postgresql://localhost:5432/geoladris -DGEOLADRIS_DB_USER=admin -DGEOLADRIS_DB_PASS=geoladris
```
