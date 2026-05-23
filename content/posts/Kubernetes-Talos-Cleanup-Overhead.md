---
title: "Kubernetes: 3TB Less Storage, 80GB Less RAM, and More Features"
date: 2026-05-22T12:00:00+01:00
draft: false
tags: ["Kubernetes", "Talos", "Homelab", "ArgoCD", "GitOps", "Prometheus", "Loki", "Grafana", "Zabbix"]
---

After getting the Talos cluster running and ArgoCD wired up, the next thing to do was actually move workloads into it. That process ended up being a general cleanup as well - going through what was running, deciding what was still needed, and getting rid of the rest. The result was 3TB less storage consumed and 80GB less RAM allocated across the hosts, while actually running more services than before.

I have been running Kubernetes for a while, but still had a lot of the "one VM per service" mindset from before. Also wasn't really confident enough in Kubernetes to just move my workloads into it and delete the old VMs.

I also started moving VMs to Containers due to all the security vulnerabilites that have been popping up around Linux and other stuff lately. So it is either patching VMs more often or moving more into containers.

<!--more-->

# What was there before

Before Kubernetes was involved, the setup was a collection of individual Debian and Windows VMs. Each one existed for one or two purposes:

- A web server
- A Zabbix proxy
- A Grafana server running inside Docker
- A general containers host for various things
- A GitHub Actions runner
- A Traefik reverse proxy
- A Windows script server

Each VM was provisioned with at least 2 vCPUs, 4GB of RAM, and an 80GB disk. That is fine for one or two things, but when you have seven of them the overhead adds up fast.

This gives a lot of overhead in terms of resources.

# The overhead problem

The "one VM per service" approach is easy to reason about, and for many years it was the way to do stuff. Each service gets its own VM, and if something is wrong its often only that VM that has han issue. The problem is that it does not scale down well. 

In my scenario fx my Zabbix Proxy and GitHub Actions Runner have allocated 4GBs of RAM each, and in my Github Actions case it sits idle most of the time.

And as i talked about earlier, the maintenance side has become more and more required, with the rising amount of security vulnerabilities. Each VM needs OS updates, Docker updates, and so on. That is a lot of overhead for something that is not doing much.

With Kubernetes you stop thinking about machines and start thinking about workloads. The Zabbix proxy is now a pod with resource requests of 50m CPU and 64MB RAM, with a limit of 500m (0.5CPU, yeah thats a thing..) and 256MB. The GitHub Actions runner scales between 0 and 10 replicas depending on how many jobs are queued, then scales back down using: [Github Actions Runner Controllers](https://github.com/actions/actions-runner-controller).

[![Kubernetes Memory Reduction](/img/Kubernetes-Memory-Reduction.png)](/img/Kubernetes-Memory-Reduction.png)


# What is running now

The cluster runs everything the old VMs did, plus things that were not practical to run before.

The monitoring stack is a good example. Previously Grafana ran in Docker on its own VM, and Prometheus did not exist in this setup at all. I had some stuff running towards Influx for graph stuff. Now my monitoring stack runs Prometheus, Grafana, Loki, Grafana Alloy and a SNMP exporter. Prometheus scrapes the Kubernetes API, nodes, pods, Cilium BGP metrics, and the Unifi switches and APs via SNMP. Alloy runs collects logs from every pod in the cluster, plus accepts syslog from network devices and forwards it into Loki. 

Everything is defined as IaaC in Git, and deployed via ArgoCD.

That is a full observability stack that did not exist before, running in the same cluster that replaced the seven VMs. The resource footprint is a fraction of what it would have taken to run all of that on separate machines.

# Firewalling at the pod level

I also had NSX running to isolate VMs with microsegmentation. NSX is a great product for VMs, you allow or deny traffic to the whole machine. But if two services share a VM, they share the same firewall posture. Also NSX-T is a bit of a resource hog, especially for a homelab. With at least 20GB of RAM allocated to the manager in itself.

In Kubernetes you can have what they call Network Policies. Each namespace and pod can have its own policy. The monitoring namespace, for example, has policies that restrict which pods can talk to which. The syslog ingestion endpoint accepts traffic from the network, but other services in the same namespace do not need to. That kind of granularity was not practical with per-VM firewalls without creating a separate VM for every service - which is obviously what the old setup had drifted toward.

# The numbers

Seven VMs gone, plus some additional cleanup of other VMs that were no longer needed. The result:

- **3TB less storage** used by VMs and their attached disks
- **80GB less RAM** allocated across the hosts
- The cluster itself runs on 3 control plane nodes and 3 workers, more services are now redundant with multiple replicas

The counter-intuitive part is that the cluster runs more software now than the old setup did. The full logging stack, Prometheus, Zabbix proxy, GitHub Actions runners, media services - all of it fits in the cluster with headroom to spare. Individual workloads just use what they actually need, rather than each owning a fixed allocation attached to a whole operating system.

All the old VMs are deleted from disks, and the cluster is running with a fraction of the resources that the old setup consumed.
