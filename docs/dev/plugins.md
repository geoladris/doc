## Descripción

Las aplicaciones Geoladris utilizan una arquitectura cliente/servidor. El servidor utiliza la API Servlet mientras que el cliente utiliza HTML+CSS+Javascript.

Podemos encontrar dos tipos básicos de plugins: servidor y cliente.

### Servidor

Un plugin servidor es un proyecto Java que aprovecha la [API Servlet 3.1](https://javaee.github.io/servlet-spec/downloads/servlet-3.1/Final/servlet-3_1-final.pdf) en el servidor. Es suficiente tener un paquete `.jar` en el _classpath_ que contenga un fichero `web-fragment.xml` con los _servlets_, filtros, etc. a utilizar por el plugin.

### Cliente

Un plugin cliente es un directorio que contiene:

* `src`: Subdirectorio con módulos RequireJS (`.js`) y/o estilos (`.css`).
* `css`: Subdirectorio con estilos (`.css`) que tiene preferencia (se aplican después) con respecto a `src`.
* `jslib`: **Deprecado**. Subdirectorio con librerías y estilos externos que **no** se pueden gestionar con `npm`.
* `geoladris.json`. Descriptor de plugin. Contiene un objeto JSON con:

	* `installInRoot`: Indica si los módulos RequireJS se instalarán en la raíz de la `baseURL` de RequireJS o dentro de un directorio con el nombre del plugin. Por defecto es `false`.
	Hay que tener en cuenta que el lugar donde se instalen los módulos afecta a la manera en la que otros módulos los referencian. Por ejemplo, un módulo llamado `mi_modulo` en un plugin `mi_plugin` se referenciará como `mi_modulo` si se instala en la raíz (`installInRoot : true`) y como  `mi_plugin/mi_modulo` en caso contrario (o como `./mi_modulo` cuando se referencia por otros módulos del mismo plugin).
	* `default-conf`: Configuración para los módulos RequireJS. Es un objeto donde los nombres de las propiedades son los nombres de los módulos a configurar y los valores la configuración a pasarles a dichos módulos. En este fichero es suficiente con especificar únicamente el nombre del módulo (sin el prefijo del _plugin_) independientemente del valor de `installInRoot`.

		La configuración se puede obtener en el módulo con la pseudodependencia `module`:

			:::js
            define([ "module" ], function(module) {
              var configuration = module.config();
              ...
            });




    * `requirejs`: Objeto con configuración de RequireJS. Únicamente tiene en cuenta [paths](http://requirejs.org/docs/api.html#config-paths) y [shim](http://requirejs.org/docs/api.html#config-shim). `paths` únicamente debería incluir rutas a `node_modules` o `jslib` (deprecado).
	* `css`: Array con las rutas a los estilos de librerías externas a incluir (`node_modules` o `jslib`).

* [`package.json`](https://docs.npmjs.com/files/package.json).

Adicionalmente puede tener otros recursos propios de cualquier proyecto JavaScript (`karma.conf.js`, `test`, `yarn.lock`, ...).

#### Traducciones

> TODO: Depender del módulo `i18n` y utilizar cadenas de `messages.properties`.

### Híbridos

Proyectos que contienen ambos tipos de recurso (Java y JavaScript).

Para incluir [recursos](https://github.com/geoladris/plugins/blob/js_deps/pom.xml#L68) correctamente en el .jar.

Para empaquetar recursos correctamente en el [package.json](https://github.com/geoladris/plugins/blob/js_deps/base/package.json#L10).

**NOTA**: Es importante mencionar que los plugins híbridos se consideran una mala práctica, ya que acoplan la funcionalidad de cliente y servidor, mezclando todos los recursos y haciendo más difícil trabajar con el plugin.

## Plugins oficiales

En el [repositorio](https://github.com/geoladris/plugins/) de Geoladris encontramos todo tipo de plugins, tanto servidor como cliente o híbridos.

Los plugins servidor son proyectos Maven con un `pom.xml` donde se pueden ejecutar los comandos Maven normalmente (`mvn clean`, `mvn package`, etc.).

Los plugins cliente, además de los subdirectorios mencionados arriba, contienen los siguientes recursos:

* `test`: Directorio con los ficheros de test. Se utiliza [Jasmine](https://jasmine.github.io/) como framework de testeo.
* `karma.conf.js`: Configuración para la ejecución de los tests. Se utiliza [Karma](https://karma-runner.github.io) como motor de testeo. Los tests se ejecutan con `yarn run test` (o `yarn run testd` para depurar).
* `yarn.lock`: Las dependencias son manejadas con `yarn`. Se considera una [buena práctica](https://yarnpkg.com/blog/2016/11/24/lockfiles-for-all/) incluir este fichero en el repositorio.
* `pom.xml`: Los plugins cliente también se gestionan con Maven (gracias al plugin Maven [frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin)) de forma que se pueden ejecutar todos los tests, tanto servidor como cliente, con `mvn test`.

## Configurar la aplicación mediante programación

En ocasiones la configuración de un plugin depende de un valor de la base de datos o, en general, de aspectos que se tienen que comprobar por programación. ¿De qué manera es posible hacer llegar estos valores a un elemento de la interfaz de usuario? La solución son los proveedores de configuración.

Los proveedores de configuración son instancias que implementan la interfaz `org.geoladris.config.PluginConfigProvider` que permiten añadir elementos a la configuración de los módulos de la misma manera que se haría manualmente modificando el fichero ``public-conf.json``.

Para que la instancia contribuya a la configuración hay que registrarla en la instancia `Config`, por lo que normalmente se registrará en un `ServletContextListener` con un código similar al siguiente:

```java
ServletContext servletContext = sce.getServletContext();
Config config = (Config) servletContext.getAttribute(Geoladris.ATTR_CONFIG);
config.addPluginConfigProvider(new MiConfigProvider());
```
