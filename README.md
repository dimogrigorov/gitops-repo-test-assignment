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

# For manual applying of app1 helm run:
helm install my-app helm-charts/app1-helm-chart -f helm-charts/app1-helm-chart/values-staging.yaml
