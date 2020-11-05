## Canary Helm Chart

This chart can be used for creating permanent, version-agnostic network endpoint for multiple and/or changing versions of and application. 

This chart can be used for deploying to EKS namespaces that are part of an App Mesh, as well as for deploying to regular, non-meshed Kubernetes namespaces. The chart has an option of allowing outside-the-cluster traffic to flow to the canary endpoint.

### To shift traffic from one version to another

When canary endpoint deployed to a plain, non-meshed Kubernetes namespace, it is done using Kubernetes Service and therefore requires different replica counts for different application versions to implement, say, 80/20 traffic split between application versions, which is less than ideal, especially on Fargate where it may lead to node over-provisioning.

When deployed to an App Mesh, VirtualRoutes allow for fine-grained traffic split regardless of the number of container replica of each application version.

### Different destinations for Mesh vs no Mesh

To specify destinations for canary traffic, for plain Kubernetes case, "`service.selectorLabels`" specifies a set of Pods that will receive the traffic routed in pseudo-random manner.

When App Mesh is employed, "`appMesh.targets`" parameter specifies VirtualNodes and their "weight" for traffic distribution.

Explore Helm Chart [values.yaml](../canary/chart/values.yaml) for details.

### Different ways to let external traffic in, depending on the presence of the App Mesh

For a regular, non-mesh installation, Ingress resource may be created using "`ingress.enabled`" parameter.

For App Mesh case, "`appMesh.outsideTrafficAllowed`" parameter could be set to true to create GatewayRoute resource.