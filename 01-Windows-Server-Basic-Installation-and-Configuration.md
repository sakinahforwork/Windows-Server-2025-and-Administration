# 01-Windows-Server-Basic-Installation-and-Configuration
This topic focuses on a basic installation and configuration of Windows Server before install any services or promote to a Domain Controller.

## Lab Environment 

**Prerequisites:**

* *Physical Host* - 16GB RAM, 512 GB SSD, Windows 11 Pro Edition
* *Virtual Machine* - Hyper-V *(Need to enable first before use)*
* *Software* - Windows Server 2025 ISO Image, download it from official [website](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)
  

**Enable VM (Hyper-V) and Create a New Virtual Machine :**

1. Press Win + R > Run `optionalfeatures` > Select/Check `Hyper-V`, click `OK` > Restart the pc.
<div align="center"><img src="Screenshots/0-Windows-Features.png" alt="My Screenshot" width="400" height="500" ></div>

2. Open `Hyper-V Manager`> At the left pane, click at the PC's name.
3. Then go to the `Actions` pane, at the right > `New` > `Virtual Machines...` , the `New Virtual Machine Wizard` appears.
<div align="center"><img src="Screenshots/1-Create-new-vm.png" alt="My Screenshot" width="600" height="600" ></div>

4. Starts create the new Windows Server VM:

* Before You Begin : Click `Next`
* Specify the Name and Location : **Name:** *Windows Server 2025* | **Location:**  *`D:\Hyper-V\`*
* Specify Generation : Generation 2
* Assign Memory : 4 GB
* Configure Networking : Default Switch

 *Note: Why `Default Switch`? It provides out-of-the-box internet connectivity using NAT without requiring manual virtual switch configuration. More importantly, it isolates the lab environment from my physical home router—preventing potential IP conflicts or rogue DHCP issues when configuring Windows Server roles later.
 
* Connect Virtual Hard Disk : 60 GB
* Installation Options : Install an operating system from a bootable file > browse the Windows Server ISO file that has been downloaded earlier.
* Click `Finish`.

<div align="center"><img src="Screenshots/6-Install-Windows-Server-2025.png" alt="My Screenshot" width="600" height="600" ></div>

5. Right click at the Windows Server VM > Click `Settings` > Go to Security > `Enable Trusted Platform Module`

<div align="center"><img src="Screenshots/8-Enable-TPM.png" alt="My Screenshot" width="600" height="600" ></div>

---

## Installation and Basic Windows Server Setup

**Install the Windows Server 2025 operating system in VM :**

1. Right click at the `Windows Server 2025` VM > click `Connect` > Press any key when the 'Press any key to start...' screen appears.

<div align="center"><img src="Screenshots/9-Installation-started.png" alt="My Screenshot" width="600" height="600" ></div>


2. Starts the installation process by selecting the language, Time & currency format, keyboard layout.
3. Choose `Windows Server 2025 Standard Evaluation (Desktop Experience)`

<div align="center"><img src="Screenshots/10-Desktop-Experience.png" alt="My Screenshot" width="600" height="600" ></div>

4. Set the password for Administrator account and click `Finish`

<div align="center"><img src="Screenshots/12-Set-Admin-Password.png" alt="My Screenshot" width="600" height="600" ></div>

---

## Setup Windows Server For The First Time

Before promoting the server or installing roles, ensure the server operating system is up to date and baseline configurations have been set up to avoid compatibility and stability issues later.


1. **Run Windows Update:**
   * Sign in to the `Windows Server 2025` VM using the `Administrator` credentials.
   * Navigate to **Settings** > **Windows Update** (or **Server Manager** > **Local Server** > **Windows Update**), click **Check for updates**, and install all available patches.
   * Restart the VM when prompted so updates apply cleanly before making further changes.

2. **Configure the Time Zone:**
   * Open **Server Manager** > **Local Server** and click on the current **Time zone** value.
   * Click **Change time zone...**, select your local time zone *(e.g., `(UTC+08:00) Kuala Lumpur, Singapore`)*, and click **OK**.
   * *Note: Accurate time synchronization is critical for Active Directory Kerberos authentication and event log tracking.*

3. **Rename the Server (Hostname):**
   * In **Server Manager** > **Local Server**, click on the randomly generated **Computer name**.
   * In the **System Properties** window, click **Change...**, enter a clear, descriptive hostname `DC01` and click **OK**.
   * Restart the VM when prompted to apply the new hostname before configuring network settings or server roles.

4. **Configure a Static IP Address:**
   * Open **PowerShell** (or **Command Prompt**) and run `ipconfig /all` to inspect the current subnet, default gateway, and DNS server assigned by the Hyper-V `Default Switch`
   
*(note: Hyper-V's Default Switch subnet can change across host reboots, so assign an IP within its active range)*.

   * Open **Server Manager** > **Local Server** and click on the **Ethernet** link next to **IPv4 address assigned by DHCP**.
   * Right-click the network adapter (`Ethernet`), select **Properties**, double-click **Internet Protocol Version 4 (TCP/IPv4)**, and select **Use the following IP address**.
   * Enter your static network details based on the `ipconfig` output:
     * **IP address:** *(e.g., `172.24.160.10` — an unused IP within your Default Switch subnet)*
     * **Subnet mask:** `255.255.240.0` *(match the exact mask shown in `ipconfig`)*
     * **Default gateway:** *(e.g., `172.24.160.1` — the exact Default Gateway shown in `ipconfig`)*
     * **Preferred DNS server:** `127.0.0.1` *(or the server's own static IP if installing Active Directory/DNS next; set **Alternate DNS server** to `8.8.8.8` or your Default Gateway to maintain internet access)*
   * Click **OK** to save, then verify connectivity by running `ping google.com` in PowerShell.
