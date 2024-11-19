---
layout: single
title: "GitHub Pages with Jekyll"
date: 2024-11-19
categories: [tutorial]
tags: [jekyll, github, pages, website]
author_profile: true
permalink: /ghpages-setup-cloudflare-jekyll/
hidden: true
---
I am documenting my experience configuring GitHub Pages with Jekyll and integrating a custom domain managed through Cloudflare. This site leverages GitHub Pages for hosting, incorporates a custom Jekyll theme, and is securely proxied behind a Cloudflare-managed custom domain. During my setup, I found a lack of comprehensive guides on using a custom Cloudflare domain within this setup. To address this gap, I am sharing a detailed walkthrough of my implementation process. Note: I'm running through this guide assuming you are using a windows client.

### Prerequisites

- [GitHub Account](https://github.com)
- [Cloudflare Account](https://www.cloudflare.com/)
- [Git](https://git-scm.com/downloads)
- [VS Code](https://code.visualstudio.com)
- [Ruby](https://rubyinstaller.org)
- Jekyll - Covered later in this post

### Install Jekyll
1. Open a Windows Terminal
2. Run the following command:
```
gem install jekyll bundler
```
3. After the command completes, check to see what versions you have installed:
```
jekyll -v
```

### Create a public GitHub Repository