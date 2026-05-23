---
title: "Kubernetes: Azure Arc in a Talos Kubernetes Cluster"
date: 2026-05-23T00:00:00+01:00
draft: false
tags: ["Kubernetes", "Talos", "Azure Arc", "ArgoCD", "GitOps", "Entra ID", "Azure", "Homelab"]
---

Azure Arc lets you connect a Kubernetes cluster to Azure and manage it from the portal, regardless of where it runs. In this case, a Talos cluster running on VMware at home, but it could be any Kubernetes cluster running anywhere. The cluster does not need any inbound ports open - the Arc agents run inside the cluster and maintain an outbound connection to Azure.

You also get MFA and Entra ID integration for authentication, and a live view of the cluster's resources in the portal.

Lets dive in.

<!--more-->

# What is Azure Arc?
Azure Arc is Microsoft's solution for managing resources outside of Azure. It allows you to connect servers, Kubernetes clusters, and databases to Azure and manage them through the Azure Portal. For Kubernetes clusters, it deploys a set of agents that maintain an outbound connection to Azure, allowing you to see the cluster's resources and manage them without needing any inbound ports open.

Azure Arc is getting a lot of traction due to it help bridge the gap between on-premise and cloud, and even 3rd party clouds.

Azure Arc for Kubernetes promises the following features:
- View all connected Kubernetes clusters for inventory, grouping, and tagging, along with your Azure Kubernetes Service (AKS) clusters.
- Configure clusters and deploy applications by using GitOps-based configuration management with Argo CD or Flux v2.
- View and monitor your clusters by using Azure Monitor.
- Enable threat protection by using Microsoft Defender for Containers.
- Manage and report on compliance by using Azure Policy.
- Connect to your Kubernetes clusters from anywhere, and manage access by using Azure role-based access control (Azure RBAC).
- Deploy machine learning workloads by using Azure Machine Learning for Kubernetes clusters.
- Deploy and manage Kubernetes applications from Microsoft Marketplace.
- Deploy services that allow you to take advantage of specific hardware, comply with data residency requirements, or enable new scenarios, such as Azure Arc-enabled data services or Event Grid on Kubernetes.
- Use Azure Kubernetes Fleet Manager and its Arc-enabled Kubernetes cluster extension to tackle hybrid and multicloud Kubernetes management challenges at scale.

Source: [Azure Arc-enabled Kubernetes overview](https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/overview)

The main features I am interested in are the portal management and Entra ID authentication. I'm not quite sure making GitOps and Flux/ArgoCD through Azure is a good idea, but it is worth looking at for new setups.

# Why did I look into it?

In my professional life I work with Azure full time. I wanted to see how the Azure Arc experience is for Kubernetes clusters, and if it could be a good way to bridge On-Premise and Cloud together. I also wanted to see how the authentication story works with Entra ID, and if it is a good alternative to static kubeconfigs and tokens for cluster access.

On-Premise is also seeing a bit of a comeback. Data sovereignty and regulatory requirements are pushing a lot of organisations to think twice about running everything in a public cloud, especially in Europe. Azure Arc fits well into that picture - you keep the workloads on your own infrastructure, but still get the management plane and identity layer from Azure. That is a reasonable trade for a lot of organisations that want the cloud tooling without the data leaving their own datacenters.

# Getting started

For almost everything, I do i use PowerShell, and Microisoft have a really good quickstart guide for connecting a cluster to Arc here: [Quickstart: Connect a Kubernetes cluster to Azure Arc](https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-powershell)

You should note that Microsoft does this through commands, and this is kinda not GitOps-y like putting it in Argo and Manifests.

## Lets break down the steps:
- Create a resource group in Azure for the cluster to live in
- Register the Microsoft.Kubernetes, Microsoft.KubernetesConfiguration and Microsoft.ConnectedKubernetes resource providers in Azure
- Install the Az.Kubernetes module in PowerShell

After that you can run a single command to connect the cluster, and Microsoft takes care of the rest:
```powershell
Install-Module -Name Az.ConnectedKubernetes
New-AzResourceGroup -Name MyResourceGroup -Location westeurope
Register-AzResourceProvider -ProviderNamespace Microsoft.Kubernetes
Register-AzResourceProvider -ProviderNamespace Microsoft.KubernetesConfiguration
Register-AzResourceProvider -ProviderNamespace Microsoft.ExtendedLocation
New-AzConnectedKubernetes -ClusterName MyCluster -ResourceGroupName MyResourceGroup -Location westeurope
```

Now that the cluster is connected, you can see it in the portal along with some basic info about the cluster:

[![Azure Arc Talos Overview](/img/Azure-Arc-Talos-Overview.png)](/img/Azure-Arc-Talos-Overview.png)

At this point the Kubernetes resources section in the portal will prompt you to sign in with a service account token. That is where the Entra ID setup comes in.

[![Azure Arc Bearer Token Prompt](/img/Azure-Arc-Talos-BearerTokenIssue.png)](/img/Azure-Arc-Talos-BearerTokenIssue.png)

# Authentication with Entra ID

Instead of a static kubeconfig or a token you have to paste every time, Arc lets you bind your Entra ID identity directly to a Kubernetes ClusterRoleBinding. From that point the portal uses your active Azure session - no separate token needed.

```powershell
# Get your current user's Entra Object ID
$upn = (Get-AzContext).Account.Id
$entityId = (Get-AzADUser -UserPrincipalName $upn).Id

# Bind it to cluster-admin
kubectl create clusterrolebinding entra-user-binding --clusterrole cluster-admin --user $entityId
```

If you want to bind a group instead of a specific user, pass the group's Object ID with `--group` instead of `--user`. That way anyone in the group gets access without repeating the process per user.

For CLI access from outside the network, `az connectedk8s proxy` opens a local tunnel through Arc using the same Azure session:

```powershell
az connectedk8s proxy -n <cluster-name> -g <resource-group>
```

No VPN, no open inbound firewall ports.

Microsoft's documentation for the Entra authentication setup is here: [Cluster connect - Microsoft Entra authentication option](https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/cluster-connect?tabs=azure-cli#microsoft-entra-authentication-option)

# What you can see

Once the Entra binding is in place, the Kubernetes resources section opens up. It covers Namespaces, Workloads (Deployments, DaemonSets, StatefulSets, Pods), Services, Storage (PVCs, StorageClasses) and Configuration (ConfigMaps, Secrets). It pulls live data from the cluster through the Arc tunnel, so what you see reflects the actual state.

[![Azure Arc Talos Namespaces Overview](/img/Azure-Arc-Talos-Namespaces-Overview.png)](/img/Azure-Arc-Talos-Namespaces-Overview.png)

# GitOps

Azure Arc has a GitOps feature built in, backed by Flux or ArgoCD. You point it at a git repository and it syncs manifests into the cluster, similar to how ArgoCD works.

I have not used it. The cluster already runs ArgoCD and everything is wired up through that. Switching would mean migrating all the existing Applications and there is no reason to do that. If you are starting from scratch and are already in the Azure ecosystem, the Arc GitOps integration is worth looking at - it is a reasonable alternative to running ArgoCD or Flux yourself.

# What did you actually get out of it?
I feel like it is a nice onboarding experience for Azure users, and the portal view of the cluster is a nice addition. For me I think that many will use this feature to easily connect and manage the Kubernetes clusters using Entra ID instead of static kubeconfigs and tokens. The portal view is nice to have, but I don't think it replaces kubectl or ArgoCD for actual management of the cluster.

Is it worth it to connect your cluster to Arc? If you are already in the Azure ecosystem and want the portal view and Entra ID integration, it is a good option. It dosen't cost anything to connect a cluster, and you can always disconnect it later if you don't like it. 

# Scripts

I have written two PowerShell scripts that handle this using Azure PowerShell (no az CLI dependency):

- `Onboard-AzureArc.ps1` - connects the cluster to Arc, registers resource providers, creates the resource group if needed, and sets up the Entra binding as part of the same run.
- `Onboard-AzureArc-Authentication.ps1` - a standalone script for clusters already onboarded, for setting up or updating the Entra ClusterRoleBinding.

Both are available in the [talos repository on GitHub](https://github.com/NPetersenDK/talos).
