# Getting starrted with the official Apache Airflow Helm chart

## Introducion

Let's follow instructions from the webinar *Getting Started With the Official Airflow Helm Chart*: https://www.youtube.com/watch?v=39k2Sz9jZ2c

Then, we can follow the official documentation of the Apache Airflow Helm Chart:
* [Quick start with kind](https://airflow.apache.org/docs/helm-chart/stable/quick-start.html)
* [Manage DAGs files](https://airflow.apache.org/docs/helm-chart/stable/manage-dags-files.html)

Currently there are at least 3 helm charts:

* [Bitnami Helm chart](https://artifacthub.io/packages/helm/bitnami/airflow)

Very complete, with plenty of versions, all the executors, it allows you to [load DAG files](https://artifacthub.io/packages/helm/bitnami/airflow#load-dag-files) from an existing config map or from a git repository.

I've found no easy way to add your own python modules or extra dependencies.

*  [Community Helm Chart](https://artifacthub.io/packages/helm/airflow-helm/airflow)

Again, several versions, but based on Helm 2, it also allows you to [add DAGs](https://artifacthub.io/packages/helm/airflow-helm/airflow#how-to-store-dags) with a git-sync sidecar, in a Kubernetes PersistentVolume, or by providing your custom image.

Not yet tested.

* [Official Apache Airflow](https://artifacthub.io/packages/helm/apache-airflow/airflow)

Of course, the one to use.

It comes with a [great documentation](https://airflow.apache.org/docs/helm-chart/stable/index.html).


## Configure cluster

Prerequisites: docker, docker-compose, kubectl, helm... and kind.

Note: useful link at https://kubernetes.io/docs/reference/kubectl/cheatsheet/ 

Let's follow these steps to install [Kind](https://kind.sigs.k8s.io/):
(Note: at the end I found useful this link: https://iximiuz.com/en/posts/kubernetes-kind-load-docker-image/ )
```
$ go get sigs.k8s.io/kind@v0.11.0

$ export PATH=$PATH:$(go env GOPATH)/bin

$ kind version
kind v0.11.0 go1.16.4 linux/amd64
```

Now let's create the cluster:
```
$ kind create cluster --name airflow-cluster --config kind-cluster.yaml
Creating cluster "airflow-cluster" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼 
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-airflow-cluster"
You can now use your cluster with:

kubectl cluster-info --context kind-airflow-cluster

Have a nice day! 👋
```

Finally, let's verify cluster:
```
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:35839
CoreDNS is running at https://127.0.0.1:35839/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get nodes -o wide
NAME                            STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION     CONTAINER-RUNTIME
airflow-cluster-control-plane   Ready    control-plane,master   3m41s   v1.21.1   172.18.0.3    <none>        Ubuntu 20.10   5.8.0-59-generic   containerd://1.5.1
airflow-cluster-worker          Ready    <none>                 3m8s    v1.21.1   172.18.0.2    <none>        Ubuntu 20.10   5.8.0-59-generic   containerd://1.5.1
airflow-cluster-worker2         Ready    <none>                 3m8s    v1.21.1   172.18.0.5    <none>        Ubuntu 20.10   5.8.0-59-generic   containerd://1.5.1
airflow-cluster-worker3         Ready    <none>                 3m8s    v1.21.1   172.18.0.4    <none>        Ubuntu 20.10   5.8.0-59-generic   containerd://1.5.1
```

## Deploy Apache airflow

Create namespace:
```
$ kubectl create namespace airflow
namespace/airflow created

$ kubectl get ns
NAME                 STATUS   AGE
airflow              Active   31s
default              Active   9m7s
kube-node-lease      Active   9m9s
kube-public          Active   9m9s
kube-system          Active   9m9s
local-path-storage   Active   9m2s
```

Add official repository to helm.

If you don't have the repo yet
```
$ helm repo add apache-airflow https://airflow.apache.org
```

Otherwise:
```
$ helm repo add apache-airflow https://airflow.apache.org
"apache-airflow" already exists with the same configuration, skipping

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...
...Successfully got an update from the "apache-airflow" chart repository
...
Update Complete. ⎈Happy Helming!⎈
```

At the end you'll have:
```
...
$ helm repo list
NAME                    URL                                                  
apache-airflow          https://airflow.apache.org                           
...
```


Now you can look for helm charts in the official Apache Airflow repo:
```
...
$ helm search repo airflow
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                       
apache-airflow/airflow  1.0.0           2.0.2           Helm chart to deploy Apache Airflow, a platform...
...
```

Note: this command will find all charts with name *airflow* in all the repos.

Once we have the chart we can deploy it:
```
$ helm install airflow apache-airflow/airflow --namespace airflow --debug --timeout 30m0s
NOTES:
Thank you for installing Apache Airflow 2.0.2!

Your release is named airflow.
You can now access your dashboard(s) by executing the following command(s) and visiting the corresponding port at localhost in your browser:

Airflow Webserver:     kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
Flower dashboard:      kubectl port-forward svc/airflow-flower 5555:5555 --namespace airflow
Default Webserver (Airflow UI) Login credentials:
    username: admin
    password: admin
Default Postgres connection credentials:
    username: postgres
    password: postgres
    port: 5432

You can get Fernet Key value by running the following:

    echo Fernet Key: $(kubectl get secret --namespace airflow airflow-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode)

$ helm ls -n airflow
NAME    NAMESPACE       REVISION        UPDATED                                         STATUS          CHART          APP VERSION
airflow airflow         1               2021-07-02 13:44:55.554129864 +0200 CEST        deployed        airflow-1.0.0  2.0.2      
javier@javier-Inspiron-13-5378:~/Projects/github.com/malsolo/airflow-chart-webinar$
```
Now you can see the deployed pods:
```
$ kubectl get pods -n airflow
NAME                                 READY   STATUS    RESTARTS   AGE
airflow-flower-54ff94c998-s8kct      1/1     Running   0          1m
airflow-postgresql-0                 1/1     Running   0          1m
airflow-redis-0                      1/1     Running   0          1m
airflow-scheduler-67c4958b86-lz885   2/2     Running   0          1m
airflow-statsd-7586f9998-kbp8m       1/1     Running   0          1m
airflow-webserver-6b447d5776-vfrzw   1/1     Running   0          1m
airflow-worker-0                     2/2     Running   0          1m
```

Since they are regular pods, you can do things such as see the logs and so on:
```
$ kubectl logs airflow-webserver-6b447d5776-vfrzw -n airflow
10.244.3.1 - - [02/Jul/2021:12:06:49 +0000] "GET /health HTTP/1.1" 200 187 "-" "kube-probe/1.21"
[2021-07-02 12:06:50 +0000] [24] [INFO] Handling signal: ttin
[2021-07-02 12:06:50 +0000] [223] [INFO] Booting worker with pid: 223

$ kubectl logs airflow-worker-0 -n airflow
error: a container name must be specified for pod airflow-worker-0, choose one of: [worker worker-gc] or one of the init containers: [wait-for-airflow-migrations]

$ kubectl get po -o jsonpath={.items[*].spec.containers[*].name} -n airflow
flower airflow-postgresql redis scheduler scheduler-gc statsd webserver worker worker-gc

$ kubectl get po airflow-worker-0 -o jsonpath='{.spec.containers[*].name}' -n airflow
worker worker-gc

$ kubectl get po airflow-scheduler-67c4958b86-lz885 -o jsonpath='{.spec.containers[*].name}' -n airflow
scheduler scheduler-gc

$ kubectl logs airflow-worker-0 -n airflow -c worker
[2021-07-02 11:48:15,664: INFO/MainProcess] Connected to redis://:**@airflow-redis:6379/0
[2021-07-02 11:48:15,701: INFO/MainProcess] mingle: searching for neighbors
[2021-07-02 11:48:16,754: INFO/MainProcess] mingle: all alone
[2021-07-02 11:48:16,776: INFO/MainProcess] celery@airflow-worker-0 ready.
[2021-07-02 11:48:19,305: INFO/MainProcess] Events of group {task} enabled by remote.
```

Now, let's make a port forward to access the airflow UI:
```
$ kubectl get svc -n airflow
NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
airflow-flower                ClusterIP   10.96.140.231   <none>        5555/TCP            79m
airflow-postgresql            ClusterIP   10.96.104.236   <none>        5432/TCP            79m
airflow-postgresql-headless   ClusterIP   None            <none>        5432/TCP            79m
airflow-redis                 ClusterIP   10.96.145.252   <none>        6379/TCP            79m
airflow-statsd                ClusterIP   10.96.10.167    <none>        9125/UDP,9102/TCP   79m
airflow-webserver             ClusterIP   10.96.139.226   <none>        8080/TCP            79m
airflow-worker                ClusterIP   None            <none>        8793/TCP            79m

$ kubectl port-forward svc/airflow-webserver 8080:8080 -n airflow
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080

```

Now go to http://localhost:8080 with **user/password** *admin/admin*

### Configure airflow

Let's update airflow to version 2.1.0

```
$ helm show values apache-airflow/airflow > values.yaml
```

Now edit:
· defaultAirflowTag to 2.1.0
· airflowVersion to 2.1.0
· executor to KubernetesExecutor

We'll also add a config map:
· extraEnvFrom <- see new values

We need to create this config map in K8s, we'll use variables.yaml for it.

```
$ kubectl apply -f variables.yaml 
configmap/airflow-variables created

$ kubectl get configmap -n airflow
NAME                     DATA   AGE
airflow-airflow-config   1      3h26m
airflow-variables        1      36s
kube-root-ca.crt         1      4h31m
```

Now, let's upgrade airflow
```
$ helm ls -n airflow
NAME    NAMESPACE       REVISION        UPDATED                                         STATUS          CHART             APP VERSION
airflow airflow         1               2021-07-02 13:44:55.554129864 +0200 CEST        deployed        airflow-1.0.0     2.0.2      

$ helm upgrade --install airflow apache-airflow/airflow --namespace airflow -f values.yaml --debug --timeout 30m0s
...
NOTES:
Thank you for installing Apache Airflow 2.1.0!

Your release is named airflow.
You can now access your dashboard(s) by executing the following command(s) and visiting the corresponding port at localhost in your browser:

Airflow Webserver:     kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
Default Webserver (Airflow UI) Login credentials:
    username: admin
    password: admin
Default Postgres connection credentials:
    username: postgres
    password: postgres
    port: 5432

You can get Fernet Key value by running the following:

    echo Fernet Key: $(kubectl get secret --namespace airflow airflow-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode)

$ helm ls -n airflow
NAME    NAMESPACE       REVISION        UPDATED                                         STATUS          CHART             APP VERSION
airflow airflow         2               2021-07-02 17:20:23.126990423 +0200 CEST        deployed        airflow-1.0.0     2.0.2      

$ kubectl get po -n airflow
NAME                                 READY   STATUS    RESTARTS   AGE
airflow-postgresql-0                 1/1     Running   0          3h54m
airflow-scheduler-5797df5d6f-mt9rn   2/2     Running   0          1m
airflow-statsd-7586f9998-kbp8m       1/1     Running   0          3h54m
airflow-webserver-7d49bc9c68-cwhx4   1/1     Running   0          1m
```

Let's access the UI. In another terminal:
```
$ kubectl port-forward svc/airflow-webserver 8080:8080 -n airflow
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

Go to http://localhost:8080 with admin/admin

Let's also verify the config map variable (actually, an airflow variable)
```
$ kubectl get po -n airflow
NAME                                 READY   STATUS    RESTARTS   AGE
airflow-postgresql-0                 1/1     Running   0          3h54m
airflow-scheduler-5797df5d6f-mt9rn   2/2     Running   0          1m
airflow-statsd-7586f9998-kbp8m       1/1     Running   0          3h54m
airflow-webserver-7d49bc9c68-cwhx4   1/1     Running   0          1m

$ kubectl exec --stdin --tty airflow-webserver-7d49bc9c68-cwhx4 -n airflow -- /bin/bash
Defaulted container "webserver" out of: webserver, wait-for-airflow-migrations (init)
airflow@airflow-webserver-7d49bc9c68-cwhx4:/opt/airflow$ python
Python 3.6.13 (default, May 12 2021, 16:48:24) 
[GCC 8.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from airflow.models import Variable
>>> Variable.get("my_s3_bucket")
'my_s3_name'
>>> exit()
airflow@airflow-webserver-7d49bc9c68-cwhx4:/opt/airflow$ exit
exit
$

```

## Next steps

Custom image with your dependencies and DAGs.

Load the mage in Kind: https://kind.sigs.k8s.io/docs/user/quick-start/#loading-an-image-into-your-cluster.

Follow https://airflow.apache.org/docs/helm-chart/stable/quick-start.html

## For the time being, let's finish it

```
$ kind get clusters
airflow-cluster

$ kind delete cluster --name airflow-cluster

$ kind get clusters
No kind clusters found.
```

