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

1. Install .NET Aspire workload.

    ```bash
    sudo dotnet workload update && sudo dotnet workload install aspire
    ```

1. Create a new Aspire starter app.

    ```bash
    dotnet new aspire-starter -n Aspir8
    ```

1. Build the app.

    ```bash
    dotnet restore && dotnet build
    ```

1. Run the app locally.

    ```bash
    dotnet run --project Aspir8.AppHost
    ```

1. Open the app in a browser.

    ```text
    http://localhost:18888
    ```

## Aspire-flavoured App Deployment to Kubernetes Cluster through Aspir8

### Use local container registry

1. Install [Distribution (formerly known as Registry)](https://github.com/distribution/distribution).

    ```bash
    docker run -d -p 6000:5000 --name registry registry:latest
    ```

   > **Note:** The port number of `6000` is just an arbitrary number. You can choose your own one.

1. Install [Aspir8](https://github.com/prom3theu5/aspirational-manifests).

    ```bash
    dotnet tool install -g aspirate --prerelease
    ```

1. Initialise Aspir8.

    ```bash
    cd Aspir8.AppHost
    aspirate init
    ```

   - Set the fall-back container registry to `localhost:6000`.
   - Skip the fall-back container tag.
   - Skip the custom directory for the kustomize manifest template.

1. Build and publish the app to the local container registry.

    ```bash
    aspirate generate
    ```

   - Choose all components while generating.
   - Select the impage pull policy to `IfNotPresent`.
   - Generate the top level kustomize manifest for the Kubernetes cluster.

1. Deploy the app to the Kubernetes cluster.

    ```bash
    aspirate apply
    ```

   - Choose the Kubernetes cluster to `docker-desktop`.

1. Check the services in the Kubernetes cluster.

    ```bash
    kubectl get services
    ```

1. Update the service type of `webfrontent` to `NodePort`.

    ```bash
    kubectl patch svc webfrontend -n default -p '{"spec": {"type": "NodePort"}}'
    ```

1. Confirm the `webfrontend` service type updated, and note the port number of the `webfrontend` service, `32689` for example.

    ```bash
    kubectl get services
    ```

1. Open the app in a browser.

    ```text
    http://localhost:32689
    ```

### Use Azure Kubernetes Services (AKS)

TBD

### Use Amazon Elastic Kubernetes Service (EKS)

TBD

### Use Google Kubernetes Engine (GKE)

TBD

### Use NHN Kubernetes Services (NKS)

TBD
