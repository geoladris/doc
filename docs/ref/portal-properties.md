## portal.properties

Algunas de las propiedades que podremos definir desde el archivo `portal.properties` son:

| Propiedad | Descripción | Valor por defecto | Ejemplo de configuración |
| -- | -- | -- | -- |
 | languages | Elemento JSON con los idiomas que soporta la aplicación | `{“en”: “English”, “fr”: “Franu00e7ais”, “es”: “Espau00f1ol”}`  | `{“en”: “English”, “fr”: “Franu00e7ais”, “es”: “Espau00f1ol”}` |
 | languages.default | Lenguaje por defecto de la aplicación | `en` | `languages.default = en` |
 
 * **TODO** **Título de la aplicación**: Se encuentra definido en los ficheros de mensajes, directorio `messages`, ficheros de nombre `messages_<lang>.properties`. Buscar la propiedad "title" en cada uno de los ficheros de idioma.