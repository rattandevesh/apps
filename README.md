# Apps

Helm charts for retail-store microservices and platform components.

## Structure

```
apps/
├── platform/              # Platform chart (ingress, certs, metrics, redis, postgres)
├── ui/                   # UI service chart
├── catalog/              # Catalog service chart
├── cart/                 # Cart service chart
├── orders/               # Orders service chart
├── checkout/             # Checkout service chart
└── README.md
```

## Usage

Each service chart includes environment-specific values files:

- `values-dev.yaml`: Development environment
- `values-stage.yaml`: Staging environment
- `values-prod.yaml`: Production environment

### Package and Lint

```bash
cd ui
helm lint .
helm package .
```

### Deploy

```bash
helm install ui ./ui -f ui/values-dev.yaml
```

## Features

- HPA (Horizontal Pod Autoscaler)
- PodDisruptionBudget
- Resource requests/limits
- Health probes (liveness, readiness)
- Ingress rules with path-based routing
- ConfigMap/Secret management
- Sealed Secrets/External Secrets integration
