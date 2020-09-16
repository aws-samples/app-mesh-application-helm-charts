## Pre-requisites

- kubeconfig pointing to an existing AWS EKS Kubernetes cluster
- [Cross-platform PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell) script snippets here compatible with Windows, MacOS and Linux. 
  (**Bash** users may replace ` with \\ instead of installing PowerShell.)

## Helm commands

### All defaults

```PowerShell
helm upgrade -i --dry-run `
 my-canary `
 ./chart
```

You should get an error saying "*Valid 'selectors' for the Service are required*".

### Add pod label selector

```PowerShell
helm upgrade -i --dry-run `
 --set service.selectorLabels.app=my-app `
 my-canary `
 ./chart
```

In real life you will want to set `selectorlabels` in the "values.yaml" file rather than override the value from the command line.

The **output** should create only Kubernetes Service:

```yaml
---
# Source: canary/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-canary
  labels:
    app: canary
    chart: canary-0.1.0
    release: my-canary
    heritage: Helm
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app: my-app
```

### With App Mesh

```PowerShell
helm upgrade -i --dry-run `
 --set service.selectorLabels.app=my-app `
 --set appMesh.enabled=true `
 color-canary `
 ./chart
```
You should get an error message saying "*At least one target VirtualNode has to be specified*".


### With App Mesh and VirtualNode targets

```PowerShell
helm upgrade -i --dry-run `
 --set service.selectorLabels.app=my-app `
 --set appMesh.enabled=true `
 --set appMesh.targets[0].vNodeName=my-app-v1 `
 --set appMesh.targets[0].weight=1 `
 --set appMesh.targets[1].vNodeName=my-app-v2 `
 --set appMesh.targets[1].weight=1 `
 color-canary `
 ./chart
```

The output should create Kubernetes Service, and AppMesh VirtualRouter and VirtualService.

```yaml
---
# Source: canary/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: color-canary
  labels:
    app: canary
    chart: canary-0.1.0
    release: color-canary
    heritage: Helm
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app: my-app
---
# Source: canary/templates/virtual-router.yaml
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualRouter
metadata:
  name: color-canary
  labels:
    app: canary
    chart: canary-0.1.0
    release: color-canary
    heritage: Helm
spec:
  listeners:
    - portMapping:
        port: 80
        protocol: http
  routes:
    - name: color-canary
      httpRoute:
        match:
          prefix: /
        action:
          weightedTargets:
            - virtualNodeRef:
                name: my-app-v1
              weight: 1
            - virtualNodeRef:
                name: my-app-v2
              weight: 1
---
# Source: canary/templates/virtual-service.yaml
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualService
metadata:
  name: color-canary
  labels:
    app: canary
    chart: canary-0.1.0
    release: color-canary
    heritage: Helm
spec:
  awsName: color-canary.default.svc.cluster.local
  provider:
    virtualRouter:
      virtualRouterRef:
        name: color-canary
```

### With App Mesh, VirtualNode targets and a GatewayRoute

Optional GatewayRoute allows traffic originating outside of the mesh to reach the canary.

```PowerShell
helm upgrade -i --dry-run `
 --set service.selectorLabels.app=my-app `
 --set appMesh.enabled=true `
 --set appMesh.targets[0].vNodeName=my-app-v1 `
 --set appMesh.targets[0].weight=1 `
 --set appMesh.targets[1].vNodeName=my-app-v2 `
 --set appMesh.targets[1].weight=1 `
 --set appMesh.outsideTrafficAllowed=true `
 color-canary `
 ./chart
```

The output should create Kubernetes Service, and AppMesh VirtualRouter and VirtualService.

```yaml
---
# Source: canary/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: color-canary
  labels:
    app: canary
    chart: canary-0.1.0
    release: color-canary
    heritage: Helm
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app: my-app
---
# Source: canary/templates/gateway-route.yaml
apiVersion: appmesh.k8s.aws/v1beta2
kind: GatewayRoute
metadata:
  name: color-canary
  labels:
    app: canary
    chart: canary-0.1.0
    release: color-canary
    heritage: Helm
spec:
  httpRoute:
    match:
      prefix: /color-canary
    action:
      target:
        virtualService:
          virtualServiceRef:
            name: color-canary
---
# Source: canary/templates/virtual-router.yaml
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualRouter
metadata:
  name: color-canary
  labels:
    app: canary
    chart: canary-0.1.0
    release: color-canary
    heritage: Helm
spec:
  listeners:
    - portMapping:
        port: 80
        protocol: http
  routes:
    - name: color-canary
      httpRoute:
        match:
          prefix: /
        action:
          weightedTargets:
            - virtualNodeRef:
                name: my-app-v1
              weight: 1
            - virtualNodeRef:
                name: my-app-v2
              weight: 1
---
# Source: canary/templates/virtual-service.yaml
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualService
metadata:
  name: color-canary
  labels:
    app: canary
    chart: canary-0.1.0
    release: color-canary
    heritage: Helm
spec:
  awsName: color-canary.default.svc.cluster.local
  provider:
    virtualRouter:
      virtualRouterRef:
        name: color-canary
```