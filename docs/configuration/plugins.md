# Configuración de los plugins

Como ya se ha comentado, Geoladris está preparado para modificar su comportamiento mediante la carga de plugins. Si se visualiza la [estructura del proyecto](../installation/project_structure.md) se verá que la mayor parte de la funcionalidad se encuentra implementada mediante el desarrollo de plugins. A continuación se introduce la configuración de los diferentes plugins.

La configuración de los plugins se puede especificar de dos maneras diferentes: mediante [ficheros](#ficheros-json) en el directorio de configuración o mediante una conexión a una [base de datos](#db).

En ambos casos, la configuración se especifica con un objeto JSON. Cada propiedad del objeto es el nombre del _plugin_ a configurar y el valor es la configuración del plugin.

La configuración del plugin es a su vez otro objeto JSON. Cada propiedad es el nombre del módulo RequireJS a configurar y el valor es la configuración del módulo, tal y como se puede obtener con ``module.config()``.

Aparte existen los siguientes psedo-módulos:

* `_enabled`: Activa (`true`) o desactiva (`false`) el plugin. Por defecto es `true`.
* `_override`: Sobreescribe (`true`) o mezcla (`false`) la configuración por defecto del plugin. Por defecto es `false`.

Ejemplo:

~~~
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
~~~

De esta forma es posible no solo cambiar la configuración de un _plugin_, sino también añadir o eliminar _plugins_ para un usuario concreto con el pseudomódulo `_enabled`.


Además es importante destacar que esta configuración pública o base puede ser adaptada en función del usuario que accede a la plataforma. Para ello, habrá que especificar un objeto JSON diferente por cada rol que pueda autenticarse en la aplicación. El formato de esos objetos JSON es el mismo que el descrito anteriormente.

**Nota**: los usuarios y roles son gestionados por los *plugins*. Actualmente en Geoladris existe un plugin `auth` encargado de la autenticación.

## <a name="ficheros-json"></a>Ficheros `.json`

En el caso de configurar los plugins mediante ficheros, bastará con crear un fichero `public-conf.json` en el directorio de configuración, con el objeto JSON descrito en el apartado anterior-

Si además se desea adaptar la configuración en función del usuario , habrá que añadir ficheros `role_conf/<rol>.json` dentro del directorio de configuración. Cada uno de los ficheros contiene la configuración específica del rol del usuario. El contenido de los ficheros sigue el mismo formato que `public-conf.json`.

## <a name="db"></a>Base de datos

Para configurar los plugins desde una base de datos, dicha base de datos deberá tener una tabla `apps` con las siguientes columnas:

- `app`: Nombre de la aplicación.
- `role`: Rol autenticado. El rol `default` está reservado para la configuración pública (análogo a `public-conf.json`).
- `conf`: Configuración de los plugins.

~~~
CREATE TABLE geoladris.apps (app text, role text, conf json NOT NULL,
  PRIMARY KEY(app, role));
~~~

También es posible configurar las propiedades y los mensajes de la aplicación desde la base de datos con las siguientes tablas:

~~~
CREATE TABLE geoladris.props (app text, key text, value text NOT NULL,
  PRIMARY KEY(app, key));
CREATE TABLE geoladris.messages (app text, lang varchar(5), key text, value text NOT NULL,
  PRIMARY KEY(app, lang, key));
~~~

La conexión a la base de datos se obtiene de la siguiente manera:

- Se busca un `Resource` llamado `jdbc/geoladris` en `context.xml`.
- Si no existe, se obtiene la conexión de las variables de entorno:
  - `JDBC_CONNECTION_URL`
  - `JDBC_CONNECTION_USER`
  - `JDBC_CONNECTION_PASS`
  - `JDBC_CONNECTION_SCHEMA`

Si se ha obtenido una conexión válida, con una configuración por defecto (`SELECT conf FROM <schema>.apps WHERE app = '<app>' AND role = 'default'`), se utiliza dicha configuración. En caso contrario, se utilizan los [ficheros .json](#ficheros-json).
