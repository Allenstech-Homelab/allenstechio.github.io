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

## The Beginning

The first step for me was identifying which code repository software I wanted utilize. I ended up settling on GitHub(github.com) to store my terraform code. I didn't necessarily care if my code was in the cloud or on-prem seeing as this was a homelab project. Your decision may be influenced on a plethora of requirements depending on if this is being setup for an enterprise organization. If you prefer to host your code locally GitHub does offer GitHub Enterprise Server which is a self-hosted version of the GitHub platform. Other recommendations include GitLab, Azure Devops Servers, and Gitea.