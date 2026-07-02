# Kubernetes 101: From Zero to Hero - Demo Guide

[Read in Thai (ภาษาไทย)](file:///Users/thapanutlurejunda/work/kubectl/README.th.md)

This guide is designed for the instructor to follow during the live demonstration. 

## Prerequisites
- Docker Desktop installed and running.
- `kubectl` CLI installed.
- `helm` CLI installed.
- `kind` CLI installed (`brew install kind` on macOS).

---

## PART 01: Kubernetes Fundamentals

We will start by deploying a basic Nginx web server using fundamental Kubernetes resources.
*Ensure Docker Desktop's built-in Kubernetes is enabled for this part, or you can use kind.*

### 1. The Pod
A Pod is the smallest deployable unit. Let's create one.
```bash
kubectl apply -f part1-fundamentals/01-pod.yaml
kubectl get pods
```

### 2. The Deployment
Pods are ephemeral. Deployments manage Pods and provide self-healing and scaling.
```bash
# First, let's delete the standalone pod to avoid confusion
kubectl delete pod nginx-pod

# Now apply the deployment
kubectl apply -f part1-fundamentals/02-deployment.yaml
kubectl get deployments
kubectl get pods
```

**Demonstrate Self-healing:**
```bash
# Delete a pod and watch the deployment recreate it instantly
kubectl delete pod <pod-name-from-previous-step>
kubectl get pods
```

### 3. The Service
Our deployment is running, but it's not accessible from our browser. We need a Service.
```bash
kubectl apply -f part1-fundamentals/03-service.yaml
kubectl get services
```
*Note: If using Docker Desktop, `LoadBalancer` type will expose it directly on `localhost:80`. Go to http://localhost in your browser to see Nginx!*

---

## PART 02: Advanced Package Management (Helm)

Managing lots of YAML files gets complicated. Helm is the package manager for Kubernetes.

```bash
cd part2-helm
```

### 1. Explore the Chart
Show the class the `my-web-app` directory structure created by `helm create`. Point out `Chart.yaml`, `values.yaml`, and the `templates/` folder.

### 2. Install the Chart (Default)
Install the default Nginx chart.
```bash
helm install my-release ./my-web-app
kubectl get pods
```

### 3. Upgrade with Custom Values (Production simulation)
We want more replicas and a LoadBalancer for our "production" environment. We use `values-prod.yaml` to override defaults.
```bash
helm upgrade my-release ./my-web-app -f values-prod.yaml
kubectl get pods  # You should now see 3 replicas
kubectl get svc   # You should see a LoadBalancer
```

### 4. Access the Application (Port Forwarding)
Because we are using a local cluster (`kind`), the `EXTERNAL-IP` for the LoadBalancer will remain `<pending>`. We must use port-forwarding to access the application from our machine.
```bash
# Run this command and leave it running
kubectl port-forward svc/my-release-my-web-app 8080:80
```
*Open http://localhost:8080 in your browser to view the app. Press `Ctrl + C` in the terminal to stop.*

```bash
# Cleanup
helm uninstall my-release
cd ..
```

---

## PART 03: Lab - Advanced Multi-Node Scheduling

Docker Desktop only provides a single node. To demonstrate multi-node features like Anti-Affinity, we will use `kind` to spin up a local multi-node cluster.

### 1. Create the Multi-Node Cluster
We will create a cluster with 1 Control Plane and 2 Worker nodes.
```bash
kind create cluster --name k8s-101-demo --config part3-multinode/kind-config.yaml
```

Verify the nodes are ready:
```bash
kubectl get nodes
```
*You should see `k8s-101-demo-control-plane`, `k8s-101-demo-worker`, and `k8s-101-demo-worker2`.*

### 2. Demonstrate Pod Anti-Affinity
We want to ensure our high-availability Nginx pods are never scheduled on the same worker node. If one node goes down, the application stays up.

```bash
kubectl apply -f part3-multinode/01-deployment-anti-affinity.yaml
```

Verify they are on different nodes:
```bash
kubectl get pods -o wide
```
*Look at the `NODE` column. You should see one pod on `worker` and one pod on `worker2`.*

### 3. Cleanup
Once the demo is complete, you can destroy the kind cluster to free up resources.
```bash
kind delete cluster --name k8s-101-demo
```
