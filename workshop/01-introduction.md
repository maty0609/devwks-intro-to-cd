## Introduction
In today's workshop we will deploy Consul with the official Helm chart on Kubernetes cluster. After deploying Consul, you will learn how to access the Consul. After you will validate that Consul works you will deploy Chuck Norris App version 1 and version 2. We will start moving slowly Chuck Norris App from version 1 to version 2 and monitor the whole migration via Grafana.

In this workshop you will have a Kubernetes cluster running on `kind` (Kubernetes in Docker).

In this workshop we will show you how blue/green deployments are being deployed with Consul.

In this part we will be doing following tasks:
- Install Consul in recommended minimal configuration - Consul, Prometheus and Consul Ingress
- Install Grafana
- Validate your Consul and Grafana

## Before we start
Please clone workshop Git repository:
```bash
cd /root
git clone https://github.com/natilik/CLUS2022-DEVWKS-1930.git
```

## Install Consul
To customize your deployment, you can pass a *yaml* file to be used during the deployment; it will override the Helm chart's default values. The chart comes with reasonable defaults, however, you will override a few values to integrate more easily with `kind` and enable useful features.

Create a custom values file called `helm-consul-values.yaml` with the following contents. This configuration will:
- Set the prefix used for all resources in the Helm chart to `consul`
- Name the Consul datacenter `dc1`
- Enable `metrics` so Envoy sidecar will be providing application metrics to Prometheus
- Configure the datacenter to run only 1 server - for our demo this is fully sufficient
- Enable the Consul UI and expose it via a `NodePort`
- Enable `metrics` in Consul UI to visualize Consul agent metrics in your Consul dashboard
- Enable Consul service mesh features by setting `connectInject.enabled` to true
- Enable Consul service mesh CRDs by setting `controller.enabled` to true
- Enable Prometheus to deploy Prometheus instance by Consul - in production you would probably use your centralized Prometheus but for this demo let’s leave Helm to create Prometheus for us
- Enable `ingressGateways` which enable Consul Ingress for our cluster with one replica which is fully sufficient for our demo environment. More on Ingress in one of the chapters.
- We will name Ingress `ingress-demo`

```bash
cat > helm-consul-values.yaml <<EOF
global:
  name: consul
  datacenter: dc1
  metrics:
    enabled: true
    enableAgentMetrics: true
    agentMetricsRetentionTime: "1m"
server:
  replicas: 1
ui:
  enabled: true
  service:
    type: 'NodePort'
  metrics:
    enabled: true
    provider: "prometheus"
    baseURL: http://prometheus-server.consul.svc.cluster.local
connectInject:
  enabled: true
controller:
  enabled: true
prometheus:
 enabled: true
ingressGateways:
  enabled: true
  defaults:
    replicas: 1
  gateways:
    - name: ingress-demo
EOF
```

You can now add official Consul Helm chart:
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
```

Now you can deploy Consul in your Kubernetes cluster using Helm:
```bash
helm install --values helm-consul-values.yaml consul hashicorp/consul --create-namespace --namespace consul
```

You can check the progress with the command:
```bash
kubectl get pods --all-namespaces -w
```

After Consul is deployed we will go to do the next steps.

## Install Grafana
We have deployed Consul with Prometheus and we are now collecting the data in Prometheus. We will install Grafana to visualize the data from Prometheus. Let’s install Grafana Helm repo first:
```bash
helm repo add grafana https://grafana.github.io/helm-charts
```

To install Grafana we will use Helm values from this file:
```bash
cat /root/CLUS2022-DEVWKS-1930/deploy/grafana/values.yaml
```

```yaml
adminPassword: password

datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.consul.svc
      access: proxy
      isDefault: true

dashboardProviders:
 dashboardproviders.yaml:
   apiVersion: 1
   providers:
   - name: 'default'
     orgId: 1
     folder: ''
     type: file
     disableDeletion: false
     editable: true
     options:
       path: /var/lib/grafana/dashboards/default

dashboards:
  default:
    app:
      json: |
{
  "annotations": {
    "list": [
```

Let's install Grafana with Helm:
```bash
helm install grafana grafana/grafana -f /root/CLUS2022-DEVWKS-1930/deploy/grafana/values.yaml
```

You can check the progress with the command:
```bash
kubectl get pods --all-namespaces -w
```

## Validate Consul and Grafana
Before we will validate Consul and Grafana let's make sure Caddy is running.

Verify Consul was deployed properly by accessing the Consul UI. Expose the Consul UI with `kubectl port-forward` with the `consul-server-0` pod name as the target.
```bash
kubectl port-forward consul-server-0 --namespace consul 8500:8500
```

Visit the Consul UI at `http://localhost:8500/` in a browser on your development machine. You will observe a list of Consul's services, nodes, and other resources. Currently, you should only find the `consul` and `ingress-demo` services listed.

Now let’s verify Grafana was deployed properly by accessing the Grafana Dashboard. Expose the Consul UI with `kubectl port-forward` with the `grafana` service name as the target:
```bash
kubectl port-forward svc/grafana 3000:80
```

Visit the Grafana Dashboard at `http://localhost:8500/` in a browser on your development machine. You have only one dashboard called Applications. Currently since we didn’t install the application yet it doesn’t contain too much of the data but we will change it now.
