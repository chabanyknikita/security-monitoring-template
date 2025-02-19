<p align="center">
  <a name="top" href="#octocat-hi-there-thanks-for-visiting-">
     <h1 align="center">Security Monitoring Stack Kubernetes</h1>
  </a>
</p>

### The easiest way to install monitoring and monitor security of your kubernetes cluster.

This stack include:

- Promtail
- Grafana
- Victoria Metrics Stack
- Victoria Logs
- Victoria Auth
- Alertmanager
- Kube-Bench Exporter
- Falco Exporter
- Trivy-Operator

<details>
  <summary><strong>Alerts From AlertManager</strong></summary>

- **Some alerts:**
  - Victoria Logs Alerts on Errors in Logs
  - Some default alerts
  - Kubernetes Node Not Ready
  - Kubernetes Memory Pressure
  - Kubernetes Disk Pressure
  - Kubernetes Network Unavailable
  - Kubernetes Out Of Capacity
  - Kubernetes Container Oom Killer
  - Kubernetes Job Failed
  - Kubernetes Cronjob Suspended
  - Kubernetes Persistentvolumeclaim Pending
  - Kubernetes Volume Out Of Disk Space
  - Kubernetes Volume Full In Four Days
  - Kubernetes Persistentvolume Error
  - Kubernetes Statefulset Down
  - Kubernetes Hpa Scaling Ability
  - Kubernetes Hpa Metric Availability
  - Kubernetes Hpa Scale Capability
  - Kubernetes Hpa Underutilized
  - Kubernetes Pod Not Healthy
  - Kubernetes Pod CrashLooping
  - Kubernetes Replicasset Mismatch
  - Kubernetes Deployment Replicas Mismatch
  - Kubernetes Statefulset Replicas Mismatch
  - Kubernetes Deployment Generation Mismatch
  - Kubernetes Statefulset Generation Mismatch
  - Kubernetes Statefulset Update Not RolledOut
  - Kubernetes Daemonset Rollout Stuck
  - Kubernetes Daemonset Misscheduled
  - Kubernetes Cronjob Too Long
  - Kubernetes Job Slow Completion
  - Kubernetes Api Server Errors
  - Kubernetes Api Client Errors
  - Kubernetes Client Certificate Expires Next Week
  - Kubernetes Client Certificate Expires Soon
  - Kubernetes Api Server Latency
  - Loki 5.. errors
  - Severity level - Error
  - Ledger Error

</details>

## :cherry_blossom: Setup

This is step-by-step how to install

Here are some details about install depends ..

- **Helm** • [Install](https://helm.sh/docs/intro/install/) :art:
- **Kubectl** • [Install](https://pwittrock.github.io/docs/tasks/tools/install-kubectl/) :shell:
- **k9s** • [Install](https://k9scli.io/topics/install/) :shaved_ice:

### Requirements: 
**2 CPU 4 GB RAM**

### Kubernetes Version which used in testing
**v1.28.2**

## Pre-requirements

#### Clone repo

```bash
git clone https://github.com/chabanyknikita/security-monitoring-template.git
cd security-monitoring-template
```

### Install helm repos for this stack
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo add stable https://charts.helm.sh/stable
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm repo update

```

#### Firstly you can disable what you want in charts/monitoring/charts/SVC/values.yaml, example:
```yaml
grafana:
  enabled: false


alertmanager:
  enabled: false

vmalert:
  enabled: false
```

#### Install nginx-Ingress and CertManger if you didn't install them

```bash
helm install \
cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--version v1.8.0 \
--set installCRDs=true
```

```bash
helm upgrade --install ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx --create-namespace
```

#### Configure AlertManger:

Go to charts/monitoring/values.yaml and change in section victoria-metrics-k8s-stack.alertmanager.config this values on your:

```
chat_id: <Chat Id> "chat_id must be integer"
bot_token: <Bot Token> "bot_token must be string"
```

- You can get this values from:

|      KEYS      |                          VALUES                           |
| :------------: | :-------------------------------------------------------: |
| TELEGRAM_ADMIN |     Your **chat id** you can get from (@userinfobot)      |
| TELEGRAM_TOKEN | Your telegram **bot token** you can get from (@botfather) |

> Only 1 user can see bot alerts

#### Configure Grafana Ingress(Optional):

If you need access to grafana via domain:

Go to charts/monitoring/values.yaml in section victoria-metrics-k8s-stack.grafana.ingress and

- Change grafana.ingress.enabled to true
- Change grafana.ingress.hosts to your domain
- Change grafana.ingress.tls.hosts to your domain

Example:

```yaml
  ingress:
    enabled: true
    annotations:
      certmanager.k8s.io/cluster-issuer: letsencrypt
      cert-manager.io/cluster-issuer: letsencrypt
      kubernetes.io/ingress.class: nginx
      kubernetes.io/tls-acme: "true"
    pathType: ImplementationSpecific
    hosts:
      - grafana.example.com
    tls: 
     - secretName: grafana-ingress-tls
       hosts:
         - grafana.example.com
```

### Upgrade your nginx-ingress for collecting metrics
```bash
helm upgrade ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx \
--set controller.metrics.enabled=true \
--set-string controller.podAnnotations."prometheus\.io/scrape"="true" \
--set-string controller.podAnnotations."prometheus\.io/port"="10254"
```

## Installation

### Install Kyverno
```bash
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```

### Install Kyverno Policies
```bash
helm install kyverno-policies charts/kyverno-policies

### Install trivy operator

```bash
helm upgrade -i trivy-operator aqua/trivy-operator --namespace trivy-system --create-namespace --version 0.20.6 --values charts/trivy-operator/trivy-values.yaml
```

### Install Falco And Falco-Exporter

```bash
helm upgrade -i falco --set falco.grpc.enabled=true --set falco.grpc_output.enabled=true --set driver.kind=ebpf falcosecurity/falco
```
```bash
helm upgrade -i falco-exporter falcosecurity/falco-exporter
```
#### Optionally Install Event-Generator for visualize how event's rules working

```bash
helm install event-generator falcosecurity/event-generator --namespace event-generator --create-namespace --set config.loop=false --set config.actions=""
```

### Install monitoring stack

```bash
helm upgrade -i monitoring charts/monitoring --values charts/monitoring/values.yaml -n monitoring
```

### Get grafana password:

```
kubectl get secret --namespace monitoring stack-grafana \
-ojsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### Access to grafana ui

- Credentials:

  - login: admin
  - password: from previous step

- If you enable ingress go to your domain and paste credentials
- If you don't enable ingress do port-forwarding and go http://localhost:3000:

```bash
kubectl port-forward service/stack-grafana -n monitoring 3000:80
```

### Now you have installed Monitoring Stack on your Kubernetes cluster!
