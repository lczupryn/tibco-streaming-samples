# Docker : Kubernetes and Helm EventFlow

This sample builds on the [main Kubernetes sample](../../../../../ef-kubernetes/ef-kubernetes-app/src/site/markdown/index.md) and adds  Helm packaging.

* [Prerequisites](#prerequisites)
* [Creating an application archive project for Helm from maven](#creating-an-application-archive-project-for-helm-from-maven)
* [Packaging with Helm](#packaging-with-helm)
* [Deployment](#deployment)

<a name="prerequisites"></a>

## Prerequisites

In addition to Docker and Kubernetes ( see [main Kubernetes sample](../../../../../ef-kubernetes/ef-kubernetes-app/src/site/markdown/index.md) ), 
Helm is also required to be installed and configured - see https://helm.sh/docs/using_helm/ .

To use helm, tiller must be started in Kubernetes :

```shell
$ helm init
Creating /Users/plord/.helm 
Creating /Users/plord/.helm/repository 
Creating /Users/plord/.helm/repository/cache 
Creating /Users/plord/.helm/repository/local 
Creating /Users/plord/.helm/plugins 
Creating /Users/plord/.helm/starters 
Creating /Users/plord/.helm/cache/archive 
Creating /Users/plord/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /Users/plord/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
```

<a name="creating-an-application-archive-project-for-kubernetes-from-maven"></a>

## Creating an application archive project for Helm from maven

The following Docker related archetypes are available :

archetypeGroupId | archetypeArtifactId                        | Fragment  | Docker | Kubernetes | Helm
---------------- |--------------------------------------------|-----------|--------|------------|-------
com.tibco.ep     | application-docker-archetype               | None      | Yes    | No         | No
com.tibco.ep     | application-kubernetes-archetype           | None      | Yes    | Yes        | No
com.tibco.ep     | application-helm-archetype                 | None      | Yes    | Yes        | Yes
com.tibco.ep     | eventflow-application-docker-archetype     | EventFlow | Yes    | No         | No
com.tibco.ep     | java-application-docker-archetype          | Java      | Yes    | No         | No
com.tibco.ep     | liveview-application-docker-archetype      | LiveView  | Yes    | No         | No
com.tibco.ep     | eventflow-application-kubernetes-archetype | EventFlow | Yes    | Yes        | No
com.tibco.ep     | eventflow-application-helm-archetype       | EventFlow | Yes    | Yes        | Yes

A maven project contain an eventflow fragment and application archive support Kubernetes and Helm can be created using the
archetype **eventflow-application-helm-archetype** :

```shell
mvn archetype:generate -B \
        -DarchetypeGroupId=com.tibco.ep -DarchetypeArtifactId=eventflow-application-helm-archetype -DarchetypeVersion=10.6.0-SNAPSHOT \
        -DgroupId=com.tibco.ep.samples.docker -DartifactId=ef-helm -Dpackage=com.tibco.ep.samples.docker -Dversion=1.0.0 -Dtestnodes=A,B,C \
        -Dname="Docker: Helm EventFlow" -Ddescription="How to deploy an EventFlow application in Docker with Kubernetes and Helm"
```

<a name="packaging-with-helm"></a>

## Packaging with Helm

The the project is built with *mvn install* a Helm chart is created :

```shell
$ mvn install
...
[INFO] --- helm-maven-plugin:4.12:package (create helm package) @ ef-helm-app ---
[INFO] Packaging chart /Users/plord/workspace/tibco-streaming-samples-plord/docker/ef-helm/ef-helm-app/src/main/helm/ef-helm-app...
[INFO] Setting chart version to 1.0.0
[INFO] Successfully packaged chart and saved it to: /Users/plord/workspace/tibco-streaming-samples-plord/docker/ef-helm/ef-helm-app/target/helm/repo/ef-helm-app-1.0.0.tgz
```

The Helm chart can be installed with the *helm install* command - this will start the application up in Kubernetes :

```shell
$ helm install ef-helm-app/target/helm/repo/ef-helm-app-1.0.0.tgz
NAME:   eerie-pike
LAST DEPLOYED: Thu Oct 31 11:24:22 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME              READY  STATUS             RESTARTS  AGE
clustermonitor-0  0/1    ContainerCreating  0         0s
ef-helm-app-0     0/1    ContainerCreating  0         0s
ef-helm-app-1     0/1    ContainerCreating  0         0s

==> v1/Service
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)          AGE
clustermonitor  NodePort   10.105.244.11  <none>       11080:30228/TCP  0s
ef-helm-app     ClusterIP  None           <none>       <none>           0s

==> v1/StatefulSet
NAME            READY  AGE
clustermonitor  0/1    0s
ef-helm-app     0/2    0s
```

```shell
$ kubectl get pod
NAME               READY   STATUS    RESTARTS   AGE
clustermonitor-0   1/1     Running   0          8m49s
ef-helm-app-0      1/1     Running   0          8m18s
ef-helm-app-1      1/1     Running   0          8m18s


NOTES:
Thank you for installing ef-helm-app - Docker: Helm EventFlow

How to deploy an EventFlow application in Docker with Kubernetes and Helm

Your release is named eerie-pike.

To learn more about the release, try:

  $ helm status eerie-pike
  $ helm get eerie-pikey
```

If the Docker image has been pushed to a remote repository, the *dockerRegistry*
value can be overridden :

```shell
$ helm --set dockerRegistry=registry.com/ install ef-helm-app/target/helm/repo/ef-helm-app-1.0.0.tgz
```

Values can be set via the *--set* argument :

```shell
$ helm --set replicaCount=3 install ef-helm-app/target/helm/repo/ef-helm-app-1.0.0.tgz
```

Substitution parameters can be passed into the start-node script in the same way :

```shell
$ helm --set substitutions="a=b\,c=d" install ef-helm-app/target/helm/repo/ef-helm-app-1.0.0.tgz
```

The *helm delete* command will stop the application and delete the chart :

```shell
$ helm list
NAME        REVISION    UPDATED                     STATUS      CHART               APP VERSION NAMESPACE
old-parrot  1           Thu Oct 31 11:34:14 2019    DEPLOYED    ef-helm-app-1.0.0               default  

$ helm delete old-parrot
release "old-parrot" deleted
```

<a name="deployment"></a>

## Deployment

The Helm chart can be deployed using the *mvn deploy* command.  Standard parameter *-Dmaven.deploy.skip=true* 
is useful to skip deploying the maven artifacts.

```shell
$ mvn -Dmaven.deploy.skip=true deploy
...
[INFO] --- helm-maven-plugin:4.12:upload (deploy helm package) @ ef-helm-app ---
[INFO] Uploading to http://server.example.com/artifactory/helm/

[INFO] Uploading /Users/plord/workspace/tibco-streaming-samples-plord/docker/ef-helm/ef-helm-app/target/helm/repo/ef-helm-app-1.0.0.tgz...
[INFO] 201 - {
  "repo" : "helm",
  "path" : "/ef-helm-app-1.0.0.tgz",
  "created" : "2019-10-31T11:23:17.332-04:00",
  "createdBy" : "deployment",
  "downloadUri" : "http://server.example.com/artifactory/helm/ef-helm-app-1.0.0.tgz",
  "mimeType" : "application/x-gzip",
  "size" : "2215",
  "checksums" : {
    "sha1" : "329d5dd4766219daa30b47df59bbc8e830e7b8a3",
    "md5" : "8deccfad3fbafd02667b9a497f1d2d71"
  },
  "originalChecksums" : {
  },
  "uri" : "http://server.example.com/artifactory/helm/ef-helm-app-1.0.0.tgz"
}
```

The repository details can be specified in the pom.xml, or more typically set in continuous integration builds in maven's settings.xml :

```xml
<settings>
    <servers>
        <server>
            <id>server.example.com:2001</id>
            <username>username</username>
            <password>password</password>
        </server>
        <server>
            <id>helm.registry</id>
            <username>username</username>
            <password>password</password>
        </server>
    </servers>
    <profiles>
        <profile>
            <id>cloud</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <docker.push.registry>server.example.com:2001</docker.push.registry>
                <helm.registry>http://server.example.com/artifactory/helm/</helm.registry>
                <helm.registry.type>ARTIFACTORY</helm.registry.type>
            </properties>
        </profile>
    </profiles>
</settings>
```

Once deployed, the application can be installed on any Kubernetes node :

```shell
$ helm install http://server.example.com/artifactory/helm/ef-helm-app-1.0.0.tgz
NAME:   gangly-lemur
LAST DEPLOYED: Thu Oct 31 15:23:12 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME              READY  STATUS             RESTARTS  AGE
clustermonitor-0  0/1    ContainerCreating  0         0s
ef-helm-app-0     0/1    ContainerCreating  0         0s
ef-helm-app-1     0/1    ContainerCreating  0         0s

==> v1/Service
NAME            TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)          AGE
clustermonitor  NodePort   10.97.10.162  <none>       11080:30384/TCP  0s
ef-helm-app     ClusterIP  None          <none>       <none>           0s

==> v1/StatefulSet
NAME            READY  AGE
clustermonitor  0/1    0s
ef-helm-app     0/2    0s

```

Note that in the above example the Helm variable dockerRegistry is set to the location of the
docker images.


