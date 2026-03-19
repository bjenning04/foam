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

### The Cleanup & Archiving Stage

Add this to the `always` block within the pipeline's `post` section. It uses `pkill -f` to target the exact command string, preventing orphaned `tcpdump` processes from running indefinitely and filling up the server's disk.

```groovy
post {
    always {
        script {
            sh """
            echo "--- CLEANING UP TCPDUMP ---"
            
            # Target and kill the specific tcpdump command
            sudo /usr/bin/pkill -f "/usr/sbin/tcpdump -i any -w ${WORKSPACE}/full_build_capture.pcap" || true
            
            # Give it a second to flush the final packets to the file
            sleep 2
            
            # Change ownership from root back to jenkins
            if [ -f "${WORKSPACE}/full_build_capture.pcap" ]; then
                sudo /usr/bin/chown jenkins:jenkins "${WORKSPACE}/full_build_capture.pcap" || true
            fi
            
            echo "--- CLEANUP COMPLETE ---"
            """
        }
        // Expose the files so they can be downloaded from the Jenkins UI
        archiveArtifacts artifacts: '*.pcap, *.log', allowEmptyArchive: true
    }
}
```

## Phase 3: Traffic Analysis

Download the `full_build_capture.pcap` artifact from the Jenkins build to your local machine. Save the following bash script as `network_reqs.sh` in the same directory as the `.pcap` file. Update the `AGENT_IP` variable to match the internal IP address of the Jenkins node that ran the build.

```bash
#!/bin/bash

# --- Configuration ---
PCAP="full_build_capture.pcap"
AGENT_IP="1.2.3.4"
OUTPUT="firewall_requirements.csv"

# --- Safety Checks ---
if ! command -v tshark &> /dev/null; then
    echo "Error: tshark is not installed or not in your PATH."
    exit 1
fi

if [ ! -f "$PCAP" ]; then
    echo "Error: Capture file '$PCAP' not found in this directory."
    exit 1
fi

echo "Analyzing $PCAP for Agent $AGENT_IP..."
echo "Target,Port,Protocol,Description" > "$OUTPUT"

# --- 1. Extract HTTPS/TLS Traffic (Domains) ---
echo "Extracting TLS Domains..."
tshark -r "$PCAP" -Y "ip.src == $AGENT_IP and tls.handshake.type == 1" -T fields \
-e tls.handshake.extensions_server_name -e tcp.dstport \
| sort -u | grep . | while read -r domain port; do
    echo "$domain,$port,TCP,HTTPS Domain" >> "$OUTPUT"
done

# --- 2. Extract TCP Traffic (Raw IPs) ---
echo "Extracting TCP IPs and performing reverse DNS lookups..."
# We filter out port 443 to avoid duplicating the domains we just grabbed above
tshark -r "$PCAP" -Y "ip.src == $AGENT_IP and tcp.flags.syn == 1 and tcp.flags.ack == 0 and tcp.dstport != 443" -T fields \
-e ip.dst -e tcp.dstport \
| sort -u | grep . | while read -r ip port; do
    
    # Set a default description
    desc="Custom TCP Service"
    target="$ip"

    # Check for known static IPs/Ports from your network
    if [[ "$ip" == "146.235.253.206" ]]; then
        desc="Tanium UEM Client"
    elif [[ "$port" == "22" ]]; then
        desc="Git SSH"
    elif [[ "$port" == "80" ]]; then
        desc="Plain HTTP"
    else
        # Attempt to reverse-resolve the IP to a hostname using the Mac's 'host' command
        resolved_name=$(host "$ip" 2>/dev/null | awk '/domain name pointer/ {print $5}' | sed 's/\.$//')
        if [[ -n "$resolved_name" ]]; then
            target="$resolved_name ($ip)"
            desc="Resolved Internal/CDN Host"
        fi
    fi
    
    echo "$target,$port,TCP,$desc" >> "$OUTPUT"
done

# --- 3. Extract UDP Traffic (Raw IPs) ---
echo "Extracting UDP background traffic..."
# Filter out port 53 to ignore standard DNS queries cluttering the list
tshark -r "$PCAP" -Y "ip.src == $AGENT_IP and udp and udp.dstport != 53" -T fields \
-e ip.dst -e udp.dstport \
| sort -u | grep . | while read -r ip port; do
    
    desc="UDP Traffic"
    target="$ip"

    if [[ "$port" == "123" ]]; then
        desc="NTP Time Sync"
    elif [[ "$port" == "514" ]]; then
        desc="Syslog Logging"
    elif [[ "$port" == "137" ]]; then
        desc="NetBIOS Name Service"
    fi
    
    echo "$target,$port,UDP,$desc" >> "$OUTPUT"
done

# --- 4. Final Polish & Deduplication ---
echo "Cleaning up output..."
temp_file=$(mktemp)
header=$(head -n 1 "$OUTPUT")
# Sort the file starting from line 2 to deduplicate any overlaps perfectly
tail -n +2 "$OUTPUT" | sort -u > "$temp_file"
echo "$header" > "$OUTPUT"
cat "$temp_file" >> "$OUTPUT"
rm "$temp_file"

echo "------------------------------------------------"
echo "Success! Final requirements saved to: $OUTPUT"
echo ""
echo "Preview of $OUTPUT:"
column -s, -t < "$OUTPUT" | head -n 15
```

Run the script. The resulting `firewall_requirements.csv` will contain a deduplicated list of domains, IPs, and ports formatted for a network security request.
