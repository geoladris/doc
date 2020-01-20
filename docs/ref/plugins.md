**IMPORTANTE**: Esta documentación está en construcción y puede estar desactualizada y/o incompleta.

## ol2

### `map.js`

**Configuración**:

* `gmaps_key`: Clave de API de Google Maps.
* `htmlId`: Identificador del elemento HTML donde colocar el mapa. Por defecto `map`.
* `numZoomLevels`: Número de niveles de zoom del mapa.

**Traducciones**:

**Eventos**:

## Base

El plugin base es el que se encarga de generar un portal básico para visualización de información geográfica. Uno de loss puntos importantes de este plugin es el archivo con el que se configuran las capas y grupos del visor. Este archivo se creará en la carpeta de confiiguración de Geoladris.

### layers.json


Define la estructura de capas del proyecto. Consiste en un elemento JSON con cuatro propiedades:

	{
		"default-server" : "http://snmb.ambiente.gob.ar/geo-server",
		"wmsLayers" : [],
		"portalLayers" : [],
		"groups" : []
	}

* `default-server` define el servidor que se usará como base en caso de que la URL de las capas no incluyan servidor. Ver atributo `baseUrl` más abajo.

#### `wmsLayers` 

Define las capas que tendrá el mapa. El orden en el que estas capas aparecen en este array define el orden de las capas en el dibujado del mapa. Cada capa consistirá en un elemento que puede ser de tres tipos. El tipo por defecto es WMS y tiene las siguientes propiedades:

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
* `queryHighlightBounds`: Sólo para el caso `"queryType": "wms"`. Si se desea resaltar solo el rectángulo que encuadra la geometría de los objetos consultados (`true`) o toda la geometría (`false`). Por defecto es `false`. En los casos en los que la geometría es muy grande puede ser conveniente ponerlo a `true` para que el proceso de resaltado sea más rápido.

	Por ejemplo:

		{
			"wmsLayers" : [
				{
					"id" : "provinces",
					"baseUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms",
					"wmsName" : "unredd:drc_provinces",
					"imageFormat" : "image/png8",
					"visible" : true,
					"sourceLink" : "http://www.wri.org/publication/interactive-forest-atlas-democratic-republic-of-congo",
					"sourceLabel" : "WRI",
					"queryable" : true
				}
			],
			...
		}

Para `osm` (Open Street Map):

* `osmUrls`: lista de las URLs de los tiles. Usando `${x}`, `${y}` y `${z}` como variables.

Por ejemplo:

		{
			"wmsLayers" : [
				{
					"id": "osm",
					"type": "osm",
					"osmUrls": [
						"http://a.tile.openstreetmap.org/${z}/${x}/${y}.png",
						"http://b.tile.openstreetmap.org/${z}/${x}/${y}.png",
						"http://c.tile.openstreetmap.org/${z}/${x}/${y}.png"
					]
				}
			],
			...
		}

Para `gmaps` (Google Maps):

* `gmaps-type`: Tipo de capa Google: `ROADMAP`, `SATELLITE`, `HYBRID` o `TERRAIN`.

> Google Maps necesita una API KEY que deberá ser configurada.

Por ejemplo:

		{
			"wmsLayers" : [
				{
					"id": "google-maps",
					"type": "gmaps",
					"gmaps-type": "SATELLITE",
					"visible": true
				}
			],
			...
		}


#### `portalLayers` 

Define las capas que aparecen visibles al usuario. Una `portalLayer` puede contener varias `wmsLayers`. Cada `portalLayer` puede contener los siguientes elementos:

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

		{
			"wmsLayers" : [
				{
					"id" : "wms_provinces",
					"baseUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms",
					"wmsName" : "unredd:drc_provinces",
					"imageFormat" : "image/png8",
					"visible" : true,
					"sourceLink" : "http://www.wri.org/publication/interactive-forest-atlas-democratic-republic-of-congo",
					"sourceLabel" : "WRI",
					"queryable" : true
				}
			],
			"portalLayers" : [
				{
					"id" : "provinces",
					"active" : true,
					"infoFile" : "provinces_def.html",
					"label" : "${provinces}",
					"layers" : [ "wms_provinces" ],
					"inlineLegendUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms?REQUEST=GetLegendGraphic&VERSION=1.0.0&FORMAT=image/png&WIDTH=20&HEIGHT=20&LAYER=unredd:drc_provinces&TRANSPARENT=true",
					"timeInstances" : "2007-03-01T00:00,2008-05-11T00:00,2005-03-01T00:00",
					"timeStyles" : "style2007,style2008,style2005",
					"date-format" : "DD-MM-YYYY"
				}
			],
			...
		}

#### `groups` 

Define la estructura final de las capas en el árbol de capas de la aplicación. Cada elemento de `groups` contiene:

* `id`: Identificador del grupo.
* `label`: Igual que en `portalLayer`
* `infoFile`: Igual que en `portalLayer`
* `infoLink`: Igual que en `portalLayer`
* `items`. Array con los identificadores de otros grupos (con la misma estructura que este elemento; recursivo) o capas (`portalLayer`).

Por ejemplo:

		{
			"wmsLayers" : [
				{
					"id" : "wms_provinces",
					"baseUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms",
					"wmsName" : "unredd:drc_provinces",
					"imageFormat" : "image/png8",
					"visible" : true,
					"sourceLink" : "http://www.wri.org/publication/interactive-forest-atlas-democratic-republic-of-congo",
					"sourceLabel" : "WRI",
					"queryable" : true,
					"wmsTime" : "2007-03-01T00:00,2008-05-11T00:00,2005-03-01T00:00"
				}
			],
			"portalLayers" : [
				{
					"id" : "provinces",
					"active" : true,
					"infoFile" : "provinces_def.html",
					"label" : "${provinces}",
					"layers" : [ "wms_provinces" ],
					"inlineLegendUrl" : "http://demo1.geo-solutions.it/diss_geoserver/wms?REQUEST=GetLegendGraphic&VERSION=1.0.0&FORMAT=image/png&WIDTH=20&HEIGHT=20&LAYER=unredd:drc_provinces&TRANSPARENT=true"
				}
			],
			"groups" : [
				{
					"id" : "base",
					"label" : "${base_layers}",
					"infoFile": "base_layers.html",
					"items" : ["provinces"]
				}
			]
		}
		
Algunas configuraciones de este plugin deberán incorporarse en el archivo `portal.properties`:

| Propiedad | Descripción | Valor por defecto | Ejemplo de configuración |
| -- | -- | -- | -- |
| map.centerLonLat | Coordenadas geográficas donde se posicionará el mapa inicialmente | `-58,-34` | `map.centerLonLat=-58,-34` |
| map.initialZoomLevel | Nivel inicial de zoom del mapa | `5` | `map.initialZoomLevel=5` |

Dentro del plugin base también se encuentran las opciones para modificar el aspecto de la aplicación:

### Cabecera de página

Veamos cómo modificar la imagen de fondo, bandera y título de la cabecera del portal:

![](images/header.png)

Partiendo del GEOLADRIS_CONFIG_DIR:

* **Imagen de fondo**: Se encuentra en `static/img/right.jpg`. Sustituir este fichero por otro de igual nombre y formato (jpeg), de 92 píxeles de alto. El ancho puede variar, aunque se recomienda que sea tan ancho como sea posible, hasta los 1920 px de una pantalla de alta definición. Para conseguir un mejor efecto junto con la bandera, se recomienda rellenar de contenido (logotipos, fotografía) la parte más a la derecha de la imagen, hasta un máximo de 500 px. Utilizar un color de fondo liso para el resto de la imagen, que ocupe toda la franja de la izquierda, y que se corresponda con el color de fondo de la bandera.

* **Bandera**: Se encuentra en `static/img/left.jpg`. Sustituir este fichero por otro de igual nombre y formato (jpeg), de 92 píxeles de alto. El ancho puede variar, aunque se recomienda alrededor de los 200 px. Utilizar un color de fondo liso, correspondiente con la parte izquierda de la imagen de fondo, para dar una sensación de continuidad.


## Feedback

La herramienta de feedback permite al usuario del portal proporcionar comentarios sobre determinadas regiones de las capas del mapa.

### Configuración

La creación y almacenamiento de estos comentarios requiere de 1) una tabla en la base de datos y 2) una cuenta de e-mail para el intercambio de mensajes entre el sistema, el usuario y el técnico que valida el comentario.

### Almacenamiento de los comentarios

Para almacenar los comentarios se utilizará una tabla llamada `redd_feedback` que debe existir en el esquema definido por la propiedad `db-schema` en el fichero `portal.properties`:

	CREATE TABLE redd_feedback (
		id serial,
		geometry geometry('GEOMETRY', 900913),
		comment varchar NOT NULL,
		layer_name varchar NOT NULL,
		layer_date varchar,
		date timestamp NOT NULL,
		email varchar NOT NULL,
		verification_code varchar,
		language varchar,
		state int
	);

Esta tabla alojará un registro por cada comentario hecho por el usuario. En ella podemos ver los siguientes campos:

* id: identificador de los comentarios
* geometry: geometría que identifica la región a la que afecta el comentario. Es dibujada por el usuario en el portal. En sistema de coordenadas EPSG:900913.
* comment: comentario realizado por el usuario
* layer_name: identificador de la `portalLayer` definida en el fichero layers.json que el usuario seleccionó para hacer el comentario.
* layer_date: instante para el cual el comentario se ha realizado. A tener en cuenta si la capa tiene varias instancias temporales.
* date: fecha en la que se realiza el comentario.
* email: email introducido por el usuario.
* verification_code: el código usado para verificar que la dirección de email proporcionada por el usuario está operativa. Ver :ref:`feedback_workflow`.
* language: el código del lenguaje seleccionado por el usuario en el momento de hacer el comentario. Por ejemplo "es" para español.
* state: estado en el flujo de trabajo. Uno de los siguientes:

	* 0 -> nuevo comentario. E-mail no confirmado.
	* 1 -> E-mail y comentario confirmado.
	* 2 -> Validado por un técnico.
	* 3 -> Validación notificada al usuario creador del comentario.

### Comunicación

Para la comunicación entre usuario y técnico es necesario primero configurar la cuenta de correo que se utilizará para realizar el intercambio. Esto se hace en el `portal.properties` mediante las siguientes propiedades:

* `feedback-mail-host`. Nombre del servidor de correo. Por ejemplo `smtp.gmail.com` si se está utilizando una cuenta de gmail.
* `feedback-mail-port`. Puerto donde escucha el servidor de correo. `587` en el caso de gmail.
* `feedback-mail-username`. Cuenta de correo. Por ejemplo: `onuredd@gmail.com`.
* `feedback-mail-password`. Contraseña de la cuenta de correo.

Esta cuenta de correo se utilizará para contactar con el usuario que crea un nuevo mensaje, pero además será donde se reciban las notificaciones de que un nuevo comentario ha sido creado y está a la espera de ser validado por un técnico. Este correo llevará como título y cuerpo los contenidos especificados en las dos propiedades siguientes:

* `feedback-admin-mail-title`. Título del correo. Por ejemplo `Hay un nuevo comentario en el portal REDD`.
* `feedback-admin-mail-text`. Texto del correo. Por ejemplo `Se ha introducido un comentario en el portal de REDD con c\u00f3digo de verificaci\u00f3n "$code`. Nótese que la variable `$code` se reemplazará por el identificador que se le haya asignado al comentario en el campo `id` de la tabla.

Por último, la notificación a los usuarios de que su comentario ha sido validado no se realiza de inmediato sino que, de forma periódica, el sistema comprueba qué comentarios han sido validados por un técnico pero no han sido notificados todavía. El tiempo de espera entre dos comprobaciones se expresa con la siguiente propiedad:
 
* `feedback-validation-check-delay`. Tiempo que el sistema espera antes de volver a releer la tabla de comentarios y notificar a los usuarios cuyo comentario ha sido validado por un técnico. Se expresa en milisegundos, por lo que un valor de 600000 correspondería a 10 minutos, valor recomendado.

Por último, todos los mensajes enviados al usuario se deben adaptar al idioma que habla el usuario por lo que para personalizarlos es necesario establecer una serie de propiedades en los ficheros de mensajes que se pueden encontrar en el directorio `messages` del directorio de configuración de la aplicación. Las propiedades con sus valores iniciales es español son:

	Feedback.all_parameters_mandatory=Todos los par\u00e1metros son obligatorios: 
	Feedback.error_sending_mail=Error enviando el correo: 
	Feedback.the_message_has_been_validated=El comentario ha sido validado.
	Feedback.comment_not_found=No se encontr\u00f3 ning\u00fan comentario con el c\u00f3digo
	Feedback.mail-title=Comentario en el portal ONU-REDD
	Feedback.verify-mail-text=Por favor, visite http://localhost:8080/unredd-portal/verify-comment?lang=$lang&verificationCode=$code para confirmar el envío.
	Feedback.validated-mail-text=El comentario con c\u00f3digo de verificaci\u00f3n "$code", ha sido validado y puede consultarse en el portal.
	Feedback.invalid-email-address=La direcci\u00f3n de correo especificada no es v\u00e1lida
	Feedback.no-geometries=Al menos se debe dibujar una geometr\u00eda
	Feedback.verify_mail_sent=Se ha enviado un mensaje a la direcci\u00f3n de correo especificada para confirmar el comentario.
	Feedback.submit_error=No se pudo realizar el env\u00edo.
	Feedback.no_layer_visible=Ninguna de las capas habilitadas para feedback est\u00e1 visible
	Feedback.no-layer-selected=No se ha seleccionado ninguna capa

### Flujo de trabajo de la herramienta Feedback


El flujo de trabajo habitual de la herramienta Feedback es el siguiente:

1. El usuario del portal abre el diálogo de feedback, dibuja una región e introduce un comentario y su dirección de e-mail y envía el formulario.
2. El sistema almacena el comentario en la tabla con `state` igual a 0 (nuevo). Se manda un correo al usuario con un enlace para confirmar el contenido del formulario y que la dirección de e-mail es correcta.
3. Cuando el usuario accede al enlace el sistema actualiza el campo `state` a 1 (confirmado) y envía un correo a la cuenta de correo configurada para que los técnicos sepan que hay un nuevo comentario en el sistema que espera a ser validado.
4. Un técnico forestal visualiza la entrada accediendo a la base de datos PostGIS, por ejemplo con QGIS, y puede opcionalmente marcar algunas entradas como validadas cambiando el valor de `state` a 2 (validado).
5. El sistema periódicamente comprueba los comentarios que hay en estado 2, validado pero no notificado al autor, y envía un mail automático indicándole que su comentario ha sido validado. Cuando consigue enviar este mensaje, actualiza el campo `state` a 3 (notificado), para no volver a procesarlo más.
6. El usuario recibe el mensaje indicándole que su entrada ha sido validada. Si se ha configurado en el portal una capa con los comentarios validados, el usuario podrá acceder al portal y ver su comentario allí.

### Recomendaciones

Una vez la funcionalidad de Feedback ha sido instalada y está en funcionamiento se recomienda:

- Configurar la cuenta de correo en el cliente de correo habitual del técnico responsable. De esta manera se evitan los olvidos y los mensajes de nuevos comentarios siempre encontrarán alguien que los lea.
- Crear una vista SQL que seleccione todos los comentarios con `state` igual a 2 y añadirla como capa en el portal. De esta manera, cuando el técnico valida una entrada, ésta se muestra automáticamente en el portal.

## Estadísticas


El sistema de estadísticas consta de dos partes. Por una parte, el servicio de estadísticas nos permite mostrar al usuario gráficas sobre objetos existentes en una capa del mapa. Por otra, el motor de cálculo nos permite generar los datos estadísticos de cobertura de forma automática.

### Servicio de estadísticas


Este servicio nos permitirá obtener gráficos sobre objetos del mapa. Los datos a presentar en dichos gráficos pueden presentar una o más variables con datos en uno o más instantes temporales.

Para utilizar el servicio es necesario partir de la tabla con los datos a graficar. Esta tabla contendrá los datos a graficar. Requerirá:

* un campo identificador del objeto sobre el que se pide la información, por ejemplo, código de provincia
* un campo con la fecha del valor
* tantos campos como valores tenga ese objeto en esa fecha: superficie de bosque, superficie deforestada, etc.

En el siguiente ejemplo tenemos un campo identificador `id` y un campo `fecha`. Para cada objeto tenemos tres fechas en las que se mide la superficie de bosque y la superficie deforestada:

	====  ===========  ================ ======
	id    sup_bosque   sup_deforestada  fecha 
	====  ===========  ================ ======
	1     400          100              2000 
	1     300          100              2005 
	1     200          100              2010 
	2     300          0                2000 
	2     350          100              2005 
	2     30           500              2010 
	====  ===========  ================ ======

Una vez se tiene la tabla con los datos hay que configurar las tablas de metadatos del servicio de estadísticas, que contiene información sobre el gráfico: título, unidades, tipo de gráfico; y permite enlazar al servicio con la tabla de datos anterior.

### Instalación

Para que el servicio de estadísticas funcione es necesario:

1. Crear un esquema en la base de datos. En dicho esquema se introducirán todas las tablas usadas por el portal tanto para estadísticas como para otras funcionalidades.
2. Crear las tablas de metadatos `redd_stats_charts` y `redd_stats_variables` que definirán el aspecto del gráfico y asociarán las capas del mapa con la tabla de datos.  


		CREATE TABLE redd_stats_charts (
			id serial PRIMARY KEY,
			title character varying,
			subtitle character varying,
			layer_name character varying,
			division_field_id character varying,
			division_field_name character varying,
			table_name_data character varying,
			data_table_id_field character varying,
			data_table_date_field character varying,
			data_table_date_field_format character varying,
			data_table_date_field_output_format character varying
		) WITH ( OIDS=FALSE );
		
		CREATE TABLE redd_stats_variables (
			id serial NOT NULL PRIMARY KEY,
			chart_id integer,
			y_label character varying,
			units character varying,
			tooltipsdecimals integer,
			variable_name character varying,
			data_table_variable_field character varying,
			graphic_type character varying,
			priority integer
		);
 
3. Configurar el esquema en el `portal.properties` existente en el directorio de configuración del portal:

		db-schema=mi_esquema

4. Tras este último paso es necesario reinicializar el portal. Esto se puede realizar mediante el comando `touch` aplicado al war. Este comando modifica la fecha y hora del fichero que se le pasa como parámetro, lo que fuerza a Tomcat a redesplegar el war y reinicializar la aplicación:

		$ touch /var/tomcat/webapps/portal.war

### Configuración de la tabla de metadatos

Cada fila de la tabla de `redd_stats_charts` especificará un gráfico para los objetos de una capa del portal:

- id: identificador, rellenado automáticamente.
- title: título del gráfico
- subtitle: subtítulo del gráfico
- layer_name: Nombre en GeoServer de la capa para la que se ofrecerá el gráfico, con la forma espaciodetrabajo:nombrecapa. Por ejemplo: bosques:provincias
- division_field_id: Nombre del campo que identifica los objetos en la capa anterior. Es posible utilizar un campo que no es único si se desea tener el mismo gráfico para más de un objeto.
- division_field_name: Nombre del campo con el nombre del objeto en la capa anterior. Se incluirá en el título del gráfico.
- table_name_data: Nombre de la tabla con los datos.
- data_table_id_field: Nombre del campo identificador en la tabla de datos `table_name_data`.
- data_table_date_field: Nombre del campo fecha en la tabla de datos `table_name_data`.
- data_table_date_field_format: Formato del campo fecha si no es de tipo `date` según la función `to_date` de [PostgreSQL](http://www.postgresql.org/docs/current/static/functions-formatting.html). NULL si el campo fecha es de tipo `date`.
- data_table_date_field_output_format: Formato de las fechas en el eje X de la gráfica  según la función `to_date` de [PostgreSQL](http://www.postgresql.org/docs/current/static/functions-formatting.html) . Si se deja a NULL se tomará DD-MM-YYYY (día-mes-año).

Una vez el registro con el gráfico ha sido creado es necesario especificar cómo se presentan los datos en el gráfico en la tabla `redd_stats_variables`:

- id: identificador, rellenado automáticamente.
- chart_id: identificador del gráfico al que pertenece esta variable. Deberá tener valor del campo `id` del gráfico.
- y_label: Magnitud medida, por ejemplo "Superficie"
- units: unidades en las que se mide la magnitud, por ejemplo "Hectáreas"
- tooltipsdecimals: Número de decimales que se presentarán cuando el usuario interactúa con la gráfica
- variable_name: Nombre de la variable que aparecerá en el gráfico, por ejemplo  "bosque cultivado".
- data_table_variable_field: Nombre del campo de la tabla de datos que contiene los valores de la variable anterior.
- graphic_type: Tipo de gráfico. Puede ser `cualquier valor aceptado por la librería highcharts <http://api.highcharts.com/highcharts#plotOptions>`_
- priority: Prioridad del eje. Para poder definir qué datos se presentan antes (p.ej.: líneas delante de gráfico de barras).

#### Caso práctico

En este ejemplo vamos a suponer que tenemos:

* Una tabla provincias con un campo `id_provincia` con tres provincias con identificador 1, 2 y 3.
* Una capa en GeoServer, publicando la tabla anterior con el nombre `provincias` en el espacio de trabajo `bosques`, es decir, con nombre `bosques:provincias`.
* La tabla convenientemente publicada en el portal, de manera es es posible mostrar el diálogo de información al pinchar en una de las provincias.

Es posible descargar los datos de ejemplo [aquí](../statistics/provincias.zip), para su carga en PostGIS y la realización del caso práctico con ellos.

Queremos publicar los siguientes datos de cobertura forestal:

	=================  ====== ====== ======
	Provincia 1         1990   2000   2005 
	=================  ====== ====== ======
	bosque nativo        100     98     78 
	bosque cultivado    1000   1100   1050 
	=================  ====== ====== ======

	=================  ====== ====== ======
	Provincia 2         1990   2000   2005 
	=================  ====== ====== ======
	bosque nativo        590     ND    208 
	bosque cultivado       0      0     50 
	=================  ====== ====== ======

	=================  ====== ====== ======
	Provincia 3         1990   2000   2005 
	=================  ====== ====== ======
	bosque nativo       2000   2300   2500 
	bosque cultivado       0    100     50 
	=================  ====== ====== ======

Lo primero será crear la tabla de datos con cualquer nombre significativo, por ejemplo `cobertura_forestal_provincias`. Suponemos que creamos todo en un esquema llamado estadísticas:

	CREATE TABLE estadisticas.cobertura_forestal_provincias(
		id_provinc varchar,
		sup_nativo varchar,
		sup_cultivado varchar,
		anio date
	);

Una vez la tabla está creada, es necesario introducir un registro por cada dato:

	INSERT INTO estadisticas.cobertura_forestal_provincias VALUES ('1', 100, 1000, '1/1/1990');
	INSERT INTO estadisticas.cobertura_forestal_provincias VALUES ('1', 98, 1100, '1/1/2000');
	INSERT INTO estadisticas.cobertura_forestal_provincias VALUES ('1', 78, 1050, '1/1/2005');

	INSERT INTO estadisticas.cobertura_forestal_provincias VALUES ('2', 590, 0, '1/1/1990');
	INSERT INTO estadisticas.cobertura_forestal_provincias VALUES ('2', null, 0, '1/1/2000');
	INSERT INTO estadisticas.cobertura_forestal_provincias VALUES ('2', 208, 50, '1/1/2005');

	INSERT INTO estadisticas.cobertura_forestal_provincias VALUES ('3', 2000, 0, '1/1/1990');
	INSERT INTO estadisticas.cobertura_forestal_provincias VALUES ('3', 2300, 100, '1/1/2000');
	INSERT INTO estadisticas.cobertura_forestal_provincias VALUES ('3', 2500, 50, '1/1/2005');

Por último crearemos el registro en la tabla de metadatos que enlaza estos datos con nuestra tabla de datos recién creada:

	INSERT INTO estadisticas.redd_stats_charts VALUES (
		DEFAULT, -- id generado automaticamente
		'Cobertura forestal', --title
		'Evolución de la cobertura forestal por provincia', --subtitle
		'bosques:provincias', --capa en geoserver
		'id_provinc', -- nombre del campo identificador de la capa
		'estadisticas.cobertura_forestal_provincias', -- nombre de la tabla de datos
		'id_provinc', -- nombre del campo id
		'anio', -- nombre del campo fecha
		NULL, -- campo fecha de tipo date
		'YYYY-MM-DD'
	);

	INSERT INTO estadisticas.redd_stats_variables VALUES (
		DEFAULT, -- id generado automaticamente
		(select currval('estadisticas.redd_stats_charts_id_seq')), --nos da el id del último INSERT en redd_stats_charts, es decir, nuestro gráfico
		'Cobertura', -- Nombre de la magnitud a medir
		'Hectáreas', -- Unidades de la magnitud a medir
		2, -- número de decimales a presentar
		'Bosque cultivado', -- Nombre de la variable
		'sup_nativo', --nombre del campo
		'line', --tipo de gráfico
		1 --prioridad
	);
	INSERT INTO estadisticas.redd_stats_variables VALUES (
		DEFAULT, -- id generado automaticamente
		(select currval('estadisticas.redd_stats_charts_id_seq')), --nos da el id del último INSERT en redd_stats_charts, es decir, nuestro gráfico
		'Cobertura', -- Nombre de la magnitud a medir
		'Hectáreas', -- Unidades de la magnitud a medir
		2, -- número de decimales a presentar
		'Bosque nativo', -- Nombre de la variable
		'sup_cultivado', --nombre del campo
		'column', --tipo de gráfico
		2 --prioridad
	);

Ahora, cuando el usuario pinche en una de las provincias:

1. el portal buscará en la tabla `estadisticas.redd_stats_charts` los registros que afectan a la capa `bosques:provincias` y encontrará el registro que acabamos de introducir.
2. el portal ofrecerá al usuario un botón para mostrar los datos de la tabla de datos asociada `estadisticas.cobertura_forestal_provincias`
3. el usuario pinchará en dicho botón
4. el portal leerá la tabla de variables, la tabla de datos y creará el gráfico que se ofrecerá al usuario

![](../_images/statistics.png)

## Motor de cálculo

El motor de cálculo son una serie de funciones PostgreSQL/PostGIS que permiten generar la tabla de datos que se presenta en los gráficos de forma automática, tomando como entrada:

* una tabla de polígonos sobre los cuales se quieren presentar las estadísticas, típicamente divisiones administrativas, con un campo identificador
* una tabla con la cobertura forestal en la que cada registro representa un area con la misma clasificación en un instante determinado.

Y produciendo:

* la tabla con los datos de cobertura en hectáreas para cada año y objeto existente en la primera capa.

### Instalación

El motor de cálculo puede [descargarse aquí](../statistics/redd_stats_calculator.sql). Para su instalación es necesario ejecutarlo en un intérprete de PostGIS, por ejemplo en línea de comandos:
redd_stats_calculator.sql

	$ psql -U spatial_user -d spatialdata -f redd_stats_calculator.sql

Esta ejecución instalará dos funciones, `redd_stats_calculo` y `redd_stats_run`. Esta última es la que se utilizará para iniciar el motor de cálculo.

Además de las funciones, el motor espera encontrar en el mismo esquema donde se encuentra la tabla de metadatos una tabla con las fajas en proyección EPSG:4326. Esta tabla deberá tener un campo `geom` con la geometria y un campo `srid` de tipo `integer` con el código SRID al que pertenece cada faja. Se puede ver un ejemplo en el caso práctico más abajo.

Una vez las funciones están instaladas y la tabla `redd_stats_fajas` está creada, podemos empezar a utilizarlo. Para hacerlo funcionar habrá que realizar dos pasos, 1) configurar la tabla de metadatos especificando esta vez TODOS los campos campos y 2) invocar al motor para que genere los gráficos.

### Configuración de la tabla de metadatos

Además de los campos especificados para el servicio, será necesario especificar:

* table_name_division: nombre de la tabla que se publica por GeoServer con el nombre especificado en el campo `layer_name`.
* class_table_name: nombre de la tabla que tiene la clasificación forestal, con los polígonos de todos los años indicando la clasificación y la fecha en la fecha en la que es válido el polígono.
* class_field_name: nombre del campo en la tabla anterior que indica el tipo de clasificación para cada registro.
* date_field_name: nombre del campo que indica la fecha en la que el polígono es válido.

### Invocación del motor para un gráfico determinado

El motor gráfico se invoca con la función `redd_stats_run`, que toma dos parámetros. El primero es el valor del campo `id` del registro de la tabla de metadatos cuyo gráfico queremos generar. El segundo es el esquema donde está esta tabla. La invocación se hace mediante una instrucción `SELECT`:

	SELECT redd_stats_run(1, 'estadisticas');
 
#### Caso práctico

En este caso se parte de

* Una tabla `provincias` con un campo `id_provincia` como identificador.
* Una capa en GeoServer, publicando la tabla anterior con el nombre `provincias` en el espacio de trabajo `bosques`, es decir, con nombre `bosques:provincias`.
* La tabla convenientemente publicada en el portal, de manera es es posible mostrar el diálogo de información al pinchar en una de las provincias.
* Una tabla `cobertura` con los polígonos de la clasificación forestal y los campos:

  * un campo `clasificac` indicando el tipo de la clasificación
  * un campo `fecha` indicando el año de esa clasificación

Es posible descargar los [datos de ejemplo aquí](../statistics/motor.zip), para su carga en PostGIS y la realización del caso práctico con ellos.

En este caso no creamos la tabla de datos, ya que ésta la creará el motor directamente, y pasamos directamente a añadir el registro en la tabla de metadatos:

	INSERT INTO estadisticas.redd_stats_metadata (
		title,
		subtitle,
		y_label,
		units,
		tooltipsdecimals,
		layer_name,
		table_name_division,
		division_field_id,
		class_table_name,
		class_field_name,
		date_field_name,
		table_name_data,
		graphic_type
	) VALUES (
		'Cobertura forestal',
		'Evolución de la cobertura forestal por provincia',
		'Cobertura',
		'Hectáreas',
		2,
		'bosques:provincias',
		'estadisticas.provincias',
		'id_provinc',
		'estadisticas.cobertura',
		'clasificac',
		'instante',
		'estadisticas.cobertura_forestal_provincias_automatica',
		'2D'
	);

Puede verse cómo en este caso se han especificado los parámetros `table_name_division`, `class_table_name`, `class_field_name`, `date_field_name`, que permitirán al motor acceder a los datos y generar la tabla automáticamente.

Para invocar el motor basta con ver el id asignado al registro recién insertado:

	spatialdata=> SELECT id, title, table_name_data FROM estadisticas.redd_stats_metadata;
	
	 id |       title        |                    table_name_data
	----+--------------------+-------------------------------------------------------
	  5 | Cobertura forestal | estadisticas.cobertura_forestal_provincias_automatica
	(1 row)

y ejecutar la función `redd_stats_run()` con el código que nos interesa y el nombre del esquema donde está la tabla redd_stats_metadata, es decir `estadisticas`:

	SELECT redd_stats_run(1, 'estadisticas');

Tras la ejecución, la tabla `estadisticas.cobertura_forestal_provincias_automatica` estará rellena con el resultado de los cálculos:

	patialdata=> select * from estadisticas.cobertura_forestal_provincias_automatica ;
	 division_id | variable  |   fecha    |    valor
	-------------+-----------+------------+-------------
	 1           | bosque    | 1999-01-01 | 6.37725e+07
	 1           | bosque    | 2004-01-01 | 5.27672e+07
	 1           | bosque    | 2010-01-01 | 8.30697e+07
	 1           | no bosque | 1999-01-01 |  1.8682e+07
	 1           | no bosque | 2004-01-01 | 2.97502e+07
	 1           | no bosque | 2010-01-01 |           0
	 2           | bosque    | 1999-01-01 |   4.982e+07
	 2           | bosque    | 2004-01-01 | 3.54705e+07
	 2           | bosque    | 2010-01-01 |           0
	 2           | no bosque | 1999-01-01 | 4.55279e+07
	 2           | no bosque | 2004-01-01 | 5.98773e+07
	 2           | no bosque | 2010-01-01 | 9.53479e+07
	 3           | bosque    | 1999-01-01 |  3.2107e+07
	 3           | bosque    | 2004-01-01 | 2.52069e+07
	 3           | bosque    | 2010-01-01 | 3.87162e+07
	 3           | no bosque | 1999-01-01 | 6.60003e+06
	 3           | no bosque | 2004-01-01 | 1.35093e+07
	 3           | no bosque | 2010-01-01 |           0
	(18 rows)
	
## Herramienta de nota al pie

El plugin "footnote" muestra un texto en la parte inferior del portal, que puede contener a su vez un enlace. El plugin soporta multiidioma.

Para incorporar este plugin a un portal, se añade como dependencia en el `pom.xml`, como cualquier otro plugin:

	<dependencies>
		...
		<dependency>
			<groupId>org.fao.unredd</groupId>
			<artifactId>footnote</artifactId>
			<version>...</version>
		</dependency>
		...
	</dependencies>


Para configurar el texto a mostrar, editar el fichero `GEOLADRIS_CONFIG_DIR/plugin-conf.json` y añadir la propiedad "footnote" bajo "default-conf":

	"footnote": {
	    "text": "Texto a pie de página",
	    "link": "http://snmb-desarrollo.readthedocs.org/",
	    "align": "center"
	}


Esta propiedad acepta tres atributos:

* "text": Obligatorio. El texto a mostrar. Para portales en un sólo idioma, puede ser el texto a mostrar. Para portales multiidioma, será una referencia a la clave en el fichero de traducción (`GEOLADRIS_CONFIG_DIR/messages/messages_*.properties`).
* "link": Opcional. Si se quiere que el texto sea un enlace, indicar aquí la página a la que enlazar.
* "align": Opcional. Indica la posición del texto: "left" (izquierda), "center" (centrado, por defecto) o "right" (derecha).

### Ejemplo de configuración multiidioma

Configurar la propiedad "text" con una clave (por ejemplo, "footnote.text"):

	"footnote": {
	    "text": "footnote.text"
	}


Añadir las traducciones usando dicha clave. Por ejemplo, en `GEOLADRIS_CONFIG_DIR/messages/messages_es.properties`:

	footnote.text=Esto es una nota a pie en espa\u00f1ol

O en `GEOLADRIS_CONFIG_DIR/messages/messages_en.properties`:

	footnote.text=This is an English footnote