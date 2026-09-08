# Troubleshooting Guide

Common issues and solutions when working with Minikube and Kubernetes.

## Table of Contents

1. [Minikube Start Issues](#minikube-start-issues)
2. [Networking Issues](#networking-issues)
3. [Resource Issues](#resource-issues)
4. [Pod Issues](#pod-issues)
5. [Docker Issues](#docker-issues)
6. [kubectl Issues](#kubectl-issues)

## Minikube Start Issues

### Minikube fails to start

**Problem:** `minikube start` returns an error

**Solutions:**

```bash
# Check Minikube status
minikube status

# Reset Minikube
minikube delete
minikube start

# Start with verbose logging
minikube start -v=7

# Check Docker daemon
docker ps

# Verify Docker is running
sudo systemctl start docker
sudo systemctl enable docker
```

### Insufficient resources

**Problem:** `Error: Not enough memory or CPU`

**Solutions:**

```bash
# Check available system resources
free -h
nproc

# Reduce Minikube resource allocation
minikube delete

minikube config set memory 2048
minikube config set cpus 2

minikube start
```

### Driver not found

**Problem:** `Error: Unsupported driver: <driver>`

**Solutions:**

```bash
# Check available drivers
minikube config view

# Install Docker driver (recommended)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Set Docker as default driver
minikube config set driver docker

# Start Minikube
minikube start
```

### Docker daemon not running

**Problem:** `Error: docker daemon not responding`

**Solutions:**

```bash
# Check Docker status
sudo systemctl status docker

# Start Docker
sudo systemctl start docker

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Test Docker
docker ps
```

## Networking Issues

### Pods cannot reach external network

**Problem:** Pods fail to download images or reach external services

**Solutions:**

```bash
# Check DNS resolution
kubectl run -it --image=busybox --restart=Never dns-test -- nslookup google.com

# Restart Minikube with Flannel CNI
minikube delete
minikube start --cni=flannel

# Enable Docker environment
eval $(minikube docker-env)
```

### Ingress not working

**Problem:** Ingress resources exist but services are not accessible

**Solutions:**

```bash
# Enable Ingress addon
minikube addons enable ingress

# Verify Ingress controller is running
kubectl get pods -n ingress-nginx

# Get Minikube IP
minikube ip

# Add to /etc/hosts for local testing
echo "$(minikube ip) myapp.local" | sudo tee -a /etc/hosts

# Test Ingress
curl http://myapp.local
```

### Service LoadBalancer not accessible

**Problem:** LoadBalancer service shows EXTERNAL-IP as `<pending>`

**Solutions:**

```bash
# Enable MetalLB addon
minikube addons enable metallb

# Get Minikube IP and configure tunnel
minikube tunnel

# In another terminal, check service status
kubectl get svc

# Test service access
curl <external-ip>:<port>
```

## Resource Issues

### Out of disk space

**Problem:** `No space left on device`

**Solutions:**

```bash
# Check disk usage
minikube ssh
df -h

# Exit Minikube
exit

# Delete old containers/images
docker prune --all

# Increase Minikube disk size
minikube delete

minikube config set disk-size 50gb

minikube start
```

### Node out of memory

**Problem:** `OOMKilled` pods or `Evicted` pods

**Solutions:**

```bash
# Check node resources
kubectl top nodes
kubectl describe node

# Check pod resource requests
kubectl describe pod <pod-name>

# Increase Minikube memory
minikube delete

minikube config set memory 8192

minikube start
```

### CPU throttling

**Problem:** Pods running slowly or timing out

**Solutions:**

```bash
# Check current CPU allocation
minikube config view

# Increase CPU cores
minikube delete

minikube config set cpus 4

minikube start

# Check pod CPU usage
kubectl top pods --all-namespaces
```

## Pod Issues

### Pod stuck in Pending

**Problem:** Pod remains in `Pending` state

**Solutions:**

```bash
# Describe the pod to see events
kubectl describe pod <pod-name>

# Check node status
kubectl get nodes
kubectl describe node

# Check resource availability
kubectl top nodes

# Delete and recreate pod
kubectl delete pod <pod-name>
kubectl apply -f <pod-manifest>
```

### Pod stuck in ImagePullBackOff

**Problem:** `Failed to pull image from registry`

**Solutions:**

```bash
# Check pod events
kubectl describe pod <pod-name>

# Verify image exists
docker images

# For local images, point Docker to Minikube's Docker daemon
eval $(minikube docker-env)
docker build -t myapp:latest .

# Set imagePullPolicy to Never for local images
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:latest
    imagePullPolicy: Never
EOF
```

### Pod CrashLoopBackOff

**Problem:** Pod container crashes and restarts repeatedly

**Solutions:**

```bash
# Check pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Check pod events
kubectl describe pod <pod-name>

# Execute debug shell
kubectl exec -it <pod-name> -- /bin/sh

# Increase pod restart policy
kubectl set env deployment <deployment-name> RESTART_POLICY=Always
```

## Docker Issues

### Cannot build images in Minikube

**Problem:** `docker build` fails or images not visible in Minikube

**Solutions:**

```bash
# Point Docker to Minikube
eval $(minikube docker-env)

# Verify Docker context
docker context ls

# Build image
docker build -t myapp:latest .

# List images
docker images

# Push to Minikube registry
docker push myapp:latest
```

### Docker socket permission denied

**Problem:** `permission denied while trying to connect to Docker daemon`

**Solutions:**

```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Apply group membership
newgrp docker

# Test Docker access
docker ps

# If still failing, check Docker socket
ls -la /var/run/docker.sock
```

## kubectl Issues

### kubectl context not set to Minikube

**Problem:** `kubectl` communicates with wrong cluster

**Solutions:**

```bash
# List available contexts
kubectl config get-contexts

# Switch to Minikube context
kubectl config use-context minikube

# Verify connection
kubectl cluster-info
kubectl get nodes
```

### kubectl commands hang

**Problem:** `kubectl` command never returns

**Solutions:**

```bash
# Check API server connectivity
kubectl cluster-info

# Restart Minikube
minikube stop
minikube start

# Check kubectl logs
kubectl logs kube-apiserver -n kube-system

# Increase kubectl timeout
kubectl --request-timeout=30s get pods
```

### Cannot connect to Minikube API server

**Problem:** `Unable to connect to the server`

**Solutions:**

```bash
# Start Minikube
minikube start

# Configure kubectl
minikube update-context

# Verify kubeconfig
cat ~/.kube/config

# Reset kubeconfig
rm ~/.kube/config
minikube start
```

---

**Last Updated:** 2024

For additional help:
- [Minikube Issues](https://github.com/kubernetes/minikube/issues)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubectl Debugging Guide](https://kubernetes.io/docs/tasks/debug-application-cluster/)
