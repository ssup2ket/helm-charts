# ssup2ket Helm Charts

Helm charts for ssup2ket.

## Infra

### MetalLB

ssup2ket uses MetalLB for load balancer type service.

#### Version & Config

* Version : v0.10.2
* Mode : Layer2
* Address : 192.168.0.36-192.168.0.39

#### Install

Install by using helm3

```
$ helm -n infra install metallb metallb
```

### NFS-client provisioner

ssup2ket uses NFS as storage class. 

#### Version & Config

* Version : v4.0.2
* NFS Server : 10.0.0.11
* NFS Server Path : /nfs_root

#### Install

Install NFS clients for all k8s cluster nodes.

```
$ apt install nfs-common 
```

Install by using helm3

```
$ helm -n infra install nfs-client nfs-subdir-external-provisioner
```

### MySQL

ssup2ket uses MySQL as RDBMS. 

#### Version & Config

* Version : 8.0.25
* Mode : 1 Master - 1 Slave
* Database : dev/prod
* ID/PW : admin/admin

#### Install

Install by using helm3

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

ssup2ket uses Kafka as event queue.

#### Version & Config

#### Install

Install by using helm3

```
$ helm dependency update kafka
$ helm -n infra install kafka kafka
```

### Redis

ssup2ket uses Redis as key-value cache. Install Redis.

```
$ helm dependency update redis
$ helm -n infra install redis redis
```

### MongoDB (Not Working)

ssup2ket uses MongoDB as document DB. Install MongoDB.

```
$ helm dependency update mongodb
$ helm -n infra install mongodb mongodb
```

## Monitoring

### Prometheus, Grafana, node-exporter, kube-state-metrics

ssup2ket uses Prometheus and Grafana to monitoring metrics.

#### Version & Config

#### Install

Install Prometheus, Grafana and related componentes.

```
$ helm dependency update kube-prometheus-stack
$ helm -n monitoring install prometheus kube-prometheus-stack
```

Change grafana service type to NodePort.

```
$ kubectl -n monitoring patch service prometheus-grafana --patch '{"spec": {"type": "NodePort", "ports": [{"port":80, "nodePort": 30091}]}}'
```

### Elasticsearch, Kibana, Fluentd

ssup2ket uses Elasticsearch, Kibana, Fluentd for log collection and analysis.

#### Version & Config

* Version
  * Elasticsearch : v7.13.2 
  * Kibana : v7.13.2
  * Fluentd : 

* Config
  * Elasticsearch 
    * Mode : 2 Replicas
	* Role : master, ingest, data
  * Kibana
    * ID/PW :

#### Install

Install Elasticsearch. Install by using helm3.

```
$ helm -n monitoring install elasticsearch elasticsearch
```

Install Kibana. Install by using helm3.

```
$ helm -n monitoring install kibana kibana
```

Install Fluentd.

```
$ helm dependency update fluentd
$ helm -n monitoring install fluentd fluentd
```

## Service Mesh

### Istio

ssup2ket uses Istio as service mesh layer. 

#### Version & Config

* Version : v1.10.2
* Components : istiod, dev ingressgateway, prod ingressgateway.

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

## CICD

### ArgoCD

ssup2ket uses ArgoCD for CD.

#### Version & Config

* Version : v2.0.4
* ID/PW : admin/`$kubectl -n cicd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

#### Install

Install by using helm3

```
$ helm dependency update argo-cd
$ helm -n cicd install argo-cd argo-cd
```

