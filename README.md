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

1) Add to `plugins.sbt`:
 
`addSbtPlugin("com.lightbend.rp" % "sbt-reactive-app" % "0.3.1")`

2) For each `impl` project, do the following in `build.sbt`:

`project("my-service-impl").enablePlugins(SbtReactiveAppPlugin)`

#### Build & Deploy

##### Start minikube

`minikube start --memory 8192 --cpus 3`

##### Setup Docker engine context to point to Minikube

`eval $(minikube docker-env)`

##### Enable Ingress Controller

minikube addons enable ingress

##### Install Helm in the Minikube

`helm init`

`helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/`

##### Install Cassandra

> You will have to wait a minute for helm to initialize. This command should suceed after a minute or so.

`helm install --namespace cassandra -n cassandra --set config.cluster_size=1 incubator/cassandra`

##### Build Project

`sbt clean docker:publishLocal`

##### View Images

`docker images`

##### Deploy chirp-impl

```
rp generate-kubernetes-deployment "lagom-java-chirper-tooling-example/chirp-impl:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=youmustchangeme1" \
  --external-service cas_native=_cql._tcp.cassandra-cassandra.cassandra.svc.cluster.local \
  --pod-controller-replicas 3 | kubectl apply -f -
```

##### Deploy friend-impl

```
rp generate-kubernetes-deployment "lagom-java-chirper-tooling-example/friend-impl:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=youmustchangeme2" \
  --external-service cas_native=_cql._tcp.cassandra-cassandra.cassandra.svc.cluster.local \
  --pod-controller-replicas 3 | kubectl apply -f -
```

##### Deploy activity-stream-impl

```
rp generate-kubernetes-deployment "lagom-java-chirper-tooling-example/activity-stream-impl:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=youmustchangeme3" | kubectl apply -f -
```

##### Deploy front-end
```
rp generate-kubernetes-deployment "lagom-java-chirper-tooling-example/front-end:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=youmustchangeme4" | kubectl apply -f -
```
##### View Results

> See the resources created for you

```bash
kubectl --namespace lagom-java-chirper-tooling-example get all
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
