# HyperV-Lab-RDS-SessionHost-Deployment

# Hyper-V Lab: Remote Desktop Services (RDS) - Session Host Deployment

## Purpose

This repository details the deployment and basic configuration of a Remote Desktop Session Host (RDSH) within a Hyper-V lab environment. It covers the creation of a dedicated RDSH VM, role installation, configuration of user access permissions, and verification of remote connectivity and session management.

## Prerequisites

Before proceeding with this lab, ensure the following core components of your Hyper-V lab are already built and verified:

* **Hyper-V Host:** Windows Server with Hyper-V role installed.
* **Virtual Switches:** `Internal_Lab_Switch` configured as an "Internal network."
* **Domain Controller:** `DC01` (and optionally `DC02`) is deployed, promoted to a Domain Controller for `mylabs.com`, and DNS is functioning.
* **Client PC:** `StanlyPC` is a domain-joined client machine, required for testing remote access.
* **Windows Admin Center (Optional but Recommended):** Installed on `StanlyPC` for centralized management.

## Lab Phases: Remote Desktop Services (RDS) Deployment

---

### Phase 1: Create `RDSH01` Virtual Machine

This creates a new, dedicated VM to serve as your Remote Desktop Session Host, adhering to best practices for role separation.

* **Action (on your Hyper-V Host):**
    1.  Open **Hyper-V Manager**.
    2.  Click **"New"** > **"Virtual Machine..."**
    3.  Follow the wizard:
        * **Name:** `RDSH01`
        * **Specify Generation:** Select **"Generation 2."**
        * **Assign Memory:** Enter `4096` MB (4 GB). (Minimum recommended; 2GB can work in very constrained labs but performance may suffer).
        * **Configure Networking:** For the "Connection" dropdown, select **`Internal_Lab_Switch`**.
        * **Connect Virtual Hard Disk:** Select "Create a virtual hard disk". Name it `RDSH01.vhdx`. Size: `80` GB (allows space for user profiles and applications).
        * **Installation Options:** Select "Install an operating system from a bootable image file" and browse to your Windows Server ISO file.
        * Click **Finish**.

---

### Phase 2: Install Windows Server and Initial Configuration on `RDSH01`

* **Action (on `RDSH01` VM):**
    1.  In Hyper-V Manager, right-click `RDSH01` and select **"Connect..."**.
    2.  Click **"Start"** in the VM window and proceed with Windows Server installation (select **Desktop Experience**).
    3.  Set the local Administrator password.
    4.  **Rename the Computer:**
        * Log in as local Administrator.
        * Right-click **Start** > **System** > **"Change settings"** next to "Computer name".
        * Click **"Change..."** and type `RDSH01`. Click OK and **Restart**.
    5.  **Configure Static IP Address:**
        * After reboot, log in as local Administrator.
        * Right-click **Start** > **Network Connections** > **"Change adapter options"** (or `ncpa.cpl`).
        * Identify the adapter connected to `Internal_Lab_Switch`. Right-click > **Properties**.
        * Select **"Internet Protocol Version 4 (TCP/IPv4)"** > **Properties**.
        * Select "Use the following IP address:"
            * **IP address:** `192.168.10.5` (Ensure this is an unused IP in your lab subnet)
            * **Subnet mask:** `255.255.255.0`
            * **Default gateway:** `192.168.10.1` (`DC01`'s IP for internet access)
            * **Preferred DNS server:** `192.168.10.1` (`DC01`'s IP)
            * **Alternate DNS server:** `192.168.10.2` (`DC02`'s IP, if powered on)
        * Click **OK**, then **Close**.
    6.  **Join `RDSH01` to the `mylabs.com` Domain:**
        * Right-click **Start** > **System**. Click **"Change settings"** next to "Computer name".
        * Click **"Change..."** under "Computer Name/Domain Changes".
        * Select "Domain:" and type `mylabs.com`.
        * Click **OK**. Provide `mylabs\Administrator` credentials when prompted.
        * Click **OK** to the "Welcome to the `mylabs.com` domain" message.
        * Click **OK**, then **Close**, and **Restart** `RDSH01`.
    7.  **Log in as Domain Administrator:** After reboot, log in to `RDSH01` as `mylabs\Administrator`.
    8.  **Perform Windows Updates (Optional but Recommended):** Go to **Settings** > **Windows Update** and install all available updates. Restart as needed.

---

### Phase 3: Install the Remote Desktop Services Role on `RDSH01`

* **Action (on `RDSH01`, logged in as `mylabs\Administrator`):**
    1.  Open **Server Manager**.
    2.  Click **"Manage"** > **"Add Roles and Features."**
    3.  Follow the wizard:
        * "Before You Begin": Click **"Next"**.
        * "Installation Type": Select **"Role-based or feature-based installation"** > Click **"Next"**.
        * "Server Selection": Ensure `RDSH01` is selected in the "Server Pool" > Click **"Next"**.
        * "Server Roles": Check **"Remote Desktop Services."**
            * When prompted "Add features that are required for Remote Desktop Services?", click **"Add Features."**
            * Click **"Next"**.
        * "Features": No additional features are typically needed here for a basic setup. Click **"Next"**.
        * "Remote Desktop Services" (Confirmation screen): Click **"Next"**.
        * "Role Services": Check **"Remote Desktop Session Host"**.
            * When prompted "Add features that are required for Remote Desktop Session Host?", click **"Add Features."**
            * Click **"Next"**.
        * "Confirmation": Review the roles and features to be installed.
            * Check **"Restart the destination server automatically if required"** (this is common for RDS installations).
            * Click **"Install"**.

---

### Phase 4: Configure Remote Desktop User Permissions

By default, only members of the local "Administrators" group can connect via Remote Desktop. To allow regular domain users (like `stanly.sunny`) to connect, they need to be added to the "Remote Desktop Users" group on the RD Session Host.

* **Action (on `RDSH01`, logged in as `mylabs\Administrator`):**
    1.  Open **Server Manager**.
    2.  Click **"Tools"** > **"Computer Management"**.
    3.  In the left pane of "Computer Management," expand **"Local Users and Groups"**, then click on **"Groups"**.
    4.  In the right pane, double-click on the **"Remote Desktop Users"** group.
    5.  In the "Remote Desktop Users Properties" dialog box, click **"Add..."**.
    6.  In the "Select Users, Computers, Service Accounts, or Groups" dialog box:
        * Click **"Locations..."** and ensure **`mylabs.com`** is selected. Click **"OK"**.
        * In the "Enter the object names to select" field, type `stanly.sunny`.
        * Click **"Check Names"**. It should resolve to `stanly.sunny`.
        * Click **"OK"**.
    7.  Click **"OK"** on the "Remote Desktop Users Properties" dialog box.

---

### Phase 5: Test Remote Desktop Access from `StanlyPC`

This verifies that the Remote Desktop Session Host is functioning and that designated users can connect.

* **Action (on `StanlyPC`, logged in as `stanly.sunny`):**
    1.  Open the **Remote Desktop Connection** client (search "Remote Desktop" in Start Menu).
    2.  In the "Computer" field, type `RDSH01.mylabs.com` (or just `RDSH01`).
    3.  Click **"Connect"**.
    4.  When prompted for credentials:
        * **Username:** Type `mylabs\stanly.sunny`
        * **Password:** Enter `stanly.sunny`'s password.
    5.  You might get a certificate warning (normal for a lab with self-signed certificates). You can safely click **"Yes"** or "Continue."

* **Verification:** You should successfully connect to the desktop of `RDSH01`, logged in as `stanly.sunny`!

---

### Phase 6: View Logged-in Sessions on `RDSH01`

It's crucial to be able to monitor who is using your RD Session Host.

* **Action (on `RDSH01`, logged in as `mylabs\Administrator` or `stanly.sunny`):**
    1.  **Using Task Manager (GUI - Easiest):**
        * Right-click on the **Taskbar** and select **"Task Manager"** (or press `Ctrl+Shift+Esc`).
        * Click on the **"Users"** tab. Here, you will see a list of all currently logged-in users, their session ID, status (Active, Disconnected), client name, and resource usage.
    2.  **Using Command Prompt (`query user` or `quser`):**
        * Open **Command Prompt as Administrator** (Right-click Start Button > "Windows Terminal (Admin)" or "Command Prompt (Admin)").
        * Type the following command and press Enter:
            ```cmd
            query user
            ```
            or the shorter version:
            ```cmd
            quser
            ```
            This will display a table with user sessions, including `USERNAME`, `SESSIONNAME`, `STATE`, and `LOGON TIME`.
    3.  **Using Windows Admin Center (from `StanlyPC`):**
        * On `StanlyPC`, open your browser to Windows Admin Center.
        * Click on your **`RDSH01`** connection in the "All connections" list.
        * In the left-hand navigation pane, scroll down and look for **"Remote Desktop"** (or "Users"). You should find a section detailing active user sessions.

---

### Troubleshooting Common Issues

* **Cannot RDP to `RDSH01`:**
    * **Firewall:** Ensure Windows Defender Firewall on `RDSH01` is allowing "Remote Desktop (TCP-In)" connections. This is usually enabled when the role is installed, but verify.
    * **IP Connectivity:** From `StanlyPC`, try `ping RDSH01.mylabs.com` and `ping 192.168.10.5`. If pings fail, check network settings on both `StanlyPC` and `RDSH01`, and ensure both are on the `Internal_Lab_Switch`.
    * **DNS Resolution:** On `StanlyPC`, run `nslookup RDSH01.mylabs.com`. Ensure it resolves to `192.168.10.5`. If not, check the DNS A record on `DC01`.
    * **Remote Desktop Service:** On `RDSH01`, open `services.msc` and ensure the "Remote Desktop Services" service is running.
    * **User Permissions:** Double-check that the connecting user (`stanly.sunny`) is a member of the **"Remote Desktop Users"** group on `RDSH01`.
* **"The user is not allowed to log on to this remote computer"**:
    * This almost always means the user account is not a member of the "Remote Desktop Users" group on the `RDSH01` server. Review Phase 4.
* **Slow performance after RDP:**
    * Check `RDSH01` VM's assigned RAM in Hyper-V Manager. If it's too low (e.g., 2GB), consider increasing it to 4GB if your host allows.
    * Check CPU usage on `RDSH01` (Task Manager > Performance tab).

---
