---
tags: software-development, jenkins
---
# Jenkins Pipeline Network Traffic Capture & Analysis Guide

This guide details how to securely capture raw network traffic (`.pcap`) during an automated build on a Jenkins agent, safely manage the background capture process, and parse the resulting binary file into a clean list of firewall requirements.

## Phase 1: Agent Preparation

The Jenkins agent requires elevated privileges to run `tcpdump` and manage the resulting files, but granting full root access is a security risk.

1. SSH into the Jenkins agent as an administrator.
2. Create a targeted `sudoers` drop-in file that strictly limits the `jenkins` user to only the commands needed for this process:

    ```bash
    sudo su -
echo "jenkins ALL=(ALL) NOPASSWD: /usr/sbin/tcpdump, /usr/bin/kill, /usr/bin/chown, /usr/bin/rm, /bin/rm, /usr/bin/pkill" > /etc/sudoers.d/jenkins-tcpdump
    ```
