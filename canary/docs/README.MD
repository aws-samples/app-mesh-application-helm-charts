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

The output should contain single Kubernetes Service:

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
