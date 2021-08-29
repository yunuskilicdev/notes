"Kustomize[1] is a standalone tool to customize Kubernetes objects through a kustomization file.
Since 1.14, Kubectl also supports the management of Kubernetes objects using a kustomization file. To view Resources found in a directory containing a kustomization file, run the following command"[0]:

````
kubectl kustomize <kustomization_directory>
````

To apply those Resources, run kubectl apply with --kustomize or -k flag:

````
kubectl apply -k <kustomization_directory>
````

**Kustomize is used for declarative object configuration.**

What are imperative and declarative object configurations?

# Imperative object configuration

In imperative object configuration, the kubectl command specifies the operation (create, replace, etc.), optional flags and at least one file name. The file specified must contain a full definition of the object in YAML or JSON format.[2]

## Examples

Create the objects defined in a configuration file:

````
kubectl create -f nginx.yaml
````

Delete the objects defined in two configuration files:
````
kubectl delete -f nginx.yaml -f redis.yaml
````

## Trade-offs

**Advantages compared to imperative commands:**

Object configuration can be stored in a source control system such as Git.

Object configuration can integrate with processes such as reviewing changes before push and audit trails.

Object configuration provides a template for creating new objects.

**Disadvantages compared to imperative commands:**

Object configuration requires basic understanding of the object schema.

Object configuration requires the additional step of writing a YAML file.

**Advantages compared to declarative object configuration:**

Imperative object configuration behavior is simpler and easier to understand.

As of Kubernetes version 1.5, imperative object configuration is more mature.

**Disadvantages compared to declarative object configuration:**

Imperative object configuration works best on files, not directories.

Updates to live objects must be reflected in configuration files, or they will be lost during the next replacement.

# Declarative object configuration

When using declarative object configuration, a user operates on object configuration files stored locally, however the user does not define the operations to be taken on the files. Create, update, and delete operations are automatically detected per-object by kubectl. This enables working on directories, where different operations might be needed for different objects.

## Trade-offs

**Advantages compared to imperative object configuration:**

Changes made directly to live objects are retained, even if they are not merged back into the configuration files.

Declarative object configuration has better support for operating on directories and automatically detecting operation types (create, patch, delete) per-object.

**Disadvantages compared to imperative object configuration:**

Declarative object configuration is harder to debug and understand results when they are unexpected.

Partial updates using diffs create complex merge and patch operations.

# Kustomize Usage

* Generating resources from other sources

* Setting cross-cutting fields for resources

* Composing and customizing collections of resources

## Generating Resources

"ConfigMaps and Secrets hold configuration or sensitive data that are used by other Kubernetes objects, such as Pods. The source of truth of ConfigMaps or Secrets are usually external to a cluster, such as a .properties file or an SSH keyfile. Kustomize has secretGenerator and configMapGenerator, which generate Secret and ConfigMap from files or literals.[0]"

### configMapGenerator

"To generate a ConfigMap from a file, add an entry to the files list in configMapGenerator[0]"

````
$ mkdir kustomizeDemo
$ cd kustomizeDemo
$ echo "ENV=DEMO" > demo.yaml
$ cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: demo-configmap
  files:
- demo.yaml
  EOF
  $ kubectl kustomize ./
  apiVersion: v1
  data:
  demo.yaml: |
  ENV=DEMO
  kind: ConfigMap
  metadata:
  name: demo-configmap-47f78c8429
  Other approaches:
  configMapGenerator:
- name: example-configmap-1
  envs:
    - .env
      EOF
      cat <<EOF >./kustomization.yaml
      configMapGenerator:
- name: example-configmap-2
  literals:
    - FOO=Bar
      EOF
````

This is an example deployment that uses a generated ConfigMap:

````
# Create a application.properties file
cat <<EOF >application.properties
FOO=Bar
EOF

cat <<EOF >deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app
        volumeMounts:
        - name: config
          mountPath: /config
      volumes:
      - name: config
        configMap:
          name: example-configmap-1
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
configMapGenerator:
- name: example-configmap-1
  files:
  - application.properties
EOF
````      
Kustomize generates ConfigMap and ConfigMap is mapped to a volume.

secretGenerator can be found at [0].

### Setting cross-cutting fields

It is quite common to set cross-cutting fields for all Kubernetes resources in a project. Some use cases for setting cross-cutting fields:
      
setting the same namespace for all Resources
      
adding the same name prefix or suffix
      
adding the same set of labels
      
adding the same set of annotations

Here is an example:

````
# Create a deployment.yaml
cat <<EOF >./deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
EOF

cat <<EOF >./kustomization.yaml
namespace: my-namespace
namePrefix: dev-
nameSuffix: "-001"
commonLabels:
  app: bingo
commonAnnotations:
  oncallPager: 800-555-1212
resources:
- deployment.yaml
EOF
````

### Composing and Customizing Resources

#### Composing

Kustomize supports composition of different resources. The resources field, in the kustomization.yaml file, defines the list of resources to include in a configuration. Set the path to a resource's configuration file in the resources list. Here is an example of an NGINX application comprised of a Deployment and a Service:

````
# Create a deployment.yaml file
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# Create a service.yaml file
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

# Create a kustomization.yaml composing them
cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
- service.yaml
EOF
````

The Resources from kubectl kustomize ./ contain both the Deployment and the Service objects.

#### Customizing

Patches can be used to apply different customizations to Resources. Kustomize supports different patching mechanisms through patchesStrategicMerge and patchesJson6902. patchesStrategicMerge is a list of file paths. Each file should be resolved to a strategic merge patch. The names inside the patches must match Resource names that are already loaded. Small patches that do one thing are recommended. For example, create one patch for increasing the deployment replica number and another patch for setting the memory limit.

````
# Create a deployment.yaml file
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# Create a patch increase_replicas.yaml
cat <<EOF > increase_replicas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
EOF

# Create another patch set_memory.yaml
cat <<EOF > set_memory.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  template:
    spec:
      containers:
      - name: my-nginx
        resources:
          limits:
            memory: 512Mi
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
patchesStrategicMerge:
- increase_replicas.yaml
- set_memory.yaml
EOF
````

Run kubectl kustomize ./ to view the Deployment:

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: nginx
        name: my-nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: 512Mi
````

0-) https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/

1-) https://github.com/kubernetes-sigs/kustomize

2-) https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/