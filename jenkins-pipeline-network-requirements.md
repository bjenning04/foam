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

## Phase 2: Jenkinsfile Integration

Before implementing any of the following steps, ensure there are no stages that call `cleanWs()` or otherwise clean or delete all of the contents of the workspace.

### The Setup Stage

Insert this stage immediately before the build steps you want to monitor.

```groovy
stage('Start Network Capture') {
    steps {
        script {
            sh """
            echo "--- STARTING TCPDUMP SETUP ---"
            
            echo "1. Cleaning up old files..."
            sudo /usr/bin/rm -f "${WORKSPACE}/full_build_capture.pcap" || true
            sudo /usr/bin/rm -f "${WORKSPACE}/tcpdump_debug.log" || true
            
            echo "2. Launching tcpdump in background..."
            JENKINS_NODE_COOKIE=dontKillMe
            nohup sudo /usr/sbin/tcpdump -i any -w "${WORKSPACE}/full_build_capture.pcap" > "${WORKSPACE}/tcpdump_debug.log" 2>&1 & 
            
            echo "--- SETUP COMPLETE ---"
            """
        }
    }
}
```
