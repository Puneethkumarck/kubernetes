# Helm

Helm is the best way to find, share, and use software built for Kubernetes.

In the following steps you will learn:

* how to install and uninstall applications,
* what a chart provides,
* how to list public charts,
* how to list and add more repositories,
* how to create a custom chart,
* how to update a chart.

## Install Helm

Helm is a cluster administration tool that manages charts on Kubernetes.

Helm relies on a packaging format called charts. 

Charts define a composition of related Kubernetes resources and values that make up a deployment solution.

Charts are source code that can be packaged, named, versioned, and maintained in version control

The chart is a collection of Kubernetes manifests in the form of YAML files along with a templating language to allow contextual values to be injected into the YAMLs.

Charts complement your infrastructure-as-code (IaC) processes.

Helm also helps you manage the complexities of dependency management

Charts can include dependencies on other charts.

A chart is a deployable unit that can be inspected, listed, updated, and removed

The Helm CLI tool deploys charts to Kubernetes.

Interaction with Helm is through its command-line tool (CLI)

```
helm version --short
```

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

The current local state of Helm is kept in your environment in the home location:

![env.png](helm/env.png)

## Search For Redis Chart

Many common and publicly available open source projects can run on Kubernetes. 

Many of these projects offer containers that package these applications and vetted Helm charts for full production installations on Kubernetes.

Up until recently in 2020, all of the most commonly used public Helm charts were being lumped into a single Git repository for incubating and stable Helm charts.

This idea of centralizing all charts in GitHub has since been abandoned, thankfully. There are too many charts now being maintained by many different organizations and projects

### Artifact Hub

Now, the canonical source for cloud native artifacts, and specifically Helm charts, is [Artifact Hub](https://artifacthub.io/) an aggregator for distributed chart repos.

This hub has risen from the need for us to have a single place for us to search for charts.

While charts are listed here, the actual charts are hosted in a growing variety of repos

If you find a chart of interest the page for the specific chart will reveal the chart name, list of versions (semver.org format) and the repo where the chart can be found.

There are over 5400 charts available and growing each day:

```
helm search hub | wc -l

helm search hub clair

helm search hub redis

helm search hub redis | grep bitnami 
```

[Redis by bitNami](https://artifacthub.io/packages/helm/bitnami/redis)

### Repos

While the chart is listed in ARtifact Hub, the Bitnami organization has a public repo of all its charts

In each Hub chart page a repo is listed for you to add for access the chart

The instructions for the Redis chart says to add the bitnami repo

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
![repoadded.png](helm/repoadded.png)

```
helm repo list
```

![repolist.png](helm/repolist.png)

Instead of searching the Hub for charts you can also search the Bitnami repo:

```
helm search repo bitnami/redis
```

![searchinrepo.png](helm/searchinrepo.png)

The Helm command can reveal additional information about the chart:

```
helm show chart bitnami/redis
helm show readme bitnami/redis
helm show values bitnami/redis
```
![searchinrepo.png](helm/show_chart.png)


### Fabric8

```
helm search repo fabric8
helm repo add fabric8 https://fabric8.io/helm
```

## Deploy Redis

Create a namespace for the installation target:

```
kubectl create namespace redis
```

With a known chart name, use the install command to deploy the chart to your cluster:

```
redis-values.yaml
replica:
   replicaCount: 2

volumePermissions:
  enabled: true

securityContext:
  enabled: true
  fsGroup: 1001
  runAsUser: 1001
```

```
helm install my-redis bitnami/redis --version 14.3.3 --namespace redis --values redis-values.yaml
```

This will name a new install called my-redis and install a specific chart name and version into the redis namespace. 

The redis-values file override the chart's default values to ensure there are just 2 replicas and some file permission configuration is performed at startup. 

With the install command Helm launches the required Deployments, ReplicaSets, Pods, Services, ConfigMaps, or any other Kubernetes resource the chart defines.

Well written charts present notes as part of the installation instructions. 

The notes provide helpful information on how to access the new services. 

We'll follow these notes in the next step, but first, view all the installed charts:

```
helm list --all-namespaces

helm ls -n redis
```
![listchart.png](helm/listchart.png)


### Chart Installation Information

For each chart deployed to the cluster its deployment information is maintained in a secret stored on the targeted Kubernetes cluster

This way multiple Helm clients can consistently list the installed charts on the cluster

The secrets are deployed to the namespace where the chart is deployed. The secret names have the sh.helm. prefix:


```
kubectl get secrets --all-namespaces | grep sh.helm

helm list -A

kubectl get secrets --all-namespaces --selector owner=helm

kubectl --namespace redis describe secret sh.helm.release.v1.my-redis.v1
```

![redis_secret.png](helm/redis_secret.png)


## Observe Redis