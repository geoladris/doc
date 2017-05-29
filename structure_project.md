# Estructura de GeoLadris

A continuación se detalla la estructura del proyecto.

Si uno revisa el repositorio de código de [GeoLadris](https://github.com/geoladris) encontrará 4 repositorios:

* [doc](https://github.com/geoladris/doc), el repositorio donde se encuentra la documentación oficial del proyecto
* [apps](https://github.com/geoladris/apps)
* [core](https://github.com/geoladris/core)
* [plugins](https://github.com/geoladris/plugins)

## core
En este repositorio se encuentra el `core` de GeoLadris. Es la parte de la funcionalidad que se encargará de poner a nuestra disposición el `message-bus` y las diferentes funcionalidades que nos permiten trabajar con GeoLadris. De cara al usuario final, sería necesario instalar `core` con alguno de los plugins para poder obtener una interfaz con la que dicho usuario pueda interactuar ya que `core` **SOLO nos aporta funcionalidad a nivel de arquitectura**.

## plugins
Toda la funcionalidad que se implementa en GeoLadris será desarrollada como plugins que incrementan las capacidades del sistema. En este respositorio se encuentran todos los plugins desarrollados de manera oficial dentro del proyecto. Para crear sus propios plugins, revise la [documentación de desarrollo]()

## apps
Las aplicaciones dentro de GeoLadris hacen referencia a la configuración de diferentes plugins dando lugar a un grupo de funcionalidades que formarian la app.
Dentro de este repositorio encontrará una aplicación de [demostración](https://github.com/geoladris/apps/tree/master/demo) así como algunos ejemplos.
