## Cliente

En primer lugar tendremos que bajarnos un servidor Tomcat sobre el que correremos la aplicación. Para esta documentación hemos utilizado la versión [8.5.27](https://tomcat.apache.org/download-80.cgi):

```bash
wget http://apache.rediris.es/tomcat/tomcat-8/v8.5.27/bin/apache-tomcat-8.5.27.tar.gz
tar xzf apache-tomcat-8.5.27.tar.gz
```

Configuraremos el directorio de Geoladris para Tomcat (ver [más](../user/config#directorio-de-configuracion)):

```bash
echo "JAVA_OPTS=-DGEOLADRIS_CONFIG_DIR=$PWD/geoladris_config_dir" > apache-tomcat-8.5.27/bin/setenv.sh
```


Después deberemos descargarnos la [aplicación](https://oss.sonatype.org/content/repositories/snapshots/com/github/geoladris/apps/develop/7.0.0-SNAPSHOT/develop-7.0.0-20180212.113905-2.war) de desarrollo sobre la que trabajaremos:

```bash
wget https://oss.sonatype.org/content/repositories/snapshots/com/github/geoladris/apps/develop/7.0.0-SNAPSHOT/develop-7.0.0-20180212.113905-2.war
mv develop-*.war apache-tomcat-8.5.27/webapps/develop.war
```

Lo siguiente que tenemos que hacer es preparar un directorio de configuración. En caso de no disponer de ninguno podemos utilizar este [ejemplo](config_dir.zip).

Una vez tenemos preparado/descomprimido el directorio de configuración, clonaremos el repositorio de plugins dentro, de forma que el portal utilice los ficheros cliente del repositorio:

```bash
git clone git@github.com:geoladris/plugins.git geoladris_config_dir/develop/plugins
```

Por último, construiremos los plugins con `yarn` antes de ejecutar:

```bash
cd geoladris_config_dir/develop/plugins
for i in `ls */package.json`; do
  pushd `dirname $i`
  yarn
  popd
done
cd ../../..
```

Y arrancaremos Tomcat:

```bash
apache-tomcat-8.5.27/bin/catalina.sh start
```

A partir de ahora podemos modificar los ficheros cliente dentro del directorio de configuración (`geoladris_config_dir/develop/plugins`) y al recargar el portal se aplicarán los cambios.

## Servidor

Para trabajar con el código del servidor antes tenemos que haber configurado el código del [cliente](#cliente). Además, en caso de que hayamos arrancado Tomcat, antes que nada deberemos pararlo:

```bash
apache-tomcat-8.5.27/bin/catalina.sh stop
```

En primer lugar clonamos el repositorio con la aplicación de desarrollo (`develop`):

```bash
git clone git@github.com:geoladris/apps.git
```

Luego nos descargamos [Eclipse IDE for Java EE Developers](https://www.eclipse.org/downloads/eclipse-packages/). Para esta guía se ha utilizado la versión Oxygen.2 (4.7.2).

Arrancamos Eclipse y configuramos el workspace. Una vez está abierto debemos importar los proyectos que vamos a utilizar. Para esto  basta con importarlos como proyectos Maven (*Project Explorer*, *Import > Import... > Maven > Existing Maven Projects*). Necesitaremos al menos la aplicación `develop` y el plugin `base`. Además, podemos importar cualquier otro plugin con el que queramos trabajar.

> Para el caso particular de los plugins, estos contienen directorios `node` y `node_modules` con una gran cantidad de recursos que ralentizan Eclipse. Para ignorar esos directorios podemos ejecutar el siguiente script bash:

```bash
for i in `find geoladris_config_dir/develop/plugins -name .project`; do
  sed -i "s#</projectDescription>#<filteredResources><filter><id>`date +%s0000`</id><name></name><type>10</type><matcher><id>org.eclipse.ui.ide.multiFilter</id><arguments>1.0-name-matches-false-false-node*</arguments></matcher></filter></filteredResources></projectDescription>#g" $i;
done
```

Ahora tenemos que configurar nuestro Tomcat en Eclipse. Para ello, lo añadimos en la vista de servidores.

Añadimos la aplicación `develop` al servidor Tomcat que acabamos de configurar y arrancamos Tomcat.

Si todo ha ido bien deberíamos poder ver un error 404 que muestra Tomcat en el navegador ([http://localhost:8080/develop](http://localhost:8080/develop)).

Esto es porque el directorio de configuración de Geoladris no está configurado correctamente. Para ello, paramos el servidor y abrimos las preferencias (`Run -> Debug configurations...`). Buscamos las preferencias de Tomcat y en la pestaña de `Environment` añadimos una nueva variable `GEOLADRIS_CONFIG_DIR` con el directorio de configuración.

Por último volvemos a arrancar Tomcat y estaremos listos para desarrollar, tanto en servidor como en cliente.

## Formateando el código

Para Java podemos utilizar el [fichero de estilo](https://google.github.io/styleguide/eclipse-java-google-style.xml) de Google.

Para JavaScript existe un [fichero de estilo](geoladris-style-js.xml) propio con algunas de las [reglas](contribute.md#formateo-del-codigo) de formateo que utiliza el proyecto.

Para aplicar los ficheros de estilos en Eclipse basta con descargar el fichero XML correspondiente e importarlo en Eclipse (_Preferences_ -> _&lt;lang&gt;_ -> _Code Style_ -> _Formatter_ -> _Import..._).

También es posible configurar JSHint en Eclipse de forma que muestre avisos para algunos (no todos) los errores de `eslint`:

```json
{
  "browser": true,
  "jquery": true,
  "node": true,
  "camelcase": true,
  "indent": 2,
  "latedef": true,
  "maxlen": 100,
  "newcap": true,
  "quotmark": "single",
  "eqeqeq": true,
  "eqnull": true,
  "undef": true,
  "unused": true,
  "eqnull": true,
  "globals" : {
    "define" : true,
    "describe" : true,
    "beforeEach" : true,
    "expect" : true,
    "spyOn" : true,
    "it" : true
  }
}
```
