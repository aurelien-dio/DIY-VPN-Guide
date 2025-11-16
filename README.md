# DIY VPN Guide

## Create your own VPN for pennies and pay only when you use it

This guide presents a step-by-step procedure to create your own personal VPN server. It was born from a real-world need: watch geo-blocked content, such as live football matches on TF1 (a French TV channel) or match highlights on YouTube from abroad without an expensive monthly subscription.

With this method, the total cost to stream an entire two-hour football match is around **4 to 5 cents**, a tiny fraction of a monthly VPN subscription.

Beyond this specific scenario, a personal VPN is a powerful tool for many other situations:

*   **Securing your connection:** Encrypt all your traffic when using untrusted networks, like public Wi-Fi in an airport, hotel, or café, protecting your data from interception.
*   **Accessing sensitive services:** Connect to your online bank or administrative sites that may block or flag connections from foreign IP addresses for security reasons.
*   **Getting a "clean" IP address:** Unlike commercial VPNs whose shared IPs are often blacklisted, your personal server gives you a dedicated IP, allowing you to avoid constant CAPTCHA verifications and access services that block known VPNs.

By using a cloud provider with hourly billing, you'll build a solution that addresses all these needs for only a few cents per hour of use, giving you total control over your connection.

The initial setup takes **approximately 10-15 minutes** to complete, after which you will have a reusable solution that can be activated **in seconds**.

## Table of contents

## Table of Contents

1.  [**Prerequisites**](#1-prerequisites)
    *   List of required tools and accounts before starting the procedure.

2.  [**Step 1: Choosing Your Cloud Provider & Server**](#2-step-1-choosing-your-cloud-provider--server)
    *   Why hourly billing is the key to an ultra-economical solution.
    *   Selecting the right server location based on your needs.
    *   Choosing the most cost-effective instance for a VPN service.

3.  [**Step 2: Creating and Accessing Your Server**](#3-step-2-creating-and-accessing-your-server)
    *   The instance creation process.
    *   **Crucial:** Generating and adding an SSH Key to guarantee secure access.
    *   First connection to the server via the SSH protocol.

4.  [**Step 3: Server-Side VPN Configuration (WireGuard)**](#4-step-3-server-side-vpn-configuration-wireguard)
    *   System update and `WireGuard` installation.
    *   Generation of server and client cryptographic keys.
    *   Creation of the `wg0.conf` configuration file.
    *   Enabling kernel IP forwarding.
    *   Starting and enabling the WireGuard service.

5.  [**Step 4: Client-Side Configuration**](#5-step-4-client-side-configuration)
    *   Installation of the WireGuard application on your local machine or device.
    *   Creation of the client configuration file.
    *   Importing the configuration and activating the VPN tunnel.

6.  [**Step 5: Verification**](#6-step-5-verification)
    *   Procedure to confirm that your IP address is correctly masked and your traffic is routed through the server.

7.  [**Step 6: Managing Your Instance & Costs**](#7-step-6-managing-your-instance--costs)
    *   **The "Power Off" Method:** Suspending the instance to reduce costs.
    *   **The "Delete" Method:** The ultimate cost-saving strategy for sporadic use.
    *   The correct procedure for shutting down and restarting the VPN service.
  
## 1. Prerequisites

Before starting, ensure you have the following elements. This guide assumes a basic familiarity with the command line interface.

*   **An account with a cloud provider offering hourly billing.**
    This guide will use the French cloud provider **Scaleway** as the primary example due to its location in France and its highly competitive pricing. The key is to choose a provider with a datacenter in the geographical region you wish to connect *from*.

*   **A valid payment method.**
    You will need a Credit Card or another valid payment option to create and activate your cloud provider account.

*   **An SSH client.**
    This tool is required to connect securely to your remote server.
    *   **macOS and Linux:** The `Terminal` application is built-in and ready to use.
    *   **Windows:** Modern versions include OpenSSH in `PowerShell` or the `Command Prompt`.

*   **An SSH key pair.**
    This is the modern and secure method for authenticating to your server, replacing traditional passwords. We will generate one during the setup process.

*   **The WireGuard client application.**
    This free software must be installed on the device(s) you will use to connect to your VPN (your computer, smartphone, etc.). It can be downloaded from the official WireGuard website or app stores.

## 2. Step 1: choosing your cloud provider & server

The choice of provider and server is the foundation of this project. It determines both the cost and the effectiveness of your VPN.

### Why hourly billing is key

Traditional VPN services charge a flat monthly or yearly fee, regardless of your usage. For occasional needs, this is inefficient.

The strategy here is to use a cloud provider that bills **by the hour** (or even by the minute). This "pay-as-you-go" model means:
*   You launch the server only when you need the VPN.
*   You shut down the server immediately after use.
*   You only pay for the exact runtime, which can amount to just a few cents per session.

This guide uses **Scaleway**, a French provider, which is ideal for accessing French content. Other excellent alternatives with hourly billing include Vultr, DigitalOcean, or Hetzner (be mindful of their datacenter locations).

### Selecting the server location

The physical location of the server's datacenter is critical. When you connect to your VPN, your internet traffic will exit from this location.

*   **Rule:** To access content restricted to a specific country, you **must** choose a server located in that country.
*   **Example:** To watch TF1 (a French TV channel), you must create a server in a datacenter located in France (e.g. Paris). A server in Germany or the USA will not work for this purpose.

### Choosing the most cost-effective instance

A VPN server is not resource-intensive. It does not require a powerful CPU or a large amount of RAM to handle the traffic of a single user.

*   **Recommendation:** Always choose the smallest and cheapest instance available.
*   **Scaleway example:** The `DEV1-S` instance (2 vCPUs, 2 GB RAM and 200 Mbps) is already more than sufficient and costs less than one cent per hour (€0.0088/hr). Even smaller "Stardust" instances, if available, are perfectly adequate.

This minimal configuration is capable of handling high-speed video streaming without any issue, while keeping costs ridiculously low.

## 3. Step 2: creating and accessing your server

This section covers the creation of the virtual server and the essential steps to establish a secure connection to it. Follow these instructions carefully to avoid being locked out.

### Instance creation process

In your cloud provider's dashboard (e.g. Scaleway), begin the process of creating a new instance with the following parameters:

*   **Instance type:** Select the most economical option, such as `DEV1-S`.
*   **Region/Zone:** Choose the geographical location you need. For our example, this would be `Paris (PAR1)`.
*   **Image (operating system):** Select a stable, long-term support version. **Ubuntu 24.04 LTS** or **22.04 LTS** is highly recommended due to its extensive documentation.
*   **Network:** Ensure that an **IPv4** address is assigned to your instance. This is critical for compatibility.

### Crucial: generating and adding an SSH key

During the creation process, you will be prompted to add an SSH key. **This is not an optional step.** While some providers offer a "Create anyway" option, this often results in a server that only accepts SSH key authentication, effectively locking you out if you haven't configured one.

**1. Generate an SSH key pair:**
If you don't already have one, open a terminal on your local machine (not the server) and run the following command. This creates a secure and modern `ed25519` key.

```bash
ssh-keygen -t ed25519 -C "a_comment_for_your_key"
```
*Example:*
```bash
ssh-keygen -t ed25519 -C "scaleway_vps_key"
```

Press `Enter` three times to accept the default file location and to create a key without a passphrase for simplicity.

**2. Display your public key:**
Your public key is the part you will give to Scaleway. Use this command to display it in your terminal:

```bash
cat ~/.ssh/id_ed25519.pub
```
*Example output:*
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBv7fR8j4e2L/iB... scaleway_vps_key
```
Copy this entire line of text.

**3. Add the public key to your provider's dashboard:**
In the Scaleway interface, navigate to the "Credentials" or "SSH Keys" section. Add a new key, give it a memorable name (e.g. "MyMacBook"), and paste the public key you just copied.

**4. Finalize instance creation:**
Return to the instance creation screen and select the SSH key you just added. Now you can safely create the server.

### First connection to the server

Once your instance is running, find its public IPv4 address in the dashboard.

**1. Connect via SSH:**
In your terminal, use the following command, replacing `YOUR_SERVER_IP` with the actual IP address:

```bash
ssh root@YOUR_SERVER_IP
```
*Example:*
```bash
ssh root@51.158.110.106
```

**2. Handle host authenticity warning:**
On the very first connection, you will see a message like: `The authenticity of host '...' can't be established.` This is normal. Type `yes` and press `Enter` to continue.

> **Troubleshooting:** If you delete and recreate an instance in secondes or minutes, you might get a `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` error. This is a security measure because the same IP now belongs to a new server. To fix this, run the following command on your local machine to remove the old key fingerprint, then try connecting again:
> ```bash
> ssh-keygen -R YOUR_SERVER_IP
> ```
> *Example:*
> ```bash
> ssh-keygen -R 51.158.110.106
> ```

If the connection is successful, your terminal prompt will change to `root@your-server-name:~#`. You are now logged into your remote server and ready for the next step.

## 4. Step 3: server-side VPN configuration (WireGuard)

Now that you are connected to your server, you will install and configure the WireGuard software. These commands are to be executed on the remote server's terminal.

### System update and wireGuard installation

First, ensure your server's software packages are up to date, then install WireGuard.

```bash
apt update && apt upgrade -y
apt install wireguard -y
```

### Enabling kernel IP forwarding

This critical step allows your server to act as a router, forwarding traffic from your VPN client (your computer) to the internet.

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

### Generation of cryptographic keys

WireGuard uses a pair of cryptographic keys for the server and for each client to establish a secure connection.

1.  **Navigate to the WireGuard directory and set secure permissions:**

    ```bash
    cd /etc/wireguard
    umask 077
    ```

2.  **Generate the server and client key pairs:**
    This command will create four files: `server_private.key`, `server_public.key`, `client_private.key`, and `client_public.key`.

    ```bash
    wg genkey | tee server_private.key | wg pubkey > server_public.key
    wg genkey | tee client_private.key | wg pubkey > client_public.key
    ```

### Creation of the `wg0.conf` configuration file

This file contains all the rules for your VPN server.

1.  **Find your server's primary network interface name:**
    This is needed for the firewall rules.

    ```bash
    ip route | grep default
    ```
    *Example output:*
    ```
    default via 62.210.0.1 dev ens2 proto dhcp src 51.158.110.106 metric 100
    ```
    In this example, the interface name is `ens2`. Note yours down.

2.  **Get the necessary keys:**
    You will need the **server's private key** and the **client's public key**. Display them now to copy them.

    ```bash
    cat server_private.key
    cat client_public.key
    ```
    *Example outputs:*
    ```
    # server_private.key
    s1rv3r_pr1v4t3_k3y_3x4mpl3_fGgHhJjKkLlMmNnOoPpQqRrSsTtUuVv=
    # client_public.key
    cl13nt_publ1c_k3y_3x4mpl3_fGgHhJjKkLlMmNnOoPpQqRrSsTtUuVv=
    ```
    Copy them into an empty text file apart from your terminal.

3.  **Create and edit the configuration file:**
    Return in your terminal to open the file `wg0.conf` using the `nano` text editor.

    ```bash
    nano /etc/wireguard/wg0.conf
    ```

4.  **Paste the following configuration:**
    Replace `[YOUR_SERVER_PRIVATE_KEY]`, `[YOUR_CLIENT_PUBLIC_KEY]`, and `[YOUR_NETWORK_INTERFACE]` with your actual values.

    ```ini
    [Interface]
    PrivateKey = [YOUR_SERVER_PRIVATE_KEY]
    Address = 10.0.0.1/24
    ListenPort = 51820
    PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o [YOUR_NETWORK_INTERFACE] -j MASQUERADE
    PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o [YOUR_NETWORK_INTERFACE] -j MASQUERADE

    [Peer]
    PublicKey = [YOUR_CLIENT_PUBLIC_KEY]
    AllowedIPs = 10.0.0.2/32
    ```
    *Example of a completed file:*
    ```ini
    [Interface]
    PrivateKey = s1rv3r_pr1v4t3_k3y_3x4mpl3_fGgHhJjKkLlMmNnOoPpQqRrSsTtUuVv=
    Address = 10.0.0.1/24
    ListenPort = 51820
    PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens2 -j MASQUERADE
    PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens2 -j MASQUERADE

    [Peer]
    PublicKey = cl13nt_publ1c_k3y_3x4mpl3_fGgHhJjKkLlMmNnOoPpQqRrSsTtUuVv=
    AllowedIPs = 10.0.0.2/32
    ```

5.  **Save and exit `nano`:**
    Press `Ctrl + O`, then `Enter` to save the file. Press `Ctrl + X` to exit.

### Starting and enabling the WireGuard service

Finally, start the VPN service and configure it to launch automatically every time the server boots.

```bash
systemctl enable wg-quick@wg0
systemctl start wg-quick@wg0
```

To verify that the service is running correctly, use the `wg show` command.

```bash
wg show
```
*Example of a successful output:*
```
interface: wg0
  public key: srv_pub_key_...
  private key: (hidden)
  listening port: 51820

peer: cl13nt_publ1c_k3y_3x4mpl3_...
  allowed ips: 10.0.0.2/32
```
If you see a similar output, your server is ready.

## 5. Step 4: client-side configuration

The server is now waiting for a connection. The final step is to configure your local device to establish the secure VPN tunnel.

### Installation of the WireGuard application

If you haven't already, install the official WireGuard client on the device you want to connect from.

*   **Download:** Go to the <a href="https://www.wireguard.com/install/" target="_blank" rel="noopener noreferrer">Wireguard official website</a> and choose the version for your operating system (Windows, macOS, Linux, Android, iOS).

### Creating the client configuration file

This involves creating a single text file with a `.conf` extension on your local machine.

**1. Gather required information from the server:**
You need three pieces of information. Two are stored on your server, and one is your server's IP. Connect to your server via SSH and use the following commands to retrieve the keys:

*   **Get the client's private key:**
    ```bash
    cat /etc/wireguard/client_private.key
    ```
    *Example output:*
    ```
    cl13nt_pr1v4t3_k3y_3x4mpl3_fGgHhJjKkLlMmNnOoPpQqRrSsTtUuVv=
    ```

*   **Get the server's public key:**
    ```bash
    cat /etc/wireguard/server_public.key
    ```
    *Example output:*
    ```
    s1rv3r_publ1c_k3y_3x4mpl3_fGgHhJjKkLlMmNnOoPpQqRrSsTtUuVv=
    ```

*   **Note your server's public IP address:**
    *Example:* `51.158.110.106`

**2. Create the `.conf` file on your local machine:**
On your computer (not the server), create a new text file. Name it something descriptive, like `vpn-scaleway.conf`.

**3. Paste and edit the following template:**
Open the file and paste this template. Replace the bracketed placeholders with the information you just gathered.

```ini
[Interface]
PrivateKey = [YOUR_CLIENT_PRIVATE_KEY]
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = [YOUR_SERVER_PUBLIC_KEY]
Endpoint = [YOUR_SERVER_IP]:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

*   `AllowedIPs = 0.0.0.0/0` is a crucial setting that tells your computer to send **all** its internet traffic through the VPN tunnel.

*Example of a completed file:*
```ini
[Interface]
PrivateKey = cl13nt_pr1v4t3_k3y_3x4mpl3_fGgHhJjKkLlMmNnOoPpQqRrSsTtUuVv=
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = s1rv3r_publ1c_k3y_3x4mpl3_fGgHhJjKkLlMmNnOoPpQqRrSsTtUuVv=
Endpoint = 51.158.110.106:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

### Importing and activating the VPN tunnel

1.  Open the WireGuard application on your computer or device.
2.  Click on **"Import tunnel(s) from file"** and select the `vpn-scaleway.conf` file you just created.
3.  A new tunnel will appear in the application. Click the **"Activate"** button.

If the status shows as "Active", your connection is established. All your internet traffic is now being securely routed through your server.

## 6. Step 5: verification

Once the WireGuard client shows an "Active" status, you need to confirm that your internet traffic is correctly and securely routed through your new server.

### Test 1: check your public IP address

This is the most straightforward test. Your public IP address, as seen by the rest of the internet, should now be that of your server.

1.  **Ensure your VPN is active.**
2.  Open a web browser and visit a site that displays your IP address, such as:
    *   `https://ifconfig.me`
    *   `https://ip.me`

The IP address displayed on the page **must** match your server's public IPv4 address.

*Example:*
If your server's IP is `51.158.110.106`, the website should display exactly `51.158.110.106`. If it still shows your home IP address, the VPN is not routing your traffic correctly.

### Test 2: check your geolocation

This test confirms that websites perceive your location as being that of the server.

1.  **With the VPN active, visit a geolocation website:**
    *   `https://www.iplocation.net`

2.  **Check the results:**
    The website should report the country, city, and ISP (Internet Service Provider) of your server.

*Example:*
For a Scaleway server located in Paris, the results should look like this:
*   **Country:** France 🇫🇷
*   **City:** Paris
*   **ISP / Provider:** Scaleway

### Test 3: the ultimate test - accessing geo-blocked content

This is the final validation. Try to access the service that was previously blocked.

1.  **Navigate to the target website.**
    *Example:* Go to the official TF1 YouTube channel (`https://www.youtube.com/@tf1`).

2.  **Attempt to view the content.**
    Click on a video (like a football match summary) that you know was blocked before.

If the video plays without any error message, your VPN is a complete success.

> **Pro Tip:** To be absolutely certain, you can deactivate your VPN in the WireGuard client and refresh the page. The geo-restriction error should reappear. This confirms that your DIY VPN is the component making the difference.

## 7. Step 6: managing your instance & costs

The primary advantage of this DIY solution is cost control. Unlike a fixed subscription, you only pay for what you use. Understanding how to manage your instance is key to maximizing savings.

There are two main strategies, depending on your frequency of use.

### The "power off" method (for regular use)

This method is like putting your server to sleep. The configuration and IP address are preserved, allowing for a quick restart.

*   **When to use it:** If you plan to use your VPN regularly (e.g. several times a week or month).
*   **Pros:** Very fast to restart (usually under a minute), keeps the same IP and configuration.
*   **Cons / ⚠️ Important Cost Warning:** Powering off the instance **does not stop all billing**. You will still be charged for reserved resources, primarily the **IPv4 address** and the **disk storage**. This can result in a fixed cost of approximately €1.50 per month, even if the server is off.

**Procedure to power off:**

1.  **Gracefully shut down the operating system from SSH.** This prevents data corruption.
    ```bash
    halt
    ```
    Your SSH connection will terminate.

2.  In your cloud provider's dashboard, navigate to your instance and click the **"Power Off"** button.

### The "delete" method (for sporadic use)

This is the ultimate cost-saving strategy. You completely destroy the instance, and all billing stops immediately.

*   **When to use it:** If you only need the VPN very occasionally (e.g. once a month or less).
*   **Pros:** The cost drops to **€0.00**. You pay absolutely nothing when you are not using it.
*   **Cons:** You must re-run the server-side configuration (Step 3) every time you create a new instance. This takes about 10-15 minutes.

**Procedure to delete:**

1.  In your cloud provider's dashboard, find your instance and click the **"Delete"** button. Confirm the action.

> **Pro Tip:** To speed up future setups, save your client configuration file (`vpn-scaleway.conf`). When you recreate the server, you will only need to generate new keys and update the `Endpoint` IP and `PublicKey` in your existing file.

### The correct procedure for shutting down and restarting

Always follow this order to avoid connection issues.

**To STOP the VPN:**

1.  **First:** Deactivate the connection in your WireGuard client application on your computer/phone.
2.  **Then:** Power off or delete the instance in your cloud provider's dashboard.
    *(If you do it in the wrong order, your device will try to send all its internet traffic to a server that is turned off, resulting in a loss of connectivity until you deactivate the client.)*

**To RESTART the VPN:**

1.  **First:** Power on (or recreate) the instance in your cloud provider's dashboard. Wait 1-2 minutes for it to boot up completely.
2.  **Then:** Activate the connection in your WireGuard client.
