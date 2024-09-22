# Aspir8 from Scratch

Let's deploy [Aspire](https://learn.microsoft.com/dotnet/aspire/get-started/aspire-overview)-flavoured apps to a [Kubernetes](https://kubernetes.io/) cluster, through [Aspir8](https://github.com/prom3theu5/aspirational-manifests)! Are you new to Kubernetes? Don't worry. Let's start from scratch.

<!-- > This document is based on MacOS Sonoma with M2 Silicon Chip. If you are using a different OS or different chipset, it might be behaving differently. -->

## Table of Contents

- [Aspir8 from Scratch](#aspir8-from-scratch)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Local Kubernetes Cluster Setup through Docker Desktop](#local-kubernetes-cluster-setup-through-docker-desktop)
  - [Kubernetes Dashboard Setup](#kubernetes-dashboard-setup)
    - [Use Kubernetes Dashboard v2.x](#use-kubernetes-dashboard-v2x)
    - [Use Helm Charts](#use-helm-charts)
  - [Aspire-flavoured App Build](#aspire-flavoured-app-build)
  - [Aspire-flavoured App Deployment to Kubernetes Cluster through Aspir8](#aspire-flavoured-app-deployment-to-kubernetes-cluster-through-aspir8)
    - [Use local container registry](#use-local-container-registry)
    - [Use Azure Kubernetes Services (AKS)](#use-azure-kubernetes-services-aks)
    - [Use Amazon Elastic Kubernetes Service (EKS)](#use-amazon-elastic-kubernetes-service-eks)
    - [Use Google Kubernetes Engine (GKE)](#use-google-kubernetes-engine-gke)
    - [Use NHN Kubernetes Services (NKS)](#use-nhn-kubernetes-services-nks)

## Prerequisites

- for Aspire
  - [.NET 8 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) 8.0.200 or higher with the [Aspire workload](https://learn.microsoft.com/dotnet/aspire/fundamentals/setup-tooling?tabs=dotnet-cli)
  - [Visual Studio Code](https://code.visualstudio.com/) with the [C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit) extension
- for local Kubernetes cluster
  - [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- for Azure
  - [Azure subscription](https://azure.microsoft.com/free)
  - [Azure CLI](https://learn.microsoft.com/cli/azure/what-is-azure-cli)
- for AWS
  - [AWS subscription](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html?nc2=h_ct&src=header_signup)
  - [AWS CLI](https://aws.amazon.com/cli/)
- for GKE
  - TBD
- for NHN Cloud
  - [NHN Cloud subscription](https://id.nhncloud.com/join)

## Local Kubernetes Cluster Setup through Docker Desktop

1. [Install Docker Desktop on you local machine](https://docs.docker.com/desktop/install/mac-install/).
1. [Enable Kubernetes in Docker Desktop](https://docs.docker.com/desktop/kubernetes/).
1. [Deploy sample app to a Kubernetes cluster](https://docs.docker.com/get-started/kube-deploy/).

## Kubernetes Dashboard Setup

### Use Kubernetes Dashboard v2.x

<!-- If you want to directly setup the Kubernetes Dashboard on your local machine, follow the steps below. Otherwise, skip this section and go to the next section, [MicroK8s Setup](#microk8s-setup). -->

> **Note:** This is only applicable for Kubernetes Dashboard v2.x.

**References**

- [Deploy and Access the Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
- [Creating sample user](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md)

1. Get dashboard version.

    ```bash
    # Bash
    dashboard_version=$(curl 'https://api.github.com/repos/kubernetes/dashboard/releases' | \
        jq -r '[.[] | select(.name | contains("-") | not)] | .[0].name')

    # PowerShell
    $dashboard_version = $($(Invoke-RestMethod https://api.github.com/repos/kubernetes/dashboard/releases) | `
        Where-Object { $_.name -notlike "*-*" } | Select-Object -First 1).name
    ```

1. Install dashboard.

    ```bash
    # Bash
    kubectl apply -f \
      https://raw.githubusercontent.com/kubernetes/dashboard/$dashboard_version/aio/deploy/recommended.yaml

    # PowerShell
    kubectl apply -f `
      https://raw.githubusercontent.com/kubernetes/dashboard/$dashboard_version/aio/deploy/recommended.yaml
    ```

<!-- 1. Install metrics server.

    ```bash
    kubectl apply -f \
      https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    ``` -->

1. Create admin user.

    ```bash
    kubectl apply -f ./admin-user.yaml
    ```

1. Get the access token. Take note the access token to access the dashboard.

    ```bash
    # Bash
    kubectl get secret admin-user \
        -n kubernetes-dashboard \
        -o jsonpath={".data.token"} | base64 -d

    # PowerShell
    kubectl get secret admin-user `
        -n kubernetes-dashboard `
        -o jsonpath='{ .data.token }' | `
        % { [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($_)) }
    ```

1. Run the proxy server.

    ```bash
    kubectl proxy
    ```

1. Access the dashboard using the following URL:

    ```text
    http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
    ```

1. Enter the access token to access the dashboard.

### Use Helm Charts

> **Note:** From Kubernetes Dashboard v3.x, use [Helm Charts](https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard) approach.

1. Install [Helm](https://helm.sh/docs/intro/install/).

1. Run the following commands to install the Kubernetes Dashboard.

    ```bash
    # Add kubernetes-dashboard repository
    helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/

    # Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
    helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
    ```

1. Create admin user.

    ```bash
    kubectl apply -f ./admin-user.yaml
    ```

1. Get the access token. Take note the access token to access the dashboard.

    ```bash
    # Bash
    kubectl get secret admin-user \
        -n kubernetes-dashboard \
        -o jsonpath={".data.token"} | base64 -d

    # PowerShell
    kubectl get secret admin-user `
        -n kubernetes-dashboard `
        -o jsonpath='{ .data.token }' | `
        % { [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($_)) }
    ```

1. Run the proxy server.

    ```bash
    kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
    ```

1. Access the dashboard using the following URL:

    ```text
    http://localhost:8443
    ```

1. Enter the access token to access the dashboard.

<!-- ## MicroK8s Setup

TBD -->

## Aspire-flavoured App Build

1. Install .NET Aspire workload.

    ```bash
    # Bash
    sudo dotnet workload update && sudo dotnet workload install aspire

    # PowerShell
    dotnet workload update && dotnet workload install aspire
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

1. Open the app in a browser, and go to the weather page to see whether the API is working or not. The port number might be different from the example below.

    ```text
    http://localhost:18888
    ```

## Aspire-flavoured App Deployment to Kubernetes Cluster through Aspir8

### Use local container registry

1. Install [Distribution (formerly known as Registry)](https://github.com/distribution/distribution) as a local Docker Hub (Container Registry).

    ```bash
    docker run -d -p 6000:5000 --name registry registry:latest
    ```

   > **Note:** The port number of `6000` is just an arbitrary number. You can choose your own one.

1. Install [Aspir8](https://github.com/prom3theu5/aspirational-manifests).

    ```bash
    dotnet tool install -g aspirate
    ```

1. Initialise Aspir8.

    ```bash
    cd Aspir8.AppHost
    aspirate init -cr localhost:6000 -ct latest --disable-secrets true --non-interactive
    ```

1. Build and publish the app to the local container registry.

    ```bash
    aspirate generate --image-pull-policy Always --include-dashboard true --disable-secrets true --non-interactive
    ```

1. Deploy the app to the Kubernetes cluster.

    ```bash
    aspirate apply -k docker-desktop --non-interactive
    ```

1. Check the services in the Kubernetes cluster.

    ```bash
    kubectl get services
    ```

1. Install a load balancer for `webfrontend` to the local Kubernetes cluster.

    ```bash
    kubectl apply -f ../load-balancer.yaml
    ```

1. Install a load balancer for `aspire-dashboard` to the local Kubernetes cluster.

    ```bash
    kubectl apply -f ../aspire-dashboard.yaml
    ```

1. Open the app in a browser, and go to the dashboard page to see the logs

    ```text
    http://localhost:18888
    ```

1. Open the app in a browser, and go to the weather page to see whether the API is working or not.

    ```text
    http://localhost/weather
    ```

### Use Azure Kubernetes Services (AKS)

> **Note:** It uses [Azure CLI](https://learn.microsoft.com/cli/azure/what-is-azure-cli), which is the imperative approach. The declarative approach using Bicep is TBD.

1. Set environment variables. Make sure that you use the closest or preferred location for provisioning resources (eg. `koreacentral`).

    ```bash
    # Bash
    export AZURE_ENV_NAME="aspir8$RANDOM"
    export AZ_RESOURCE_GROUP=rg-$AZURE_ENV_NAME
    export AZ_NODE_RESOURCE_GROUP=rg-$AZURE_ENV_NAME-mc
    export AZ_LOCATION=koreacentral
    export ACR_NAME=acr$AZURE_ENV_NAME
    export AKS_CLUSTER_NAME=aks-$AZURE_ENV_NAME

    # PowerShell
    $AZURE_ENV_NAME = "aspir8$(Get-Random -Minimum 1000 -Maximum 9999)"
    $AZ_RESOURCE_GROUP = "rg-$AZURE_ENV_NAME"
    $AZ_NODE_RESOURCE_GROUP = "rg-$AZURE_ENV_NAME-mc"
    $AZ_LOCATION = "koreacentral"
    $ACR_NAME = "acr$AZURE_ENV_NAME"
    $AKS_CLUSTER_NAME = "aks-$AZURE_ENV_NAME"
    ```

1. Create a resource group.

    ```bash
    az group create -n $AZ_RESOURCE_GROUP -l $AZ_LOCATION
    ```

1. Create an [Azure Container Registry (ACR)](https://learn.microsoft.com/azure/container-registry/container-registry-intro).

    ```bash
    # Bash
    az acr create \
        -g $AZ_RESOURCE_GROUP \
        -n $ACR_NAME \
        -l $AZ_LOCATION \
        --sku Basic \
        --admin-enabled true

    # PowerShell
    az acr create `
        -g $AZ_RESOURCE_GROUP `
        -n $ACR_NAME `
        -l $AZ_LOCATION `
        --sku Basic `
        --admin-enabled true
    ```

1. Get ACR credentials.

    ```bash
    # Bash
    export ACR_LOGIN_SERVER=$(az acr show \
        -g $AZ_RESOURCE_GROUP \
        -n $ACR_NAME \
        --query "loginServer" -o tsv)
    export ACR_USERNAME=$(az acr credential show \
        -g $AZ_RESOURCE_GROUP \
        -n $ACR_NAME \
        --query "username" -o tsv)
    export ACR_PASSWORD=$(az acr credential show \
        -g $AZ_RESOURCE_GROUP \
        -n $ACR_NAME \
        --query "passwords[0].value" -o tsv)

    # PowerShell
    $ACR_LOGIN_SERVER = $(az acr show `
        -g $AZ_RESOURCE_GROUP `
        -n $ACR_NAME `
        --query "loginServer" -o tsv)
    $ACR_USERNAME = $(az acr credential show `
        -g $AZ_RESOURCE_GROUP `
        -n $ACR_NAME `
        --query "username" -o tsv)
    $ACR_PASSWORD = $(az acr credential show `
        -g $AZ_RESOURCE_GROUP `
        -n $ACR_NAME `
        --query "passwords[0].value" -o tsv)
    ```

1. Create an [AKS](https://learn.microsoft.com/azure/aks/intro-kubernetes) cluster.

   > **Note:** Depending on the location you create the cluster, the VM size might vary.

    ```bash
    # Bash
    az aks create \
        -g $AZ_RESOURCE_GROUP \
        -n $AKS_CLUSTER_NAME \
        -l $AZ_LOCATION \
        --node-resource-group $AZ_NODE_RESOURCE_GROUP \
        --node-vm-size Standard_B2s \
        --network-plugin azure \
        --generate-ssh-keys \
        --attach-acr $ACR_NAME

    # PowerShell
    az aks create `
        -g $AZ_RESOURCE_GROUP `
        -n $AKS_CLUSTER_NAME `
        -l $AZ_LOCATION `
        --node-resource-group $AZ_NODE_RESOURCE_GROUP `
        --node-vm-size Standard_B2s `
        --network-plugin azure `
        --generate-ssh-keys `
        --attach-acr $ACR_NAME
    ```

1. Connect to the AKS cluster.

    ```bash
    # Bash
    az aks get-credentials \
        -g $AZ_RESOURCE_GROUP \
        -n $AKS_CLUSTER_NAME \

    # PowerShell
    az aks get-credentials `
        -g $AZ_RESOURCE_GROUP `
        -n $AKS_CLUSTER_NAME `
    ```

1. Connect to ACR.

   > **Note:** This is the demo purpose only. You should manually enter username and password from your input.

    ```bash
    docker login $ACR_LOGIN_SERVER -u $ACR_USERNAME -p $ACR_PASSWORD
    ```

1. Install [Aspir8](https://github.com/prom3theu5/aspirational-manifests).

    ```bash
    dotnet tool install -g aspirate
    ```

1. Initialise Aspir8.

    ```bash
    cd Aspir8.AppHost
    aspirate init -cr $ACR_LOGIN_SERVER -ct latest --non-interactive
    ```

   > **Note:** If you are asked to enter or skip the repository prefix, enter `n` to skip it.

1. Build and publish the app to ACR.

    ```bash
    aspirate generate --image-pull-policy IfNotPresent --non-interactive
    ```

1. Deploy the app to the AKS cluster.

    ```bash
    aspirate apply -k $AKS_CLUSTER_NAME --non-interactive
    ```

1. Install a load balancer to the AKS cluster.

    ```bash
    kubectl apply -f ../load-balancer.yaml
    ```

1. Confirm the `webfrontend-lb` service type is `LoadBalancer`, and note the external IP address of the `webfrontend-lb` service.

    ```bash
    kubectl get services
    ```

1. Open the app in a browser, and go to the weather page to see whether the API is working or not.

    ```text
    http://<EXTERNAL_IP_ADDRESS>
    ```

1. Once you are done, delete the entire resources from Azure.

    ```bash
    az group delete -n $AZ_RESOURCE_GROUP -f Microsoft.Compute/virtualMachineScaleSets -y --no-wait
    ```

### Use Amazon Elastic Kubernetes Service (EKS)

> **Note:**
> 
> - It uses both [AWS Console](https://console.aws.amazon.com/) and [AWS CLI](https://aws.amazon.com/cli/) to provision resources to AWS.
> - It uses the Root account for this demo purpose only. You should use the IAM user with the least privilege.

1. Set environment variables. Make sure that you use the closest or preferred location for provisioning resources (eg. `ap-northeast-2`).

    ```bash
    # Bash
    export AWS_ENV_NAME="aspir8$RANDOM"
    export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)
    export AWS_LOCATION=ap-northeast-2 # Seoul
    export ECR_LOGIN_SERVER=$AWS_ACCOUNT_ID.dkr.ecr.AWS_LOCATION.amazonaws.com
    export EKS_STACK_NAME=aspir8-stack
    export EKS_CLUSTER_NAME=eks-$AWS_ENV_NAME
    export EKS_NODE_GROUP_NAME=aspir8-nodegroup

    # PowerShell
    $AWS_ENV_NAME = "aspir8$(Get-Random -Minimum 1000 -Maximum 9999)"
    $AWS_ACCOUNT_ID = $(aws sts get-caller-identity --query "Account" --output text)
    $AWS_LOCATION = "ap-northeast-2" # Seoul
    $ECR_LOGIN_SERVER = "$($AWS_ACCOUNT_ID).dkr.ecr.$($AWS_LOCATION).amazonaws.com"
    $EKS_STACK_NAME = "aspir8-stack"
    $EKS_CLUSTER_NAME = "eks-$AWS_ENV_NAME"
    $EKS_NODE_GROUP_NAME = "aspir8-nodegroup"
    ```

1. Create a VPC stack for EKS.

    ```bash
    # Bash
    aws cloudformation create-stack \
        --region $AWS_LOCATION \
        --stack-name $EKS_STACK_NAME \
        --template-url https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
    
    # PowerShell
    aws cloudformation create-stack `
        --region $AWS_LOCATION `
        --stack-name $EKS_STACK_NAME `
        --template-url https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
    ```

1. Create an EKS cluster role and attach it to the policy.

    ```bash
    # Bash
    aws iam create-role \
        --role-name Aspir8AmazonEKSClusterRole \
        --assume-role-policy-document file://"eks-cluster-role-trust-policy.json"
    aws iam attach-role-policy \
        --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
        --role-name Aspir8AmazonEKSClusterRole
    
    # PowerShell
    aws iam create-role `
        --role-name Aspir8AmazonEKSClusterRole `
        --assume-role-policy-document file://"eks-cluster-role-trust-policy.json"
    aws iam attach-role-policy `
        --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy `
        --role-name Aspir8AmazonEKSClusterRole
    ```

1. Create an EKS cluster node role and attach it to the policies.

    ```bash
    # Bash
    aws iam create-role \
        --role-name Aspir8AmazonEKSNodeRole \
        --assume-role-policy-document file://"eks-node-role-trust-policy.json"
    aws iam attach-role-policy \
        --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
        --role-name Aspir8AmazonEKSNodeRole
    aws iam attach-role-policy \
        --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly \
        --role-name Aspir8AmazonEKSNodeRole
    aws iam attach-role-policy \
        --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
        --role-name Aspir8AmazonEKSNodeRole
    
    # PowerShell
    aws iam create-role `
        --role-name Aspir8AmazonEKSNodeRole `
        --assume-role-policy-document file://"eks-node-role-trust-policy.json"
    aws iam attach-role-policy `
        --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy `
        --role-name Aspir8AmazonEKSNodeRole
    aws iam attach-role-policy `
        --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly `
        --role-name Aspir8AmazonEKSNodeRole
    aws iam attach-role-policy `
        --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy `
        --role-name Aspir8AmazonEKSNodeRole
    ```

1. Create an EKS cluster by following this [document](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html#eks-create-cluster).

1. Create an EKS cluster nodes by following this [document](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html#eks-launch-workers).

1. Connect to the EKS cluster.

    ```bash
    aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_LOCATION
    ```

1. Connect to ECR.

    ```bash
    aws ecr get-login-password --region $AWS_LOCATION | docker login --username AWS --password-stdin $ECR_LOGIN_SERVER
    ```

1. Create repositories in ECR.

    ```bash
    aws ecr create-repository --repository-name apiservice --region $AWS_LOCATION
    aws ecr create-repository --repository-name webfrontend --region $AWS_LOCATION
    ```

1. Install [Aspir8](https://github.com/prom3theu5/aspirational-manifests).

    ```bash
    dotnet tool install -g aspirate
    ```

1. Initialise Aspir8.

    ```bash
    cd Aspir8.AppHost
    aspirate init -cr $ECR_LOGIN_SERVER -ct latest --non-interactive
    ```

   > **Note:** If you are asked to enter or skip the repository prefix, enter `n` to skip it.

1. Build and publish the app to ECR.

    ```bash
    aspirate generate --image-pull-policy IfNotPresent --non-interactive
    ```

1. Deploy the app to the EKS cluster.

    ```bash
    aspirate apply -k $EKS_CLUSTER_NAME --non-interactive
    ```

1. Install a load balancer to the EKS cluster.

    ```bash
    kubectl apply -f ../load-balancer.yaml
    ```

1. Confirm the `webfrontend-lb` service type is `LoadBalancer`, and note the URL under the external IP address column of the `webfrontend-lb` service.

    ```bash
    kubectl get services
    ```

1. Open the app in a browser, and go to the weather page to see whether the API is working or not.

    ```text
    http://<xxxx.ap-northeast-2.elb.amazonaws.com>
    ```

1. Once you are done, delete the entire resources from AWS.

    ```bash
    # Delete EKS node group
    aws eks delete-nodegroup --nodegroup-name $EKS_NODE_GROUP_NAME --cluster-name $EKS_CLUSTER_NAME

    # Delete EKS cluster
    aws eks delete-cluster --name $EKS_CLUSTER_NAME

    # Delete ECR repositories
    aws ecr delete-repository --repository-name apiservice --force --region $AWS_LOCATION
    aws ecr delete-repository --repository-name webfrontend --force --region $AWS_LOCATION

    # Delete CloudFormation stack
    aws cloudformation delete-stack --stack-name $EKS_STACK_NAME
    ```

   > **Note:**
   > 
   > - Deleting the EKS node group takes 5-10 mins.
   > - Only after the EKS node group is deleted, the EKS cluster can be deleted.
   > - While deleting the CloudFormation stack, you might be failing the deletion process. It's highly likely because of Elastic Load Balancer. Go to [EC2 Dashboard](https://ap-northeast-2.console.aws.amazon.com/ec2/home), and delete the existing load balancer instance first.

### Use Google Kubernetes Engine (GKE)

TBD

### Use NHN Kubernetes Services (NKS)

> **Note:**
> 
> - It uses [NHN Cloud Console](https://console.nhncloud.com/) to manage NHN Kubernetes Service (NKS).
> - It uses [NHN Container Registry (NCR)](https://www.nhncloud.com/kr/service/container/nhn-container-registry-ncr) as the container registry.

1. Add the following Docker Hub repository details to `Aspir8.ApiService/Aspir8.ApiService.csproj`.

    ```xml
    <PropertyGroup>
      <ContainerRepository>{{DOCKER_USERNAME}}/apiservice</ContainerRepository>
    </PropertyGroup>
    ```

1. Add the following Docker Hub repository details to `Aspir8.Web/Aspir8.Web.csproj`.

    ```xml
    <PropertyGroup>
      <ContainerRepository>{{DOCKER_USERNAME}}/webfrontend</ContainerRepository>
    </PropertyGroup>
    ```

1. Set environment variables.

    ```bash
    export NHN_ENV_NAME="aspir8$RANDOM"
    export NKS_CLUSTER_NAME=nks-$NHN_ENV_NAME
    ```

1. Create an NKS cluster from the console.
1. Get the `kubeconfig` of the NKS cluster from the console.
1. Connect to the NKS cluster using the `kubeconfig`.

    ```bash
    export KUBECONFIG=~/.kube/config:~/path/to/downloaded/kubeconfig
    kubectl config view --merge --flatten > ~/.kube/merged_kubeconfig
    mv ~/.kube/config ~/.kube/config.bak
    mv ~/.kube/merged_kubeconfig ~/.kube/config
    ```

1. Change the context to the NKS cluster.

    ```bash
    kubectl config use-context default
    ```

1. Connect to Docker Hub.

   > **Note:** This is the demo purpose only. You should manually enter username and password from your input.

    ```bash
    docker login registry.hub.docker.com -u <DOCKER_USERNAME> -p <DOCKER_PASSWORD>
    ```

1. Initialise Aspir8.

    ```bash
    cd Aspir8.AppHost
    aspirate init -cr registry.hub.docker.com -ct latest --non-interactive
    ```

1. Build and publish the app to Docker Hub.

    ```bash
    aspirate generate --image-pull-policy IfNotPresent --non-interactive
    ```

1. Deploy the app to the NKS cluster.

    ```bash
    aspirate apply -k toast-$NKS_CLUSTER_NAME --non-interactive
    ```

1. Install a load balancer to the NKS cluster.

    ```bash
    kubectl apply -f ./load-balancer.yaml
    ```

1. Confirm the `webfrontend` service type is `LoadBalancer`, and note the external IP address of the `webfrontend` service.

    ```bash
    kubectl get services
    ```

1. Open the app in a browser, and go to the weather page to see whether the API is working or not.

    ```text
    http://<EXTERNAL_IP_ADDRESS>
    ```

1. Once you are done, delete the entire resources from the console and container images from Docker Hub.
