* Cómo configurar el workspace de Eclipse.

* Cosas útiles, como filtrar los recursos node* para los plugins:

```bash
for i in `find geoladris/plugins -name .project`; do
  sed -i "s#</projectDescription>#<filteredResources><filter><id>`date +%s0000`</id><name></name><type>10</type><matcher><id>org.eclipse.ui.ide.multiFilter</id><arguments>1.0-name-matches-false-false-node*</arguments></matcher></filter></filteredResources></projectDescription>#g" $i;
done
```
