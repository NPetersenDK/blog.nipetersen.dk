---
title: "Building a VMware SelfService Portal with the power of Github Copilot, PowerShell and Azure"
date: 2026-04-04T13:00:00+01:00
draft: false
tags: ["VMware", "PowerShell", "Azure", "vCenter", "SelfService", "AI", "Project", "Github", "Open Source"]
---

Last month I built a SelfService Portal for VMware vCenter as an alternative to a full-fledged automation platform. The idea was simple: let users provision their own VMs in minutes without manual work needed and without paying for an expensive platform to do it. The goal was to have a maintainable, cost-effective solution that could be easily deployed in any environment with a vCenter and some Azure services for under $15 per month.

I wanted to see how far I could come with a "vibecoded" solution, but still work in something I know, and that is something I'm sure I can maintain with and without the need of AI.

<!--more-->

# Why did I try to build this?

I do not work with any VMware components in my professional life anymore, but I have a background as a VMware System Administrator, and I still have access to a lab running vSphere. I wanted to try to bridge the gap between my current work with Cloud solutions and my past experience with VMware, and this project was a perfect way to do that. Some of the things we do here are things I wouldn't have done without my experience with Cloud, working with Developers and the offerings from Azure.

vRealize Automation (now Aria Automation) and other automation platforms are powerful, but they often come with a price tag and a maintenance burden that not every organisation can justify. If all you want is to let users spin up VMs from a set of approved templates, on approved clusters and networks, without involving the helpdesk every time - you do not need a full-blown platform to do that.

I was a VMware System Administrator for years. I know the PowerCLI commands. I know what vCenter can do and how to talk to it, so the project was more about putting the pieces together in a new way, rather than learning new technologies from scratch. I also wanted to see how far I could get with a "vibecoded" solution, using GitHub Copilot to generate large amounts of the code quickly.

## My goals for the project were:
- Build a Self-Service portal on plain HTML and CSS from Bootstrap (Bootstrap is a great way to get a decent-looking UI without needing to be a designer or set up a complex frontend framework).
- Use Azure Static WebApps for hosting the frontend, which helps you get authentication and authorization out of the box with Entra ID, and also makes deployment super easy.
- Azure Functions for the backend API using PowerShell. By doing this we keep it simple, because we use the same PowerCLI commands we already know. We do not need to learn VMware's REST API or a new programming language.
- Use Azure Table Storage for configuration and state management. This keeps the backend stateless and simple, and allows us to manage configuration without touching files or redeploying code. Azure Table Storage is also very cost-effective for this kind of use case.
- Keep costs low. My goal is that it shouldn't exceed a monthly cost of around $15 to run.
- Cloud-Init was a requirement for my Linux Templates. I wanted to be able to inject the cloud-init configuration through guestinfo properties, and have the backend handle that when provisioning the VM.
- A user should be able to log in, see the available templates, clusters, and networks, and provision a VM with a few clicks. They should also be able to see their existing VMs and delete them if needed.
- An admin should be able to manage the allowed templates, clusters, networks, and system names through the frontend without needing to touch any config files or code.

I wanted to have this in 2 repositories, one for the backend and one for the frontend, to keep things organized and separate. 

## How it works

The backend is an Azure Functions app written in PowerShell, running in a Docker container. The idea is that you run it close to your vCenter - on-premises in a VM with network access to vCenter. 

The frontend is plain HTML with Bootstrap for styling, deployed to an Azure Static WebApp. Authentication and authorization is handled by Azure Static WebApps and Entra ID, so you do not need to build any login logic yourself.

The configuration templates, clusters, networks, and system names that are available are stored in Azure Table Storage. The admin section of the frontend lets you manage these without touching any config files. When a user creates a VM, the request goes to the backend API, which reads the allowed config from Table Storage, provisions the VM on vCenter, and records it back to Table Storage.

The API itself is protected by a function key, which the Static WebApp picks up from a Key Vault secret or a Static WebApp environment secret.

By separating the frontend and backend with a REST API, this architecture isn't limited to just the web interface. Other applications, scripts, or automation tools can consume the API directly. This means you could integrate VM provisioning into whatever you want. CI/CD Pipelines, build custom tooling, make it able to be consumed by others - all without touching the web frontend.

## The backend in a bit more detail

The PowerShell part is what made this fast to build. The commands I needed:  `New-VM`, `Get-Template`, `Get-Cluster`, `Get-VirtualPortGroup` - are things I had written dozens of times before. Wrapping them in an Azure Function with a JSON request body was straightforward.

The backend exposes a REST API with several endpoints for different actions: creating VMs, listing existing VMs, deleting VMs, and managing configuration (templates, clusters, networks, and system names).

There is a `StorageModule` that handles all Table Storage operations using the REST API directly with key authentication, and a `VMwareModule` that handles the actual provisioning. Cloud-init is also supported for Linux templates. From a public source like GitHub Gist, the hostname is injected, and it is passed to the VM via guestinfo properties.

Since vSphere 7 and later support cloud-init through guestinfo, the backend takes care of fetching the cloud-init configuration from the provided URL, injecting the hostname, and passing it to the VM during provisioning. This allows for seamless integration of cloud-init with your Linux templates, making it easier to manage and customize your VMs, and also update the cloud-init templates without needing to touch VM Templates and redeploy the backend.

William Lam has a great blog post on how to set up cloud-init: [Cloud-Init on vSphere](https://williamlam.com/2022/07/exploring-the-cloud-init-datasource-for-vmware-guestinfo-using-vsphere.html).

## The frontend

I am by no means a frontend developer, but I have been writing HTML and a bit of CSS for years. I did not want to learn a frontend framework and to be honest, I have always just used [Bootstrap](https://getbootstrap.com/) for quick and simple UIs. It is not the most modern or fancy way to build a frontend, but it gets the job done without needing to learn React, Angular, Vue, or whatever the latest framework is.

It also means that the frontend is just plain HTML and CSS, which makes it super easy to edit and customize. The frontend is backed by Azure Static WebApps that gives some nice features:

- Authentication and Authorization with Entra ID out of the box. You can easily set up role-based access control to restrict who can provision VMs and who can manage the configuration.
- An easy deployment pipeline. You basically get the Github Actions pipeline when you create the WebApp. So there is CI/CD out of the box.
- Custom domains and SSL are also super easy to set up with Static WebApps.
- Managed by Microsoft

## Cost

This is where it gets interesting. The Azure Static WebApp in the Standard tier is around $9 per month. The Azure Storage Account used for Table Storage and configuration will be well under $2 per month for most organisations. The Docker container running the backend can run on any VM or server you already have near vCenter - there is no additional Azure cost for that part if you run it on-premises.

So the total cost to run this platform is roughly $10-11 per month, plus whatever you already spend on infrastructure close to vCenter. Compare that to a vRealize Automation licence.

## Repositories

- Backend: [NPetersenDK/VMware-SelfService-Backend](https://github.com/NPetersenDK/VMware-SelfService-Backend)
- Frontend: [NPetersenDK/VMware-SelfService-Frontend](https://github.com/NPetersenDK/VMware-SelfService-Frontend)

Both repos include a disclaimer: this is a starting point, not a production-ready solution. Review the code, harden it for your environment, and make it your own. That is the whole point. I only built the foundation and the basic functionality, it still needs work to be production-ready, but it is a great starting point for anyone looking to build something similar.

Also if it wasn't clear the first time: This is coded with large amounts of AI.

## What did I learn from the vibecoding approach?
This project was built heavily using the vibecoding approach and GitHub Copilot CLI. I had a clear vision of what I wanted to build, and I used Copilot to generate the first draft of the code for both the backend and frontend. 

I quickly learned that giving Copilot CLI access to both the backend and frontend repos allowed it to fix things one place, and adjust it another. For example, I asked for a change in how the template configuration was stored, and it adjusted both the backend and frontend in one go, and made sure everything was working.

I was kind of amazed by how quickly I had something. 

My first prompt was this:

> I want to build a self-service portal for VMware vCenter using Azure Static WebApps for the frontend and Azure Functions with PowerShell for the backend. The backend should use PowerCLI to talk to vCenter and Azure Table Storage for configuration. The frontend should be simple HTML with Bootstrap. Can you help me get started with the backend code?. The frontend should have a form to create VMs and an admin section to manage templates and clusters. The backend should have endpoints for creating VMs, listing VMs, and managing config. I want to keep it simple and cost-effective, so no complex frameworks or databases. The form should include the following:
> - VM Name
> - Template (dropdown) (A Display Name that maps to a Template in vCenter and a Cloud-Init URL)
> - Cluster (dropdown) (A Display Name that maps to a Cluster in vCenter)
> - Network (dropdown) (A Display Name that maps to a Network in vCenter)
> - System Name (dropdown)

I think i have used around 5 hours in total on this project. And now the platform is ready for the next prompts to make it the specific needs for the environment it will be used in.

### What did i notice?
You still need to understand whats going on, and whats being generated. You cant just ask for "Make me a self-service portal" and expect it to be perfect. You need to guide it, review the code, and make sure it fits your needs. But if you do that, you can get something up and running in no time.

I also quickly saw some issues different places, that i would not have noticed if I didn't have the experience with the technologies I asked for. Some errors came up in the backend code, but it was actually more of a issue with how it PowerCLI and VMware. 

So having the experience with the technologies you are asking for is still important, because it allows you to review the generated code and make sure it makes sense, and also to guide the generation in the right direction.

## Video

Here is a short video showing the portal in action:
{{< youtube CJ7mupdZHSo >}}