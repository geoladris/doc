## Configurar el workspace

Para importar los proyectos de Geoladris, basta con importarlos como proyectos Maven (*Project Explorer*, *Import > Import... > Maven > Existing Maven Projects*).

Para el caso particular de los plugins, estos contienen directorios `node` y `node_modules` con una gran cantidad de recursos que ralentizan Eclipse. Para ignorar esos directorios podemos ejecutar el siguiente script bash:

```bash
for i in `find geoladris/plugins -name .project`; do
  sed -i "s#</projectDescription>#<filteredResources><filter><id>`date +%s0000`</id><name></name><type>10</type><matcher><id>org.eclipse.ui.ide.multiFilter</id><arguments>1.0-name-matches-false-false-node*</arguments></matcher></filter></filteredResources></projectDescription>#g" $i;
done
```

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
