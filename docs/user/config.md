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
		<param-name>GEOLADRIS_CONF_DIR</param-name>
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

> TODO Subdirectorio `messages`

## Añadir plugins después de desplegar

> TODO Subdirectorio `plugins`

## Recursos estáticos

> TODO Subdirectorio `static`

## `<config_dir>/portal.properties`

Fichero de propiedades. Cada plugin define sus propiedades (ver [referencia](../ref/plugins.md)). Estas propiedades son privadas y nunca abandonan el servidor, por lo que es un buen sitio para almacenar contraseñas y otra información sensible.

## Configurar caché

> TODO
>
> * GEOLADRIS_CONFIG_CACHE
> * GEOLADRIS_CACHE_TIMEOUT
