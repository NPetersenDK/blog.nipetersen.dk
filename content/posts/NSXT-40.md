---
title: "NSX-T 4.0.0.1 - Whats new?"
date: 2022-08-17T00:10:00+01:00
draft: false
---

In NSX-T 4.0.0.1, VMware changed their naming scheme (again, would some say), to just be NSX.
One of the biggest features, and long overdue is IPv6 support for management. There is also a new feature for Blocking Malicious IPs, which we will look at below.

Here are the Release Notes: https://docs.vmware.com/en/VMware-NSX/4.0/rn/vmware-nsx-4001-release-notes/index.html

# Block Malicious IPs:
In the Release Notes the following is written:

- Block Malicious IPs in Distributed Firewall is a new capability that allows the ability to block traffic to and from Malicious IPs.
- Block Malicious IPs in Distributed Firewall is a new capability that allows the ability to block traffic to and from Malicious IPs. This is achieved by ingesting a feed of Malicious IPs provided by Vmware Contexa. This feed is automatically updated multiple times a day so that the environment is protected with the latest malicious IPs. For existing environments the feature will need to be turned on explicitly. For new environments, the feature will be default enabled

My LAB enviorment is a existing installation, so it will need to be turned off explicitly as the release notes says. Luckily thats quite easy in NSX-T.

We also know its a part of VMware Contexa, that is VMwares take on a cloud security platform. I actually didnt know about Contexa before this update, it looks cool. We might see more of Contexa in later NSX-Releases, maybe within NSX-Intelligence where more of the Contexa looks to be already.

## Lets setup auto update:
As you can see below, you will right away after upgrading the NSX-T Manager to 4.x the warnings telling you: 
*Auto Update Malicious IPs is turned off. All rules containing groups with malicious IPs might not work at all or work with outdated data if available.* 

[![NSX-T DFW Warnings](/img/NSX-T_DFW-Warnings-MaliciousIPS.png)](/img/NSX-T_DFW-Warnings-MaliciousIPS.png)


<!--more-->


If you click settings you will see this:

[![NSX-T Malicious IP Auto Update](/img/NSX-T_DFW-MaliciousIPS-AutoUpdate.png)](/img/NSX-T_DFW-MaliciousIPS-AutoUpdate.png)

So there is not that much to set up actually:
- Enable Auto Updating
- It will update every 12 hours (The changelog says "Multilpe times a day", so there is the answer to that)
- Last updated
- Manually download/update

If you enable it, the warning will go away.

## How does the firewall rules look?
In the Distributed Firewall the new firewall rules will be under "Infrastructure" as the top section/policy, named "Default Malicious IP Block Rules"

Limitations:
- You cannot move the section/policy
- You can not add/remove things to the rules

What you can do:
- Enable logging (you really should, for trouble shooting). The rules come with a predefined LOG-LABEL which can be changed to your liking.
- Enable/disable rules (disabled per default if you upgraded to 4.x)
- Change the action

[![NSX-T DFW Rules](/img/NSX-T_DFW-MaliciousIPS-DFWRules.png)](/img/NSX-T_DFW-MaliciousIPS-DFWRules.png)

## How does the group look?
The group with the Malicious IPS are called "DefaultMaliciousIpGroup", and is a IP Address Only group.

Limitations:
- It should be possible to see the Malicious IPs but i didnt find away.
- You cannot add IPs you think is Malicious to the group.

What you can do:
- Add Exception IP Addresses

# What do I think of it?
This is really valuable feature that many might look forward to try out, and a good feature for NSX as a Sofware-Defined FW. I would have liked to be able to see the IP-Addresses, in case of troubleshooting, but that might come in either a later update, or a way to do it in the API. When I tried looking up the members through the API it seemed empty.

# UPDATE 03-09-2022:
I tried downloading somthing from Github (https://github.com/*), and was denied somehow. After some troubleshooting i found out that Github was a part of the Malicious IP's, oops. I excluded the IP-Addresses that my nslookup gave, in the "DefaultMaliciousIpGroup" and that solved it. Because this might be different IP's, it could be an idea to create a rule over this using FQDN Firewalling to get around this. As of writing this, my IPs for Github was: 140.82.121.3 and 140.82.121.4
