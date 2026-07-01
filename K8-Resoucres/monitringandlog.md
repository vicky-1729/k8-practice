# Minikube Observability Setup: Prometheus + Grafana

This guide installs the **kube-prometheus-stack** on a **Minikube** cluster using **Helm**. The stack includes Prometheus, Grafana, Alertmanager, Prometheus Operator, Node Exporter, and kube-state-metrics.

---

# Architecture

```text
                   +--------------------+
                   |      Grafana       |
                   | Dashboards & Alerts|
                   +---------+----------+
                             |
                             |
                   +---------v----------+
                   |    Prometheus      |
                   | Time Series Store  |
                   +---------+----------+
                             |
          --------------------------------------------
          |                  |                       |
+---------v--------+ +--------v-------+ +------------v------------+
| Node Exporter    | | kube-state     | | Kubernetes Components   |
| Node Metrics     | | Metrics        | | API Server, Pods, Nodes |
+------------------+ +----------------+ +-------------------------+

                 Minikube Kubernetes Cluster
```

---

# Prerequisites

Install these tools before you begin:

* Docker
* Minikube
* kubectl
* Helm

---

# Step 1: Start Minikube

Start Minikube:

```bash
minikube start
```

If Minikube is already running:

```bash
minikube status
```

## Verify

```bash
kubectl get nodes
```

Expected output:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...
```

---

# Step 2: Install Helm

If Helm is not installed, download the install script:

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
sudo ./get_helm.sh
```

## Verify

```bash
helm version
```

Expected output includes `v3.x.x`.

---

# Step 3: Add the Prometheus Helm repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## Verify

```bash
helm repo list
```

Expected output includes:

```text
NAME                    URL
prometheus-community    https://prometheus-community.github.io/helm-charts
```

---

# Step 4: Create the monitoring namespace

```bash
kubectl create namespace monitoring
```

## Verify

```bash
kubectl get ns monitoring
```

Expected output includes `monitoring`.

---

# Step 5: Create a Helm values file

Create `stack.yaml` with the following values:

```yaml
grafana:
  adminPassword: prom-operator

alertmanager:
  alertmanagerSpec:
    alertmanagerConfigSelector:
      matchLabels:
        release: monitoring
    replicas: 2
    alertmanagerConfigMatcherStrategy:
      type: None
```

This file customizes Grafana and Alertmanager for the monitoring namespace.

---

# Step 6: Install kube-prometheus-stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f stack.yaml
```

## Verify installation

```bash
helm list -n monitoring
```

Expected output:

```text
NAME         STATUS
monitoring   deployed
```

---

# Step 7: Verify pods

```bash
kubectl get pods -n monitoring
```

Wait until all monitoring pods are `Running`.

Expected pods include:

```text
alertmanager-monitoring-kube-prometheus-alertmanager-0    Running
alertmanager-monitoring-kube-prometheus-alertmanager-1    Running
monitoring-grafana                                        Running
monitoring-kube-prometheus-operator                       Running
monitoring-kube-state-metrics                             Running
monitoring-prometheus-node-exporter                       Running
prometheus-monitoring-kube-prometheus-prometheus          Running
```

---

# Step 8: Verify services

```bash
kubectl get svc -n monitoring
```

Expected services include:

* `monitoring-grafana`
* `monitoring-kube-prometheus-prometheus`
* `monitoring-kube-state-metrics`
* `alertmanager-operated`

---

# Step 9: Verify Helm release

```bash
helm list -n monitoring
```

Expected output:

```text
NAME
monitoring
```

---

# Step 10: Retrieve the Grafana admin password

```bash
kubectl get secret monitoring-grafana \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 -d
```

Default credentials:

* Username: `admin`
* Password: `prom-operator`

---

# Step 11: Access Grafana via port forwarding

Use port forwarding for local browser access:

```bash
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```

Open:

```text
http://localhost:3000
```

---

# Optional: Access Grafana via NodePort

If you prefer a NodePort service, patch Grafana:

```bash
kubectl patch svc monitoring-grafana -n monitoring -p '{"spec":{"type":"NodePort"}}'
kubectl get svc monitoring-grafana -n monitoring
```

Then get the Minikube IP:

```bash
minikube ip
```

Example URL:

```text
http://<MINIKUBE_IP>:<NODE_PORT>
```

---

# Step 12: Access Prometheus via NodePort

```bash
kubectl patch svc monitoring-kube-prometheus-prometheus -n monitoring -p '{"spec":{"type":"NodePort"}}'
kubectl get svc monitoring-kube-prometheus-prometheus -n monitoring
```

Example URL:

```text
http://<MINIKUBE_IP>:<NODE_PORT>
```

---

# Step 13: Verify Prometheus

```bash
curl http://$(minikube ip):<NODE_PORT>
```

Expected output contains a redirect or HTML landing page.

---

# Step 14: Verify Grafana

```bash
curl http://$(minikube ip):<NODE_PORT>
```

Expected output contains `HTTP/1.1 302 Found` or HTML response.

---

# Step 15: Inspect Kubernetes resources

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl get deployments -n monitoring
kubectl get statefulsets -n monitoring
kubectl get daemonsets -n monitoring
```

---

# Step 16: Check Helm release status

```bash
helm status monitoring -n monitoring
```

---

# Step 17: Uninstall

```bash
helm uninstall monitoring -n monitoring
kubectl delete namespace monitoring
```

---

# Components installed

| Component           | Purpose                                                            |
|---------------------|--------------------------------------------------------------------|
| Prometheus          | Collects and stores metrics                                        |
| Grafana             | Visualizes metrics and dashboards                                  |
| Alertmanager        | Handles and routes alerts                                          |
| Prometheus Operator | Manages Prometheus resources                                       |
| Node Exporter       | Collects node-level CPU, memory, disk, and network metrics         |
| kube-state-metrics  | Exposes Kubernetes object metrics (Pods, Deployments, Nodes, etc.) |

---

# Useful verification commands

```bash
kubectl get all -n monitoring
kubectl get pods -n monitoring
kubectl get svc -n monitoring
helm list -n monitoring
kubectl top nodes
kubectl top pods -A
kubectl logs -n monitoring deployment/monitoring-kube-prometheus-operator
kubectl describe pod <pod-name> -n monitoring
```

---

# Note for remote Minikube on EC2

If Minikube runs inside a remote EC2 instance, the Minikube IP is not directly reachable from your laptop. In that case:

* Use `kubectl port-forward` and an SSH tunnel.
* Use VS Code Remote SSH with forwarded ports.
* Expose the service through a reachable network path only if browser access from outside the instance is required.

If Minikube runs locally on your laptop, NodePort URLs using `minikube ip` should work directly.


---

# Step 10: Get Grafana Password

```bash
kubectl get secret monitoring-grafana \
-n monitoring \
-o jsonpath="{.data.admin-password}" | base64 -d
```

Username

```text
admin
```

Password

```text
prom-operator
```

---

# Step 11: Access Grafana (Port Forward)

```bash
kubectl port-forward svc/monitoring-grafana \
-n monitoring 3000:80
```

Open

```
http://localhost:3000
```

---

# Step 12: Access Grafana (NodePort)

Convert the service.

```bash
kubectl patch svc monitoring-grafana \
-n monitoring \
-p '{"spec":{"type":"NodePort"}}'
```

Verify.

```bash
kubectl get svc monitoring-grafana -n monitoring
```

Example Output

```text
NAME                 TYPE       PORT(S)
monitoring-grafana   NodePort   80:31463/TCP
```

Get Minikube IP.

```bash
minikube ip
```

Example

```text
192.168.49.2
```

Grafana URL

```
http://192.168.49.2:31463
```

---

# Step 13: Access Prometheus (NodePort)

Convert Prometheus service.

```bash
kubectl patch svc monitoring-kube-prometheus-prometheus \
-n monitoring \
-p '{"spec":{"type":"NodePort"}}'
```

Verify.

```bash
kubectl get svc monitoring-kube-prometheus-prometheus -n monitoring
```

Example

```text
NAME                                    TYPE       PORT(S)
monitoring-kube-prometheus-prometheus   NodePort   9090:30218/TCP
```

Prometheus URL

```
http://192.168.49.2:30218
```

---

# Step 14: Verify Prometheus

Check from the Minikube host.

```bash
curl http://$(minikube ip):30218
```

Expected Output

```text
<a href="/query">Found</a>
```

---

# Step 15: Verify Grafana

```bash
curl http://$(minikube ip):31463
```

Expected Output

```text
HTTP/1.1 302 Found
```

or HTML output.

---

# Step 16: Check Kubernetes Resources

Pods

```bash
kubectl get pods -n monitoring
```

Services

```bash
kubectl get svc -n monitoring
```

Deployments

```bash
kubectl get deployments -n monitoring
```

StatefulSets

```bash
kubectl get statefulsets -n monitoring
```

DaemonSets

```bash
kubectl get daemonsets -n monitoring
```

---

# Step 17: Check Helm Release

```bash
helm status monitoring -n monitoring
```

---

# Step 18: Uninstall

Uninstall the monitoring stack.

```bash
helm uninstall monitoring -n monitoring
```

Delete the namespace.

```bash
kubectl delete namespace monitoring
```

---

# Components Installed

| Component           | Purpose                                                            |
| ------------------- | ------------------------------------------------------------------ |
| Prometheus          | Collects and stores metrics                                        |
| Grafana             | Visualizes metrics with dashboards                                 |
| Alertmanager        | Handles and routes alerts                                          |
| Prometheus Operator | Manages Prometheus resources                                       |
| Node Exporter       | Collects node-level CPU, memory, disk, and network metrics         |
| kube-state-metrics  | Exposes Kubernetes object metrics (Pods, Deployments, Nodes, etc.) |

---

# Useful Verification Commands

```bash
kubectl get all -n monitoring
```

```bash
kubectl get pods -n monitoring
```

```bash
kubectl get svc -n monitoring
```

```bash
helm list -n monitoring
```

```bash
kubectl top nodes
```

```bash
kubectl top pods -A
```

```bash
kubectl logs -n monitoring deployment/monitoring-kube-prometheus-operator
```

```bash
kubectl describe pod <pod-name> -n monitoring
```

---

## Note for Remote Minikube on EC2

If Minikube is running **inside a remote EC2 instance**, the Minikube IP (for example, `192.168.49.2`) is **not directly reachable from your laptop**. In that case, use one of these approaches:

* `kubectl port-forward` and an SSH tunnel.
* VS Code Remote SSH with forwarded ports.
* Expose the service through a suitable network path if you specifically need browser access from outside the EC2 instance.

If Minikube is running **on your local laptop**, the NodePort URLs using `minikube ip` will work directly.
