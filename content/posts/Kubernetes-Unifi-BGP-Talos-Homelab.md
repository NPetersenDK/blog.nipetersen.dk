---
title: "Kubernetes, Unifi, BGP and Talos in a Homelab"
date: 2026-05-13T12:00:00+01:00
draft: false
tags: ["Kubernetes", "Talos", "Cilium", "BGP", "Unifi", "ArgoCD", "GitOps", "Homelab", "PowerShell", "Traefik"]
---

Last year I started looking into Kubernetes for the first time. There are a lot of options - k3s, k0s, Talos, kubeadm - and a lot of new terms to get your head around. I started with k3s, and have been running it for half a year. The last week I have been slowly but surely transitioning to Talos. A fully automated GitOps-driven cluster that I can wipe and rebuild from scratch in under an hour. 
- Real LoadBalancer IPs with BGP peering to my Unifi UDM-PRO. 
- Traefik with Gateway API for routing. cert-manager for TLS. 
- External Secrets Operator with Azure Key Vault for secrets management. 


The whole stack is in a single git repository and ArgoCD takes care of the syncing.
But how did we get here?

<!--more-->

# The backstory

When I started looking at Kubernetes the number of options was overwhelming. k3s, k0s, kubeadm, Talos. I looked at Talos Linux early on. If you don't know it, is an OS with no shell, no SSH, and you configure everything through an API and Bootstrap it from the start. That surely sounds great. But when you're even discussing with yourself what a Kubernetes Pod is, then Talos might be a bit much.

So I started with k3s instead. k3s is Kubernetes but with a lot of the sharp edges smoothed off and it runs anywhere. It has been my Kubernetes cluster for over 6 months. I have learned a lot about Kubernetes using this. k3s is fine but it does require more maintenance in some ways. You still have the OS to worry about, the Kubernetes software stack, installing stuff and making sure something outside Kubernetes dosen't cause problems.

# Why did I bother if k3s was fine?

Talos Linux is an OS that exists for one purpose: running Kubernetes. There is no package manager, no SSH, no shell. You configure it entirely through an API, and the configuration is YAML. That aligns really well with how I want to run things, my daily work is a DevOps role, so I should also be running my homelab that way. The other thing I like about Talos is that it is immutable. Nodes don't drift. If something is wrong with a node, you reset it and it comes back clean. 

Also that they provide a lot of ways to build the cluster. They do provide VMware Templates, stuff for Bare Metal and Cloud Providers. I also do belive that this is the closest you get to what you get from an Cloud Provider, but locally.

# The stack

Before I get into the interesting parts, here is what I ended up with:

- **Talos Linux 1.13.2** on VMware vSphere - 3 control plane nodes + 3 workers
- **Cilium** as the CNI - replaces kube-proxy entirely, handles BGP routing
- **BGP peering** to Unifi UDM-PRO - real LoadBalancer IPs, no MetalLB
- **ArgoCD** for GitOps - everything in a single git repository
- **Traefik** with Gateway API - HTTPRoutes instead of Ingress
- **cert-manager** - wildcard TLS certs via Cloudflare and Let's Encrypt
- **External Secrets Operator** with Azure Key Vault

# PowerShell and not doing it by hand

With k3s you have to install the OS, then install k3s on top. Talos provides an OVA Template that you can deploy, and also some govc VMware commands in their [documentation](https://docs.siderolabs.com/talos/v1.13/platform-specific-installations/virtualized-platforms/vmware).

I ended up building my own PowerShell scripts to deploy the cluster. The Talos documentation is good [(Talos on VMware documentation)](https://docs.siderolabs.com/talos/v1.13/platform-specific-installations/virtualized-platforms/vmware), but it is really just a collection of one-off commands. I wanted something that I could run from start to finish without having to copy-paste commands and wait for things to be ready in between.

My scripts are available here: [GitHub Repository](https://github.com/NPetersenDK/talos)

I have created a YAML format that contains all the environment-specific configuration called `environment.yaml` - IP addresses, network config, what clusters in VMware to use, portgroups etc. That file is being used in the below script. An example file is here: [environment.example.yaml](https://github.com/NPetersenDK/talos/blob/main/exampleconfigs/environment.example.yaml).

`Deploy-TalosCluster.ps1` handles uploading the Talos OVA and provisioning the six VMs in vSphere. `Bootstrap-TalosCluster.ps1` picks up from there - it waits for the Talos API to come up on all control plane nodes, bootstraps etcd, retrieves the kubeconfig, and installs Cilium via Helm.

After running the 2 scripts, you end up with this. Or whatever you put inside the `environment.yaml` file:

```bash
kubectl get nodes
NAME             STATUS     ROLES           AGE   VERSION
talos-cp-1       NotReady   control-plane   56s   v1.36.0
talos-cp-2       NotReady   control-plane   61s   v1.36.0
talos-cp-3       NotReady   control-plane   62s   v1.36.0
talos-worker-1   NotReady   <none>          39s   v1.36.0
talos-worker-2   NotReady   <none>          48s   v1.36.0
talos-worker-3   NotReady   <none>          58s   v1.36.0
```

The nodes in my setup will all be `NotReady` at this point. That is expected as we dont have the CNI installed yet. The next step is to install Cilium, which also handles BGP peering and LoadBalancer IPs.

# BGP with Unifi - the part that actually makes this useful

When it comes to getting traffic into the cluster, there are a few options. The most basic is NodePort - you expose your services on high ports on each node, and then forward traffic to those ports. The other solution is MetalLB in layer 2 mode - it fakes a floating IP on one of the nodes and moves it around as needed. 

BGP is a different approach. Instead of faking a floating IP on a node, Cilium peers directly with my Unifi UDM-PRO and advertises routes to a dedicated LoadBalancer IP pool (`10.100.0.0/24`). The router learns those routes and forwards traffic directly - it knows exactly which node to send each service's traffic to.

BGP has been a feature since 2024 on Unifi UDM-PROs. It uses FRRouting (FRR) under the hood - the same open-source routing stack you find in a lot of enterprise gear. You configure it through the Unifi Network application and it looks like standard FRR config:

```
router bgp 65100
  bgp bestpath as-path multipath-relax
  no bgp ebgp-requires-policy
  bgp router-id 10.13.37.1
  neighbor BGP-TALOS-01 peer-group
  neighbor BGP-TALOS-01 remote-as external
  neighbor BGP-TALOS-01 timers 3 9
  neighbor 10.13.37.11 peer-group BGP-TALOS-01
  neighbor 10.13.37.12 peer-group BGP-TALOS-01
  neighbor 10.13.37.13 peer-group BGP-TALOS-01
  neighbor 10.13.37.21 peer-group BGP-TALOS-01
  neighbor 10.13.37.22 peer-group BGP-TALOS-01
  neighbor 10.13.37.23 peer-group BGP-TALOS-01
```

Each node in the cluster is listed as a neighbor. `remote-as external` means EBGP - the router and the cluster use different ASNs, so the router treats each node as an external peer. `multipath-relax` lets the router ECMP load-balance across all nodes that advertise the same prefix, which is what you want with a DaemonSet like Traefik where every node can handle traffic.

On the Kubernetes side, Cilium BGP is configured through CRDs. There are four pieces: the cluster config (which nodes participate and which peer to talk to), the peer config (timers, graceful restart), the advertisement (which services to announce), and the IP pool (the actual address range):

```yaml
apiVersion: cilium.io/v2
kind: CiliumBGPClusterConfig
metadata:
  name: cilium-bgp
spec:
  nodeSelector:
    matchLabels:
      bgp-policy: default
  bgpInstances:
  - name: "instance-65110"
    localASN: 65110
    peers:
    - name: "UDM-PRO"
      peerASN: 65100
      peerAddress: 10.13.37.1
      peerConfigRef:
        name: "cilium-peer"
---
apiVersion: cilium.io/v2
kind: CiliumBGPAdvertisement
metadata:
  name: bgp-advert
  labels:
    advertise: bgp
spec:
  advertisements:
    - advertisementType: "Service"
      service:
        addresses:
          - LoadBalancerIP
      selector:
        matchExpressions:
          - { key: bgp-announce, operator: In, values: [ "true" ] }
---
apiVersion: cilium.io/v2
kind: CiliumLoadBalancerIPPool
metadata:
  name: "lb-pool"
spec:
  blocks:
  - cidr: "10.100.0.0/24"
  serviceSelector:
    matchLabels:
      bgp: "pool-a"
```

The advertisement selector means only services with the label `bgp-announce: "true"` get announced via BGP - everything else stays internal. Services get a LoadBalancer IP from the pool automatically when you add the two labels:

```yaml
labels:
  bgp-announce: "true"
  bgp: pool-a
```

ArgoCD ends up at `10.100.0.0`, Traefik at `10.100.0.1`, and so on. The UDM-PRO announces them onward into the rest of the network. From anywhere on my network these addresses just work.

One thing worth noting: I had to set the VXLAN tunnel port to `4789` (the IANA standard) explicitly in the Cilium config. ESXi 8.0U2 has a bug where the VMXNET3 driver fails hardware offload on non-standard VXLAN ports and silently drops packets. There is no obvious error - things just don't work. Broadcom has a [KB article](https://knowledge.broadcom.com/external/article/432932/connectivity-issues-with-cilium-custom-v.html) about it.

# GitOps with ArgoCD and Gateway API

Everything beyond Cilium is managed by ArgoCD. The git repository has one ArgoCD Application manifest per type of workload. For example, there is one for Traefik, one for cert-manager, one for External Secrets Operator, and so on. Each Application points to a different path in the repository where the manifests for that app live.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: external-secrets
  namespace: argocd
spec:
  sources:
    - repoURL: https://github.com/YourName/your-configs.git
      targetRevision: main
      path: argocd/external-secrets
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

For routing I went with Traefik and the **Gateway API** instead of the traditional Ingress resource. NGINX announced they deprecated Ingress in favor of Gateway API, and i would assume others are heading the same way. Each service gets an HTTPRoute that declares which hostname maps to which backend. TLS is terminated at the Gateway using wildcard certs from cert-manager.

# Wiping and rebuilding

The real test of any automation setup is whether you can throw it away and get it back. I deleted all six VMs and started from nothing.

The sequence:

1. Run `Deploy-TalosCluster.ps1` - VMs up in around ten minutes
2. Run `Bootstrap-TalosCluster.ps1` - Talos bootstrapped, Cilium installed
3. Apply the ArgoCD install manifest, then apply the Application YAMLs
4. Create the one manual bootstrap secret for External Secrets
5. Wait

The "wait" part is real. cert-manager uses DNS-01 to issue certs for `*.services.netdc.dk` and `*.monitoring.netdc.dk`, and that involves Cloudflare propagating DNS records. Traefik's HTTPS listeners show `InvalidCertificateRef` until those certs exist.

Total time from no VMs to everything green: around 1 hour.

# What sucked

- **Talos first impressions**: It looked like too much on top of already not understanding Kubernetes. k3s first was the right call - it got me familiar with Kubernetes concepts and terminology, so when I looked at Talos again it was more about learning the Talos way of doing things rather than learning Kubernetes and Talos at the same time.
- **The VXLAN port**: ESXi 8.0U2 silently drops packets on non-standard VXLAN ports due to a VMXNET3 hardware offload bug. No obvious error message. You just get no pod connectivity and spend time eliminating everything else first.

# Is it worth it?

k3s was the right move at the time. Running it taught me what I needed to know before I could appreciate what Talos was actually offering. I don't think I would have gotten this setup working cleanly without that background.

The GitOps approach - everything in a git repository, ArgoCD doing the syncing - is what makes it maintainable as a one-person homelab. If a node dies I reset it. If the whole cluster dies I run two PowerShell scripts and wait. That's the state I wanted to reach.

The BGP part is genuinely one of my favourite things about this setup. Having real LoadBalancer IPs that the router knows about, the same way cloud providers work, at home on Unifi gear - it's one of those things that seems more complex than it is once you've done it once.
