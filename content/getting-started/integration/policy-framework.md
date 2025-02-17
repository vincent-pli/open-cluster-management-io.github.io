---
title: Policy framework
weight: 1
geekdocHidden: true
---

After the cluster manager and klusterlet are installed, you can install the policy framework components to the hub and the managed clusters.

<!-- spellchecker-disable -->

{{< toc >}}

<!-- spellchecker-enable -->

## Prerequisite

You must meet the following prerequisites to install the policy framework:

* Ensure [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl) and [`kustomize`](https://kubernetes-sigs.github.io/kustomize/installation) are installed.

* Ensure [Golang](https://golang.org/doc/install) is installed, if you are planning to install from the source.

* Prepare one Kubernetes cluster to function as the hub cluster. For example, use [kind](https://kind.sigs.k8s.io/docs/user/quick-start) to create a hub cluster. To use kind, you must install and run [Docker](https://docs.docker.com/get-started).

* Ensure the `open-cluster-management` _cluster manager_ is installed. See [Cluster Manager](/getting-started/core/cluster-manager) for more information.

* Ensure the `open-cluster-management` _klusterlet_ is installed. See [Klusterlet](/getting-started/core/register-cluster) for more information.

* Ensure the `open-cluster-management` _application_ is installed. See [Application management](/getting-started/integration/app-lifecycle) for more information.

## Install from prebuilt images on Quay.io

Complete the following steps to install the policy framework from prebuild images on Quay.io:

1. Clone the `governance-policy-framework` repository:

   ```Shell
   git clone https://github.com/open-cluster-management/governance-policy-framework.git
   ```

2. Deploy the policy framework components to the hub cluster with the following commands: 

   ```Shell
   kubectl config use-context <hub cluster context> # kubectl config use-context kind-hub
   cd governance-policy-framework
   make deploy-community-policy-framework-hub
   ```

   * The previous command deploys the [policy-propagator](https://github.com/open-cluster-management/governance-policy-propagator).

3. Ensure the pods are running on hub with the following command:

   ```Shell
   $ kubectl get pods -n open-cluster-management 
   NAME                                           READY   STATUS    RESTARTS   AGE
   governance-policy-propagator-8c77f7f5f-kthvh   1/1     Running   0          94s
   ```

4. Export the hub cluster `kubeconfig` with the following command:

   For `kind` cluster:

   ```Shell
   kind get kubeconfig --name <cluster name> --internal > $PWD/kubeconfig_hub
   ```

   For non-`kind` clusters:

   ```Shell
   kubectl config view --context=<hub cluster context> --minify --flatten > $PWD/kubeconfig_hub
   ```

5. Deploy the policy framework components to the managed cluster. Run the following commands: 

   ```Shell
   export MANAGED_CLUSTER_NAME=<managed cluster name> # export MANAGED_CLUSTER_NAME=cluster1
   kubectl config use-context <managed cluster context> # kubectl config use-context kind-$MANAGED_CLUSTER_NAME
   kubectl create ns open-cluster-management-agent-addon
   kubectl -n open-cluster-management-agent-addon create secret generic hub-kubeconfig --from-file=kubeconfig=$PWD/kubeconfig_hub
   make deploy-community-policy-framework-managed
   ```

   * The previous command deploy following components:
     -  [policy-spec-sync](https://github.com/open-cluster-management/governance-policy-spec-sync)
     -  [policy-status-sync](https://github.com/open-cluster-management/governance-policy-status-sync)
     -  [policy-template-sync](https://github.com/open-cluster-management/governance-policy-template-sync)

6. Verify that the pods are running with the following command:

   ```Shell
   $ kubectl get pods -n open-cluster-management-agent-addon 
   NAME                                               READY   STATUS    RESTARTS   AGE
   governance-policy-spec-sync-6474b6d898-tmkw6       1/1     Running   0          2m14s
   governance-policy-status-sync-84cbb795df-pgbgt     1/1     Running   0          2m14s
   governance-policy-template-sync-759b9b556f-mx46t   1/1     Running   0          2m14s
   ```
