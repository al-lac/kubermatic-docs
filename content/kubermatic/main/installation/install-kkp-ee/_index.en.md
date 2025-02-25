+++
title = "Install Kubermatic Kubernetes Platform (KKP) EE"
linkTitle = "Install Enterprise Edition"
date = 2018-04-28T12:07:15+02:00
description = "Learn the installation procedure of KKP Enterprise Edition (EE) into a pre-existing Kubernetes cluster"
weight = 20
enterprise = true
+++

This chapter explains the installation procedure of KKP Enterprise Edition (EE) into a pre-existing
Kubernetes cluster.

{{% notice note %}}
At the moment you need to be invited to get access to Kubermatic's EE Docker repository before you can try it out.
Please [contact sales](mailto:sales@kubermatic.com) to receive your credentials.
{{% /notice %}}

## Terminology

In this chapter, you will find the following KKP-specific terms:

* **Master Cluster** -- A Kubernetes cluster which is responsible for storing central information about users, projects and SSH keys. It hosts the KKP master components and might also act as a seed cluster.
* **Seed Cluster** -- A Kubernetes cluster which is responsible for hosting the control plane components (kube-apiserver, kube-scheduler, kube-controller-manager, etcd and more) of a User Cluster.
* **User Cluster** -- A Kubernetes cluster created and managed by KKP, hosting applications managed by users.

It is also recommended to make yourself familiar with our [architecture documentation]({{< ref "../../architecture/" >}}).

## Installation

The installation procedure is identical to the [installation process for the Community Edition]({{< ref "../install-kkp-ce" >}}),
with the exception that a different installer needs to be downloaded and that the Docker credentials need to be configured.

When downloading the installer, make sure to choose the `-ee-` variant on GitHub. Extract it like documented in the CE install
guide.

During configuration, it's required to set the Docker Pull Secret, which allows the local Docker daemons to pull the KKP
images from the private Docker repository. The Docker Pull Secret is a tiny JSON snippet and needs to be configured in the
KubermaticConfiguration (e.g. in the `kubermatic.yaml`):

```yaml
apiVersion: kubermatic.k8c.io/v1
kind: KubermaticConfiguration
metadata:
  name: kubermatic
  namespace: kubermatic
spec:
  # skipping everything else in this file
  # for demonstration purposes

  # This is where the JSON snippet needs to be configured. It does not need to be
  # a multiline JSON string.
  imagePullSecret: |
    {
      "auths": {
        "quay.io": {....}
      }
    }
```

Follow the CE install guide as normal, the remaining steps apply equally to the Enterprise Edition.

### Next Steps

* [Add a Seed cluster]({{< ref "./add-seed-cluster" >}}) to start creating user clusters.
* Install the [Master / Seed Monitoring, Logging & Alerting Stack]({{< ref "../../tutorials-howtos/monitoring-logging-alerting/master-seed/installation" >}}) to collect cluster-wide metrics in a central place.
