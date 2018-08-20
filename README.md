
# 0 Prerequisites

If using google cloud for kubernetes as a service:

Install gcloud CLI on mac: https://cloud.google.com/sdk/docs/quickstart-macos

Then run:

```
gcloud container clusters get-credentials test-cluster-1
```

to get access to the k8s cluster

After that, you can try to run:

```
kubectl config get-contexts
```

to confirm that the cluster is in the output.

In google cloud kubernetes service, RBAC (role based access control) is enabled by default. In order to deploy the EFK stack into k8s, you will need to grant user permission to create roles in k8s.

Read more about it here: https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control

In short, you need to run:

```
kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=guotiexin@gmail.com
```

# 1 Get the Repository for Deploying EFK

```
git clone https://github.com/kubernetes/kubernetes.git
```

Actually here only `cluster/addons/fluentd-elasticsearch` folder is needed, contents of which are already in this repo.

This contains the yaml file for deploying elastic search, fluentd, and kibana.

# 2 Deploy Fluentd

## 2.1 Prerequisites

In order for Fluentd to work, every Kubernetes node must be labeled with beta.kubernetes.io/fluentd-ds-ready=true, as otherwise the Fluentd DaemonSet will ignore them.

## 2.2 Deploy Fluentd

At the root directory of the kubernetes repository, do:

```
cd cluster/addons/fluentd-elasticsearch
kubectl create -f fluentd-es-configmap.yaml
kubectl create -f fluentd-es-ds.yaml
```

This creates the config map for fluentd, and deploys fluentd as a daemon set.

# 3 Deploy Kibana

Under the same directory (`repo_root/cluster/addons/fluentd-elasticsearch`), do:

```
kubectl create -f kibana-deployment.yaml
kubectl create -f kibana-service.yaml
```

This deploys Kibana using "deployment", and exposes it as an internal service.

# 4 Deploy Elastic Search

Under the same directory (`repo_root/cluster/addons/fluentd-elasticsearch`), do:

```
kubectl create -f es-statefulset.yaml
kubectl create -f es-service.yaml
```

This deploys Kibana using a stateful set, and exposes it as an internal service.

# 5 Test and Verify

## 5.1 Verify services/deployment/pods

```
kubectl get services --namespace kube-system
kubectl get deployments --namespace kube-system
kubectl get daemonset --namespace kube-system
kubectl get statefulset --namespace kube-system
kubectl get pods --namespace kube-system
```

You should be able to see

- "elasticsearch-logging" and "kibana-logging" in the service output
- "kibana-logging" in the deployment
- "fluentd-es-v2.2.0" in the daemonset
- elasticsearch-logging in the statefulset

And the pods should be in READY state.

## 5.2 Using Kibana

In this setup, Kibana is only exposed as an internal service, no internal/external load balancer is created by default, so you need kubernetes proxy in order to access Kibana:

```
kubectl proxy
```

Then access:

http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kibana-logging/proxy/
