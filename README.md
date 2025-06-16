# Hub Standard Services
This repo contains helm charts and general automation for installing and configuring the [standard services](https://github.com/RedHatEdge/patterns/blob/main/patterns/rh-hub-standard-services/README.md) that run on a Hub, which is used to manage [ACPs](https://github.com/RedHatEdge/patterns/blob/main/patterns/acp-standardized-architecture-ha/README.md) at scale.

This repo is intended to be used with the declarative state management service on a hub. Bootstrapping that service may be required.

[Helm](https://helm.sh/) is used to offer templating functionality around the installation and configuration of the services. Helm clients can be downloaded [here](https://helm.sh/docs/intro/install/).

## DISCLAIMER
This repository is currently a work in progress. The included charts are not guaranteed to work, however work is being done to bring them to a production-ready state.

## Bootstrapping the Declarative State Management Service
Refer to the [ACP Standard Services](https://github.com/RedHatEdge/acp-standard-services-public?tab=readme-ov-file#bootstrapping-the-declarative-state-management-service) repository to bootstrap the declarative state management service, and for the available automation.

The installation should only take a few moments.

## Structure
Within the charts directory, a parent chart named 'hub-standard-services' will create the [parent application](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/) which then installs the child applications, which are the individual services.

The deployment of these services is controlled by template variables, allowing for fine-tuning of what services should be deployed.

## Services
Below are the services deployed by the parent `acp-standard-services` application.

### Advanced Cluster Management
[Advanced Cluster Management](https://www.redhat.com/en/technologies/management/advanced-cluster-management) provides the zero-touch provisioning, management, and policy functionality used to manage ACPs at scale. In addition, [Red Hat Edge Manager](https://www.redhat.com/en/about/press-releases/red-hat-introduces-red-hat-edge-manager-overseeing-fleets-devices) can be installed and managed by ACM. To deploy this service, define the following:

```yaml
---
advancedClusterManagement:
  # Be sure to use 2.13+ if Red Hat Edge Manager is desired
  version: 2.13
  # If the hub should manage itself
  disableHubSelfManagement: false
  # If RHEM should be installed as well
  enableRHEM: true
```

### Central Infrastructure Management
The [Central Infrastructure Management](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.13/html/clusters/cluster_mce_overview#cluster-lifecycle-arch) service of ACM backs the ACP lifecycle management services, and requires storage to be enabled. To deploy this functionality, define the following:

```yaml
---
AgentServiceConfig:
  database:
    storage: 100Gi
    storageClass: ocs-storagecluster-ceph-rbd
  filesystem:
    storage: 100Gi
    storageClass: ocs-storagecluster-cephfs
  imageStorage:
    storage: 100Gi
    storageClass: ocs-storagecluster-ceph-rbd
```

### Cluster Infrastructure Environments
Cluster infrastructure environments represent locations where ACPs will be operated, which share network settings and use the same base credentials for accessing ACP content from an upstream or mirrored source.

To create a cluster infrastructure environment, define the following.

```yaml
clusterInfrastructureEnvironment:
  pullSecret: 'your-pull-secret'
  sshKey: 'your-ssh-key'

  environments:
    # Name of infra env
  - name: msp-store-1
    # Optional location used as a tag
    location: US-Central
  - name: msp-store-2
    location: US-Central
  - name: msp-store-3
    location: US-Central
  - name: bos-store-1
    location: US-Eastern
  - name: bos-store-2
    location: US-Eastern
  - name: bos-store-3
    location: US-Eastern
  - name: lax-store-1
    location: US-Pacific
  - name: lax-store-2
    location: US-Pacific
  - name: lax-store-3
    location: US-Pacific
```

### Assisted Cluster Install
Assisted cluster installs leverage services within ACM to provide a low/no-touch experience for building bare-metal ACPs.

To initiate a cluster install, define the following:

```yaml
assistedClusterInstalls:
  openshiftToolsImage: quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:96f217fd794eed50ee2c075d44ec56b7f8cfed4370662df59dc06c70ac2d67c0
  clusters:
    - name: new-cluster
      baseDomain: site1.company.com
      env: site1
      platformType: BareMetal
      version: 4.18.6
      networking:
      machineNetwork:
        - cidr: 192.168.0.0/24
      clusterNetwork:
        - cidr: 10.128.0.0/14
          hostPrefix: 23
          networkType: OVNKubernetes 
      serviceNetwork:
        - 172.30.0.0/16
      controlPlaneNodes: 3
      workerNodes: 0
      apiVIP: 192.168.0.10
      ingressVIP: 192.168.0.11
      sshPublicKey: 'you-ssh-key'
      pullSecret: 'your-pull-secret'
      nodes:
        node0:
          spec:
            installation_disk_id: /dev/disk/by-id/nvme-eui.00000000000000000026b768704fd5a5
        node1:
          spec:
            installation_disk_id: /dev/disk/by-id/nvme-eui.00000000000000000026b768704fb795
        node2:
          spec:
            installation_disk_id: /dev/disk/by-id/nvme-eui.00000000000000000026b768704fb745
```
