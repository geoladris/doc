# Supuestos Prácticos
A continuación se plantean unos supuestos prácticos para revisar los diferentes aspectos de la configuración del portal.


## Configuración de una nueva capa

La definición de las capas a mostrar en el Portal se encuentra en el fichero `layers.json`.

Contiene la información para asociar los elementos de la interfaz de usuario (panel con la lista de capas en la parte izquierda de la página)
con las capas WMS publicadas en GeoServer, personalizar las leyendas, y definir cuáles de las capas son interrogables. También clasifica las capas
por grupos.

El formato utilizado para este fichero de configuración es JSON (JavaScript Object Notation), que es un formato para la representación de datos. Está fuera del objetivo de esta guía el aprendizaje de JSON, pero se exponen a continuación algunas nociones básicas:

* Los valores en JSON pueden ser: números, cadenas de carácteres, booleanos, arrays, objetos y el valor nulo. Por ejemplo: 13, "hola mundo", true, [12, 5, 2], {"id":3}.

* Los objetos están delimitados por llaves (`{}`) y contienen una serie de pares atributo-valor separados por comas. Los pares atributo/valor consisten en un nombre de propiedad entrecomillado, dos puntos y el valor. Por ejemplo podemos tener el siguiente elemento:

~~~

	{
		"id":12,
		"nombre":"paco",
		"edad":55
	}

  o incluso un elemento dentro de otro:

~~~

	{
		"empresa":"zapatos smith",
		"propietario":{
			"id":12,
			"nombre":"john smith",
			"edad":55
		},
		"pais":"Argentina"
	}


* Los arrays especifican sus valores entre corchetes ([]) y separados por comas.

~~~

	[1, 2, 3, 4, 5]

	[
		{
			"id":12,
			"nombre":"john smith",
			"edad":34
		},
		{
			"id":12,
			"nombre":"sarah smith",
			"edad":22
		},
		{
			"id":12,
			"nombre":"Clark Kent",
			"edad":43
		}
	]
~~~

<aside class="notice">

  * `Introducción al formato JSON <http://www.json.org/>`_
  * `Validador de JSON <http://jsonformatter.curiousconcept.com/>`_
  * Validador en línea de comandos: python -mjson.tool <fichero.json>

</aside>

El fichero `layers.json` contiene tres secciones:

* `wmsLayers`
* `portalLayers`
* `groups`

En este apartado vamos a realizar dos ejercicios:

* En primer lugar, vamos a añadir la capa de límites administrativos al grupo existente de "admin_areas".

* En segundo lugar, añadiremos la capa "roads" en un nuevo grupo de capas.


## Conexión WMS

Cada "wmsLayer" se corresponde con una de las capas publicadas en GeoServer, y describe la manera de conectarse al servidor para obtener los datos:

TODO link the reference and complete the reference if necessary

.. code-block:: js

  "wmsLayers": [
     {
      "id": "limites_administrativos",
      "baseUrl": "http://172.16.250.131/geoserver/gwc/service/wms",
      "wmsName": "capacitacion:limites_administrativos",
      "imageFormat": "image/png",
      "visible": true
    }
  ],


* Es posible copiar y pegar un elemento existente y reemplazar :

  * el nuevo "id" será distinto a todos los otros, por ejemplo: "limites_administrativos".
  * el nuevo "wmsName" será "capacitacion:limites_administrativos" (el nombre de la capa publicada en GeoServer).
  * la baseUrl debe apuntar al servidor geoserver donde hemos cargado la capa.


## Capas del portal

Cada "portalLayer" representa una capa en el árbol de capas del portal y por tanto añade nuevos datos necesarios para mostrar la información en la interfaz gráfica.

~~~

  "portalLayers": [
    {
      "id": "limites_administrativos",
      "active": true,
      "label": "${limites_administrativos}",
      "infoFile": "limites_def.html",
      "layers": ["country"],
      "inlineLegendUrl": "http://172.16.250.131/geoserver/wms?REQUEST=GetLegendGraphic&VERSION=1.0.0&FORMAT=image/png&WIDTH=20&HEIGHT=20&LAYER=unredd:country&TRANSPARENT=true"
    }
  ],

~~~

* Añadir un nuevo objeto en "context", de igual estructura y valores que "country", excepto los siguientes cambios:

  * el nuevo "id" será "regions".
  * como "label" se utilizará "${limites_administrativos}". De nuevo, esta etiqueta de sintaxis ${...} será sustituida por un texto en el idioma que
    corresponda, según los contenidos de "messages". Es la etiqueta que se mostrará en la interfaz gráfica.
  * en "infoFile" pondremos "administrative_boundaries_def.html". Esto creará un enlace a un documento con información sobre
    los datos (localizado en :file:`static/loc/<idioma>/html/`).
  * en "layers" pondremos ["limites_administrativos"], haciendo referencia al nuevo *layer*.
  * en "inlineLegendUrl" estableceremos el parámetro LAYER así `LAYER=capacitacion:limites_administrativos`. Esto generará
    una imagen con la leyenda.


## Grupos

Los "Groups" son una estructura recursiva (multinivel) para agrupar visualmente las capas en el panel.
El "group" de primer nivel construye cada uno de los grupos de capas en forma de persiana desplegable, conteniendo una lista de
"items" que hacen referencia a los contextos definidos anteriormente.

~~~

	"groups" : [
		{
			"id" : "admin",
			"label" : "${admin_areas}",
			"items" : [ "countryBoundaries", "provinces" ]
		},
		...
	]

~~~

Nótese que en la propiedad "items", se hace referencia a las "portalLayers" definidas anteriormente. También, es posible dentro de dicha propiedad, añadir varios subgrupos de manera que las capas contenidas en éstos se visualicen dentro de una misma pestaña, pero agrupados visualmente bajo un título.

~~~

	"groups" : [
		{
			"id" : "admin",
			"label" : "${admin_areas}",
			"items" : [
				{
					"id" : "admin1",
					"label" : "Nacional",
					"items": ["limite_nacional"]
				}, {
					"id" : "admin2",
					"label" : "Regional",
					"items": [ "provincias" ]
				}
			]
		},
		...
	]

~~~

* Añadir un nuevo elemento `{ "context": "limites_administrativos" }` a continuación de `{ "context": "country" }`. Esto incluirá la capa
  en el grupo de áreas administrativas.

* Finalmente, utilizar un validador JSON, para comprobar que la sintaxis del nuevo :file:`layers.json` es correcta, y recargar la página.

## Posición inicial del mapa y prefijo capas

Antes de añadir la capa de carreteras vamos a proceder a configurar la posición inicial del mapa. Para ello tenemos que editar el fichero
`static/custom.js` y que contiene al principio del todo una declaración con los valores que nos interesa cambiar::

	UNREDD.maxExtent = new OpenLayers.Bounds(-20037508, -20037508, 20037508, 20037508);
	UNREDD.restrictedExtent = new OpenLayers.Bounds(-20037508, -20037508, 20037508, 20037508);
	UNREDD.maxResolution = 4891.969809375;
	UNREDD.mapCenter = new OpenLayers.LonLat(-9334782,-101119);
	UNREDD.defaultZoomLevel = 0;

	UNREDD.wmsServers = [
	    "http://demo1.geo-solutions.it",
	    "http://incuweb84-33-51-16.serverclienti.com"
	];

Para la posición central del mapa tendremos que modificar el valor *UNREDD.mapCenter* y poner la coordenada central en Google
Mercator (EPSG:900913 o EPSG:3857), que es el sistema de referencia que se usa en la aplicación web.

  * Obtener la coordenada central del mapa en el sistema de coordenadas usado en el portal.

Para regular el nivel de zoom inicial es posible cambiar el valor *UNREDD.defaultZoomLevel*. Cuanto mayor es el nivel de
zoom, más cercano es el zoom inicial.

Por último, es posible configurar en `UNREDD.wmsServers` una o más URLs correspondientes a nuestro servidor de manera que en el fichero `layers.json` basete especificar los atributos *baseUrl* con URLs relativas comenzando por el carácter "/". Estas URLs se componen prefijando los servidores especificados en el valor *UNREDD.wmsServers*. Por otra parte, si el servidor tiene más de una dirección, es conveniente especificarlas todas, ya que algunos navegadores limitan la cantidad de peticiones que se pueden hacer simultáneamente a un servidor y éste sería un método para sobrepasar ese límite.

Ejercicio:

  * Poner el servidor local en *UNREDD.wmsServers* y poner todas las capas del servidor como relativas.


## Configuración de un nuevo grupo de capas

Repetiremos el ejercicio anterior para añadir la capa de ciudades, teniendo en cuenta que:

* Para el nuevo "layer", usaremos el id "ciudades" y la capa wms "capacitacion:ciudades". Además, añadiremos un nuevo
  atributo `"legend": "ciudades.png"` para mostrar la leyenda de la capa. Este atributo hace referencia a una imagen
  localizada en :file:`static/loc/<idioma>/images/`.

* En el nuevo "context", será más sencillo, sólo contendrá los tres elementos `"id": "roads", "label": "${ciudades}", "layers": ["ciudades"]`.

* En "contextGroups", crearemos un nuevo grupo llamado "otros", con esta sintaxis:

~~~

  {
    "group": {
      "label": "${other}",
      "items": [
          { "context": "roads" }
      ]
  }

~~~

* Tras validar el JSON, y recargar la página, obtendremos la capa de carreteras bajo el nuevo grupo "Otros".



## Más ejercicios configuración:

  * Buscar el elemento `title` en `messages_es.properties`.

Otro ejercicio:

  * Traducir el texto del enlace añadido en `footer.tpl`

De la misma manera, Para añadir un nuevo idioma (por ejemplo, el guaraní):

 * Editar `portal.properties` y añadir el elemento `"gn": "Guaraní"` a la propiedad `languages`::

    languages = {"gn": "Guaraní", "es": "Español", "en": "English"}

 * Copiar el fichero `messages_es.properties` con el nuevo nombre `messages_gn.properties`.
 * Traducir los textos en `messages_gn.properties`.
 * Reiniciar la aplicación para aplicar los cambios. Desde la linea de comandos::

	sudo service tomcat6 restart

