---
title: "My Homelab"
date: 2020-02-14T00:16:52+01:00
draft: false
---

My homelab is based on 4 hosts and one fileserver (running Windows, for now).

## My VMware Installation
I try to be a front runner for running the newest version of everything. So as of now i'm on 6.7U3 on everything.
I have 5 hosts in my installation, which one of them is in France (i'm living in Denmark) - i have a 500/500 Mbit connection to it.
<!--more-->

### The 4 hosts is locally is:
- Dell R210ii (Xeon E-1265LV2) - 32GB RAM
- HP MicroServer Gen8 (Xeon E-1265L V2) - 16GB RAM
- Intel NUC NUC8i5BEH (i5-8259U) - 32GB RAM
- Intel NUC (Some ancient shit, running the VCSA in a cluster for it self) - 16GB

The 3 of the hosts is a part of a cluster named "ProductionCluster" which is my primary cluster for testing and stuff. 
Right now i'm thinking about making a vSAN cluster out of some newer kind of hosts, since i have the network to do it - and by doing that i can lab pretty much everything.

## Networking:
They are connected to a EdgeSwitch 16XG (10g) switch, and a EdgeSwitch 24L.
Everything is redundant so i can loose a switch, and the biggest impact it would get is running 1g instead of 10g. Which is fine, because quite frankly 1g is enough for my enviorment. But go big or go home :)

I have VLAN'ed everything into 7 VLAN's for each kind of network. Management, Server, Guest, Wifi Users, Clients, VPN and more.

### NSX
Running NSX-T on this cluster although as of now without any routing, but just for getting the micro-segmentation into it.

If you have any questions, please let me know :)
