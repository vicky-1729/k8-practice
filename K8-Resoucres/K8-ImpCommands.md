# Kubernetes Important Commands

This file explains each important Kubernetes command in simple words:
- What it is
- Why we use it

## 1) Cluster And Namespace

### Command: kubectl cluster-info
What it is:
Shows the cluster control-plane and service endpoints.

Why we use it:
To confirm your cluster is reachable and running.

### Command: kubectl get ns
What it is:
Lists all namespaces in the cluster.

Why we use it:
To know where your resources are organized.

### Command: kubectl create namespace demo-app
What it is:
Creates a new namespace.

Why we use it:
To separate resources by project/environment.

## 2) Apply And Delete Manifests

### Command: kubectl apply -f <file.yaml>
What it is:
Creates or updates resources from a YAML file.

Why we use it:
To deploy manifests in a declarative way.

### Command: kubectl apply -f <folder>/
What it is:
Applies all manifest files from a folder.

Why we use it:
To deploy multiple resources in one step.

### Command: kubectl delete -f <file.yaml>
What it is:
Deletes resources defined in a YAML file.

Why we use it:
To remove deployed resources cleanly.

## 3) Inspect Resources

### Command: kubectl get pods
What it is:
Lists pods in the current namespace.

Why we use it:
To quickly check pod names and status.

### Command: kubectl get pods -n demo-app
What it is:
Lists pods in a specific namespace.

Why we use it:
To avoid confusion across namespaces.

### Command: kubectl get all -n demo-app
What it is:
Shows major resources (pods, services, deployments, etc.) in one view.

Why we use it:
To get a fast overall health snapshot.

### Command: kubectl describe pod <pod-name> -n demo-app
What it is:
Displays detailed pod information (events, image, mounts, probes).

Why we use it:
To troubleshoot when a pod is not running correctly.

## 4) Logs And Debugging

### Command: kubectl logs <pod-name> -n demo-app
What it is:
Shows logs from a pod's main container.

Why we use it:
To inspect app output and errors.

### Command: kubectl logs <pod-name> -c <container-name> -n demo-app
What it is:
Shows logs from a specific container in a multi-container pod.

Why we use it:
To debug the correct container when a pod has sidecars.

### Command: kubectl exec -it <pod-name> -n demo-app -- sh
What it is:
Opens an interactive shell inside a running container.

Why we use it:
To verify files, env variables, network, and process state.

## 5) ConfigMap And Secret

### Command: kubectl get configmap -n demo-app
What it is:
Lists ConfigMaps.

Why we use it:
To verify non-sensitive configuration objects exist.

### Command: kubectl describe configmap app-config -n demo-app
What it is:
Shows ConfigMap keys and metadata.

Why we use it:
To confirm correct config values are mounted/injected.

### Command: kubectl get secret -n demo-app
What it is:
Lists Secrets.

Why we use it:
To verify sensitive config objects exist.

### Command: kubectl describe secret app-secret -n demo-app
What it is:
Shows Secret metadata and key names (not plain values).

Why we use it:
To confirm secret object structure safely.

## 6) Useful Exam/Practice Shortcuts

### Command: kubectl get pods -o wide
What it is:
Shows extra pod details like node and pod IP.

Why we use it:
To troubleshoot scheduling and networking quickly.

### Command: kubectl get events -n demo-app --sort-by=.metadata.creationTimestamp
What it is:
Lists namespace events in time order.

Why we use it:
To identify recent failures and their exact reason.

### Command: kubectl config current-context
What it is:
Shows active kubeconfig context.

Why we use it:
To avoid applying commands to the wrong cluster.

## Quick Note

Use these commands in this order during troubleshooting:
1. kubectl get pods -n demo-app
2. kubectl describe pod <pod-name> -n demo-app
3. kubectl logs <pod-name> -n demo-app
4. kubectl get events -n demo-app --sort-by=.metadata.creationTimestamp
