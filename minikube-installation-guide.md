# Docker, kubectl, and Minikube Installation Guide (Ubuntu EC2)


---

## Estimated Time

- Update System: 2 minutes
- Install Docker: 5 minutes
- Configure Docker Permissions: 2 minutes
- Install kubectl: 2 minutes
- Install Minikube: 2 minutes
- Start Minikube: 5 minutes
**Total Time:** Approximately 15–20 minutes

---

# Step 1: Update the System

```
sudo apt update
sudo apt upgrade -y
```

---

# Step 2: Install Required Packages

```
sudo apt install -y curl wget apt-transport-https ca-certificates software-properties-common
```

---

# Step 3: Install Docker

```
curl -fsSL https://get.docker.com | sudo sh
```

---

# Step 4: Start Docker

```
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```
You should see:

```
Active: active (running)
```

---

# Step 5: Give Docker Permission to Your User

```
sudo usermod -aG docker $USER
```
Apply the new group without logging out:

```
newgrp docker
```
Verify:

```
groups
```
You should see:

```
docker
```

---

# Step 6: Verify Docker

```
docker version
docker ps
```
If these commands work without sudo, Docker is configured correctly.

---

# Step 7: Install kubectl

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

```
chmod +x kubectl
```

```
sudo mv kubectl /usr/local/bin/
```
Verify:

```
kubectl version --client
```

---

# Step 8: Install Minikube
Download Minikube:

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
Make it executable:

```
chmod +x minikube-linux-amd64
```
Move it:

```
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```
Verify:

```
minikube version
```

---

# Step 9: Start Minikube

```
minikube start --driver=docker
```
This may take 3–5 minutes.

---

# Step 10: Verify Kubernetes

```
kubectl get nodes
```
Expected output:

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   xxm   v1.xx.x
```
Check pods:

```
kubectl get pods -A
```

---

# Step 11: Check Minikube

```
minikube status
```
Expected:

```
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

---

# Useful Commands
Start Minikube:

```
minikube start
```
Stop Minikube:

```
minikube stop
```
Delete Cluster:

```
minikube delete
```
Dashboard:

```
minikube dashboard
```
List Docker Containers:

```
docker ps
```
Cluster Info:

```
kubectl cluster-info
```
List Nodes:

```
kubectl get nodes
```
List All Pods:

```
kubectl get pods -A
```

---

# Troubleshooting
If you get:

```
permission denied while trying to connect to the Docker daemon socket
```
Run:

```
sudo usermod -aG docker $USER
newgrp docker
docker ps
```
If Docker is not running:

```
sudo systemctl restart docker
```
If Minikube fails:

```
minikube delete
minikube start --driver=docker
```

---

# Installation Complete
Verify everything:

```
docker version
kubectl version --client
minikube version
kubectl get nodes
minikube status
```
If all commands execute successfully, Docker, kubectl, and Minikube are installed and ready to use.

---

## Basic Commands You Must Know for Course Labs

### Check cluster context

```bash
kubectl config current-context
```

Expected output: `minikube`.

### See all nodes

```bash
kubectl get nodes -o wide
```

### See system pods

```bash
kubectl get pods -n kube-system
```

### Open Kubernetes dashboard (optional)

```bash
minikube dashboard
```

### Access service quickly (NodePort labs)

```bash
minikube service <service-name>
```

---

## Course Workflow with Minikube

Use this flow in every lab:

1. Write YAML file.
2. Apply file with `kubectl apply -f file.yaml`.
3. Verify with `kubectl get`.
4. Inspect with `kubectl describe`.
5. Troubleshoot with `kubectl logs`.
6. Clean old resources when done.

Example:

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod my-pod
kubectl logs my-pod
kubectl delete -f pod.yaml
```

---

## Common Errors and Quick Fixes

### Error 1: `Exiting due to DRV_NOT_HEALTHY`

Cause: Docker is not running.
Fix: Start Docker Desktop, then run `minikube start --driver=docker` again.

### Error 2: `kubectl get nodes` shows connection error

Cause: Minikube cluster is stopped.
Fix:

```bash
minikube start --driver=docker
```

### Error 3: Image pull issues / slow start

Cause: Network problem.
Fix: Check internet and retry.

### Error 4: Resource shortage

Cause: Low RAM/CPU allocation.
Fix:

```bash
minikube stop
minikube start --driver=docker --cpus=4 --memory=8192
```

---

## Start, Stop, Restart, Delete

Use these lifecycle commands during course:

```bash
# Start cluster
minikube start --driver=docker

# Stop cluster
minikube stop

# Restart (stop + start)
minikube stop
minikube start --driver=docker

# Delete cluster completely
minikube delete
```

Use `minikube delete` only when you want a fresh cluster.

---

## Why This Is Important for Exam and Interviews

Many interviews ask practical Kubernetes basics:

1. How to create local cluster?
2. How to deploy YAML in cluster?
3. How to check pod logs and events?
4. How to expose app in Minikube?

If you practice these in Minikube during this course, you can answer confidently.

---

## One-Time Setup Checklist

Complete this checklist before starting labs:

1. Docker Desktop installed and running.
2. `kubectl` installed.
3. `minikube` installed.
4. `minikube start --driver=docker` successful.
5. `kubectl get nodes` shows `Ready`.
6. Able to run one sample pod.

Sample test pod:

```bash
kubectl run test-pod --image=nginx --port=80
kubectl get pods
kubectl delete pod test-pod
```

If all these work, your system is fully ready for this Kubernetes fundamentals course.
