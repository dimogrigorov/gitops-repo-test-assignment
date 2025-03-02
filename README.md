# Repository Structure Example

```plaintext
gitops-repo-test-assignment/  # Dedicated GitOps repository for Flux
├── my-cluster/
│   ├── staging/
│   │   ├── app1/
│   │   │   ├── kustomization.yaml   # Flux Kustomization resource
│   │   │   ├── helmrelease.yaml     # Flux HelmRelease for App1
│   │   │   ├── values-staging.yaml  # Staging-specific Helm values
│   ├── production/
│   │   ├── app1/
│   │   │   ├── kustomization.yaml   # Flux Kustomization resource
│   │   │   ├── helmrelease.yaml     # Flux HelmRelease for App1
│   │   │   ├── values-prod.yaml     # Production-specific Helm values
├── helm-charts/
│   ├── app1/
│   │   ├── Chart.yaml
│   │   ├── values.yaml              # Default Helm values
│   │   ├── templates/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   ├── ingress.yaml
```

## Implementing Automated Rollbacks in Case of Failures
To implement automated rollbacks, we use Helm, Kubernetes health checks, and Flux’s/ArgoCD’s rollback mechanisms.

### 1. Helm Rollback
Helm provides a built-in rollback command that can be triggered when a deployment fails.

**Enable Helm Rollback in Flux:**

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: my-app
  namespace: my-namespace
spec:
  chart:
    spec:
      chart: my-app
      sourceRef:
        kind: HelmRepository
        name: my-app-repo
  interval: 5m
  install:
    remediation:
      retries: 3
  upgrade:
    remediation:
      retries: 3
      strategy: rollback
```

**Manual Helm Rollback:**

```bash
helm rollback my-app <previous-release-revision> -n my-namespace
```

### 2. Kubernetes Readiness & Liveness Probes
Proper liveness and readiness probes ensure that only healthy pods serve traffic. A fully working `/health` endpoint is assumed.

**Define probes in Helm `values.yaml`:**

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

If the pod fails, Kubernetes will automatically restart it. Combined with Helm’s rollback strategy, this ensures failed deployments are automatically undone.

### 3. Kubernetes Deployment Strategy
A rolling update strategy avoids downtime and ensures new pods are only created if the previous ones are running.

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

### 4. ArgoCD Automated Rollback
*(Not yet implemented)*

If using ArgoCD, it can automatically roll back a failed deployment.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  destination:
    server: https://kubernetes.default.svc
    namespace: my-namespace
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
  retry:
    limit: 5
    backoff:
      duration: "5s"
      factor: 2
      maxDuration: "3m"
```

The retry policy allows failed deployments to retry a few times before rolling back.


