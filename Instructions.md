# URL Shortener Project: Setup & Operations Guide

This guide provides step-by-step instructions to set up and run the URL shortener application with Kubernetes, including Horizontal Pod Autoscaler (HPA) and stress testing.

## Setup Instructions

### Step 1: Start Minikube
```bash
minikube start --driver=docker
minikube status
```

### Step 2: Configure Docker to Use Minikube's Docker Daemon
```bash
eval $(minikube docker-env)
```

### Step 3: Build the URL Shortener Image
```bash
docker build -t urlshortener:latest .
```

### Step 4: Deploy Kubernetes Resources
Apply the Kubernetes configurations in the following order:

```bash
# Apply MySQL configurations
kubectl apply -f k8s/mysql-secret.yaml
kubectl apply -f k8s/mysql-pv.yaml
kubectl apply -f k8s/mysql-configmap.yaml

# Apply URL shortener configurations
kubectl apply -f k8s/urlshortener-config.yaml
kubectl apply -f k8s/urlshortener-hpa.yaml
kubectl apply -f k8s/mysql-deployment.yaml
kubectl apply -f k8s/urlshortener-deployment.yaml
```

## Operations Guide

### Accessing the Services

You'll need multiple terminal windows to properly run and monitor the application:

#### Terminal 1: Create External Access with Minikube Tunnel
```bash
minikube tunnel
```
Keep this terminal running for the duration of your testing.

#### Terminal 2: Port-Forward MySQL Service (for Database Access)
```bash
kubectl port-forward service/mysql-service 3307:3306
```
This allows connecting to the MySQL database from your local machine using port 3307.

#### Terminal 3: Application Testing and Monitoring
Get the service URL:
```bash
kubectl get service urlshortener-service
```

Run API tests:
```bash
./test_api.sh
```

Run stress tests:
```bash
./test_api.sh stress
```

#### Terminal 4: Monitor CPU Usage and HPA
Check HPA status:
```bash
kubectl get hpa urlshortener-hpa
```

Monitor CPU usage in real-time:
```bash
watch -n 2 "kubectl top pods | grep urlshortener"
```

## Monitoring and Troubleshooting

### Check Pod Status
```bash
kubectl get pods
```

### View Logs for a Specific Pod
```bash
kubectl logs <pod-name>
```

### Get Detailed Information About the HPA
```bash
kubectl describe hpa urlshortener-hpa
```

### Access MySQL Database
You can connect to the MySQL database using a MySQL client:
```
Host: 127.0.0.1
Port: 3307
User: root
Password: <password-from-mysql-secret>
Database: url_shortener
```

## Cleanup

When you're done testing:

1. Stop port-forwarding and monitoring (Ctrl+C in respective terminals)
2. Stop the minikube tunnel (Ctrl+C)
3. Delete the Kubernetes resources:
   ```bash
   kubectl delete -f k8s/
   ```
4. Optionally, stop minikube:
   ```bash
   minikube stop
   ``` 