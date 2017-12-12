# lagom-java-chirper-example

## Overview

A Lagom Java example showcasing a Twitter-like application

## Local JVM

* Start all services using `mvn lagom:runAll` or `sbt runAll`.

## Minikube

#### Prerequisites

* [Docker](https://www.docker.com/)
* [Helm](https://github.com/kubernetes/helm)
* [Minikube](https://github.com/kubernetes/minikube)
* [SBT](http://www.scala-sbt.org/)

#### A note on project setup

To enable the tooling for this project, the following steps were performed:

1) Add to `project/plugins.sbt`:
 
`addSbtPlugin("com.lightbend.rp" % "sbt-reactive-app" % "0.4.0")`

2) For each `impl` project, do the following in `build.sbt`:

`project("my-service-impl").enablePlugins(SbtReactiveAppPlugin)`

#### Build & Deploy

##### Install reactive-cli

See [Tooling Documentation](https://s3-us-west-2.amazonaws.com/rp-tooling-temp-docs/deployment-setup.html#install-the-cli)

Ensure you're using `reactive-cli` 0.4.2 or newer. You can check the version with `rp version`.

##### Start minikube

> It's recommended to use Kubernetes 1.8 or newer so make sure your Minikube is up-to-date. If necessary, you can delete your Minikube and start refresh via `minikube delete`

`minikube start --memory 8192 --cpus 3`

##### Setup Docker engine context to point to Minikube

`eval $(minikube docker-env)`

##### Enable Ingress Controller

`minikube addons enable ingress`

##### Install Reactive Sandbox

The `reactive-sandbox` includes development-grade installations of Cassandra, Elasticsearch, Kafka, and ZooKeeper. It's packaged as a Helm chart for easy installation into your Kubernetes cluster. It's a great way to provide the cassandra dependency.

> Note that if you have an external Cassanda cluster, you can skip this step. You'll need to change the `cassandra_svc` variable (defined below) if this is the case.

```bash
helm init
helm repo add lightbend-helm-charts https://lightbend.github.io/helm-charts
helm update
```

Verify that Helm is available (this takes a minute or two):

```bash
kubectl --namespace kube-system get deploy/tiller-deploy
```

```
NAME            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
tiller-deploy   1         1         1            1           3m
```

Install the sandbox:
```bash
helm install lightbend-helm-charts/reactive-sandbox --name reactive-sandbox
```

Verify that it is available (this takes a minute or two):

```bash
kubectl get deploy/reactive-sandbox
```

```
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
reactive-sandbox   1         1         1            1           1m
```

##### Build Project

`sbt clean docker:publishLocal`


##### View Images

`docker images`

##### Deploy Projects

First, ensure `reactive-sandbox` is ready. It should indicate `1` in the AVAILABLE column:

`kubectl get deploy/reactive-sandbox`

Now, you can start deploying the projects:

```bash
# Be sure to change these secret values

chirp_secret="youmustchangeme"
friend_secret="youmustchangeme"
activity_stream_secret="youmustchangeme"
front_end_secret="youmustchangeme"

# Default address for reactive-sandbox, change if using external Cassandra

cassandra_svc="_cql._tcp.reactive-sandbox-cassandra.default.svc.cluster.local"

# deploy chirp-impl

rp generate-kubernetes-deployment "lagom-java-chirper-tooling-example/chirp-impl:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=$chirp_secret" \
  --external-service "cas_native=$cassandra_svc" \
  --pod-controller-replicas 2 | kubectl apply -f -

# deploy friend-impl

rp generate-kubernetes-deployment "lagom-java-chirper-tooling-example/friend-impl:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=$friend_secret" \
  --external-service "cas_native=$cassandra_svc" \
  --pod-controller-replicas 2 | kubectl apply -f -
  
# deploy activity-stream-impl

rp generate-kubernetes-deployment "lagom-java-chirper-tooling-example/activity-stream-impl:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=$activity_stream_secret" | kubectl apply -f -
  
# deploy front-end

rp generate-kubernetes-deployment "lagom-java-chirper-tooling-example/front-end:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=$front_end_secret" | kubectl apply -f -
```

##### View Results

> See the resources created for you

```bash
kubectl get all
```

```
NAME                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/activityservice-v1-5-0   1         1         1            1           2h
deploy/chirpservice-v1-5-0      3         3         3            3           2h
deploy/friendservice-v1-5-0     3         3         3            3           2h
deploy/front-end-v1-5-0         1         1         1            1           2h

NAME                                   DESIRED   CURRENT   READY     AGE
rs/activityservice-v1-5-0-659877cd49   1         1         1         2h
rs/chirpservice-v1-5-0-6548865dc5      3         3         3         2h
rs/friendservice-v1-5-0-66f688897b     3         3         3         2h
rs/front-end-v1-5-0-87c5b6b79          1         1         1         2h

NAME                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/activityservice-v1-5-0   1         1         1            1           2h
deploy/chirpservice-v1-5-0      3         3         3            3           2h
deploy/friendservice-v1-5-0     3         3         3            3           2h
deploy/front-end-v1-5-0         1         1         1            1           2h

NAME                                   DESIRED   CURRENT   READY     AGE
rs/activityservice-v1-5-0-659877cd49   1         1         1         2h
rs/chirpservice-v1-5-0-6548865dc5      3         3         3         2h
rs/friendservice-v1-5-0-66f688897b     3         3         3         2h
rs/front-end-v1-5-0-87c5b6b79          1         1         1         2h

NAME                                         READY     STATUS    RESTARTS   AGE
po/activityservice-v1-5-0-659877cd49-59z4z   1/1       Running   0          2h
po/chirpservice-v1-5-0-6548865dc5-2fjn6      1/1       Running   0          2h
po/chirpservice-v1-5-0-6548865dc5-kgbbb      1/1       Running   0          2h
po/chirpservice-v1-5-0-6548865dc5-zcc6l      1/1       Running   0          2h
po/friendservice-v1-5-0-66f688897b-d22ph     1/1       Running   0          2h
po/friendservice-v1-5-0-66f688897b-fvndd     1/1       Running   0          2h
po/friendservice-v1-5-0-66f688897b-j9tzj     1/1       Running   0          2h
po/front-end-v1-5-0-87c5b6b79-mvnh8          1/1       Running   0          2h

NAME                  TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                         AGE
svc/activityservice   ClusterIP   None         <none>        10000/TCP,10001/TCP,10002/TCP   2h
svc/chirpservice      ClusterIP   None         <none>        10000/TCP,10001/TCP,10002/TCP   2h
svc/friendservice     ClusterIP   None         <none>        10000/TCP,10001/TCP,10002/TCP   2h
svc/front-end         ClusterIP   None         <none>        10000/TCP                       2h
```

> Open the URL this command prints in the browser

`echo "http://$(minikube ip)"`
