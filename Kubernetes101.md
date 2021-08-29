“Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.[0]”

You need to know the below terms at least.

###### Pods

The smallest unit that can be deployed in Kubernetes.

In reality, pods are one or more containers working together to service a part of your system.
Each pod has a unique IP address.

Containers in a pod can talk to each other via localhost.

But mostly each pod contains one container.

“Note: Grouping multiple co-located and co-managed containers in a single Pod is a relatively advanced use case. You should use this pattern only in specific instances in which your containers are tightly coupled”[3].

###### Replica Set

“A ReplicaSet’s purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.”

Introduces state management

###### Deployments

Level of abstraction above ReplicaSets

Deployments create and update ReplicaSets

###### ConfigMaps

Used to override container-specific data like

Configuration files

Environment variables

Entire directories of data

“Apps sometimes store config as constants in the code. This is a violation of twelve-factor, which requires strict separation of config from code. Config varies substantially across deploys, code does not.”[5] That’s why ConfigMap is being used to handle this case.

Automatically updated within the container when changed

The best practice is to version ConfigMaps and perform a rolling update

“ConfigMaps can be mounted as data volumes. ConfigMaps can also be used by other parts of the system, without being directly exposed to the Pod. For example, ConfigMaps can hold data that other parts of the system should use for configuration.”[4]
Using ConfigMaps as files from a Pod

To consume a ConfigMap in a volume in a Pod:

-Create a ConfigMap or use an existing one. Multiple Pods can reference the same ConfigMap.

-Modify your Pod definition to add a volume under .spec.volumes[]. Name the volume anything, and have a .spec.volumes[].configMap.name field set to reference your ConfigMap object.

-Add a .spec.containers[].volumeMounts[] to each container that needs the ConfigMap. Specify .spec.containers[].volumeMounts[].readOnly = true and .spec.containers[].volumeMounts[].mountPath to an unused directory name where you would like the ConfigMap to appear.

-Modify your image or command line so that the program looks for files in that directory. Each key in the ConfigMap data map becomes the filename under mountPath.

This is an example of a Pod that mounts a ConfigMap in a volume:

````
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
  configMap:
  name: myconfigmap
````

“When a ConfigMap currently consumed in a volume is updated, projected keys are eventually updated as well. The kubelet checks whether the mounted ConfigMap is fresh on every periodic sync. However, the kubelet uses its local cache for getting the current value of the ConfigMap. The type of the cache is configurable using the ConfigMapAndSecretChangeDetectionStrategy field in the KubeletConfiguration struct. A ConfigMap can be either propagated by watch (default), ttl-based, or by redirecting all requests directly to the API server. As a result, the total delay from the moment when the ConfigMap is updated to the moment when new keys are projected to the Pod can be as long as the kubelet sync period + cache propagation delay, where the cache propagation delay depends on the chosen cache type (it equals to watch propagation delay, ttl of cache, or zero correspondingly).”[4]

###### Services

Defines a DNS entry that can be used to refer to a group of pods

Provide a consistent endpoint for the group of pods

Different types ClusterIP, NodePort, LoadBalancer

**ClusterIP**: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.
  
**NodePort**: Exposes the Service on each Node's IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You'll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.
  
**LoadBalancer**: Exposes the Service externally using a cloud provider's load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.

###### Ingresses

Define how traffic outside the cluster is routed to inside the cluster.

Used to expose Kubernetes services to the world

Route traffic to internal services based on factors such as host and path

Ingress is usually implemented by a load balancer(Nginx, HAProxy, etc)

Like a Layer 7 Load balancer.

Ingresses were developed to decrease the load balancer cost in the cloud. If we use a load balancer for each service(Services type -> LoadBalancer), it costs very much. That’s why we are using ingresses.
  
0-) https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/
  
1-) https://www.youtube.com/watch?v=5h1TCrh_hZ0
  
2-) https://www.cncf.io/blog/2019/12/16/kubernetes-101/
  
3-) https://kubernetes.io/docs/concepts/workloads/pods/
  
4-) https://kubernetes.io/docs/concepts/configuration/configmap/
  
5-) https://12factor.net/config