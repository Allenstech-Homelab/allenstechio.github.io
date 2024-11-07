---
layout: single
title: "Leveraging Terraform Cloud with Proxmox"
date: 2024-11-06
categories: [tutorial]
tags: [devops, terraform, cloud, proxmox]
author_profile: true
permalink: /tf-cloud-proxmox/
hidden: true
---

## Overview

One of my favorite aspects of technology is automation. In the infrastructure realm, automation shines by freeing engineers from mundane tasks, enabling them to focus on more impactful work. Beyond saving time, automation introduces reliable, repeatable, and secure processes, ensuring infrastructure is robust and resilient. Several years ago I was introduced to the concept of IaC (Infrastructure as code) and I found it to be fascinating.

Roughly a year ago, I decided to explore using Terraform to streamline my personal Proxmox environment. My goal was to set up CI/CD pipelines to fully automate the provisioning of VMs and LXCs. While I was already familiar with using Terraform via the CLI, I wanted to take it a step further by incorporating elements of the SDLC, aiming for a truly seamless, automated process.

Follow along below for an overview of how I set up Terraform Cloud with pipelines to bring this vision to life. I’ll walk through my configuration, the integrations I used, and some tips I picked up along the way for optimizing the automation process.

## Terraform Code Repository

My first step was to determine which code repository platform best suited my needs. Ultimately, I chose GitHub (github.com) to store my Terraform code. Since this was a homelab project, I was flexible about hosting in the cloud or on-premises. However, your decision may depend on various requirements, particularly if you're setting up for an enterprise environment.

For those preferring on-premises hosting, GitHub offers GitHub Enterprise Server, a self-hosted version of the platform. Other solid alternatives include GitLab, Azure DevOps Server, and Gitea, each with unique features and configurations tailored to different use cases.

## Why Terraform Cloud vs Azure, AWS, GCP?

Terraform is a stateful infrastructure-as-code (IaC) tool, meaning it uses a tfstate file to track and manage infrastructure changes. Since GitHub Actions doesn’t natively store this file, additional storage solutions are necessary, such as storing the state in cloud providers like Azure, AWS, or GCP. However, Terraform Cloud offers several compelling advantages over these alternatives.

One key benefit of Terraform Cloud is its built-in state management with a locking feature, which prevents multiple users or pipelines from modifying the state simultaneously. This greatly reduces the risk of conflicts and state corruption.

Another major advantage is collaboration. Terraform Cloud is purpose-built for team environments, enabling multiple users to work on infrastructure efficiently with shared visibility and access.

Other notable features include version control integration that triggers Terraform runs directly from supported VCS platforms (such as GitHub and GitLab) with every push to a branch. It also offers role-based access control (RBAC), enhancing security by allowing fine-grained permissions. Plus, for smaller teams, Terraform Cloud offers a free tier, making it a cost-effective solution for getting started.

You'll need to sign up for a free Terraform Cloud account here (https://app.terraform.io/session)
If you are interested, the free and paid Terraform cloud plan comparison can be found here (https://developer.hashicorp.com/terraform/cloud-docs/overview)

## Self-hosted TF Cloud Execution Agent

Since im hosting Proxmox on-prem in my homelab im going to be using a self-hosted Proxmox LXC to execute my code locally on-prem within my network.

# Here's How I did it

## Prerequisites
- Terraform CLI (version 1.1.0+) installed locally (https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
- Sign up for a GitHub account and create a new repository (https://github.com)
- Clone your repository locally by either using a Git Client or command line. I prefer to use VSCode (https://code.visualstudio.com/) If you don't have a terraform repository or are just learning terraform the first time, hashicorp provides an example learning repository you can use. (git clone https://github.com/hashicorp/learn-terraform.git)
- Sign up for a HCP Terraform Cloud account here (https://app.terraform.io/session)

## Step by Step
1. Create an Organization in TF Cloud
2. Create a workspace in TF Cloud
3. Connect your GitHub to TF Cloud VCS (OAuth - https://developer.hashicorp.com/terraform/cloud-docs/vcs/github)
4. Using a CLI issue the "terraform login" command
5. A browser will pop-up automatically. Enter a token name, click Create API Token, paste the token value in your CLI interface. If you need to save the token for any reason, be sure to store it in a secure location. If all goes well you should see a "Welcome to HCP Terraform!" message.
6. Create a credentials variable set in Settings -> Variable Sets -> Create Variable Set. I recommend creating a variable set rather than workspace specific variables so that they can be used across different workspaces you create in the future. Choose the "apply globally" setting, and when adding your credential variables be sure to choose the environment variable category. You will also see a checkbox to mark the variable as sensitive. I highly recommend using the sensitive label for any credential related environment variables. To save the credential set click "Create variable set"
7. Update your main.tf to include your TF Cloud organization name and workspace that you created earlier. I provided code for you at the bottom of this post that you can use as a reference for building your main.tf and vars.tf files.
8. Access your CLI and change directories to where your terraform files are are run the following command:
```
terraform init
```
9. Navigate to your new workspace, Select Variables in the workspace's menu, Under Variable sets, click Apply variable set.
10. At this point from your CLI you can run:
```
terraform plan
``` 
11. You can check your terraform plan in the CLI or you can view this in the Terraform Cloud UI.
12. If you are satisfied with the changes that are planned you can run:
```
terraform apply
```
This will apply the proposed changes to your infrastructure.

## Proxmox Terraform file examples (main.tf & vars.tf)

### Main.tf
```
terraform {
  required_version = "~> 1.9"
  cloud {
    organization = "Blog-Organization"

    workspaces {
      name = "Blog-Workspace"
    }
  }

  required_providers {
    proxmox = {
      source = "Telmate/proxmox"
      version = "3.0.1-rc4"
    }
  }
}

provider "proxmox" {
  # URL is the hostname (FQDN if you have one) for the proxmox host you'd like to connect to to issue the commands.
  pm_api_url = "https://1.1.1.1:8006/api2/json"

  # API token id from Proxmox
  pm_api_token_id = var.token_id

  # This is the full secret wrapped in quotes.
  pm_api_token_secret = var.token_secret

  # Leave tls_insecure set to true unless you have your proxmox SSL certificate situation fully sorted out.
  pm_tls_insecure = true
}

resource "proxmox_vm_qemu" "pelican-01" {

  # VM settings
  vm_state                            = "running"
  vmid                                = 106
  tags                                = "linux"
  name                                = "pelican-01" 
  target_node                         = "pve-01"
  clone                               = "2204-TPL-PKR"
  full_clone                          = "true"
  agent                               = 1
  os_type                             = "cloud-init"
  cores                               = 4
  sockets                             = 1
  cpu                                 = "host"
  memory                              = 16384
  scsihw                              = "virtio-scsi-single"
  qemu_os                             = "other"
  desc                                = "Managed by Terraform"
  disks {
    scsi {
      scsi0 {
        disk {
          size = 60
          storage = "ceph-storage-01"
          emulatessd = "true"
          discard = "true"
          iothread = "false"
          backup  = "true"
          cache   = "none"
        }
      }
    }
    ide {
      ide0 {
        cloudinit {
          storage = "ceph-storage-01"
        }
      }
    }
  }

  network {
    model = "virtio"
    bridge = "vmbr1"
    tag = 120
  }
  ##########################################
  ########### CLOUD-INIT SETTINGS ##########
  ##########################################
  ipconfig0 = "ip=dhcp"
  ssh_user = "michael"
  sshkeys = <<EOF
  ${var.ssh_key}
  EOF
}
```

### Vars.tf
```
#SSH key used during cloud-init configuration
variable "ssh_key" {
    type        = string
    description = "Public SSH Key to install inside VM"
    default     = "ssh key output goes here"
}

#Token ID from Proxmox API (Passed thru and handled by TF Cloud Variables)
variable "token_id" {
    type        = string
    description = "API Token ID from Proxmox"
    sensitive   = true
}

##Token secret from Proxmox API Account (Passed thru and handled by TF Cloud Variables)
variable "token_secret" {
    type        = string
    description = "API Token Secret from Proxmox"
    sensitive   = true
}
```