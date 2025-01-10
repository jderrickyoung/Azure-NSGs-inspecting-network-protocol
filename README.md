<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups. <br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04

<h2>High-Level Steps</h2>

- Step 1 - Observe ICMP Traffic
- Step 2 - Configure a Firewall
- Step 3 - Observe SSH Traffic
- Step 4 - Observe DNS Traffic
- Step 5 - Observe RDP Traffic

# Inspecting Network Protocols with Wireshark

## Create Virtual Machines and Download Wireshark

Before inspecting network protocols, set up two virtual machines (VMs) in Azure. One will be a Windows VM, and the other will be a Linux VM. Ensure both are in the same resource group and region. Apply the following settings:

### Windows VM
- **Image:** Windows 10 Pro 22H2
- **Size:** Standard with at least 2 vCPUs and 8 GiB memory
- **Authentication:** New username/password
- **Networking:** New Virtual Network

### Linux VM
- **Image:** Ubuntu Server 24.04
- **Size:** Standard with at least 2 vCPUs and 8 GiB memory
- **Authentication:** New username/password
- **Networking:** Connect to the same Virtual Network as the Windows VM

1. Launch both VMs and log in to your Windows VM using its IP address.

   ![Login](https://i.imgur.com/cK3bQba.png)
   ![Azure VMs](https://i.imgur.com/vpn580y.png)

2. Open a web browser on the Windows VM and go to [Wireshark Downloads](https://www.wireshark.org/download.html). Download the Windows x64 Installer.

   ![Wireshark Download](https://i.imgur.com/EvSVdJc.png)
   ![Installation](https://i.imgur.com/TUuNiBy.png)

3. Install Wireshark (no additional components are required).

   ![Wireshark Installed](https://i.imgur.com/i4FQrmz.png)

4. Open Wireshark and PowerShell. In Wireshark, start a packet capture to inspect network traffic. This will allow us to filter through all the network traffic and examine specific protocols in action.

   ![Wireshark Capture](https://i.imgur.com/kic5QxQ.png)

---

## Step 1: Observe ICMP Traffic

1. In Wireshark, filter for ICMP traffic by typing `ICMP` in the filter bar and restarting the capture. Filtering ICMP traffic helps focus on "ping" requests and replies, which are often used for network diagnostics.

   ![Filter ICMP](https://i.imgur.com/N3es0Uo.png)

2. Retrieve the private IP address of the Linux VM from Azure.

   ![Linux IP](https://i.imgur.com/M4aobuZ.png)

3. Ping the Linux VM from the Windows VM using PowerShell:

   ```powershell
   ping 10.0.0.5
   ```

   ![Ping Linux VM](https://i.imgur.com/82YLMTT.png)

4. Observe the ICMP traffic in Wireshark to verify the ping requests and replies. Filtering ICMP traffic allows you to see if "ping" requests (ICMP Echo Request/Reply) are reaching the destination and receiving responses, helping diagnose potential network issues or firewalls blocking traffic.

   ![ICMP Traffic](https://i.imgur.com/rEckb1j.png)

---

## Step 2: Configure a Firewall (Network Security Group)

1. Initiate a perpetual ping to the Linux VM:

   ```powershell
   ping 10.0.0.5 -t
   ```

   ![Perpetual Ping](https://i.imgur.com/N6lQ10k.png)

2. In Azure, open the **Network Security Group** (NSG) for the Linux VM:
   - Navigate to `Virtual Machines > linux-vm > Network settings > linux-vm-nsg`.

   ![NSG Settings](https://i.imgur.com/zGVE3ph.png)

3. Add an inbound security rule:
   - **Destination Port Ranges:** `*`
   - **Protocol:** `ICMPv4`
   - **Action:** `Deny`
   - **Priority:** Set below 300.

   ![NSG Rule](https://i.imgur.com/tDDzJ0r.png)

4. Once the rule is active, the ping requests will time out. This happens because the firewall now blocks the ICMP traffic. This demonstrates how firewalls can control specific types of network traffic.

   ![Ping Timeout](https://i.imgur.com/Dc1Q8mf.png)

5. Delete the rule when no longer needed, and stop the ping activity by pressing `Ctrl+C` in PowerShell.

---

## Step 3: Observe SSH Traffic

1. In Wireshark, start a new packet capture and filter for SSH traffic. Filtering SSH traffic is useful for monitoring or troubleshooting secure remote connections, such as ensuring proper access or detecting unauthorized attempts.

2. From PowerShell, SSH into the Linux VM using its private IP address:

   ```powershell
   ssh username@10.0.0.5
   ```

   ![SSH Command](https://i.imgur.com/OCHQLVp.png)

3. Follow the prompts to connect and enter your password (not visible while typing). If entered correctly, you'll be logged into the Linux VM. Each keystroke in the session generates its own packet, visible in Wireshark.

   ![SSH Traffic](https://i.imgur.com/EErcK9u.png)

4. Exit the Linux VM:

   ```bash
   exit
   ```

---

## Step 4: Observe DNS Traffic

1. In Wireshark, start a new packet capture and filter for DNS traffic. The DNS (Domain Name System) resolves domain names to IP addresses, making it a key protocol for internet communication.

2. Use the `nslookup` command in PowerShell to look up a website's IP address:

   ```powershell
   nslookup example.com
   ```

   ![DNS Lookup](https://i.imgur.com/QRj22Ay.png)

3. Knowing the IP address is important because it directly identifies a device or server on a network. While DNS makes it easier for humans to access websites, IP addresses are what devices use to actually route traffic.

---

## Step 5: Observe RDP Traffic

1. In Wireshark, start a new packet capture and filter for RDP traffic using the following filter:

   ```
   tcp.port==3389
   ```

2. Monitor the flow of RDP traffic during a remote desktop session. Filtering for RDP (Remote Desktop Protocol) traffic can help diagnose connection issues or monitor active sessions. Notice the constant flow of traffic within Wireshark, as data is continuously transmitted between the client and the server.

   ![RDP Traffic](https://i.imgur.com/2O8GXHO.png)

---

This guide provides a hands-on demonstration of inspecting various network protocols using Wireshark. Each step highlights essential concepts for network monitoring and troubleshooting while maintaining practical examples to enhance understanding.

