---
layout: post
title: Setting Up a WireGuard VPN Client on Linux
featured: false
giscus_comments: true
authors:
  - name: Joshua Rothe
    url: "https://portfolio.rothellc.com"
excerpt: "Setting up a Wireguard VPN for privacy and security involves setting up both server and client side systems. This guide explains how to set up a client side Linux system - with or without [Pi-hole DNS filtering](https://pi-hole.net/) on the home network - and then configure the system so that Wireguard settings will switch depending on if the client system is on the home network or not. This is necessary because the Wireguard client will break your network connection if you are on your home network, and there is no need to manually switch your VPN on and off when automation exists."
date: 2025-09-27
description: Guide for configuring a client-side Linux (Debian) system for WireGuard VPN, automating network settings on both home and away networks.
tags: [linux, vpn, wireguard, pihole, devops]
categories: [linux, devops]
toc:
  sidebar: left
citation: true
---

![WireGuard Network Diagram](../../../assets/img/diagrams/2025-09-27-setting-up-a-wireguard-vpn-client-on-linux/wireguard-vpn-client-diagram.svg){: .img-fluid}

---

Setting up a WireGuard VPN for privacy and security involves setting up both server and client side systems. This guide explains how to set up a client side Linux system - with or without [Pi-hole DNS filtering](https://pi-hole.net/) on the home network - and then configure the system so that WireGuard settings will switch depending on if the client system is on the home network or not. This is necessary because the WireGuard client will break your network connection if you are on your home network, and there is no need to manually switch your VPN client on and off when automation exists.

This guide assumes a [WireGuard VPN server](https://www.wireguard.com/quickstart/) is set up, and port forwarding is configured on the home router (UDP, port 51820 should be forwarding to your WireGuard server). This guide is also written for Linux Mint - while this should also work for most Debian systems, you may need to modify some file paths depending on your distro.

Note: This post was updated Oct 18 2025 that fixes critical issues with the original approach - it would break upon switching network types while the client was shut down, and did not check if the WireGuard server itself was reachable when on an external network. This improved method fixes these issues and keeps the WireGuard client disabled until the following is verified:

1. The client is not on the home network.
2. The WireGuard server is reachable.

The old method is kept in the Appendix for historical purposes, but is depreciated.

---

## Background

VPNs are a great tool for security and privacy, with key benefits being: 

**Privacy and Security:**
- **Location Privacy**: All traffic appears to be from your home IP address. Your actual location is obscured to anyone tracking your browsing activity.
- **Data Protection**: On public wireless networks, your VPN tunnel ensures packet sniffers and other malicious actors can't see your activity.
- **Audited:** WireGuard is open source and is [audited regularly](https://courses.csail.mit.edu/6.857/2018/project/He-Xu-Xu-WireGuard.pdf) by both organizations and individuals. All of its code is viewable by any developer curious enough to know how it functions.

**Functionality:**
- **Ad Blocking:** If you run a [Pi-Hole](https://pi-hole.net/) system at home for ad blocking, you can use that same tool while off your home network.
- **Home Network Access:** This secure tunnel to your home network means you can access devices on your home network such as cameras, NAS systems, and IoT devices without needing to expose them to the internet.
- **Speed:** [WireGuard is fast](https://www.wireguard.com/performance/), faster than most other comparable VPN services.

**Practicality:**
- **Low Overhead:** Works on virtually all modern hardware.
- **Cryptographically Secure:** Uses proven, modern cryptographic primitives detailed on the [WireGuard homepage](https://www.wireguard.com/).
- **Free:** Self explanatory. No risk of subscription fees in the future, either. Open source comes with benefits!

If you do not have a WireGuard VPN server set up and find this interesting, I've worked through the [official WireGuard documentation](https://www.wireguard.com/quickstart/) and found it more than sufficient. Make sure this is complete before setting up clients! Windows and MacOS have [dedicated programs](https://www.wireguard.com/install/) for clients, but Linux is a bit more complicated; hence this guide.

---

## Prerequisites

- WireGuard server, configured with port forwarding on your home router.
- Linux client with `sudo` access (tested here on Mint/Debian).
- Basic command line familiarity.

---

## Client Setup

### SSH Key Generation

This process should be familiar (for general server access, not WireGuard-specific handshakes), but direction is provided below just in case. On the client, run:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

Hit enter twice - use default location and no passkey (unless you really want one).

There are two ways to copy your key to the server, the **Easy Way** and the **More Likely Way**.

---

{::options parse_block_html="true" /}

<div class="row">
<div class="col-md-6">

#### The Easy Way:

The easy way only works if your client can access the server. Since we are generating keys, you should probably have password authentication enabled for SSH. In case you forgot, you modify these lines in `/etc/ssh/sshd_config`:
```sh
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

Then on the client side you can simply run:
```bash
ssh-copy-id username@server-ip
```
This copies SSH key settings to the host, if security settings allow it.

</div>
<div class="col-md-6">

#### The More Likely Way:

Run this on the client:
```bash
cat ~/.ssh/id_ed25519.pub
```

Get this clipboard item to the server however you like, probably using another system that does have SSH access, and append it to `~/.ssh/authorized_keys`. No need to disable the server's security requirements this way.

</div>
</div>

---

### Installation

Run on the client to install necessary dependencies:
```bash
sudo apt update
sudo apt install wireguard resolvconf netcat-openbsd
```

Once that is done, generate WireGuard keys (these are different than SSH keys - the former are needed to access the WireGuard server for configuration):

```bash
cd /etc/wireguard
sudo umask 077
sudo wg genkey | sudo tee privatekey | sudo wg pubkey | sudo tee publickey
```

Next, create the WireGuard config file on your client using:
```bash
sudo vim /etc/wireguard/wg0.conf
```

And insert the following text to create your client's WireGuard network configuration file. This facilitates the key handshake your client system will do with the server:
```sh
[Interface]
PrivateKey = # Paste the contents of /etc/wireguard/privatekey here
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = # Your server's public key goes here.
Endpoint = your-server-ip:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```
Note that you have three items to fill in above. You will need the PublicKey from the server, not from what you have here on the client. So hold off on that for now.

The private key on the client can be acquired using:
```bash
sudo cat /etc/wireguard/privatekey
```

The public key on the client (which will be needed on the server) can be acquired using:
```bash
sudo cat /etc/wireguard/publickey
```

Now SSH into the WireGuard server, and edit the WireGuard config using:
```bash
sudo vim /etc/wireguard/wg0.conf
```

Append the following to the file:
```sh
[Peer]
PublicKey = # Paste your client's public key here.
AllowedIPs = 10.0.0.x/32
```
For `AllowedIPs`, you will need to replace the `x`. For my setup, the server was .1, and I had two previous clients taking up .2 and .3, so I used .4. These are the IPs the WireGuard server uses to identify the peers on its network.

Back on the client, edit the client's WireGuard config using:
```bash
sudo vim /etc/wireguard/wg0.conf
```

Remember how we didn't have the server's public key last time? Fix that now:
```sh
[Interface]
PrivateKey = # Paste(d) the contents of /etc/wireguard/privatekey here
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = # Your server's public key goes here.
Endpoint = your-server-ip:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

Three items to check, above. Make sure the `Address` on your client side matches `AllowedIps` on the server side.

---

## Automated VPN Management

Run this first to set the WireGuard client as disabled by default:
```bash
sudo systemctl disable wg-quick@wg0
sudo systemctl stop wg-quick@wg0
```

Run:
```bash
sudo vim /etc/NetworkManager/dispatcher.d/99-wireguard-auto
```

Replace the script text with the following, adjusting the `HOME_*` values as needed:
```sh
#!/bin/bash

INTERFACE=$1
ACTION=$2

# Configuration. Change these as needed.
HOME_NETWORK_GATEWAY="192.168.1.1" # Router's IP.
HOME_WIREGUARD_SERVER="192.168.1.114" # WireGuard server IP.
WIREGUARD_PORT="51820"
HOME_EXTERNAL_IP="YOUR.EXTERNAL.IP" # Your external home network IP.
LOG_FILE="/var/log/wireguard-auto.log"

PING_TIMEOUT=5
NC_TIMEOUT=10

log_message() {
    echo "$(date): [$INTERFACE/$ACTION] $1" | tee -a "$LOG_FILE"
}

is_wireguard_server_reachable() {
    local server_ip="$1"
    local port="$2"
    
    if command -v nc >/dev/null 2>&1; then
        if nc -u -z -w 5 "$server_ip" "$port" >/dev/null 2>&1; then
            log_message "Server $server_ip is reachable and responding"
            return 0
        else
            log_message "WireGuard port $port on $server_ip is not responding"
            return 1
        fi
    else
        # Fallback if nc is not available.
        log_message "Error: nc not available"
        return 1
    fi
}

is_home_network() {
    if ping -c 3 -W "$PING_TIMEOUT" "$HOME_NETWORK_GATEWAY" >/dev/null 2>&1; then
        if ping -c 3 -W "$PING_TIMEOUT" "$HOME_WIREGUARD_SERVER" >/dev/null 2>&1; then
            local_ip=$(ip route get 8.8.8.8 2>/dev/null | awk '{print $7; exit}')
            if [[ "$local_ip" =~ ^192\.168\.1\. ]]; then
                return 0
            fi
        fi
    fi
    return 1
}

enable_wireguard() {
    if ! systemctl is-active --quiet wg-quick@wg0; then
        log_message "Enabling WireGuard"
        systemctl enable wg-quick@wg0
        systemctl start wg-quick@wg0
    fi
}

disable_wireguard() {
    if systemctl is-active --quiet wg-quick@wg0; then
        log_message "Disabling WireGuard"
        systemctl disable wg-quick@wg0
        systemctl stop wg-quick@wg0
    fi
}

# Only act on interface up events.
if [ "$2" != "up" ]; then
    exit 0
fi

log_message "Network change detected on $1"

sleep 10

if is_home_network; then
    log_message "Home network confirmed - ensuring WireGuard is disabled"
    disable_wireguard
else
    log_message "External network confirmed - testing VPN server"
    if is_wireguard_server_reachable "$HOME_EXTERNAL_IP" "$WIREGUARD_PORT"; then
        log_message "VPN server reachable - enabling WireGuard"
        enable_wireguard
    else
        log_message "VPN server unreachable - WireGuard remains disabled"
        disable_wireguard
    fi
fi
```

Next, run `sudo vim /etc/systemd/system/wireguard-boot-check.service` and add:
```sh
[Unit]
Description=WireGuard Boot Config
After=network-online.target
Wants=network-online.target
Before=wg-quick@wg0.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/wireguard-boot-check.sh
RemainAfterExit=yes
TimeoutStartSec=150

[Install]
WantedBy=multi-user.target
```

Then run `sudo vim /usr/local/bin/wireguard-boot-check.sh` and add the following adjusting variables as needed:
```sh
#!/bin/bash

# Configuration - adjust these to match your setup.
HOME_NETWORK_GATEWAY="192.168.1.1"
HOME_WIREGUARD_SERVER="192.168.1.114"
HOME_EXTERNAL_IP="YOUR.EXTERNAL.IP.HERE"
WIREGUARD_PORT="51820"
LOG_FILE="/var/log/wireguard-boot-check.log"

# Timeout after 2 min.
NETWORK_WAIT_TIMEOUT=120
PING_TIMEOUT=5
NC_TIMEOUT=10

log_message() {
    echo "$(date): $1" | tee -a "$LOG_FILE"
}

is_wireguard_server_reachable() {
    local server_ip="$1"
    local port="$2"

    # Give network time to settle on boot.
    sleep 10
    check_timeout
    
    if command -v nc >/dev/null 2>&1; then
        if nc -u -z -w 5 "$server_ip" "$port" >/dev/null 2>&1; then
            log_message "Server $server_ip is reachable and responding"
            return 0
        else
            log_message "WireGuard port $port on $server_ip is not responding"
            return 1
        fi
    else
        # Fallback if nc is not available.
        log_message "Error: nc not available"
        return 1
    fi
}

is_home_network() {
    # Check if home gateway is reachable.
    if ping -c 3 -W "$PING_TIMEOUT" "$HOME_NETWORK_GATEWAY" >/dev/null 2>&1; then
        # Second check by pinging WireGuard server directly.
        if ping -c 3 -W "$PING_TIMEOUT" "$HOME_WIREGUARD_SERVER" >/dev/null 2>&1; then
            # Third check by seeing if client IP is in the home range.
            local_ip=$(ip route get 8.8.8.8 2>/dev/null | awk '{print $7; exit}')
            if [[ "$local_ip" =~ ^192\.168\.1\. ]]; then
                log_message "Confirmed home network (gateway: $HOME_NETWORK_GATEWAY, server: $HOME_WIREGUARD_SERVER, local IP: $local_ip)"
                return 0
            fi
        fi
    fi
    log_message "Not on home network"
    return 1
}

enable_wireguard() {
    log_message "Enabling WireGuard"
    systemctl enable wg-quick@wg0
    systemctl start wg-quick@wg0
}

log_message "Boot check: Starting (WireGuard disabled by default)"

# Wait for stable network connectivity.
elapsed=0
while [ $elapsed -lt $NETWORK_WAIT_TIMEOUT ]; do
    if ping -c 2 -W 3 8.8.8.8 >/dev/null 2>&1; then
        log_message "Network connectivity established after ${elapsed}s"
        break
    fi
    log_message "Waiting for network connectivity... (${elapsed}s)"
    sleep 10
    elapsed=$((elapsed + 10))
done

# If still no connectivity, exit.
if ! ping -c 2 -W 3 8.8.8.8 >/dev/null 2>&1; then
    log_message "No network connectivity after ${elapsed}s - WireGuard remains disabled"
    exit 0
fi

log_message "Allowing network to settle..."
sleep 15

# Network location detection.
if is_home_network; then
    log_message "Home network confirmed - WireGuard remains disabled"
else
    log_message "External network confirmed - testing VPN server accessibility"
    if is_wireguard_server_reachable "$HOME_EXTERNAL_IP" "$WIREGUARD_PORT"; then
        log_message "VPN server confirmed reachable - enabling WireGuard"
        enable_wireguard
    else
        log_message "VPN server not accessible - WireGuard remains disabled"
    fi
fi

log_message "Boot check: WireGuard configuration complete"
```

Make the script executable and enable it.
```bash
sudo chmod +x /usr/local/bin/wireguard-boot-check.sh
sudo systemctl enable wireguard-boot-check.service
```

Lastly, run:

```bash
sudo vim /etc/wireguard/wg0.conf
```

And add this to the file:
```sh
[Interface]
PrivateKey = # Your laptop's private key.
Address = 10.0.0.2/24
DNS = 1.1.1.1 # Or 192.168.1.xxx for Pi-Hole.

[Peer]
PublicKey = # Your server's public key.
Endpoint = your-server-external-ip:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```
Note for Endpoint above, you need your server's external IP. This will be your home router's IP, port forwarded to your VPN server.

---

### Verification

After setup is complete, verify your VPN is working correctly.

On an 'away' network, run:
```bash
sudo wg show
sudo systemctl status wg-quick@wg0
ip addr show wg0
```

You should see your WireGuard client configuration in the output.

You should also see your home external IP displayed when you try to view your client's public IP. Verify this using:
```bash
curl -s ifconfig.me
```

Verify DNS resolution using:
```bash
nslookup google.com
```

The logs should also populate as your client switches networks or reboots. This can be viewed using:
```bash
sudo tail -f /var/log/wireguard-auto.log
sudo tail -f /var/log/wireguard-boot-check.log
```

---

## Conclusion

That should get you a working VPN client! If you notice any issues with this guide, please feel free to reach out or leave a comment.

---

## Appendix

### Dual WireGuard Configurations - Old Method

The following method is depreciated and is kept for historical purposes only.

On your client side, run:
```bash
sudo cp /etc/wireguard/wg0.conf /etc/wireguard/wg0-away.conf
sudo cp /etc/wireguard/wg0.conf /etc/wireguard/wg0-home.conf
```
Both of these files will become your home and away configurations, which will be copied into `wg0.conf` when your client detects a network change.

Edit the home configuration using:
```bash
sudo vim /etc/wireguard/wg0-home.conf
```

The only change you need to make below is to AllowedIPs: `your-local-ip` (e.g. 192.168.1.0/24) is critical here, as you want the initial `192.168.1.*` to match the home network IPs your router issues. Also, don't forget to adjust `Address` if needed.
```sh
[Interface]
PrivateKey = # Your laptop's private key.
Address = 10.0.0.2/24
DNS = 1.1.1.1 # Or 192.168.1.xxx for Pi-Hole.

[Peer]
PublicKey = # Your server's public key.
Endpoint = your-server-local-ip:51820
AllowedIPs = your-local-ip/24
PersistentKeepalive = 25
```

Next, the away config should be edited using:
```bash
sudo vim /etc/wireguard/wg0-away.conf
```

And should contain:
```sh
[Interface]
PrivateKey = # Your laptop's private key.
Address = 10.0.0.2/24
DNS = 1.1.1.1 # Or 192.168.1.xxx for Pi-Hole.

[Peer]
PublicKey = # Your server's public key.
Endpoint = your-server-external-ip:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```
Note for above, you need your server's external IP. This will be your home router's IP, port forwarded to your VPN server. This was likely set up when the WireGuard server was set up, and you only need a service like [this](https://whatismyip.com) to check your external IP.

As a reminder, these two `wg0-*` scripts will replace wg0.conf dynamically depending on what network the client is on. This will be triggered by the detection script, which we will now build.

---

#### Detection Script

Run:
```bash
sudo vim /etc/NetworkManager/dispatcher.d/99-wireguard-auto
```

Add the following, adjusting the `HOME_*` values as needed:
```sh
#!/bin/bash

INTERFACE=$1
ACTION=$2

# Filter out interfaces we don't care about (loopback, virtual bridges, docker).
case "$INTERFACE" in
    lo|virbr*|docker*|br-*)
        exit 0
        ;;
esac

LOG_FILE="/var/log/wireguard-auto.log"

log_message() {
    echo "$(date): [$INTERFACE/$ACTION] $1" | tee -a "$LOG_FILE"
}

# Only act on interface up/down events for primary connections.
if [ "$ACTION" != "up" ] && [ "$ACTION" != "down" ]; then
    exit 0
fi

# Skip if this is the WireGuard interface itself.
if [ "$INTERFACE" = "wg0" ]; then
    exit 0
fi

log_message "Network change detected on $INTERFACE"

# Let network settle.
sleep 2

# Configuration. Change these as needed.
HOME_NETWORK_GATEWAY="192.168.1.1" # Router's IP.
HOME_WIREGUARD_SERVER="192.168.1.114" # WireGuard server IP.
WG_INTERFACE="wg0"

is_home_network() {
    # Check if home gateway is reachable.
    if ping -c 1 -W 3 "$HOME_NETWORK_GATEWAY" >/dev/null 2>&1; then
        # Double-check by pinging WireGuard server directly.
        if ping -c 1 -W 3 "$HOME_WIREGUARD_SERVER" >/dev/null 2>&1; then
            # Triple-check by seeing if our IP is in the home range.
            local_ip=$(ip route get 8.8.8.8 2>/dev/null | awk '{print $7; exit}')
            if [[ "$local_ip" =~ ^192\.168\.1\. ]]; then
                return 0 # Home.
            fi
        fi
    fi
    return 1 # Away.
}

get_wg_status() {
    if systemctl is-active --quiet wg-quick@wg0; then
        echo "active"
    else
        echo "inactive"
    fi
}

copy_config() {
    local source="$1"
    if [ -f "$source" ]; then
        cp "$source" /etc/wireguard/wg0.conf
        log_message "Copied $source to wg0.conf"
        return 0
    else
        log_message "ERROR: $source not found!"
        return 1
    fi
}

# Only proceed if network connection is present.
if [ "$ACTION" = "up" ]; then
    if is_home_network; then
        log_message "Detected home network."
        current_status=$(get_wg_status)
        
        if [ "$current_status" = "active" ]; then
            # Check if using away config (away config has AllowedIPs = 0.0.0.0/0).
            if grep -q "AllowedIPs = 0.0.0.0/0" /etc/wireguard/wg0.conf 2>/dev/null; then
                log_message "Switching to home VPN config."
                systemctl stop wg-quick@wg0
                if copy_config "/etc/wireguard/wg0-home.conf"; then
                    systemctl start wg-quick@wg0
                fi
            else
                log_message "Already using home VPN config."
            fi
        else
            log_message "Starting home VPN config."
            if copy_config "/etc/wireguard/wg0-home.conf"; then
                systemctl start wg-quick@wg0
            fi
        fi
    else
        log_message "Detected external network."
        current_status=$(get_wg_status)
        
        if [ "$current_status" = "active" ]; then
            # Check if we're using the home config (home config has restricted AllowedIPs).
            if grep -q "AllowedIPs = 192.168.1.0/24" /etc/wireguard/wg0.conf 2>/dev/null; then
                log_message "Switching to away VPN config."
                systemctl stop wg-quick@wg0
                if copy_config "/etc/wireguard/wg0-away.conf"; then
                    systemctl start wg-quick@wg0
                fi
            else
                log_message "Already using away VPN config."
            fi
        else
            log_message "Starting away VPN config."
            if copy_config "/etc/wireguard/wg0-away.conf"; then
                systemctl start wg-quick@wg0
            fi
        fi
    fi
elif [ "$ACTION" = "down" ]; then
    # Uncomment the next line if wireguard should stop when network goes down.
    # systemctl stop wg-quick@wg0
    log_message "Network disconnected."
fi

exit 0
```

Once complete, make it executable:
```bash
sudo chmod +x /etc/NetworkManager/dispatcher.d/99-wireguard-auto
```

Then, add the following (adjust DNS IP as needed) using:
```bash
sudo vim /etc/NetworkManager/dispatcher.d/99-wireguard-dns.sh
```

Add:
```sh
#!/bin/bash

INTERFACE=$1
ACTION=$2

if [[ "$INTERFACE" == "wg0" && "$ACTION" == "up" ]]; then
    # Set Pi-hole as DNS server for WireGuard interface.
    resolvectl dns wg0 192.168.1.223 # Change! Pi-Hole DNS address or 1.1.1.1 1.0.0.1 (Cloudflare) or 9.9.9.9 149.112.112.112 (Quad9)
    resolvectl domain wg0 "~."
    
    # Remove default route from WiFi interface to prioritize VPN DNS.
    resolvectl default-route <replace-me> false
    
    # Ensure only VPN has default route for DNS.
    resolvectl default-route wg0 true
    
    # Flush DNS cache to apply changes.
    resolvectl flush-caches
    
    logger "WireGuard DNS configured: Pi-hole via VPN with WiFi DNS disabled."
fi
```
To know what to enter in `<replace-me>`, run `ip link show`. You need your actual WiFI interface name here.

Make it executable:
```bash
sudo chmod +x /etc/NetworkManager/dispatcher.d/99-wireguard-dns.sh
```

To finalize, restart WireGuard on both machines. For the server first, run:
```bash
sudo systemctl restart wg-quick@wg0
```

For the client, run:
```bash
sudo resolvectl default-route wlp4s0 false
sudo systemctl restart wg-quick@wg0
```
This restarts WireGuard and fixes the initial VPN prioritization issue. Some Linux queries might go through your ISP's DNS instead of the proper tunnel due to how systemd-resolve handles queries.