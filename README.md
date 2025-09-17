# Kubernetes Nginx Deployment Project

A complete Kubernetes deployment project that runs a custom Nginx web server with SSL termination on Google Kubernetes Engine (GKE).

## Project Overview

This project deploys a custom Nginx application to Kubernetes with the following features:
- **Custom HTML content** with pod information display
- **SSL/TLS termination** using Google-managed certificates
- **Load balancing** across multiple replicas
- **Health checks** for monitoring
- **Custom Nginx configuration**

## Architecture

```
Internet → GKE Ingress (SSL) → Service → Deployment (3 replicas) → Nginx Pods
```

- **Domain**: `k8s.deployment.sh` and `www.k8s.deployment.sh`
- **SSL**: Google-managed certificates (automatic provisioning and renewal)
- **Load Balancer**: GCP HTTP(S) Load Balancer via Ingress
- **Replicas**: 3 Nginx pods for high availability

## Files Structure

```
k8s/
├── README.md              # This documentation
├── my-dep.yml            # Nginx deployment configuration
├── nginx-cm.yml          # ConfigMap with custom HTML and Nginx config
├── nginx-service.yml     # Service to expose the deployment
└── ingress.yml           # Ingress with SSL and routing rules
```

## Components

### 1. Deployment (`my-dep.yml`)
- **Image**: `nginx:latest`
- **Replicas**: 3 pods for high availability
- **Volumes**: ConfigMap mounted for custom configuration and HTML content

### 2. ConfigMap (`nginx-cm.yml`)
- **Custom HTML page**: Shows pod hostname and timestamp
- **Nginx configuration**: Custom server block with health endpoint
- **Health endpoint**: `/health` for monitoring

### 3. Service (`nginx-service.yml`)
- **Type**: ClusterIP (internal to cluster)
- **Port**: 80 (HTTP traffic within cluster)
- **Selector**: Routes traffic to pods with `app: my-dep` label

### 4. Ingress (`ingress.yml`)
- **Domains**: `k8s.deployment.sh` and `www.k8s.deployment.sh`
- **SSL**: Google-managed certificates
- **Path routing**: All traffic (`/`) to the Nginx service

## Prerequisites

- Google Kubernetes Engine (GKE) cluster
- `kubectl` configured to connect to your cluster
- Domain `k8s.deployment.sh` pointing to your GKE ingress IP

## Deployment Instructions

### 1. Apply the configurations in order:

```bash
# Create the ConfigMap first (contains HTML and Nginx config)
kubectl apply -f nginx-cm.yml

# Create the deployment
kubectl apply -f my-dep.yml

# Create the service
kubectl apply -f nginx-service.yml

# Create the ingress and managed certificate
kubectl apply -f ingress.yml
```

### 2. Verify the deployment:

```bash
# Check pod status
kubectl get pods -l app=my-dep

# Check service
kubectl get service my-dep-service

# Check ingress
kubectl get ingress my-dep-ingress

# Check managed certificate status
kubectl get managedcertificate my-ssl-cert
```

### 3. Monitor SSL certificate provisioning:

```bash
# Check certificate status (takes 10-60 minutes)
kubectl describe managedcertificate my-ssl-cert
```

## Testing

### Health Check
```bash
# Test health endpoint
curl http://k8s.deployment.sh/health
# Expected: "healthy"
```

### Load Balancing
```bash
# Multiple requests to see different pods
curl -H "X-Show-Pod: true" https://k8s.deployment.sh/
```

### SSL Verification
```bash
# Check SSL certificate
curl -I https://k8s.deployment.sh/
```

## Features

- **Custom HTML**: Interactive page showing pod information
- **Health Monitoring**: `/health` endpoint for load balancer health checks
- **SSL Termination**: Automatic HTTPS with Google-managed certificates
- **High Availability**: 3 replicas with load balancing
- **Custom Headers**: Pod name injection for debugging

## Troubleshooting

### SSL Certificate Issues
```bash
# Check certificate events
kubectl describe managedcertificate my-ssl-cert

# Common issues:
# - DNS not pointing to ingress IP
# - Domain verification pending
# - Certificate provisioning in progress (can take up to 60 minutes)
```

### Pod Issues
```bash
# Check pod logs
kubectl logs -l app=my-dep

# Check pod status
kubectl describe pods -l app=my-dep
```

### Service Connectivity
```bash
# Test service endpoint directly
kubectl port-forward service/my-dep-service 8080:80
# Then visit: http://localhost:8080
```

## Scaling

### Scale the deployment:
```bash
# Scale to 5 replicas
kubectl scale deployment my-dep --replicas=5

# Auto-scaling (optional)
kubectl autoscale deployment my-dep --cpu-percent=50 --min=3 --max=10
```

## Cleanup

```bash
# Remove all resources
kubectl delete -f ingress.yml
kubectl delete -f nginx-service.yml
kubectl delete -f my-dep.yml
kubectl delete -f nginx-cm.yml
```

## Notes

- SSL certificate provisioning can take 10-60 minutes
- Make sure your domain DNS points to the ingress IP address
- The managed certificate will automatically renew
- Health checks are configured at `/health` endpoint

## Monitoring

The application includes:
- Custom headers showing pod names
- Health check endpoint at `/health`
- Timestamp display on the web page
- Pod hostname display for load balancing verification
