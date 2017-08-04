## Configurar el workspace

> TODO

## Formateando el código

> TODO Java

Existe un [fichero de estilo](geoladris-style-js.xml) para Eclipse con algunas de las [reglas](contribute.md#formateo-del-codigo) de formateo de JavaScript  que utiliza el proyecto.

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

## Trucos útiles


** Script bash para filtrar los recursos `node*` para los plugins**:

```bash
for i in `find geoladris/plugins -name .project`; do
  sed -i "s#</projectDescription>#<filteredResources><filter><id>`date +%s0000`</id><name></name><type>10</type><matcher><id>org.eclipse.ui.ide.multiFilter</id><arguments>1.0-name-matches-false-false-node*</arguments></matcher></filter></filteredResources></projectDescription>#g" $i;
done
```
