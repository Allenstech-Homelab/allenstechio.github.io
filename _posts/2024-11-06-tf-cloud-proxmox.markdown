---
layout: single
title: "Leveraging Terraform Cloud with Proxmox"
date: 2024-11-06
categories: [tutorial]
tags: [devops, terraform, cloud, proxmox]
author_profile: true
---

## Overview

One of my favorite aspects of technology is automation. In the infrastructure realm, automation shines by freeing engineers from mundane tasks, enabling them to focus on more impactful work. Beyond saving time, automation introduces reliable, repeatable, and secure processes, ensuring infrastructure is robust and resilient. Several years ago I was introduced to the concept of IaC (Infrastructure as code) and I found it to be fascinating.

Roughly a year ago, I decided to explore using Terraform to streamline my personal Proxmox environment. My goal was to set up CI/CD pipelines to fully automate the provisioning of VMs and LXCs. While I was already familiar with using Terraform via the CLI, I wanted to take it a step further by incorporating elements of the SDLC, aiming for a truly seamless, automated process.

Follow along below for an overview of how I set up Terraform Cloud with pipelines to bring this vision to life. I’ll walk through my configuration, the integrations I used, and some tips I picked up along the way for optimizing the automation process.

## Terraform Code Repository

My first step was to determine which code repository platform best suited my needs. Ultimately, I chose GitHub (github.com) to store my Terraform code. Since this was a homelab project, I was flexible about hosting in the cloud or on-premises. However, your decision may depend on various requirements, particularly if you're setting up for an enterprise environment.

For those preferring on-premises hosting, GitHub offers GitHub Enterprise Server, a self-hosted version of the platform. Other solid alternatives include GitLab, Azure DevOps Server, and Gitea, each with unique features and configurations tailored to different use cases.

## HCP Terraform
There were a few reasons why I decided to use the Terraform cloud platform instead of solely using GitHub Actions. Terraform is a stateful IaC tool. It uses the tfstate file in order to determine what changes should be made to the infrastructure you are managing within in it. GitHub actions alone does not natively store the tfstate file. There are alternatives to using the Terraform Cloud such as storing your state in Azure or AWS however Terraform Cloud offers many benefits since its sole purpose is focused towards Terraform and managing the state properly. The Terraform Cloud platform also natively integrates with GitHub using a VCS connection.
You'll need to sign up for a free Terraform Cloud account here (https://app.terraform.io/session) If you are interested, the free and paid Terraform cloud plan comparison can be found here (https://developer.hashicorp.com/terraform/cloud-docs/overview)

## Self-hosted GitHub Runner & TF Cloud Execution Agent
Since im hosting proxmox on-prem in my homelab im going to be using a self-hosted GitHub Runner (Proxmox LXC) to execute my code locally within my network. The same goes for the terraform cloud as I will be using the same self-hosted Proxmox LXC as the TF Cloud agent for plans and applies.

## GitHub Actions Setup
Once you connect your GitHub repository to the TF Cloud platform it will automatically generate a Github actions yaml file for you and place it within the ".github/workflows" directory.

## GitHub Secrets