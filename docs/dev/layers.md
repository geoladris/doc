### layers.json

La estructura de capas del proyecto se define en el fichero `layers.json`.
Este elemento JSON tiene cuatro propiedades:

* `default-server`
* `wmsLayers`
* `portalLayers`
* `groups`

### `default-server` 

Define el servidor por defecto en el caso en que no se especifique un servidor base en el atributo `baseUrl` de la propiedad `wmsLayers`.

### `wmsLayers`

Es un array que define las capas que tendrá el mapa. El orden de las capas en el array definen el orden de las capas en el dibujado del mapa. Cada capa consistirá en un elemento que puede ser de tres tipos, `WMS`(por defecto),`osm`(Open Street Map) y `gmaps`(Google Maps). Cada capa tiene las siguientes propiedades:

* `id`: identificados de la capa
* `type`: tipo de la capa (`wms`, `osm`, `gmaps`)
* `legend`: nombre del fichero de imagen con la leyenda de la capa. Estos ficheros se acceden en `static/loc/{lang}/images`.Es posible poner la cadena de carácteres `auto` para obtener la imagen automaticamente de GeoServer usando la petición `GetLegendGraphic` de WMS.
* `sourceLink`: URL del proveedor de datos.
* `sourceLabel`: texto con el que presentar el enlace especificado en `sourceLink`.

Además existen otras propiedades en función del tipo de la capa. Para `wms`:

* `baseUrl`: URL del servidor WMS. Si no se especifica se usará `default-server`.
* `wmsName`: nombre de la capa en el servicio WMS
* `imageFormat`: formato de imagen a utilizar en las llamadas WMS.
* `queryType`: protocolo usado para la herramienta de información `wfs` o `wms`. Parámetro obligatorio para que la capa sea consultable. En el caso de ser `wfs` los siguientes parámetros son obligatorios: `queryGeomFieldName`, `queryFieldName`, `queryFieldAliases` y `queryTimeField` (si es capa temporal). En caso de ser `wms` no hay ningún parámetro obligatorio. El único requisito es que el servidor codifique en EPSG:4326 la geometría de la respuesta al `GetFeatureInfo` para que el objeto consultado se pueda localizar en el mapa mediante zoom y resaltado.
* `queryUrl`: URL base a utilizar para la petición de información. Base del servidor WMS o WFS a utilizar (según `queryType`). Si no se especifica se toma `baseUrl`.
* `queryGeomFieldName`: obligatorio en el caso de `queryType: wfs`. El nombre del campo geométrico. Tipicamente `geom`, `the_geom`, `geometry`, etc.
* `queryFieldNames`: obligatorio en el caso de `queryType: wfs`. Nombres de los campos que se quieren obtener en la petición de info.
* `queryFieldAliases`: obligatorio en caso de especificar `queryFieldNames`. Aliases de los especificados en `queryFieldNames`.
* `queryTimeFieldName`: obligatorio en caso de especificar `queryType: wfs` si la capa tiene varias instancias temporales.
* `queryHighlightBounds`: solo para el caso que `queryType: wms`. Si se desea resaltar solo el rectángulo que encuadra la geometría de los objetos consultados (`true`) o toda la geometría (`false`). Por defecto es `false`. En los casos en los que la geometría es muy grande es conveniente ponerlo a `true` para que el proceso de resaltado sea más rápido.

Para `osm` (Open Street Map):

* `osmUrls`: lista de las URLs de los tiles. Usando `${x}`, `${y}` y `${z}` como variables.

Para `gmaps`(Google Maps):

* `gmaps-type`: tipo de carga de Google: `ROADMAP`, `SATELLITE`, `HYBRID` o `TERRAIN`.


### `portalLayers` 
Define las capas que aparecen visibles al usuario. Cada `portalLayer` puede contener varias `wmsLayers`. Cada `portalLayer` puede contener los siguientes elementos:

* `id`: identificador de la capa.
* `label`: texto con el nombre de la capa a usar en el portal. Si se especifica entre `$[]`, se intentará obtener la traducción de los ficheros `.properties` existentes en el directorio `messages` del directorio de configuración del portal.
* `infoFile`: nombre del fichero HTML con información sobre la capa. El fichero se accede en `static/loc/{lang}/html`. En la interfaz gráfica se representa con un botón de información al lado del nombre de la capa.
* `infoLink`: URL con la información sobre la capa. Igual que `infoFile` pero especificando una ruta absoluta. `infoFile` tiene preferencia sobre `infoLink`, por lo que si se define el primero, `infoLink` se ignorará.
* `inlineLegendUrl`: URL con una imagen pequeña que situar al lado del nombre de la capa en el árbol de capas. También es posible poner la cadena de carácteres `auto` y el portal intentará obtener la imagen automáticamente de GeoServer usando la petición `GetLegendGraphic` de WMS.
* `active`: Si la capa está inicialmente visible o no.
* `layers`: Array con los identificadores de las `wmsLayers` a las que se accede a través de esta capa.
* `timeInstances`: Instantes de tiempo en ISO8601 separados por comas.
* `timeStyles`: Nombres de los estilos a utilizar para cada instancia temporal. Cada estilo se corresponde con aquella instancia temporal que ocupa la misma posición en la lista. Si no se especifica este parámetro se utilizará el estilo por defecto para todos los estilos.
* `date-format`: Formato de la fecha para cada capa. Según la librería `Moment <http://momentjs.com/docs/#/displaying>_`. Por ejempo: `DD-MM-YYYY`. Por defecto sólo el año (`YYYY`).
* 'feedback`: En el caso de que la herramienta de feedback esté instalada, si se quiere o no que la capa aparezca en dicha herramienta para permitir al usuario hacer comentarios sobre la capa.

### `groups` 
Define la estructura final de las capas en el árbol de capas. Cada elmento de `groups` contiene:

* `id`: Identificador del grupo.
* `label`: Igual que en `portalLayer`.
* `infoFile`: Igual que en `portalLayer`.
* `infoLink`: Igual que en `portalLayer`.
* `items`. Array con los identificadores de otros grupos (con la misma estructura que este elemento; recursivo) o capas (`portalLayer`).
