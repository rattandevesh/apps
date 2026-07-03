# Apps Runbook

This runbook provides step-by-step instructions for packaging and deploying applications with Helm charts.

## Prerequisites

- Helm >= 3.0 installed
- kubectl configured
- Access to apps repository
- Kubernetes cluster running

## Initial Setup

### 1. Clone Repository

```bash
cd /Users/devesh/workspace/assignment
git clone <apps-repo> apps
cd apps
```

### 2. Add Helm Repositories

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add jetstack https://charts.jetstack.io
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## Chart Development

### Create New Chart

```bash
helm create my-service
```

### Lint Chart

```bash
cd my-service
helm lint .
```

### Test Chart Template

```bash
helm template my-service . -f values-dev.yaml
```

### Package Chart

```bash
helm package .
```

## Deployment

### Deploy to Development

```bash
cd ui
helm install retail-store-ui . -f values-dev.yaml --namespace retail-store --create-namespace
```

### Deploy to Staging

```bash
helm install retail-store-ui . -f values-stage.yaml --namespace retail-store --create-namespace
```

### Deploy to Production

```bash
helm install retail-store-ui . -f values-prod.yaml --namespace retail-store --create-namespace
```

## Day 2 Operations

### Upgrade Application

```bash
helm upgrade retail-store-ui . -f values-dev.yaml --namespace retail-store
```

### Rollback Application

```bash
helm rollback retail-store-ui -n retail-store
```

### Scale Application

Edit values file and upgrade:

```bash
# Edit values-dev.yaml to change replicaCount
helm upgrade retail-store-ui . -f values-dev.yaml --namespace retail-store
```

### Update Image

```bash
# Edit values-dev.yaml to change image.tag
helm upgrade retail-store-ui . -f values-dev.yaml --namespace retail-store
```

### Pin Image Digest

```bash
# Get image digest
docker pull public.ecr.aws/aws-containers/retail-store-sample-ui:1.0.0
docker inspect public.ecr.aws/aws-containers/retail-store-sample-ui:1.0.0 | grep Digest

# Edit values-prod.yaml to set imageDigest
helm upgrade retail-store-ui . -f values-prod.yaml --namespace retail-store
```

## Validation

### Check Deployment Status

```bash
kubectl get deployments -n retail-store
kubectl get pods -n retail-store
kubectl get svc -n retail-store
```

### Check HPA Status

```bash
kubectl get hpa -n retail-store
kubectl describe hpa retail-store-ui -n retail-store
```

### Check Ingress

```bash
kubectl get ingress -n retail-store
kubectl describe ingress retail-store-ui -n retail-store
```

### Check Resource Usage

```bash
kubectl top pods -n retail-store
kubectl top nodes
```

## Troubleshooting

### Pod Not Starting

```bash
kubectl describe pod <pod-name> -n retail-store
kubectl logs <pod-name> -n retail-store
kubectl logs <pod-name> -n retail-store --previous
```

### Image Pull Errors

```bash
kubectl describe pod <pod-name> -n retail-store
# Check imagePullSecrets if using private registry
```

### HPA Not Scaling

```bash
kubectl describe hpa <hpa-name> -n retail-store
kubectl describe deployment <deployment-name> -n retail-store
# Check metrics server is running
kubectl get pods -n kube-system | grep metrics-server
```

### Ingress Not Working

```bash
kubectl describe ingress <ingress-name> -n retail-store
kubectl get svc -n ingress-nginx
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
```

### Resource Limits Exceeded

```bash
kubectl describe pod <pod-name> -n retail-store
# Check OOMKilled in events
# Adjust resources in values file
```

## Rollback Procedures

### Application Rollback

```bash
# View history
helm history retail-store-ui -n retail-store

# Rollback to specific revision
helm rollback retail-store-ui <revision> -n retail-store
```

### Emergency Rollback

```bash
# Quick rollback to previous version
helm rollback retail-store-ui -n retail-store

# Verify
kubectl rollout status deployment/retail-store-ui -n retail-store
```

## Monitoring

Set up monitoring for:
- Pod health
- Resource utilization
- HPA scaling events
- Ingress metrics
- Application metrics (if instrumented)

## Security

### Scan Images

```bash
trivy image public.ecr.aws/aws-containers/retail-store-sample-ui:1.0.0
```

### Check for Vulnerabilities

```bash
helm audit retail-store-ui -n retail-store
```

### Validate RBAC

```bash
kubectl auth can-i list pods --as=system:serviceaccount:retail-store:default
```

## Best Practices

1. **Always use image digests in production**
2. **Set appropriate resource requests and limits**
3. **Enable HPA for stateless services**
4. **Use PodDisruptionBudgets for high availability**
5. **Implement health checks (liveness, readiness)**
6. **Use ConfigMaps for configuration**
7. **Use Secrets for sensitive data**
8. **Never use :latest tags**
9. **Test in dev before promoting to stage/prod**
10. **Keep charts in version control**

## Promotion Workflow

### Dev to Stage

1. Update image tag in values-stage.yaml
2. Test in stage environment
3. Verify functionality
4. Promote to prod if successful

### Stage to Prod

1. Update image tag in values-prod.yaml
2. Pin image digest
3. Deploy to prod
4. Monitor for issues
5. Rollback if needed
