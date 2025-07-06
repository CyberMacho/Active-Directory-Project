# Active Directory Project

###
This project creates a virtual lab environment using Oracle VM VirtualBox with multiple virtual machines (VMs), including Windows 10, Kali Linux, Windows Server, and Ubuntu Server. The network is configured with IP addressing and NAT networks to enable seamless communication between all VMs.

Security Monitoring is established using a Splunk Server for centralized log analysis, supported by a Universal Forwarder to collect and forward logs, alongside Sysmon for detailed endpoint monitoring. For Testing, tools like Crowbar simulate brute-force attacks while the Atomic Red Team (ART) framework executes a variety of cybersecurity tests, all analyzed through Splunk logs.

The lab includes a full Active Directory Setup where Windows machines join a domain and have Remote Desktop enabled for remote access. Additionally, Automation is implemented through PowerShell scripting to streamline routine tasks.

Overall, this hands-on environment offers practical experience with key cybersecurity tools and concepts within a secure and isolated setting.



![Active Directory Lab Diagram](https://github.com/user-attachments/assets/ba3e74f0-0cbc-4ae4-bb14-4ca792cd3ac0)

---
## Objective

The objective of this lab is to provide a hands-on learning experience in setting up a virtualized environment tailored for cybersecurity testing and exploration. By creating and configuring multiple virtual machines (VMs) — including **Windows 10**, **Kali Linux**, **Windows Server**, and **Ubuntu Server** — participants will develop skills in:

- Network configuration and management  
- Installation and use of security tools such as **Splunk** and **Sysmon**  
- Endpoint monitoring and log analysis  
- Security testing, including brute force attack simulations using **Crowbar**  
- Integrating Windows machines into an **Active Directory** domain  
- Enabling and managing **Remote Desktop** access  
- Automating tasks using **PowerShell scripting**

## Skills Learned

- Setting up virtual machines (Windows 10, Kali Linux, Windows Server, Ubuntu Server) in **Oracle VM VirtualBox**  
- Configuring IP addresses and NAT networks for VMs  
- Troubleshooting network connectivity issues using tools like **ping** and configuring **DNS settings**  
- Installing and configuring **Splunk Server** and **Universal Forwarder** for log collection and analysis  
- Deploying **Sysmon** for advanced endpoint monitoring  
- Using **Crowbar** to perform brute force attack simulations  
- Leveraging **Atomic Red Team (ART)** to simulate various cybersecurity tests  
- Analyzing security logs, focusing on critical event codes such as **4625** (failed logon) and **4624** (successful logon) within Splunk  
- Joining Windows machines to an **Active Directory domain**  
- Enabling and managing **Remote Desktop** access on Windows systems  
- Writing and running **PowerShell scripts** to automate tasks (e.g., `Invoke-WebRequest`, `Set-ExecutionPolicy`)  

## Tools Used

- **Oracle VM VirtualBox Manager:** For creating and managing virtual machines (VMs)  
- **Splunk Server:** For centralized log analysis and monitoring  
- **Splunk Universal Forwarder:** For forwarding data to the Splunk Server  
- **Sysmon:** Endpoint monitoring tool for Windows machines  
- **Crowbar:** Tool to simulate brute force attacks  
- **Atomic Red Team (ART):** Framework for security testing and validation  
- **PowerShell:** Scripting and automation tasks  
- **Microsoft Windows Event Logs:** Source of security logs analyzed within Splunk  
- **Windows Server 2022:** Operating system used to set up Active Directory Domain Services  
- **Ubuntu Server:** Deployed as the Splunk server in the lab environment  
- **Microsoft Windows 10:** Operating system for target machines in the lab  
- **Kali Linux:** Used as the attacker machine in the lab setup  

# Steps

## Part 1 - VM Installation

### 1. Install Oracle VM VirtualBox Manager  
Navigate to [https://www.virtualbox.org/](https://www.virtualbox.org/), download the version compatible with your OS, and install it along with any required dependencies.

### 2. Install Windows 10  
- Go to [Microsoft's Windows 10 download page](https://www.microsoft.com/en-ca/software-download/windows10).  
- Select **Create Windows 10 installation media**, then click **Download tool now**.  
- Choose **Create installation media (USB flash drive, DVD, or ISO file) for another PC**, select **ISO file**, and save it locally.  
- In Oracle VM VirtualBox Manager, click **New** to create a new VM:  
  - Name the VM.  
  - Select the downloaded Windows 10 ISO file.  
  - Allocate **4096 MB RAM**, **1 CPU**, and a **50 GB virtual hard disk**.  
- Start the VM and follow the installation prompts: choose **Custom: Install Windows only (advanced)** and complete the installation.

### 3. Install Kali Linux  
- Download Kali Linux VM image from [https://www.kali.org/](https://www.kali.org/).  
- Download and install [7-Zip](https://www.7-zip.org/) to extract the Kali Linux archive.  
- Extract the Kali Linux files using 7-Zip.  
- Import the extracted Kali Linux VM into Oracle VM VirtualBox Manager.  
- Start the VM.

### 4. Install Windows Server  
- Download Windows Server 2022 ISO from [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022).  
- Fill out the required form and download the **64-bit edition**.  
- Create a new VM in Oracle VM VirtualBox Manager using the downloaded ISO, with:  
  - **4096 MB RAM**  
  - **1 CPU**  
  - **50 GB virtual disk**  
- Start the VM, select **Install now**.  
- Choose **Windows Server 2022 Standard Evaluation (Desktop Experience)**, customize settings, create a password, and complete the setup.

### 5. Install Ubuntu Server  
- Visit [https://ubuntu.com/server](https://ubuntu.com/server) and download **Ubuntu Server 22.04.4 LTS**.  
- Create a new VM in Oracle VM VirtualBox Manager with the Ubuntu ISO, allocating:  
  - **8192 MB RAM**  
  - **2 CPUs**  
  - **100 GB virtual disk**  
- Start the VM, select **Try or Install Ubuntu Server**, and proceed through the setup steps.  
- Complete the installation form as prompted and reboot when finished. (*Note: Some error messages during installation are expected.*)  
- After reboot, log in and update the system by running:  
  ```bash
  sudo apt-get update && sudo apt-get upgrade -y


### Summary  
You should now have Oracle VM VirtualBox Manager installed with four VMs running:

- Windows 10  
- Kali Linux  
- Windows Server  
- Splunk Server  

---

## Part 2 - Configure the Network

### 1. Setup Communications

1. In Oracle VM VirtualBox Manager, navigate to:  
   **Tools > Network > NAT Networks > Create**.  
2. Provide a name for the NAT Network and set the **IPv4 Prefix** to `192.168.10.0/24` (used throughout this lab), then apply the settings.  
3. For each VM, go to:  
   **Settings > Network**,  
   change **Attached to:** to **NAT Network**,  
   and select the NAT Network you just created.  

4. On the Splunk VM, open a terminal and edit the netplan configuration:  
   ```bash
   sudo nano /etc/netplan/00-installer-config.yaml

### Modify the file and save
```yaml
network: 
  ethernet:
    enp0s3:
      dhcp4: no
      addresses: [192.168.10.10/24]
      nameservers:
          addresses: [8.8.8.8]
      routes:
          - to: default
            via: 192.168.10.1
  version: 2

sudo netplan apply
ip a
ping google.com
```

### 2. Install Splunk

Navigate to [https://www.splunk.com/](https://www.splunk.com/) and download the free trial of Splunk Enterprise for Linux (.deb). On your Splunk VM, install VirtualBox guest additions by running `sudo apt-get install virtualbox-guest-additions-iso`. In VirtualBox Manager, go to **Devices > Shared Folders > Shared Folders Settings > Create New Shared Folder**, select the directory where the Splunk `.deb` file is located, check all three options (Auto-mount, Make Permanent, Read-only as needed), and save. Reboot the VM with `sudo reboot`. After reboot, install guest utilities and add your user to the shared folder group by running `sudo apt-get install virtualbox-guest-utils`, then `sudo reboot`, followed by `sudo adduser <your-username> vboxsf`. Create a directory to mount the shared folder by running `mkdir share`. Mount the shared folder (replace `<shared-folder-name>` with the folder name you set in VirtualBox) with `sudo mount -t vboxsf -o uid=1000,gid=1000 <shared-folder-name> share/`. Verify the mount by running `ls -la`, then `cd share/` and `ls -la` again to see the files including the Splunk `.deb` installer. Install Splunk using `dpkg` by running `sudo dpkg -i splu<TAB>` to auto-complete the filename. Navigate to the Splunk installation directory with `cd /opt/splunk/` and list files with `ls -la`. Switch to the Splunk user with `sudo -u splunk bash`, go to the `bin/` directory by running `cd bin/`, and start Splunk using `./splunk start`. When prompted, press `q` to exit the license agreement, then type `y` and press **Enter** to accept. To finalize setup, exit the Splunk user shell with `exit`, navigate back to the `bin` directory if needed with `cd /opt/splunk/bin`, and enable Splunk to start on boot by running `sudo ./splunk enable boot-start -user splunk`.

### 3. Configure Windows Server

In the Start Menu, search for **About** > select **Rename this PC** and rename it to your preferred name (in this lab, we use "target-PC"). Restart the system. Open Command Prompt, run `ipconfig` to view the current IPv4 address. Click the network icon at the bottom-right, right-click it, then select **Open Network & Internet Settings > Change adapter options**. Right-click your network adapter, choose **Properties**, then double-click **Internet Protocol Version 4 (TCP/IPv4)**. Select **Use the following IP address** and set the IP Address to `192.168.10.100`, Subnet mask to `255.255.255.0`, Default gateway to `192.168.10.1`, and Preferred DNS server to `8.8.8.8`. Running `ipconfig` again should confirm the new IP.

If your Splunk server is running, open a browser on the target machine and navigate to `http://192.168.10.10:8000`. To install the Universal Forwarder, visit [https://www.splunk.com/](https://www.splunk.com/), go to **Products > Free Trials & Downloads > Universal Forwarder**, download the appropriate version for your target machine, and run the MSI installer. Provide basic setup info but skip creating a password and the deployment server setup. For Receiving Indexer, enter `192.168.10.10:9997`, then complete the installation.

Next, install Sysmon by downloading it from [Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon). Then, download the `sysmonconfig.xml` file from [Sysmon Modular GitHub](https://github.com/olafhartong/sysmon-modular) by clicking **Raw** and saving the file. Extract the Sysmon files, copy the path of the extracted directory, open PowerShell as Administrator, navigate to that directory, and run `.\Sysmon64.exe -i ..\sysmonconfig.xml`. Accept the license agreement.

Now, navigate to `C:\Program Files\SplunkUniversalForwarder\etc\system\local`, open Notepad as Administrator, and create a file named `inputs.conf` with the following content:

```ini
[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

```

### 4. Summary

When viewing your Splunk server from either of your Windows machines, under **Selected fields > Host**, you should see two values representing your VMs: **TARGET-PC** and **ADDC01**.

## Part 3 - Active Directory and Control Domain

### 1. Setup Active Directory Domain Services on the Windows Server

On your Windows Server, open **Server Manager**, then navigate to **Manage > Add Roles and Features**. Click **Next > Next**, check **Active Directory Domain Services** and select **Add Features**. Continue advancing until you can click **Install**.

Once you receive the message **"Configuration required. Installation succeeded on ADDC01"**, proceed to the next steps. Locate the flag icon at the top of the window and select **Promote this server to a domain controller**. Choose **Add a new forest** since this is a new domain. Name the domain **demodomain.local**. On the next page, leave all settings as default and create a secure password. Continue until the Configuration Wizard validates prerequisites, then click **Install**. The server will sign you out and restart.

After reboot, log in as **DEMODOMAIN\Administrator**. Open **Server Manager**, then go to **Tools > Active Directory Users and Computers**. Right-click **demodomain.local**, select **New > Organizational Unit**, and name it **IT**. Right-click inside this new OU, select **New > User**, and create a user with First name: **Jenny**, Last name: **Smith**, Full name: **Jenny Smith**, and User logon name: **jsmith**. Create a password and finish. Repeat to create a new OU called **HR** and add another user.

### 2. Windows 10 Machine Joins New Domain

On the target Windows 10 machine, open Windows Search and type **Advanced System Properties**. Go to the **Computer Name** tab and click **Change**. Select **Domain:** and enter **DEMODOMAIN.LOCAL**. Next, right-click the network icon in the bottom right, choose **Open Network & Internet Settings > Change Adapter Options**, right-click your adapter, select **Properties**, then double-click **Internet Protocol Version 4 (TCP/IPv4)**. Set the Preferred DNS server to `192.168.10.7`. Open Command Prompt and run `ipconfig /all` to verify the DNS Servers value is `192.168.10.7`.

Return to the Computer Name/Domain Changes window and click **OK**. When prompted, enter administrator credentials to join the domain and restart the machine. At the login screen, select **Other User** and verify the domain is **DEMODOMAIN**. Log in using the credentials of one of the users you created under the IT or HR organizational units.

## Part 4 - Kali Linux Brute Force Attack on the Target Machine

### 1. Configure Network
Power on the Kali Linux VM and log in. Right-click the network icon (top-right) > **Edit Connections** > select **Wired connection 1** > **Edit selected connection**. Under **IPv4 Settings**, set **Method** to `Manual`, then click **Add** and input the following:

- Address: `192.168.10.250`
- Netmask: `24`
- Gateway: `192.168.10.1`
- DNS servers: `8.8.8.8`

Click **Save**, disconnect from the network, then reconnect. Open a terminal and run `ip a` to verify the new IP. Test connectivity using:
```bash
ping google.com
ping 192.168.10.10


### *2. Setting Up the Attack*

On the **Kali Linux** machine:

1. Create a working directory and install Crowbar:
   ```bash
   mkdir ad-project
   sudo apt-get install -y crowbar
#Navigate to the default wordlist directory and verify that rockyou.txt.gz exists:
cd /usr/share/wordlists
ls
#Unzip the wordlist:
sudo gunzip rockyou.txt.gz
#Copy the extracted wordlist to your project directory:
cp rockyou.txt ~/Desktop/ad-project
cd ~/Desktop/ad-project
#Create a new file named passwords.txt containing the first 20 entries from the rockyou.txt wordlist:
head -n 20 rockyou.txt > passwords.txt
```
### *3. Enable Remote Desktop*

On the **target Windows 10 machine**:

1. Search for **"This PC"**, then right-click and select **Properties**.
2. Click **Advanced System Settings** on the left panel.
3. Go to the **Remote** tab.
4. Under **Remote Desktop**, check **"Allow remote connections to this computer"**.
5. Click **Select Users** > **Add**, and enter the usernames you created earlier (e.g., `jsmith`, `tsmith`).
6. Click **Apply** and **OK** to confirm.

This enables Remote Desktop Protocol (RDP) access for specified users, which is required for the brute force simulation.

---

### *4. Perform the Brute Force Attack*

On the **Kali Linux** machine:

1. Run the following command to view Crowbar usage options:
   ```bash
   crowbar -h
Launch the brute force attack using the prepared password list (passwords.txt) and a known username (e.g., tsmith):
 ```bash
crowbar -b rdp -u tsmith -C passwords.txt -s 192.168.10.100/32
 ```

### *5. Analyze the Attack Using Splunk*

To analyze the brute force attack, open Splunk and use the following search query:

```spl
index=endpoint
index=endpoint tsmith

### *6. Run Tests on the Target Machine Using Atomic Red Team (ART)*

Begin by opening **PowerShell as Administrator** on the target machine and setting the execution policy:

```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser
#install Atomic Red Team with the following command
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics
Invoke-AtomicTest T1136.001
index=endpoint
```

After executing this test, check Splunk for related logs. If no events appear for actions like `NewLocalUser`, it highlights gaps in your detection capabilities. Running multiple tests with different technique IDs helps strengthen your security posture. This approach makes Atomic Red Team an invaluable tool for continuous detection validation.

*Credit: Thanks to [MyDFIR](https://mydfir.com/) for valuable insights and resources on Atomic Red Team testing.*
