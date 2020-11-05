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

It will produce K8s manifest with only a Deployment and a Service


### Bare minimum: regular K8s Deployment and Service - no Ingress, no Mesh

```PowerShell
helm upgrade -i -n default --dry-run `
 --set image.repository=fake/repo `
 --set env.Settings_Color=yellow `
 my-awesome-app-v1-0 `
 ./chart
```

Same as above only the Deployment will have `env` section with an environment variable.

### K8s Deployment, Service and Ingress - no Mesh

```PowerShell
helm upgrade -i -n default --dry-run `
 --set image.repository=fake/repo `
 --set ingress.enabled=true `
 my-awesome-app-v1-0 `
 ./chart
```

Should produce Ks manifest with Deployment, Service and AWS ALB Ingress.

### With App Mesh

```PowerShell
helm upgrade -i -n default --dry-run `
  --set image.repository=fake/repo `
  --set appMesh.enabled=true `
  my-awesome-app-v1-0 `
  ./chart
```

The output will have K8s Deployment and Service, plus App Mesh VirtualNode and a VirtualService.

### App Mesh with a GatewayRoute and a VirtualNode with a Backend

```PowerShell
helm upgrade -i -n default --dry-run `
  --set image.repository=fake/repo `
  --set appMesh.enabled=true `
  --set appMesh.backendVirtualServices[0]=back-svc-1 `
  --set appMesh.backendVirtualServices[1]=back-svc-2 `
  my-awesome-app-v1-0 `
  ./chart
```

The output will have K8s Deployment and Service, plus App Mesh VirtualNode having a Backend, a VirtualService and a GatewayRoute.