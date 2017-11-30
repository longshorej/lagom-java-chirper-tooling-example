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

##### Install Cassandra

> You will have to wait a minute for helm to initialize. This command should suceed after a minute or so.

`helm install --namespace cassandra -n cassandra --set config.cluster_size=1 incubator/cassandra`

##### Build Project

`sbt clean docker:publishLocal`

##### View Images

`docker images`

##### Deploy chirp-impl

```
rp generate-kubernetes-deployment "activator-lagom-java-chirper/chirp-impl:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=youmustchangeme1" \
  --external-service cas_native=_cql._tcp.cassandra-cassandra.cassandra.svc.cluster.local \
  --pod-controller-replicas 3 | kubectl apply -f -
```

##### Deploy friend-impl

```
rp generate-kubernetes-deployment "activator-lagom-java-chirper/friend-impl:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=youmustchangeme2" \
  --external-service cas_native=_cql._tcp.cassandra-cassandra.cassandra.svc.cluster.local \
  --pod-controller-replicas 3 | kubectl apply -f -
```

##### Deploy activity-stream-impl

```
rp generate-kubernetes-deployment "activator-lagom-java-chirper/activity-stream-impl:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=youmustchangeme3" | kubectl apply -f -
```

##### Deploy front-end
```
rp generate-kubernetes-deployment "activator-lagom-java-chirper/front-end:1.0.0-SNAPSHOT" \
  --env JAVA_OPTS="-Dplay.crypto.secret=youmustchangeme4" | kubectl apply -f -
```
##### View Results

> Open the URL this command prints in the browser

`echo "http://$(minikube ip)"`