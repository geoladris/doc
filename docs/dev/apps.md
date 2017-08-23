Existe un [script](../user/create.md) para la generación de aplicaciones de manera sencilla. Esta documentación es útil para crear aplicaciones con necesidades específicas que el script no cubre o para entender el funcionamiento de las aplicaciones del [repositorio](https://github.com/geoladris/apps) de Geoladris.

## Estructura de una aplicación

Una aplicación Geoladris contiene los siguientes ficheros/directorios:

* `package.json`: Fichero de configuración de yarn. Aquí se deben incluir los plugins [cliente](plugins.md#cliente) como dependencias:

        :::json
        "dependencies": {
          "@csgeoladris/ui": "csgis/geoladris-ui#master",
          "@geoladris/core": "geoladris/core#master",
          "@geoladris/base": "file:../../plugins/base",
          ...
        }

    Además se debe incluir un script para el empaquetado de recursos cliente:

        :::json
        "scripts": {
          "build": "gl-build-app.js && r.js -o .requirejs-build.js"
        }

* `pom.xml`: Fichero de configuración de Maven. Aquí se deben incluir los plugins [servidor](plugins.md#servidor) como dependencias:

        :::xml
        <dependencies>
          ...
          <dependency>
            <groupId>org.fao.unredd</groupId>
            <artifactId>layers-editor</artifactId>
            <version>${plugins.version}</version>
          </dependency>
          ...
        </dependencies>

    También se debe incluir el plugin [frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin) para ejecutar el script de `build` de `yarn`:

        :::xml
        <build>
            <plugins>
                <plugin>
                    <groupId>com.github.eirslett</groupId>
                    <artifactId>frontend-maven-plugin</artifactId>
                    <version>1.4</version>
                    <executions>
                        <execution>
                            <id>install-node-and-yarn</id>
                            <goals>
                              <goal>install-node-and-yarn</goal>
                            </goals>
                            <configuration>
                                <nodeVersion>${node.version}</nodeVersion>
                                <yarnVersion>${yarn.version}</yarnVersion>
                            </configuration>
                        </execution>
                        <execution>
                            <id>yarn install</id>
                            <goals>
                              <goal>yarn</goal>
                            </goals>
                            <configuration>
                              <arguments>install</arguments>
                            </configuration>
                        </execution>
                        <execution>
                            <id>yarn-build</id>
                            <goals>
                                <goal>yarn</goal>
                            </goals>
                            <phase>prepare-package</phase>
                            <configuration>
                              <arguments>run build</arguments>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>


* `yarn.lock`: [Siempre](https://yarnpkg.com/blog/2016/11/24/lockfiles-for-all/) se deben incluir estos ficheros en el repositorio.
* `src/main/webapp/META-INF/context.xml`: Fichero de configuración de [Tomcat](https://tomcat.apache.org/tomcat-8.5-doc/config/context.html#Defining_a_context).
* `src/main/webapp/WEB-INF/web.xml`: [Descriptor](https://cloud.google.com/appengine/docs/standard/java/config/webxml) de aplicación.
* `src/main/webapp/WEB-INF/default_config`: [Directorio de configuración](../user/config.md) por defecto. Opcional.

## Empaquetar

El `core` de Geoladris se encarga de empaquetar todos los recursos tanto Java (servlets, filtros, listeners,...) como JavaScript (copiar módulos, estilos, dependencias, etc. a `src/main/webapp` para empaquetarlos dentro del war y servirlos con Tomcat) y minificar los recursos estáticos.

Los **plugins JavaScript** se gestionan con herramientas de JavaScript. El flujo de trabajo es:

* Incluir plugins en `dependencies` dentro del `package.json`.
* `yarn install` para gestionar todas las dependencias.
* `gl-build-app.js`. Es un script propio de `core` que:
    * Copia los plugins definidos en la opción `dependencies` del `package.json` dentro de `src/main/webapp`; también copia las dependencias (`*.js` y `*.css`).
    * Genera el fichero `app.min.css` con  todos los estilos (de menos a más prioridad: dependencias, `src` y `css`).
    * Genera el fichero `main.js` de RequireJS que se sirve en la aplicación final (solo con `debug=true`).
    * Genera el fichero `index.html`.
    * Genera un fichero `.requirejs-build.js` para minificar módulos RequireJS, teniendo en cuenta la configuración de `requirejs` en los `geoladris.json` de los plugins.
* `r.js -o build.js`. Genera `src/main/webapp/app.min.js` con la minificación de los recursos JavaScript a partir del `.requirejs-build.js` generado por `gl-build-app.js`.

Los **plugins Java** se gestionan simplemente incluyéndolos como dependencias en el `pom.xml`.

El build de plugins JavaScript se incluye como parte del [build](https://github.com/geoladris/apps/blob/js_deps/demo/pom.xml#L119) de Maven con un plugin. Por tanto, basta con hacer `mvn package` para empaquetarlo todo.

## Servir

Una vez desplegada la aplicación `.war` en Tomcat, el `core` de Geoladris también se encarga de servir todos los recursos estáticos (plugins empaquetados en el war, plugins en el directorio de configuración, ficheros estáticos en el directorio de configuración,...) y dinámicos (`config.js`, ...). Son los siguientes:

Recursos estáticos gestionados directamente por Tomcat (por estar en `src/main/webapp`):

* `src/main/webapp/app.min.js`
* `src/main/webapp/app.min.css`
* `src/main/webapp/index.html`
* `src/main/webapp/*.js`
* `src/main/webapp/geoladris/*`

Recursos estáticos fuera del `war`. Configurado automáticamente para Tomcat 8.x y 9.x:

* `<config_dir>/plugins/`. Bajo el path `/plugins/*`.
* `<config_dir>/static/`. Bajo el path `/static/*`. Inhabilita `src/main/webapp/static`.

`config.js` (con un servlet), que es dinámico porque puede depender, por ejemplo, del usuario.
