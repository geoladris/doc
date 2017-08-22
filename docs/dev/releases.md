### Frecuencia de publicaciones

### Proceso

* Acordar una fecha por la lista de correo.
* Preparar la rama de release (principalmente copiado de [GeoServer](http://docs.geoserver.org/stable/en/developer/release-guide/index.html#if-you-are-cutting-the-first-rc-of-a-series-create-the-stable-branch)) y de **documentación**. Necesitaremos ramas en el repo de documentación para mantener documentación de diferentes versiones con readthedocs. Dos casos:

  - Cambios menores (únicamente cambia el patch number). No se hace nada, se utiliza la rama existente (queremos publicar la `7.0.1`, utilizamos la rama `7.0.x` existente).
  - Cambios mayores (cambia el minor y/o major number). Se crea una rama de release:

        :::
        git checkout <rama_base>
        git checkout -b <rama_release>


    `rama_release` se tiene que corresponder con el número de versión en el `pom.xml`. Por ejemplo, si en el `pom.xml` está `7.0.0-SNAPSHOT`, se crea la rama `7.0.x`. Para ello, se elige la `rama_base` correspondiente a la versión que se quiere publicar.

    Se actualiza `rama_base` con el siguiente minor number:

        :::
        git checkout rama_base
        mvn versions:set -DnewVersion=7.1.0-SNAPSHOT
        yarn version --no-git-tag-version --new-version 7.1.0-SNAPSHOT
        git commit -m "Bump version to 7.1.0-SNAPSHOT"
        git push


* Probar bien y hacer fixes de última hora sobre la rama de release.
* Actualizar versiones a la versión definitiva:

        :::
        git checkout rama_release
        mvn versions:set -DnewVersion=7.0.0
        yarn version --no-git-tag-version --new-version 7.0.0
        git commit -m "Bump version to 7.0.0"
        git push


* Desplegar

        :::
        mvn deploy -P release


* Actualizar documentación (versiones en changelog, descargas, etc.) y hacer un tag.

* Añadir una versión en readthedocs para el tag creado.

* Actualizar la versión en la rama de release:

        :::
        git checkout rama_release
        mvn versions:set -DnewVersion=7.1.0-SNAPSHOT
        yarn version --no-git-tag-version --new-version 7.1.0-SNAPSHOT
        git commit -m "Bump version to 7.1.0-SNAPSHOT"
        git push


* Enviar mensaje a la lista de correo.
