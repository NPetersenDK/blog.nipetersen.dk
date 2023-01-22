---
title: "Tjest"
date: 2023-01-20T00:02:00+01:00
draft: false
---

NSX-T Version: 4.0.0.1.0.20159689

# Header 1
# Header 2
# Header 3

I was experiencing a high latency in NSX-T for all my VMs, and i couldnt figure out why. VMs on the same host, that wasnt on $
I was pinging from a VM on VLAN 10 to a VM thats part of my NSX environment on VLAN 20. Both VMs was on the same host, and I $

Let me first show you how the latency was fluctuating:
![NSX-T high VM latency](/blog/img/NSX-T_latency_issue.png)

As you can see above the latency was in the low end at 8ms and to the very high end of 150+ ms per ping. Thats not acceptable$


<!--more-->



Because my NSX-T networks are based on VLANs, what I did was create a Distributed Port Group in vCenter on the VLAN, and move the VM there. 

And that fixed the issue - so with that i know its a thing in my VMware/NSX environment and not my switching/routing. As you can see below, the latency is now sub 1ms which is more acceptable.
![NSX-T high VM latency DVPG fix](/blog/img/NSX-T_latency_issue_dvpg_fix.png)

After a lot of trial and error, i tried removing my IDS DFW rule - and that was the issue all along:

![NSX-T high VM latency IDS fix](/blog/img/NSX-T_latency_issue_IDS_Fix.png)

My hosts in my LAB is not the biggest, and as I just said - this is a LAB environment. So it would be fun to see if this is a problem in a production environment aswell. I have had some IDS alerts in the manager, so you might look into this, if you get those type of errors.
