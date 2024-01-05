# Aspir8 from Scratch

Let's deploy [Aspire](https://learn.microsoft.com/dotnet/aspire/get-started/aspire-overview)-flavoured apps to a [Kubernetes](https://kubernetes.io/) cluster, through [Aspir8](https://github.com/prom3theu5/aspirational-manifests)! Are you new to Kubernetes? Don't worry. Let's start from scratch.

> This document is based on MacOS Sonoma with M2 Silicon Chip. If you are using a different OS or different chipset, it might be behaving differently.

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [.NET 8 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) with the [Aspire workload](https://learn.microsoft.com/dotnet/aspire/fundamentals/setup-tooling?tabs=dotnet-cli)
- [Visual Studio Code](https://code.visualstudio.com/) with the [C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit) extension
- [Azure Account](https://azure.microsoft.com/free)

## Local Kubernetes Cluster Setup through Docker Desktop

1. [Install Docker Desktop on you local machine](https://docs.docker.com/desktop/install/mac-install/).
1. [Enable Kubernetes in Docker Desktop](https://docs.docker.com/desktop/kubernetes/).
1. [Deploy sample app to a Kubernetes cluster](https://docs.docker.com/get-started/kube-deploy/).

## Kubernetes Dashboard Setup

> **Note:** This is only applicable for Kubernetes Dashboard v2.x. From v3.x, use [Helm Charts](https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard) approach.

**References**

- [Deploy and Access the Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
- [Creating sample user](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md)

1. Get dashboard version.

    ```bash
    dashboard_version=$(curl 'https://api.github.com/repos/kubernetes/dashboard/releases' | \
      jq -r '[.[] | select(.name | contains("-") | not)] | .[0].name')
    ```

1. Install dashboard.

    ```bash
    kubectl apply -f \
      https://raw.githubusercontent.com/kubernetes/dashboard/$dashboard_version/aio/deploy/recommended.yaml
    ```

<!-- 1. Install metrics server.

    ```bash
    kubectl apply -f \
      https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    ``` -->

1. Create admin user.

    ```bash
    kubectl apply -f - <<EOF
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: admin-user
      namespace: kubernetes-dashboard
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kubernetes-dashboard
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: admin-user
      namespace: kubernetes-dashboard
      annotations:
        kubernetes.io/service-account.name: "admin-user"
    type: kubernetes.io/service-account-token
    EOF
    ```

1. Get the access token.

    ```bash
    kubectl get secret admin-user \
      -n kubernetes-dashboard \
      -o jsonpath={".data.token"} | base64 -d
    ```

1. Run the proxy server.

    ```bash
    kubectl proxy
    ```

1. Access the dashboard using the following URL:

    ```text
    http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
    ```

## Aspire-flavoured App Build

TBD

## Aspire-flavoured App Deployment to Kubernetes Cluster through Aspir8

TBD
