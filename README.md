# ssup2ket-helm-charts

ssup2ket-service-charts is the git repo for helm charts used by ssup2ket services. ssup2ket-service-charts includes DB, message queue, cache, proxy, log analyzer/collector, metric collector, and CI/CD tools. 


## Infra

SQL/NoSQL DB, Message Queue and Storage Charts

### MySQL

ssup2ket services use MySQL as RDBMS. 

#### Version & Config

* Version : v8.0.25
* Mode : 1 Master / 1 Slave
* Database : dev/prod
* Namespace : infra
* ID/PW : admin/admin
* External Access Point
  * Primary : NodePort 31010
  * Secondary : NodePort 31012

#### Install

Install 

```
$ helm dependency update mysql
$ helm -n infra install mysql mysql
```

Create database for dev/prod environments.

```
$ kubectl -n infra exec -it mysql-primary-0 bash
(mysql-primary-0)$ mysql -u root -p
mysql> create database dev;
mysql> create database prod;
```

### Kafka

ssup2ket service use Kafka as event queue.

#### Version & Config

* Version : v2.8.0
* Partition Count : 1
* Replica Count : 1
* Namespace : infra
* External Access Point : NodePort 31012

#### Install

Install 

```
$ helm dependency update kafka
$ helm -n infra install kafka kafka
```

### Redis

ssup2ket services use Redis as key-value cache. 

#### Version & Config 

* Version : v6.2.4
* Mode : 1 Master / 1 Slave
* Password : redis
* Namespace : infra
* External Access Point
  * Master : NodePort 31013
  * Slave : NodePort 31014

#### Install

```
$ helm dependency update redis
$ helm -n infra install redis redis
```

### MongoDB (WIP)

ssup2ket services use MongoDB as document DB.

#### Version & Config 

* Version : v4.4.6
* Mode : Replicaset
* Namespace : infra

#### Install

```
$ helm dependency update mongodb
$ helm -n infra install mongodb mongodb
```

### MetalLB

ssup2ket K8s cluster uses MetalLB for load balancer type service. If K8s Cluster runs on the Cloud, there is no need to install MetalLB.

#### Version & Config

* Version : v0.10.2
* Mode : Layer2
* Address : 192.168.0.36-192.168.0.39
* Namespace : infra

#### Install

```
$ helm -n infra install metallb metallb
```

### NFS-client provisioner

ssup2ket K8s cluster uses NFS as storage class. If K8s Cluster runs on the Cloud, there is no need to install NFS-client provisioner. The values.yaml file must be set appropriately according to the NFS Server IP/Path.

#### Version & Config

* Version : v4.0.2
* NFS Server IP : 10.0.0.11
* NFS Server Path : /nfs_root
* Namespace : infra

#### Install

Install NFS clients for all k8s cluster nodes.

```
$ apt install nfs-common 
```

Install NFS-client provisioner.

```
$ helm -n infra install nfs-client nfs-subdir-external-provisioner
```


## Service Mesh

### Istio

ssup2ket services use Istio as service mesh layer. 

#### Version & Config

* Version : v1.10.2
* Components : istiod, dev ingressgateway, prod ingressgateway.
* Namespace : istio-system, istio-operator
* External Access Point
  * Istio Grafana : NodePort 32510
  * Istio Jaeger : NodePort 32513
  * kiali : NodePort 32511

#### Install

Install Istio operator and Istio. profile.yml set istiod, dev ingressgateway, prod ingressgateway. 

```
$ export PATH=$PWD/istio/bin:$PATH
$ istioctl operator init
$ kubectl apply -f istio/profile.yaml
```

Install Istio plugins. Run twice because of MonitoringDashboard CRD.

```
$ kubectl apply -f istio/samples/addons
$ kubectl apply -f istio/samples/addons
```

Set app-dev, app-prod namespace to istio injection.

```
$ kubectl create ns app-dev
$ kubectl create ns app-prod
$ kubectl label ns app-dev istio-injection=enabled
$ kubectl label ns app-prod istio-injection=enabled
```

## Monitoring

### Prometheus, Grafana, node-exporter, kube-state-metrics

ssup2ket services use Prometheus and Grafana to monitoring metrics.

#### Version & Config

* Prometheus 
  * Version : v2.27.1
  * Namespace : monitoring
  * External Access Point : NodePort 30090

* Grafana
  * Version : v8.0.3
  * Namespace : monitoring
  * External Access Point : NodePort 30091

* Node-exporter
  * Version : v1.1.2
  * Namespace : monitoring

#### Install

Install Promtheus, Grafana, node-exporter, kube-state-metrics.

```
$ helm dependency update kube-prometheus-stack
$ helm -n monitoring install prometheus kube-prometheus-stack
```

Change grafana service type to NodePort.

```
$ kubectl -n monitoring patch service prometheus-grafana --patch '{"spec": {"type": "NodePort", "ports": [{"port":80, "nodePort": 30091}]}}'
```

### Elasticsearch, Kibana, Fluentd

ssup2ket services use Elasticsearch, Kibana, Fluentd for log collection and analysis.

#### Version & Config

* Elasticsearch 
  * Version : v7.13.2
  * Mode : 2 Replicas (master, ingest, data)
  * Namespace : monitoring

* Kibana
  * Version : v7.13.2
  * Namespace : monitoring
  * External Access Point : NodePort 30092

* Fluentd
  * Version : v1.13.1
  * JSON Parsing
  * Namespace : monitoring

#### Install

Install Elasticsearch.

```
$ helm -n monitoring install elasticsearch elasticsearch
```

Install Kibana.

```
$ helm -n monitoring install kibana kibana
```

Install Fluentd.

```
$ helm dependency update fluentd
$ helm -n monitoring install fluentd fluentd
```


## CICD

### ArgoCD

ssup2ket services use ArgoCD for CD.

#### Version & Config

* Version : v2.0.4
* Namespace : cicd
* ID/PW : admin/`$kubectl -n cicd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
* External Access Point : NodePort 32010

#### Install

Install ArgoCD.

```
$ helm dependency update argo-cd
$ helm -n cicd install argo-cd argo-cd
```

### ArgoCD Image Updater

ssup2ket uses ArgoCD Image Updater to easily change app images.

#### Version & Config

* Version : v0.9.5
* Namespace : cicd

#### Install

Install ArgoCD Image Updater.

```
kubectl -n cicd apply -f argo-cd-image-updater/install.yaml 
```
