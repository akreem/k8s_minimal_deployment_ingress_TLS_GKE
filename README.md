# Kubernetes Minimal Deployment on GKE

A compact, production-minded Kubernetes example that deploys Nginx behind a GKE Ingress with Google-managed TLS.

This repository is intentionally small but complete: it demonstrates the minimum building blocks needed to run and expose a stateless web workload on Kubernetes.

## Why this project is useful

- Shows the full request path from Internet to pod
- Uses managed TLS certificates on GKE (no manual cert renewal)
- Keeps configuration externalized with ConfigMap
- Includes health endpoints and readiness/liveness probes
- Uses resource requests and limits for safer scheduling

## Architecture

Internet -> GKE Ingress (HTTPS) -> ClusterIP Service -> Deployment (3 replicas) -> Nginx pods

Domains:
- k8s.deployment.sh
- www.k8s.deployment.sh

## Repository structure

- README.md: project guide and runbook
- nginx-cm.yml: ConfigMap with Nginx config + HTML content
- my-dep.yml: Deployment definition
- nginx-service.yml: internal Service for pod traffic
- ingress.yml: external Ingress + ManagedCertificate

## What gets created

1. ConfigMap named nginx-config with:
- default.conf
- index.html

2. Deployment named my-dep with:
- 3 replicas
- nginx container
- mounted ConfigMap files
- health probes
- resource requests/limits

3. Service named my-dep-service:
- type ClusterIP
- port 80 -> targetPort 80

4. Ingress named my-dep-ingress:
- host-based routing for both domains
- Google-managed certificate attachment

## Prerequisites

- A running GKE cluster
- kubectl configured for that cluster
- A DNS zone where you can point A records to the Ingress IP
- These DNS records prepared:
	- k8s.deployment.sh
	- www.k8s.deployment.sh

## Quick start

Apply resources:

kubectl apply -f nginx-cm.yml
kubectl apply -f my-dep.yml
kubectl apply -f nginx-service.yml
kubectl apply -f ingress.yml

Check status:

kubectl get pods -l app=my-dep
kubectl get svc my-dep-service
kubectl get ingress my-dep-ingress
kubectl get managedcertificate my-ssl-cert

Fetch ingress address and update DNS if needed:

kubectl get ingress my-dep-ingress

Note: managed certificate provisioning usually takes 10-60 minutes after DNS is correct.

## Validation checklist

1. Pods are Running and Ready

kubectl get pods -l app=my-dep

Expected:
- 3/3 pods in Running state
- Ready column should be 1/1 per pod

2. Service has endpoints

kubectl get endpoints my-dep-service

Expected:
- one endpoint per ready pod

3. Health endpoint responds

curl http://k8s.deployment.sh/health

Expected output:
- healthy

4. TLS is active

curl -I https://k8s.deployment.sh/

Expected:
- HTTP response over HTTPS
- valid certificate for configured host

## Troubleshooting

Certificate stuck in Provisioning:
- Verify DNS points to the current Ingress IP
- Check events:
	kubectl describe managedcertificate my-ssl-cert

Ingress has no external IP yet:
- Wait a few minutes and re-check
- Confirm Ingress class/controller is correct for GKE

Pods not becoming ready:
- Inspect events and logs:
	kubectl describe pods -l app=my-dep
	kubectl logs -l app=my-dep

## Clean up

kubectl delete -f ingress.yml
kubectl delete -f nginx-service.yml
kubectl delete -f my-dep.yml
kubectl delete -f nginx-cm.yml

## Possible next improvements

- Add HorizontalPodAutoscaler manifest
- Add PodDisruptionBudget for safer node maintenance
- Add CI linting with kubeconform or kubeval
- Add Kustomize overlays for dev/stage/prod
