# OVerview

TBD

## "helm install" examples

(Examples are in PowerShell. **Bash** users may replace ` with \.)

To run examples, please `cd` to the "application" directory.

### Failure: missing image repository

```PowerShell
helm upgrade -i -n default --dry-run `
 my-awesome-app-v1-0 `
 ./chart
```
Should get error message about missing image repository.

### Bare minimum: regular K8s Deployment and Service - no Ingress, no Mesh

```PowerShell
helm upgrade -i -n default --dry-run `
 --set image.repository=fake/repo `
 my-awesome-app-v1-0 `
 ./chart
```

Should produce K8s manifest with only a Deployment and a Service

### K8s Deployment, Service and Ingress - no Mesh

```PowerShell
helm upgrade -i -n default --dry-run `
 --set image.repository=fake/repo `
 --set ingress.enabled=true `
 my-awesome-app-v1-0 `
 ./chart
```

Should produce Ks manifest with Deployment, Service and AWS ALB Ingress.
