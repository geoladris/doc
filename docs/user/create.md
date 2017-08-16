Para crear una aplicación propia, Geoladris proporciona un [script](https://raw.githubusercontent.com/geoladris/core/master/build-tools/geoladris_build.sh) bash (Linux). Este script utiliza un fichero `build.json` que describe la aplicación y (opcionalmente) un directorio de configuración (llamado `config` junto a `build.json`) y produce un paquete `.war` listo para desplegar.

Por ejemplo, se puede crear una aplicación que contenga únicamente los plugins `base` y `layers-editor` utilizando este fichero `build.json`:

```json
{
  "name": "demo",
  "plugins": ["base", "layers-editor"]
}
```
Si se ejecuta el script sin argumentos:

```bash
$ geoladris_build.sh
```

obtendremos un fichero `demo.war` listo para desplegar. La versión de los plugins será la última versión estable. Para especificar una versión concreta para todos los plugins de Geoladris se puede utilizar `-v`:

```bash
$ geoladris_build.sh -v 6.0.1
```

También se puede utilizar el script para generar una aplicación con plugins que no son de Geoladris. Basta con añadir `<groupId>:<artifactId>:<version>` para los plugins servidor y `<dependency>:<version>` para los plugins cliente. Por ejemplo:

```json
{
  "name": "demo",
  "version": "1.0.0",
  "plugins": ["base", "layers-editor", "@csgis/ui:1.0.0", "de.csgis.geoladris:myplugin:1.0.1"]
}
```

Por último, es posible obtener una ayuda con todas las opciones del script y la referencia del fichero `build.json` con `-h`:

```bash
$ geoladris_build.sh -h
```
