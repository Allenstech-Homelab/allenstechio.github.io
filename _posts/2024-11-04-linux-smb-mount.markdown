---
layout: single
title: "Create a systemd SMB mount on (Debian) Linux"
date: 2024-11-04
categories: [blog]
tags: [Jekyll, Minimal Mistakes]
author_profile: true
---
Recently while I was working in my homelab I came across the need to mount some SMB shares to my LXC containers. The shares needed to persist through reboots as well and I wanted to create a systemd unit to accomplish these tasks.

1. Install cifs-utils via apt:
```
apt install cifs-utils -y
```
2. Next we need to create a set of credentials for the CIFS share to use:
```
nano /root/.cifs_credentials
```
```
username=usernamehere
password=insertpasswordhere
```
3. Create a directory to mount your CIFS share on:
```
mkdir -p /mnt/exampledir
```
4. Identify the id of the user and group of your currently logged in user. In my case I was logged in as root.
```
id -u root && id -g root
```
Save the id from this command to be used in the next step.
5. Create the systemd unit and insert the following contents into the file:
```
nano /etc/systemd/system/mnt-media.mount
```
6. Insert the following into the new .mount file:

    ```
    [Unit]
     Description=Media Server files
     Requires=network-online.target
     After=network-online.target

    [Mount]
     What=//192.168.0.1/media/
     Where=/mnt/media
     Options=credentials=/root/.cifs_credentials,uid=0,gid=0
     Type=cifs

    [Install]
     WantedBy=multi-user.target
    ```
7. Enable systemd-networkd-wait-online. systemd-networkd-wait-online is a oneshot system service that waits for the network to be configured. Without this enabled our new systemd service will fail to run on a server boot up.
```
sudo systemctl enable systemd-networkd-wait-online.service
```
8. Enable the new systemd service.
```
systemctl enable mnt-media.mount
```
9. Reboot your server for changes to take affect. If you experience any issues check the journalctl logs.