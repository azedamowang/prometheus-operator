# kube-prometheus

[![Build Status](https://github.com/prometheus-operator/kube-prometheus/workflows/ci/badge.svg)](https://github.com/prometheus-operator/kube-prometheus/actions)
[![Slack](https://img.shields.io/badge/join%20slack-%23prometheus--operator-brightgreen.svg)](http://slack.k8s.io/)
[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/prometheus-operator/kube-prometheus)

> Note that everything is experimental and may change significantly at any time.

This repository collects Kubernetes manifests, [Grafana](http://grafana.com/) dashboards, and [Prometheus rules](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/) combined with documentation and scripts to provide easy to operate end-to-end Kubernetes cluster monitoring with [Prometheus](https://prometheus.io/) using the Prometheus Operator.

The content of this project is written in [jsonnet](http://jsonnet.org/). This project could both be described as a package as well as a library.

Components included in this package:

* The [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)
* Highly available [Prometheus](https://prometheus.io/)
* Highly available [Alertmanager](https://github.com/prometheus/alertmanager)
* [Prometheus node-exporter](https://github.com/prometheus/node_exporter)
* [Prometheus Adapter for Kubernetes Metrics APIs](https://github.com/DirectXMan12/k8s-prometheus-adapter)
* [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
* [Grafana](https://grafana.com/)

This stack is meant for cluster monitoring, so it is pre-configured to collect metrics from all Kubernetes components. In addition to that it delivers a default set of dashboards and alerting rules. Many of the useful dashboards and alerts come from the [kubernetes-mixin project](https://github.com/kubernetes-monitoring/kubernetes-mixin), similar to this project it provides composable jsonnet as a library for users to customize to their needs.

## Warning

If you are migrating from `release-0.7` branch or earlier please read [what changed and how to migrate in our guide](https://github.com/prometheus-operator/kube-prometheus/blob/main/docs/migration-guide.md).

## Table of contents

- [kube-prometheus](#kube-prometheus)
  - [Warning](#warning)
  - [Table of contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
    - [minikube](#minikube)
  - [Compatibility](#compatibility)
    - [Kubernetes compatibility matrix](#kubernetes-compatibility-matrix)
  - [Quickstart](#quickstart)
    - [Access the dashboards](#access-the-dashboards)
  - [Customizing Kube-Prometheus](#customizing-kube-prometheus)
    - [Installing](#installing)
    - [Compiling](#compiling)
    - [Apply the kube-prometheus stack](#apply-the-kube-prometheus-stack)
    - [Containerized Installing and Compiling](#containerized-installing-and-compiling)
  - [Update from upstream project](#update-from-upstream-project)
    - [Update jb](#update-jb)
    - [Update kube-prometheus](#update-kube-prometheus)
    - [Compile the manifests and apply](#compile-the-manifests-and-apply)
  - [Configuration](#configuration)
  - [Customization Examples](#customization-examples)
    - [Cluster Creation Tools](#cluster-creation-tools)
    - [Internal Registry](#internal-registry)
    - [NodePorts](#nodeports)
    - [Prometheus Object Name](#prometheus-object-name)
    - [node-exporter DaemonSet namespace](#node-exporter-daemonset-namespace)
    - [Alertmanager configuration](#alertmanager-configuration)
    - [Adding additional namespaces to monitor](#adding-additional-namespaces-to-monitor)
      - [Defining the ServiceMonitor for each additional Namespace](#defining-the-servicemonitor-for-each-additional-namespace)
    - [Monitoring all namespaces](#monitoring-all-namespaces)
    - [Static etcd configuration](#static-etcd-configuration)
    - [Pod Anti-Affinity](#pod-anti-affinity)
    - [Stripping container resource limits](#stripping-container-resource-limits)
    - [Customizing Prometheus alerting/recording rules and Grafana dashboards](#customizing-prometheus-alertingrecording-rules-and-grafana-dashboards)
    - [Exposing Prometheus/Alermanager/Grafana via Ingress](#exposing-prometheusalermanagergrafana-via-ingress)
    - [Setting up a blackbox exporter](#setting-up-a-blackbox-exporter)
  - [Minikube Example](#minikube-example)
  - [Continuous Delivery](#continuous-delivery)
  - [Troubleshooting](#troubleshooting)
    - [Error retrieving kubelet metrics](#error-retrieving-kubelet-metrics)
      - [Authentication problem](#authentication-problem)
      - [Authorization problem](#authorization-problem)
    - [kube-state-metrics resource usage](#kube-state-metrics-resource-usage)
  - [Contributing](#contributing)
  - [License](#license)

## Prerequisites

You will need a Kubernetes cluster, that's it! By default it is assumed, that the kubelet uses token authentication and authorization, as otherwise Prometheus needs a client certificate, which gives it full access to the kubelet, rather than just the metrics. Token authentication and authorization allows more fine grained and easier access control.

This means the kubelet configuration must contain these flags:

* `--authentication-token-webhook=true` This flag enables, that a `ServiceAccount` token can be used to authenticate against the kubelet(s).  This can also be enabled by setting the kubelet configuration value `authentication.webhook.enabled` to `true`.
* `--authorization-mode=Webhook` This flag enables, that the kubelet will perform an RBAC request with the API to determine, whether the requesting entity (Prometheus in this case) is allowed to access a resource, in specific for this project the `/metrics` endpoint.  This can also be enabled by setting the kubelet configuration value `authorization.mode` to `Webhook`.

This stack provides [resource metrics](https://github.com/kubernetes/metrics#resource-metrics-api) by deploying the [Prometheus Adapter](https://github.com/DirectXMan12/k8s-prometheus-adapter/).
This adapter is an Extension API Server and Kubernetes needs to be have this feature enabled, otherwise the adapter has no effect, but is still deployed.

### minikube

To try out this stack, start [minikube](https://github.com/kubernetes/minikube) with the following command:

```shell
$ minikube delete && minikube start --kubernetes-version=v1.20.0 --memory=6g --bootstrapper=kubeadm --extra-config=kubelet.authentication-token-webhook=true --extra-config=kubelet.authorization-mode=Webhook --extra-config=scheduler.address=0.0.0.0 --extra-config=controller-manager.address=0.0.0.0
```

The kube-prometheus stack includes a resource metrics API server, so the metrics-server addon is not necessary. Ensure the metrics-server addon is disabled on minikube:

```shell
$ minikube addons disable metrics-server
```

## Compatibility

### Kubernetes compatibility matrix

The following versions are supported and work as we test against these versions in their respective branches. But note that other versions might work!

| kube-prometheus stack | Kubernetes 1.18 | Kubernetes 1.19 | Kubernetes 1.20 | Kubernetes 1.21 |
|-----------------------|-----------------|-----------------|-----------------|-----------------|
| `release-0.5`         | ✔               | ✗               | ✗               | ✗               |
| `release-0.6`         | ✗               | ✔               | ✗               | ✗               |
| `release-0.7`         | ✗               | ✔               | ✔               | ✗               |
| `release-0.8`         | ✗               | ✗               | ✔               | ✔               |
| `HEAD`                | ✗               | ✗               | ✔               | ✔               |

## Quickstart

>Note: For versions before Kubernetes v1.20.z refer to the [Kubernetes compatibility matrix](#kubernetes-compatibility-matrix) in order to choose a compatible branch.

This project is intended to be used as a library (i.e. the intent is not for you to create your own modified copy of this repository).

Though for a quickstart a compiled version of the Kubernetes [manifests](manifests) generated with this library (specifically with `example.jsonnet`) is checked into this repository in order to try the content out quickly. To try out the stack un-customized run:
 * Create the monitoring stack using the config in the `manifests` directory:

```shell
# Create the namespace and CRDs, and then wait for them to be available before creating the remaining resources
kubectl create -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl create -f manifests/
```

We create the namespace and CustomResourceDefinitions first to avoid race conditions when deploying the monitoring components.
Alternatively, the resources in both folders can be applied with a single command
`kubectl create -f manifests/setup -f manifests`, but it may be necessary to run the command multiple times for all components to
be created successfullly.

 * And to teardown the stack:
```shell
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

### Installing


安装文件基本基于release-0.8这个版本,后期做了修改 新增kube-controller-namager-svc-ep.yaml  kube-scheduler-svc-ep.yaml  默认是没有添加 controller-namager scheduler 的EP 所以alertmanager 监控报警。

文件已经修改grafana alertmanager prometheus 的访问方式为nodeport.

########基础准备#########

下载资源包之后先部署setup目录文件，创建资源对象 以及monitoring namespace


#########################

这是为了防止prometheus无法监听controller scheduler 的metrics地址

修改 /etc/kubernetes/manifests/kube-controller-manager.yaml 文件的监听地址为 0.0.0.0 # - --bind-address=0.0.0.0

修改 /etc/kubernetes/manifests/kube-scheduler.yaml  文件的监听地址为 0.0.0.0 # - --bind-address=0.0.0.0

#########################
#添加alertmanager-main confmap secret 报警配置项

kubectl create secret generic  alertmanager-main --from-file=alertmanager.yaml --from-file=email.tmpl -n monitoring

#alertmanager目录为confmap 和email 模板文件 

创建底层nfs动态存储 


#kube-controller-namager-svc-ep.yaml  kube-scheduler-svc-ep.yaml  需要修改IP为自己服务器的IP


###################################自动发现模块#######################################

kubectl create secret generic additional-configs --from-file=prometheus-additional.yaml -n monitoring

创建完成后，会将上面配置信息进行 base64 编码后作为 prometheus-additional.yaml 这个 key 对应的值存在：

kubectl get secret additional-configs -n monitoring -o yaml

![image](https://user-images.githubusercontent.com/55533886/146626869-b3a13052-16ef-4cd0-9a48-6c1b4b1f0dc4.png)

然后我们只需要在声明 prometheus 的资源对象文件中添加上这个额外的配置：(prometheus-prometheus.yaml)

![image](https://user-images.githubusercontent.com/55533886/146626893-897ae724-5c64-43ff-87f4-4a7bd9548b7e.png)


  
添加完成后，直接更新 prometheus 这个 CRD 资源对象：

kubectl apply -f prometheus-prometheus.yaml
  
隔一小会儿，可以前往 Prometheus 的 Dashboard 中查看配置是否生效：
  
![image](https://user-images.githubusercontent.com/55533886/146626915-264f500f-8e3e-481a-b7e1-54220934a574.png)


修改prometheus-clusterRole.yaml 增加权限避免prometheus参数错误日志 都是xxx is forbidden.

更新prometheus-clusterRole.yaml 稍等一会就可以看到

![image](https://user-images.githubusercontent.com/55533886/146626976-6e25059a-c4d7-43da-920f-242cf3bf3602.png)
########基础准备完成#########

####################部署应用程序##########################

这些目录下文件可以一次性执行（无先后顺序） kubectl apply -f ./   

alertmanager

blackbox

grafana

kubernetes

node-export

prometheus

##############自动发现验证#####################

部署 test目录下  redis-prom.yaml，待程序启动之后可以验证.

kubectl get pods,svc -n monitoring | grep redis

curl 10.244.0.81:9121/metrics|grep "redis_up 1"  #验证是否可以正常访问.

curl 10.244.0.81:9121/metrics|grep -v ^#  查询reids metrics下可以查询到的指标.

![image](https://user-images.githubusercontent.com/55533886/146627052-c6ff7994-08a3-46ad-a312-2815320287fe.png)

##############自动发现验证完成#####################



prometheus-operator 自动发现部署完成。



1.20.X 使用以下NFS镜像 修改deployment的image地址就可以了 不然会出现pending的问题PV无法创建。

vbouchaud/nfs-client-provisioner




Apache License 2.0, see [LICENSE](https://github.com/prometheus-operator/kube-prometheus/blob/main/LICENSE).
