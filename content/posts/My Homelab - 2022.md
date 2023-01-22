---
title: "My Homelab (2022 edition)"
date: 2022-08-16T10:00:00+01:00
draft: false
---
I thought I would give an updated look at my HomeLab where i obviously lab most of my stuff.

# Why?
My Homelab started as I think it does for many, a curiousity and passion to know more. For me it started way back when I was very young, and wanted to learn the enterprise stuff. It have made me get jobs where i havent had professional experiences, so I think it has paid off multiple times. Now its more of a playground, and to learn and try new stuff, beta releases and more.

# What?
I work as a VMware and Datacenter Administrator in my daily work, so it helps me learn: Routing, VLANS, Switching (even L3 Switching), Datacenter management, VMware and more.
I LAB many things, but primarily most of the VMware stack, with licenses from VMUG.


<!--more-->
VMware consists of:
- 2 ESXi hosts with VSAN (Selfbuild - Xeon E-2236, 128GB RAM, 2x Capacity disks for VSAN, 2x 10G networking)
- 1 ESXi host (Intel NUC8i5BEH - i5-8259U, 32GB RAM, 1TB NVMe, 1x 10G networking via TB)
- 1 Nested VSAN Witness host
- iSCSI storage as a supplement for VSAN using my Windows Server NAS (Storage Spaces, yes - it sucks)
- vCenter (1)
- NSX-T Manager (1), with no routing (VLAN Backed NSX-T Microsegmentation).
- VMware vRA Suite
- VMware vRLi

# Networking:

## Physical:

- 1x ES-48-500W (Ubiquiti)
- 1x ES-16-XG (Ubiquiti)
- 1x Unifi UDM

All ESXi hosts has 10g from the ES-16-XG.
Unfortunately its impossible to get a hold of EdgeSwitches as of writing this, and i really want redundancy for my VSAN, so getting a ES-16-XG more would be lovely. I really dont like switches in Unifi, as they are very limited.

- Multiple VM's with VyOS with Wireguard for other sites.

## Network setup:
- 15 VLANS
- 1 VLAN for NSX VMs
- Everything routed in Unifi UDM
