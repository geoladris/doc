# Introducción

## ¿Qué es GeoLadris?

GeoLadris, es un framework para construir aplicaciones web modulares, basado en [RequireJS](http://requirejs.org/) y el patrón `eventbus` en el cliente, y en la especificación [Java Servlet 3.0](http://download.oracle.com/otndocs/jcp/servlet-3.0-fr-oth-JSpec/) en el servidor.

## La Arquitectura Cliente-Servidor
La arquitectura cliente-servidor es un modelo de aplicación distribuida en el que las tareas se reparten entre los proveedores de recursos o servicios, llamados servidores, y los demandantes, llamados clientes. Un cliente realiza peticiones a otro programa, el servidor, quien le da respuesta. Esta idea también se puede aplicar a programas que se ejecutan sobre una sola computadora, aunque es más ventajosa en un sistema operativo multiusuario distribuido a través de una red de computadoras. (Fuente: [Wikipedia](https://es.wikipedia.org/wiki/Cliente-servidor))

## Tecnologías usadas en GeoLadris

### Servidor
La parte servidora de *GeoLadris* está desarrollada mediante el uso de [Java](https://www.java.com). Java deberá encontrarse instalado en el servidor. La instalación  de Java puede realizarse siguiendo los pasos para el sistema operativo sobre el que se vaya a instalar ([¿Cómo puedo instalar Java?](https://www.java.com/es/download/help/download_options.xml)).

Una vez instalado Java podremos comprobar si esta se encuentra instalado mediante:

En entornos Linux:
```
$ java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```

En Windows:
```
C:\Users\username>java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```

GeoLadris necesitará de un contenedor de aplicaciones para su puesta en marcha como servidor. Para ello haremos uso de [Tomcat](http://tomcat.apache.org/).

Se deberá instalar Tomcat en la máquina servidora. Se realizará la instalación dependiendo del sistema operativo de la máquina siguiendo las instrucciones al respecto.

Una vez instalado Tomcat en la máquina comprobaremos que se encuentra corriendo en la misma accediendo desde un navegador a la URL [http://localhost:8080](http://localhost:8080), siendo está la dirección de instalación por defecto de Tomcat.

En esa URL deberemos ver la página de inicio de Tomcat:

![Página de inicio de Tomcat](_images/tomcat.png)

### Cliente
La parte cliente de GeoLadris hace uso de las tecnologías habituales del desarrollo de aplicaciones web. Para su uso será necesario disponer de cualquiera de las versiones modernas de los navegadores actuales, Google Chrome, Internet Explorer, Firefox o Safari. Una vez instalado y configurado podremos acceder a nuestras aplicaciones haciendo uso de cualquiera de estos navegadores.