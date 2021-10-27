## ConfigMaps

ConfigMap is a map containing key/value pairs with the values ranging from short literals to full config files. The whole point of an app’s configuration is to keep the config options that vary between environments, or change frequently, separate from the application’s source code. 

<img src=".\images\p3_configmaps.jpg"/>

An application doesn’t need to read the ConfigMap directly or even know that it exists. The contents of the map are passed to containers as either environment variables or as files in a volume.

### Creating ConfigMaps

* By passing literals to the kubectl command

        kubectl create configmap myconfigmap --from-literal=foo=bar --from-literal=bar=baz --from-literal=one=two
* By defining in a yaml file
* From individual files or a directory

<img src=".\images\p3_configmaps_createfromdirectory.jpg"/>

## Secret

TODO