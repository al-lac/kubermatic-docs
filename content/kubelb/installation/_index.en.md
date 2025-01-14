+++
title = "Installation guide for KubeLB"
date = 2023-10-27T10:07:15+02:00
+++

## Installation

### Prerequisites for KubeLB

{{% notice note %}}
KubeLB is an enterprise offering by Kubermatic. To use KubeLB, you need to have a valid license.
Please [contact sales](mailto:sales@kubermatic.com) for more information.
{{% /notice %}}

#### Consumer cluster

* KubeLB manager cluster API access.
* Registered as a tenant in the KubeLB manager cluster.

#### Load balancer cluster

* Service type `LoadBalancer` implementation. This can be a cloud solution or a self-managed implementation like [MetalLB](https://metallb.universe.tf).
* Network access to the consumer cluster nodes with node port range (default: 30000-32767). This is required for the envoy proxy to be able to connect to the consumer cluster nodes.

### Installation for KubeLB manager

KubeLB manager is deployed as a Kubernetes application. It can be deployed using the KubeLB manager Helm chart in the following way:

#### Prerequisites

* Create a namespace `kubelb` for the CCM to be deployed in.
* Create `imagePullSecrets` for the chart to pull the image from the registry.

At this point a minimal values.yaml should look like this:

```yaml
imagePullSecrets:
  - name: <imagePullSecretName>
```

#### Install helm chart for KubeLB manager

Now, we can install the helm chart:

```sh
helm registry login quay.io --username ${REGISTRY_USER} --password ${REGISTRY_PASSWORD}
helm pull oci://quay.io/kubermatic/helm-charts/kubelb-manager --version=v0.4.0 --untardir "kubelb-manager" --untar
## Create and update values.yaml with the required values.
helm install kubelb-manager kubelb-manager --namespace kubelb -f values.yaml
```

#### Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` |  |
| autoscaling.enabled | bool | `false` |  |
| autoscaling.maxReplicas | int | `10` |  |
| autoscaling.minReplicas | int | `1` |  |
| autoscaling.targetCPUUtilizationPercentage | int | `80` |  |
| autoscaling.targetMemoryUtilizationPercentage | int | `80` |  |
| fullnameOverride | string | `""` |  |
| image.pullPolicy | string | `"IfNotPresent"` |  |
| image.repository | string | `"quay.io/kubermatic/kubelb-manager-ee"` |  |
| image.tag | string | `"v0.4.0"` |  |
| imagePullSecrets | list | `[]` |  |
| kubelb.debug | bool | `false` |  |
| kubelb.enableLeaderElection | bool | `true` |  |
| kubelb.envoyProxyReplicas | int | `3` |  |
| kubelb.envoyProxySinglePodPerNode | bool | `true` |  |
| kubelb.envoyProxyTopology | string | `"shared"` |  |
| kubelb.envoyProxyUseDaemonset | bool | `false` |  |
| nameOverride | string | `""` |  |
| nodeSelector | object | `{}` |  |
| podAnnotations | object | `{}` |  |
| podLabels | object | `{}` |  |
| podSecurityContext.runAsNonRoot | bool | `true` |  |
| podSecurityContext.seccompProfile.type | string | `"RuntimeDefault"` |  |
| rbac.allowLeaderElectionRole | bool | `true` |  |
| rbac.allowMetricsReaderRole | bool | `true` |  |
| rbac.allowProxyRole | bool | `true` |  |
| rbac.enabled | bool | `true` |  |
| replicaCount | int | `1` |  |
| resources.limits.cpu | string | `"100m"` |  |
| resources.limits.memory | string | `"128Mi"` |  |
| resources.requests.cpu | string | `"100m"` |  |
| resources.requests.memory | string | `"128Mi"` |  |
| securityContext.allowPrivilegeEscalation | bool | `false` |  |
| securityContext.capabilities.drop[0] | string | `"ALL"` |  |
| securityContext.runAsUser | int | `65532` |  |
| service.port | int | `8001` |  |
| service.protocol | string | `"TCP"` |  |
| service.type | string | `"ClusterIP"` |  |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |
| serviceMonitor.enabled | bool | `false` |  |
| tolerations | list | `[]` |  |

### Installation for KubeLB CCM

#### Pre-requisites

* Create a namespace `kubelb` for the CCM to be deployed in.
* Create `imagePullSecrets` for the chart to pull the image from the registry.
* The agent expects a `Secret` with a kubeconf file named `kubelb` to access the load balancer cluster. To create such run: `kubectl --namespace kubelb create secret generic kubelb-cluster --from-file=<path to kubelb kubeconf file>`. The name of secret can't be overridden using `.Values.kubelb.clusterSecretName`
* Update the `tenantName` in the `values.yaml` to a unique identifier for the tenant. This is used to identify the tenant in the manager cluster. This can be any unique string that follows [lower case RFC 1123](https://www.rfc-editor.org/rfc/rfc1123).

At this point a minimal `values.yaml` should look like this:

```yaml
imagePullSecrets:
  - name: <imagePullSecretName>
kubelb:
    clusterSecretName: kubelb-cluster
    tenantName: <unique-identifier-for-tenant>
```

#### Install helm chart for KubeLB CCM

Now, we can install the helm chart:

```sh
helm registry login quay.io --username ${REGISTRY_USER} --password ${REGISTRY_PASSWORD}
helm pull oci://quay.io/kubermatic/helm-charts/kubelb-ccm --version=v0.4.0 --untardir "kubelb-ccm" --untar
## Create and update values.yaml with the required values.
helm install kubelb-ccm kubelb-ccm --namespace kubelb -f values.yaml
```

#### Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` |  |
| autoscaling.enabled | bool | `false` |  |
| autoscaling.maxReplicas | int | `10` |  |
| autoscaling.minReplicas | int | `1` |  |
| autoscaling.targetCPUUtilizationPercentage | int | `80` |  |
| autoscaling.targetMemoryUtilizationPercentage | int | `80` |  |
| extraVolumeMounts | list | `[]` |  |
| extraVolumes | list | `[]` |  |
| fullnameOverride | string | `""` |  |
| image.pullPolicy | string | `"IfNotPresent"` |  |
| image.repository | string | `"quay.io/kubermatic/kubelb-manager-ee"` |  |
| image.tag | string | `"v0.4.0"` |  |
| imagePullSecrets | list | `[]` |  |
| kubelb.clusterSecretName | string | `"kubelb-cluster"` |  |
| kubelb.enableLeaderElection | bool | `true` |  |
| kubelb.nodeAddressType | string | `"InternalIP"` |  |
| kubelb.tenantName | string | `nil` |  |
| nameOverride | string | `""` |  |
| nodeSelector | object | `{}` |  |
| podAnnotations | object | `{}` |  |
| podLabels | object | `{}` |  |
| podSecurityContext.runAsNonRoot | bool | `true` |  |
| podSecurityContext.seccompProfile.type | string | `"RuntimeDefault"` |  |
| rbac.allowLeaderElectionRole | bool | `true` |  |
| rbac.allowMetricsReaderRole | bool | `true` |  |
| rbac.allowProxyRole | bool | `true` |  |
| rbac.enabled | bool | `true` |  |
| replicaCount | int | `1` |  |
| resources.limits.cpu | string | `"100m"` |  |
| resources.limits.memory | string | `"128Mi"` |  |
| resources.requests.cpu | string | `"100m"` |  |
| resources.requests.memory | string | `"128Mi"` |  |
| securityContext.allowPrivilegeEscalation | bool | `false` |  |
| securityContext.capabilities.drop[0] | string | `"ALL"` |  |
| securityContext.runAsUser | int | `65532` |  |
| service.port | int | `8443` |  |
| service.protocol | string | `"TCP"` |  |
| service.type | string | `"ClusterIP"` |  |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |
| serviceMonitor.enabled | bool | `false` |  |
| tolerations | list | `[]` |  |
