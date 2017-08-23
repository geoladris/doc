**IMPORTANTE**: Esta documentación está en construcción y puede estar desactualizada y/o incompleta.

## ol2

### `map.js`

**Configuración**:

* `gmaps_key`: Clave de API de Google Maps.
* `htmlId`: Identificador del elemento HTML donde colocar el mapa. Por defecto `map`.
* `numZoomLevels`: Número de niveles de zoom del mapa.

**Traducciones**:

**Eventos**:


## layers.json

Define la estructura de capas del proyecto. Consiste en un elemento JSON con cuatro propiedades:

```js
{
    "default-server" : "http://demo1.geo-solutions.it",
    "wmsLayers" : [],
    "portalLayers" : [],
    "groups" : []
}
```

* `default-server` define el servidor que se usará como base en caso de que la URL de las capas no incluyan servidor. Ver atributo `baseUrl` más abajo.

* `wmsLayers` define las capas que tendrá el mapa. El orden en el que estas capas aparecen en este array define el orden de las capas en el dibujado del mapa. Cada capa consistirá en un elemento que puede ser de tres tipos. El tipo por defecto es WMS y tiene las siguientes propiedades:

    * `id`: Identificador de la capa.
    * `type`: Tipo de la capa. Puede ser `wms`, `osm` (para Open Street Map) o `gmaps` (para Google Maps). Por defecto es `wms`.
    * `legend`: Nombre del fichero imagen con la leyenda de la capa. Estos ficheros se acceden en `static/loc/{lang}/images`. También es posible poner la cadena de carácteres `auto` y el portal intentará obtener la imagen automáticamente de GeoServer usando la petición `GetLegendGraphic` de WMS.
    * `sourceLink`: URL del proveedor de los datos.
    * `sourceLabel`: Texto con el que presentar el enlace especificado en `sourceLink`.

      En función del tipo de la capa se especificarán además otras propiedades. Para `wms`:

    * `baseUrl`: URL del servidor WMS que sirve la capa. Si se especifica una URL sin servidor, por ejemplo  `/diss_geoserver/gwc/service/wms`, se usará `default-server`.
    * `wmsName`: Nombre de la capa en el servicio WMS.
    * `imageFormat`: Formato de imagen a utilizar en las llamadas WMS.
    * `queryType`: Protocolo usado para la herramienta de información: `wfs` o `wms`. En caso de ser `wfs` los siguientes parámetros son obligatorios: `queryGeomFieldName`, `queryFieldNames`, `queryFieldAliases` y `queryTimeField` (en caso de ser una capa temporal). En caso de ser `wms` no hay ningún parámetro adicional obligatorio, por lo que una capa que quiera usar WMS para la herramienta de información puede configurarse sólo con: `"queryType": "wms"`. El único requisito para capas con `"queryType": "wms"` es que el servidor codifique en EPSG:4326 la geometría de la respuesta al `GetFeatureInfo`; en caso contrario el objeto consultado no se podrá localizar en el mapa mediante zoom y resaltado. Si la capa no tiene un parámetro `queryType` la capa no será consultable.
    * `queryUrl`: URL base a utilizar en la petición de información. Base del servidor WMS o WFS a utilizar (según `queryType`). Si no se especifica se toma `baseUrl`.
    * `queryGeomFieldName`: Obligatorio en el caso de `"queryType": "wfs"`. El nombre del campo geométrico. Típicamente `geom`, `the_geom`, `geometry`, etc.
    * `queryFieldNames`: Obligatorio en el caso de `"queryType": "wfs"`. Nombres de los campos que se quieren obtener en la petición de info.
    * `queryFieldAliases`: Obligatorio en el caso de especificar `queryFieldNames`. Aliases de los campos especificados en `queryFieldNames`.
    * `queryTimeFieldName`: Obligatorio en el caso de `"queryType": "wfs"` si la capa tiene varias instancias temporales. Sin uso en el caso de `"queryType": "wms"`.
    * `queryHighlightBounds`: Sólo para el caso `"queryType": "wms"`. Si se desea resaltar solo el rectángulo que encuadra la geometría de los objetos consultados (`true`) o toda la geometría (`false`). Por defecto es `false`. En los casos en los que la geometría es muy grande puede ser conveniente ponerlo a `true` para que el proceso de resaltado sea más rápido. Por ejemplo:

            :::js
            {
              "wmsLayers" : [{
                "id" : "provinces",
                "baseUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms",
                "wmsName" : "unredd:drc_provinces",
                "imageFormat" : "image/png8",
                "visible" : true,
                "sourceLink" : "http://www.wri.org/publication/interactive-forest-atlas-democratic-republic-of-congo",
                "sourceLabel" : "WRI",
                "queryable" : true
              }],
              ...
            }


      Para `osm` (Open Street Map):

    * `osmUrls`: lista de las URLs de los tiles. Usando `${x}`, `${y}` y `${z}` como variables. Por ejemplo:

            :::js
            {
              "wmsLayers" : [{
                "id" : "openstreetmap",
                "type" : "osm",
                "osmUrls" : [
                  "http://a.tile.openstreetmap.org/${z}/${x}/${y}.png",
                  "http://b.tile.openstreetmap.org/${z}/${x}/${y}.png",
                  "http://c.tile.openstreetmap.org/${z}/${x}/${y}.png"
                ]
              }],
              ...
            }

      Para `gmaps` (Google Maps):

    * `gmaps-type`: Tipo de capa Google: `ROADMAP`, `SATELLITE`, `HYBRID` o `TERRAIN`. Por ejemplo:

            :::js
            {
              "wmsLayers" : [{
                "id" : "google-maps",
                "type" : "gmaps",
                "gmaps-type" : "SATELLITE"
              }],
              ...
            }

* `portalLayers` define las capas que aparecen visibles al usuario. Una `portalLayer` puede contener varias `wmsLayers`. Cada `portalLayer` puede contener los siguientes elementos:

    * `id`: Identificador de la capa.
    * `label`: Texto con el nombre de la capa a usar en el portal. Si se especifica entre ${ }, se intentará obtener la traducción de los ficheros `.properties` existentes en el directorio `messages` del  directorio de configuración del portal.
    * `infoFile`: Nombre del fichero HTML con información sobre la capa. El fichero se accede en  `static/loc/{lang}/html`. En la interfaz gráfica se representa con un botón de información al lado del nombre de la capa.
    * `infoLink`: URL con la información sobre la capa. Igual que `infoFile` pero especificando una ruta absoluta. `infoFile` tiene preferencia sobre `infoLink`, por lo que si se define el primero, `infoLink` se ignorará.
    * `inlineLegendUrl`: URL con una imagen pequeña que situar al lado del nombre de la capa en el árbol de capas. También es posible poner la cadena de carácteres `auto` y el portal intentará obtener la imagen automáticamente de GeoServer usando la petición `GetLegendGraphic` de WMS.
    * `active`: Si la capa está inicialmente visible o no.
    * `layers`: Array con los identificadores de las `wmsLayers` a las que se accede a través de esta capa.
    * `timeInstances`: Instantes de tiempo en ISO8601 separados por comas.
    * `timeStyles`: Nombres de los estilos a utilizar para cada instancia temporal. Cada estilo se corresponde con aquella instancia temporal que ocupa la misma posición en la lista. Si no se especifica este parámetro se utilizará el estilo por defecto para todos los estilos.
    * `date-format`: Formato de la fecha para cada capa. Según la librería `Moment <http://momentjs.com/docs/#/displaying>_`. Por ejempo: `DD-MM-YYYY`. Por defecto sólo el año (`YYYY`).
    * `feedback`: En el caso de que la herramienta de feedback esté instalada, si se quiere o no que la capa aparezca en dicha herramienta para permitir al usuario hacer comentarios sobre la capa.

        Por ejemplo:

            :::js
            {
              "wmsLayers" : [{
                "id" : "wms_provinces",
                "baseUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms",
                "wmsName" : "unredd:drc_provinces",
                "imageFormat" : "image/png8",
                "visible" : true,
                "sourceLink" : "http://www.wri.org/publication/interactive-forest-atlas-democratic-republic-of-congo",
                "sourceLabel" : "WRI",
                "queryable" : true
              }],
              "portalLayers" : [{
                "id" : "provinces",
                "active" : true,
                "infoFile" : "provinces_def.html",
                "label" : "${provinces}",
                "layers" : [ "wms_provinces" ],
                "inlineLegendUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms?REQUEST=GetLegendGraphic&VERSION=1.0.0&FORMAT=image/png&WIDTH=20&HEIGHT=20&LAYER=unredd:drc_provinces&TRANSPARENT=true",
                "timeInstances" : "2007-03-01T00:00,2008-05-11T00:00,2005-03-01T00:00",
                "timeStyles" : "style2007,style2008,style2005",
                "date-format" : "DD-MM-YYYY"
              }],
              ...
            }

* `groups` define la estructura final de las capas en el árbol de capas de la aplicación. Cada elemento de `groups` contiene:

    * `id`: Identificador del grupo.
    * `label`: Igual que en `portalLayer`
    * `infoFile`: Igual que en `portalLayer`
    * `infoLink`: Igual que en `portalLayer`
    * `items`. Array con los identificadores de otros grupos (con la misma estructura que este elemento; recursivo) o capas (`portalLayer`).

        Por ejemplo:

            :::js
            {
            "wmsLayers" : [{
                "id" : "wms_provinces",
                "baseUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms",
                "wmsName" : "unredd:drc_provinces",
                "imageFormat" : "image/png8",
                "visible" : true,
                "sourceLink" : "http://www.wri.org/publication/interactive-forest-atlas-democratic-republic-of-congo",
                "sourceLabel" : "WRI",
                "queryable" : true,
                "wmsTime" : "2007-03-01T00:00,2008-05-11T00:00,2005-03-01T00:00"
              }],
              "portalLayers" : [{
                "id" : "provinces",
                "active" : true,
                "infoFile" : "provinces_def.html",
                "label" : "${provinces}",
                "layers" : [ "wms_provinces" ],
                "inlineLegendUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms?REQUEST=GetLegendGraphic&VERSION=1.0.0&FORMAT=image/png&WIDTH=20&HEIGHT=20&LAYER=unredd:drc_provinces&TRANSPARENT=true"
              }],
              "groups" : [{
                "id" : "base",
                "label" : "${base_layers}",
                "infoFile": "base_layers.html",
                "items" : ["provinces"]
              }]
            }
