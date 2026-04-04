---
title: "Building a VMware SelfService Portal with PowerShell and Azure"
date: 2026-04-04T13:00:00+01:00
draft: false
tags: ["VMware", "PowerShell", "Azure", "vCenter", "SelfService", "AI", "Project", "Github", "Open Source"]
---

Last month I built a SelfService Portal for VMware vCenter as an alternative to the VMware vRealize Automation platform. The idea was simple: let users provision their own VMs without raising tickets, while admins retain control over where and how VMs can be created — without paying for an expensive platform to do it. The goal was to have a maintanable, cost-effective solution that could be easily deployed in any environment with a vCenter and some Azure services for under $15 per month.

I wanted to see how far i could come with a "vibecoded" solution, but still work in something i know of, and that is something im sure i can maintain with and without the need of AI.

<!--more-->

# Why did I try to build this?

I do not work with any VMware components in my professional life anymore, but I have a background as a VMware System Administrator, i still have access to a lab running vSphere. I wanted to try to bridge the gap between my current work with Cloud solutions and my past experience with VMware, and this project was a perfect way to do that. some of the things we do here, is things i wouldnt have done without my experience with Cloud and the offerings from Azure.

vRealize Automation (now Aria Automation) and other automation platforms are powerful, but they often come with a price tag and a maintenance burden that not every organisation can justify. If all you want is to let users spin up VMs from a set of approved templates, on approved clusters and networks, without involving the helpdesk every time - you do not need a full-blown platform to do that.

I was a VMware System Administrator for years. I know the PowerCLI commands. I know what vCenter can do and how to talk to it, so the project was more about putting the pieces together in a new way, rather than learning new technologies from scratch. I also wanted to see how far I could get with a "vibecoded" solution, using GitHub Copilot to generate large amounts of the code quickly.

## My goals for the project were:
- Build a Self-Service portal on plain HTML and CSS from Bootstrap (Bootstrap is a great way to get a decent-looking UI without needing to be a designer or set up a complex frontend framework).
- Use Azure Static WebApps for hosting the frontend
- Azure Functions for the backend API using PowerShell. By doing this we keep it simple, due to we use the same PowerCLI commands we already know. We do not need to learn VMwares REST API or a new programming language.
- Use Azure Table Storage for configuration and state management. This keeps the backend stateless and simple, and allows us to manage configuration without touching files or redeploying code.
- Keep costs low. My goal is that it shouldnt exceed a monthly cost of around $15 to run.

## How it works

The backend is an Azure Functions app written in PowerShell, running in a Docker container. The idea is that you run it close to your vCenter — on-premises in a VM with network access to vCenter. 

The frontend is plain HTML with Bootstrap for styling, deployed to an Azure Static WebApp. Authentication and authorization is handled by Azure Static WebApps and Entra ID, so you do not need to build any login logic yourself.

Configuration — which templates, clusters, networks, and system names are available — is stored in Azure Table Storage. The admin section of the frontend lets you manage these without touching any config files. When a user creates a VM, the request goes to the backend API, which reads the allowed config from Table Storage, provisions the VM on vCenter, and records it back to Table Storage.

The API itself is protected by a function key, which the Static WebApp picks up from a Key Vault secret or a Static WebApp environment secret.

The project ended up being two repositories: a [backend](https://github.com/NPetersenDK/VMware-SelfService-Backend) and a [frontend](https://github.com/NPetersenDK/VMware-SelfService-Frontend).

## The backend in a bit more detail

The PowerShell part is what made this fast to build. The commands I needed — `New-VM`, `Get-Template`, `Get-Cluster`, `Get-VirtualPortGroup` — are things I had written dozens of times before. Wrapping them in an Azure Function with a JSON request body was straightforward.

There is a `StorageModule` that handles all Table Storage operations using the REST API directly with SharedKeyLite authentication, and a `VMwareModule` that handles the actual provisioning. Cloud-init is also supported for Linux templates: userdata is fetched from a GitHub Gist, the hostname is injected, and it is passed to the VM via guestinfo properties.

The API endpoints are:

| Endpoint | Description |
|---|---|
| `GET /api/ping` | Health check |
| `GET /api/GetOptions` | Returns available templates, clusters, networks, system names |
| `POST /api/CreateVM` | Provisions a VM from a template |
| `GET /api/ListVMs` | Lists VMs for a given responsible |
| `GET/POST/DELETE /api/manage-config` | Admin config management |

## The frontend

The frontend is as simple as I could make it without it looking bad. Bootstrap handles the layout and styling, which means making changes to the look is straightforward — you do not need a build pipeline or a framework, just edit the HTML. There are two sections: a self-service section for users to create and view their own VMs, and an admin section for managing the allowed options.

## Cost

This is where it gets interesting. The Azure Static WebApp in the Standard tier is around $9 per month. The Azure Storage Account used for Table Storage and configuration will be well under $2 per month for most organisations. The Docker container running the backend can run on any VM or server you already have near vCenter — there is no additional Azure cost for that part if you run it on-premises.

So the total cost to run this platform is roughly $10-11 per month, plus whatever you already spend on infrastructure close to vCenter. Compare that to a vRealize Automation licence.

## Video

Here is a short video showing the portal in action:

{{< youtube CJ7mupdZHSo >}}

## Repositories

- Backend: [NPetersenDK/VMware-SelfService-Backend](https://github.com/NPetersenDK/VMware-SelfService-Backend)
- Frontend: [NPetersenDK/VMware-SelfService-Frontend](https://github.com/NPetersenDK/VMware-SelfService-Frontend)

Both repos include a disclaimer: this is a starting point, not a production-ready solution. Review the code, harden it for your environment, and make it your own. That is the whole point. I only built the foundation and the basic functionality, it still needs work to be production-ready, but it is a great starting point for anyone looking to build something similar.

Also if it didn't come clear the first time: This is coded with large amounts of AI.

## What did i Learn of the vibecoding approach?
This project was built heavily using the vibecoding approach and Github Copilot CLI. I had a clear vision of what I wanted to build, and I used Copilot to generate code snippets, handle boilerplate, and even write some of the logic.

I was kind of amazed of how quickly i had something. 

My first prompt was this "I want to build a self-service portal for VMware vCenter using Azure Static WebApps for the frontend and Azure Functions with PowerShell for the backend. The backend should use PowerCLI to talk to vCenter and Azure Table Storage for configuration. The frontend should be simple HTML with Bootstrap. Can you help me get started with the backend code?. The frontend should have a form to create VMs and an admin section to manage templates and clusters. The backend should have endpoints for creating VMs, listing VMs, and managing config. I want to keep it simple and cost-effective, so no complex frameworks or databases. The form should include the following:
- VM Name
- Template (dropdown)
- Cluster (dropdown)
- Network (dropdown)
- System Name (dropdown)
- OS Type (dropdown)

I think i have used around 5 hours in total on this project. And now the platform is ready for the next prompts to make it the specific needs for the environment it will be used in.