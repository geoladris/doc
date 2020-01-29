En primer lugar hay que clonar los repositorios de Geoladris (ver [repositorios](source_code.md#repositorios)):

```
git clone git@github.com:geoladris/core.git
git clone git@github.com:geoladris/plugins.git
git clone git@github.com:geoladris/apps.git
```

Posteriormente hay que instalar `core` y `plugins` (en ese orden) localmente con Maven (puede tardar un poco):

```
cd core
mvn install
cd ../plugins
mvn install
```

Por último, nos falta por empaquetar las aplicaciones utilizando `core` y `plugins`:

```
cd ../apps
mvn package
```

Una vez hecho esto tendremos los paquetes `.war` listos para ser desplegados en Tomcat:

```
cp apps/demo/target/demo.war $CATALINA_BASE/webapps
```
