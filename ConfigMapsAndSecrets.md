# Introduction to Kubernetes

Editors: **Kaan Keskin, Sezen Erdem**

Date: November 2021

Available at: https://github.com/kaan-keskin/introduction-to-kubernetes

**Resources:**

> - Kubernetes Documentation - https://kubernetes.io/docs/home/
> - Kubernetes in Action - Marko Lukša 
> - Kubernetes Fundamentals (LFS258) - The Linux Foundation
> - Kubernetes for Developers (LFD259) - The Linux Foundation
> - Getting Started with Kubernetes - Sander van Vugt - Addison-Wesley Professional

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial purposes. If you find this document useful in any means please support the original authors for ethical reasons.** 

[Return to the README page.](README.md)

# ConfigMaps and Secrets

## ConfigMaps

A ConfigMap is an API object used to store non-confidential data in key-value pairs. It is a map containing key/value pairs with the values ranging from short literals to full config files. A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.
The whole point of an app’s configuration is to keep the config options that vary between environments, or change frequently, separate from the application’s source code. 

An application doesn’t need to read the ConfigMap directly or even know that it exists. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

<img src=".\images\p3_configmaps.jpg"/>

### Portable Data with ConfigMaps

A similar API resource to Secrets is the ConfigMap, except the data is not encoded. In keeping with the concept of decoupling in Kubernetes, using a ConfigMap decouples a container image from configuration artifacts. 

They store data as sets of key-value pairs or plain configuration files in any format. The data can come from a collection of files or all files in a directory. It can also be populated from a literal value. 

A ConfigMap can be used in several different ways. A container can use the data as environmental variables from one or more sources. The values contained inside can be passed to commands inside the pod. A Volume or a file in a Volume can be created, including different names and particular access modes. In addition, cluster components like controllers can use the data. ​

Let's say you have a file on your local filesystem called **config.js**. You can create a ConfigMap that contains this file. The **configmap** object will have a data section containing the content of the file:

```shell
$ kubectl get configmap foobar -o yaml 

kind: ConfigMap 
apiVersion: v1 
metadata: 
    name: foobar 
data: 
    config.js: | 
         { 
...
```

ConfigMaps can be consumed in various ways:
- Pod environmental variables from single or multiple ConfigMaps
- Use ConfigMap values in Pod commands
- Populate Volume from ConfigMap
- Add ConfigMap data to specific path in Volume
- Set file names and access mode in Volume from ConfigMap data
- Can be used by system components and controllers.

### Creating ConfigMaps

* By passing literals to the kubectl command

```shell
kubectl create configmap myconfigmap --from-literal=foo=bar --from-literal=bar=baz --from-literal=one=two

```
* By defining in a yaml file

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true    
```
* From individual files or a directory

<img src=".\images\p3_configmaps_createfromdirectory.jpg"/>

### Using ConfigMaps

You can use ConfigMaps as environment variables or using a volume mount. They must exist prior to being used by a Pod, unless marked as optional. They also reside in a specific namespace.

In the case of environment variables, your pod manifest will use the valueFrom key and the configMapKeyRef value to read the values. 

For instance:

```yaml
env: 
- name: SPECIAL_LEVEL_KEY 
  valueFrom: 
    configMapKeyRef: 
      name: special-config 
      key: special.how
```

With volumes, you define a volume with the configMap type in your pod and mount it where it needs to be used.

```yaml
volumes: 
    - name: config-volume 
      configMap: 
        name: special-config
```

There are four different ways that you can use a ConfigMap to configure a container inside a Pod:

1. Inside a container command and args
2. Environment variables for a container
3. Add a file in read-only volume, for the application to read
4. Write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap

These different methods lend themselves to different ways of modeling the data being consumed. For the first three methods, the kubelet uses the data from the ConfigMap when it launches container(s) for a Pod. The fourth method means you have to write code to read the ConfigMap and its data.

Here's an example Pod that uses values from game-demo to configure a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # Define the environment variable
        - name: PLAYER_INITIAL_LIVES # Notice that the case is different here
                                     # from the key name in the ConfigMap.
          valueFrom:
            configMapKeyRef:
              name: game-demo           # The ConfigMap this value comes from.
              key: player_initial_lives # The key to fetch.
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
    # You set volumes at the Pod level, then mount them into containers inside that Pod
    - name: config
      configMap:
        # Provide the name of the ConfigMap you want to mount.
        name: game-demo
        # An array of keys from the ConfigMap to create as files
        items:
        - key: "game.properties"
          path: "game.properties"
        - key: "user-interface.properties"
          path: "user-interface.properties"
```

## Secret

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Using a Secret means that you don't need to include confidential data in your application code. Secrets are similar to ConfigMaps but are specifically intended to hold confidential data.

A Secret can be used with a Pod in three ways:

1. As files in a volume mounted on one or more of its containers.
2. As container environment variable.
3. By the kubelet when pulling images for the Pod.

Pods can access local data using volumes, but there is some data you don't want readable to the naked eye. Passwords may be an example. Using the Secret API resource, the same password could be encoded or encrypted.

You can create, get, or delete secrets:

```shell
$ kubectl get secrets 
```

Secrets can be encoded manually or via '**kubectl create secret**':​

```shell
$ kubectl create secret generic --help

$ kubectl create secret generic mysql --from-literal=password=root
```

A secret is not encrypted, only base64-encoded, by default. You must create an **EncryptionConfiguration** with a key and proper identity. Then, the kube-apiserver needs the **--encryption-provider-config** flag set to a previously configured provider, such as **aescbc** or **ksm**. Once this is enabled, you need to recreate every secret, as they are encrypted upon write. 

Multiple keys are possible. Each key for a provider is tried during decryption. The first key of the first provider is used for encryption. To rotate keys, first create a new key, restart (all) kube-apiserver processes, then recreate every secret. 

You can see the encoded string inside the secret with **kubectl**. The secret will be decoded and be presented as a string saved to a file. The file can be used as an environmental variable or in a new directory, similar to the presentation of a volume.

A secret can be made manually as well, then inserted into a YAML file:

```shell
$ echo LFTr@1n | base64

TEZUckAxbgo= 

$ vim secret.yaml

apiVersion: v1
kind: Secret
metadata: 
  name: lf-secret
data: 
  password: TEZUckAxbgo=
```

### Creating Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  USER_NAME: YWRtaW4=
  PASSWORD: MWYyZDFlMmU2N2Rm
```
```shell
kubectl apply -f mysecret.yaml
```
### Using Secrets

Secrets can be mounted as data volumes or exposed as environment variables to be used by a container in a Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

### Using Secrets via Environment Variables

A secret can be used as an environmental variable in a Pod. You can see one being configured in the following example: 

```yaml
... 
spec: 
    containers: 
    - image: mysql:5.5
      name: dbpod
      env: 
      - name: MYSQL_ROOT_PASSWORD 
        valueFrom: 
            secretKeyRef: 
                name: mysql 
                key: password 
```

There is no limit to the number of Secrets used, but there is a 1MB limit to their size. Each secret occupies memory, along with other API objects, so very large numbers of secrets could deplete memory on a host.

They are stored in the **tmpfs** storage on the host node, and are only sent to the host running Pod. All volumes requested by a Pod must be mounted before the containers within the Pod are started. So, a secret must exist prior to being requested.

### Mounting Secrets as Volumes

You can also mount secrets as files using a volume definition in a pod manifest. The mount path will contain a file whose name will be the key of the secret created with the **kubectl create secret** step earlier.

```yaml
... 
spec: 
    containers: 
    - image: busybox 
      command: 
        - sleep 
        - "3600" 
      volumeMounts: 
      - mountPath: /mysqlpassword 
        name: mysql 
      name: busy 
    volumes: 
    - name: mysql 
        secret: 
            secretName: mysql
```

Once the pod is running, you can verify that the secret is indeed accessible in the container:

```shell
$ kubectl exec -ti busybox -- cat /mysqlpassword/password

LFTr@1n
```
