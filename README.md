# Docker EE 2.1 & Apache Spark

#### Security in Spark is OFF by default. This could mean you are vulnerable to attack by default. Please see Spark Security and the specific advice below before running Spark.  https://spark.apache.org/docs/latest/security.html

##### User Identity
Images built from the project provided Dockerfiles do not contain any USER directives. This means that the resulting images will be running the Spark processes as root inside the container. On unsecured clusters this may provide an attack vector for privilege escalation and container breakout. Therefore security conscious deployments should consider providing custom images with USER directives specifying an unprivileged UID and GID.

Alternatively the Pod Template feature can be used to add a Security Context with a runAsUser to the pods that Spark submits. Please bear in mind that this requires cooperation from your users and as such may not be a suitable solution for shared environments. Cluster administrators should use Pod Security Policies if they wish to limit the users that pods may run as.

##### Volume Mounts
As described later in this document under Using Kubernetes Volumes Spark on K8S provides configuration options that allow for mounting certain volume types into the driver and executor pods. In particular it allows for hostPath volumes which as described in the Kubernetes documentation have known security vulnerabilities.

Cluster administrators should use Pod Security Policies to limit the ability to mount hostPath volumes appropriately for their environments.

> podSecurityPolicy available in k8s API 1.14beta


##### Prerequisites
- A runnable distribution of Spark 2.3 or above.
- A running Kubernetes cluster at version >= 1.6 with access configured to it using kubectl. recommend 3 CPUs and 4g of memory to be able to start a simple Spark application with a single executor.
- You must have appropriate permissions to list, create, edit and delete pods in your cluster. You can verify that you can list these resources by running kubectl auth can-i <list|create|edit|delete> pods.
- The service account credentials used by the driver pods must be allowed to create pods, services and configmaps.
- You must have Kubernetes DNS configured in your cluster.


#### Cluster Mode
To launch Spark Pi in cluster mode,

```
$ bin/spark-submit \
    --master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=5 \
    --conf spark.kubernetes.container.image=<spark-image> \
    local:///path/to/examples.jar
```

The Spark master, specified either via passing the --master command line argument to spark-submit or by setting spark.master in the application’s configuration, must be a URL with the format k8s://<api_server_url>. Prefixing the master string with k8s:// will cause the Spark application to launch on the Kubernetes cluster, with the API server being contacted at api_server_url. If no HTTP protocol is specified in the URL, it defaults to https. For example, setting the master to k8s://example.com:443 is equivalent to setting it to k8s://https://example.com:443, but to connect without TLS on a different port, the master would be set to k8s://http://example.com:8080.

In Kubernetes mode, the Spark application name that is specified by spark.app.name or the --name argument to spark-submit is used by default to name the Kubernetes resources created like drivers and executors. So, application names must consist of lower case alphanumeric characters, -, and . and must start and end with an alphanumeric character.

If you have a Kubernetes cluster setup, one way to discover the apiserver URL is by executing kubectl cluster-info.

```
$ kubectl cluster-info
Kubernetes master is running at http://127.0.0.1:6443
```

In the above example, the specific Kubernetes cluster can be used with spark-submit by specifying --master k8s://http://127.0.0.1:6443 as an argument to spark-submit. Additionally, it is also possible to use the authenticating proxy, kubectl proxy to communicate to the Kubernetes API.

The local proxy can be started by: `$ kubectl proxy`

If the local proxy is running at localhost:8001, --master k8s://http://127.0.0.1:8001 can be used as the argument to spark-submit. Finally, notice that in the above example we specify a jar with a specific URI with a scheme of local://. This URI is the location of the example jar that is already in the Docker image.

##### Deploy



#### Source repositories

https://github.com/kubernetes/examples/tree/master/staging/spark

https://github.com/aseigneurin/spark-ui-proxy

https://github.com/apache/spark
