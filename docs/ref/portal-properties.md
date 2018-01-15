## portal.properties

Algunas de las propiedades que podremos definir desde el archivo `portal.properties` son:

| Propiedad | Descripción | Valor por defecto | Ejemplo de configuración |
| -- | -- | -- | -- |
 | languages | Elemento JSON con los idiomas que soporta la aplicación | `{“en”: “English”, “fr”: “Franu00e7ais”, “es”: “Espau00f1ol”}`  | `{“en”: “English”, “fr”: “Franu00e7ais”, “es”: “Espau00f1ol”}` |
 | languages.default | Lenguaje por defecto de la aplicación | `en` | `languages.default = en` |
 | db-schema | Nombre del esquema donde están las tablas necesarias para las distintas funcionalidades como estadísticas y feedback | `db-schema=integration_tests` | `db-schema=integration_tests` |
| map.centerLonLat | Coordenadas geográficas donde se posicionará el mapa inicialmente | `-58,-34` | `map.centerLonLat=-58,-34` |
| map.initialZoomLevel | Nivel inicial de zoom del mapa | `5` | `map.initialZoomLevel=5` |
| auth.roles | Nombre del grupo de roles (Mas info [+](../user/config/#autenticacion-y-configuracion-especifica-de-usuario)) | `viewer` | auth.roles=viewer |

Por defecto ya vienen definidos los valores de configuración para el plugin de [Feedback]()

| Propiedad | Descripción | Valor por defecto | Ejemplo de configuración |
| -- | -- | -- | -- |
| feedback-mail-host | Servidor desde el que se enviarán los correos | `smtp.gmail.com` | `feedback-mail-host=smtp.gmail.com` |
| feedback-mail-port | Puerto | `587` | |
| feedback-mail-username| Dirección de correo desde la que se enviarán |  `fake@mail.invalid` | |
| feedback-mail-password | Contraseña del correo definido | `a_fake_pass` | |
| feedback-admin-mail-title | Título del correo | `Nuevo comentario en el portal REDD` | |
| feedback-admin-mail-text | Texto que irá en el cuerpo del correo | `Se ha introducido un comentario en el portal de REDD con c\u00f3digo de verificaci\u00f3n "$code" ` | |
| feedback-validation-check-delay | Tiempo máximo de espera para la respuesta del servidor de correo | `600000` | |