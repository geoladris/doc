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

Con Geoladris se pueden configurar las cadenas de texto que aparecen en el visor e incluso escribirlas en diferentes idiomas.

Para ello, basta con crear un directorio `<config_dir>/messages` que contenga ficheros `messages_<lang>.properties`. Por ejemplo, para tener traducciones en inglés y español habría que crear los ficheros `messages_en.properties` y `messages_es.properties`. Además, existe un fichero `messages.properties` (sin el sufijo del idioma) con las traducciones por defecto, que puede ser una copia de los anteriores.

Un ejemplo de fichero de traducciones:

```
...
base_layers=Capas Base
cancel=Cancelar
chart=Gr\u00e1fico
data_source=Fuente de datos
...
```

Cada plugin define sus propias traducciones. Para ver qué traducciones hay disponibles, ver [referencia](../ref/plugins.md).

Para añadir la posibilidad de cambiar de un idioma a otro en el visor, existe el plugin `language-buttons`.

### Soporte multiidioma

En los casos anteriores vemos algunas cadenas de texto entre los símbolos `${` y `}`. Estos elementos son sustituidos por mensajes de texto traducidos a cada idioma.

En el directorio `messages` contamos con un fichero `messages.properties` que contiene los mensajes por defecto. Son los textos que se usarán en caso de no encontrar mensajes traducidos a una lengua específica. Los ficheros para los distintos idiomas soportados llevan el código del idioma al final del nombre, según la `[nomenclatura ISO 639-1 de dos letras](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)`

Para añadir un nuevo idioma (por ejemplo, el guaraní):

 * Editar ``portal.properties`` y añadir el elemento ``"gn": "Guaraní"`` a la propiedad ``languages``::

    languages = {"gn": "Guaraní", "es": "Español", "en": "English"}

 * Copiar el fichero ``messages_es.properties`` con el nuevo nombre ``messages_gn.properties``.
 * Traducir los textos en ``messages_gn.properties``.
 * Reiniciar la aplicación para aplicar los cambios. Desde la linea de comandos::

~~~
	$ sudo service tomcat6 restart
~~~

## Añadir plugins después de desplegar

> TODO Subdirectorio `plugins`

## Recursos estáticos

En ocasiones es útil poder servir recursos estáticos directamente desde la aplicación web. Para eso existe el directorio `<config_dir>/static`. Cualquier fichero que se incluya en este directorio se servirá automáticamente a través de la URL `http://<host>/<app>/static/*`. Por ejemplo, se podría acceder a un fichero `<config_dir>/static/header.png` a través de `http://localhost:8080/demo/static/header.png`.

### Adaptación del aspecto gráfico

#### Cabecera de página

Veamos cómo modificar la imagen de fondo, bandera y título de la cabecera del portal:

![](../_images/header.png)

Partiendo del PORTAL_CONFIG_DIR (generalmente en /var/portal):

* **Imagen de fondo**: Se encuentra en `static/img/right.jpg`. Sustituir este fichero por otro de igual nombre y formato (jpeg), de 92 píxeles de alto. El ancho puede variar, aunque se recomienda que sea tan ancho como sea posible, hasta los 1920 px de una pantalla de alta definición. Para conseguir un mejor efecto junto con la bandera, se recomienda rellenar de contenido (logotipos, fotografía) la parte más a la derecha de la imagen, hasta un máximo de 500 px. Utilizar un color de fondo liso para el resto de la imagen, que ocupe toda la franja de la izquierda, y que se corresponda con el color de fondo de la bandera.

* **Bandera**: Se encuentra en `static/img/left.jpg`. Sustituir este fichero por otro de igual nombre y formato (jpeg), de 92 píxeles de alto. El ancho puede variar, aunque se recomienda alrededor de los 200 px. Utilizar un color de fondo liso, correspondiente con la parte izquierda de la imagen de fondo, para dar una sensación de continuidad.

* **Título**: Se encuentra definido en los ficheros de mensajes, directorio `messages`, ficheros de nombre `messages_<lang>.properties`. Buscar la propiedad "title" en cada uno de los ficheros de idioma.

#### Favicon

Se conoce como *favicon* al icono que se muestra en el navegador en la barra de direcciones. Para personalizar el *favicon*
del portal, basta con copiar la imagen en el directorio `static/img`. El nombre de la imagen sólo puede ser `favicon.ico` o `favicon.png`.

#### Estilos predefinidos (CSS)

En ciertos casos se requiere modificar los estilos que vienen predefinidos para OpenLayers, jQuery o cualquier otro. En estos casos,
en lugar de modificar los estilos directamente en el fichero que se encuentra en `/var/tomcat/webapps/portal`, se ha de crear
un nuevo fichero `overrides.css` en el directorio `/var/portal/static/css` que contenga las reglas CSS que se desean modificar.

De esta manera, tendrán preferencia las reglas que se escriban en `overrides.css` frente a cualquier otra que se encuentre en
`/var/tomcat/webapps/portal`.

Además, cuando se despliegue una actualización del portal en Tomcat, el fichero `overrides.css` no se modificará, manteniendo
así la personalización.

## `<config_dir>/portal.properties`

Fichero de propiedades. Cada plugin define sus propiedades (ver [referencia](../ref/plugins.md)). Estas propiedades son privadas y nunca abandonan el servidor, por lo que es un buen sitio para almacenar contraseñas y otra información sensible.

## Configurar caché

> TODO
>
> * GEOLADRIS_CONFIG_CACHE
> * GEOLADRIS_CACHE_TIMEOUT
