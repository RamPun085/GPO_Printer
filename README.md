# GPO_Printer
# Enterprise-Style Printer Deployment Using Active Directory and Group Policy

This guide documents the full process of setting up a printer deployment infrastructure similar to large organizations. It covers Active Directory setup, print server configuration, printer sharing, and automated printer deployment using Group Policy with Point and Print Restrictions configured.

---

## Table of Contents

- [Prerequisites](#prerequisites)  
- [Step 1: Prepare Active Directory Environment](#step-1-prepare-active-directory-environment)  
- [Step 2: Configure Print Server](#step-2-configure-print-server)  
- [Step 3: Share and Publish the Printer](#step-3-share-and-publish-the-printer)  
- [Step 4: Create Group Policy for Printer Deployment](#step-4-create-group-policy-for-printer-deployment)  
- [Step 5: Configure Point and Print Restrictions](#step-5-configure-point-and-print-restrictions)  
- [Step 6: Test Printer Deployment](#step-6-test-printer-deployment)  
- [Optional: Batch File Fallback](#optional-batch-file-fallback)  
- [Troubleshooting](#troubleshooting)  
- [Further Enhancements](#further-enhancements)

---

## Prerequisites

- Windows Server as Domain Controller with Active Directory Domain Services (AD DS), DNS, and DHCP configured.
- Print Server role installed on a server joined to the domain.
- Printer installed and connected to the Print Server.
- Client machines joined to the Active Directory domain.
- Appropriate administrative permissions to manage AD, Print Server, and GPOs.

---

## Step 1: Prepare Active Directory Environment

- Ensure your Domain Controller is set up with AD DS, DNS, and DHCP.
- Use **Active Directory Users and Computers (ADUC)** to organize users and computers into Organizational Units (OUs) as needed.
- Confirm DNS is resolving your domain and print server names correctly.

---

## Step 2: Configure Print Server

- Install the **Print and Document Services** role on your designated print server.
- Add and install the printer drivers for your physical printer.
- Connect the printer and verify it works locally on the print server.

---

## Step 3: Share and Publish the Printer

1. Open **Print Management** on the print server.
2. Navigate to **Printers**, right-click your printer, and select **Properties**.
3. Go to the **Sharing** tab:
   - Check **Share this printer**.
   - Give it a share name (e.g., `Home Printer`).
   - Check **List in the directory** to publish the printer in Active Directory.
4. Apply changes.

---

## Step 4: Create Group Policy for Printer Deployment

1. Open **Group Policy Management Console (GPMC)** on the Domain Controller.
2. Right-click the OU containing your users (or computers) and select **Create a GPO in this domain, and Link it here...**
3. Name the GPO, e.g., `Deploy Home Printer`.
4. Edit the GPO:
   - For **User-Based Deployment**:
     ```
     User Configuration > Preferences > Control Panel Settings > Printers
     ```
   - For **Computer-Based Deployment**:
     ```
     Computer Configuration > Preferences > Control Panel Settings > Printers
     ```
5. Right-click **Printers** > **New** > **Shared Printer**.
6. Set the following:
   - **Action**: Create
   - **Share Path**: `\\PR-PRN1-PRINTER02\Home Printer`
   - (Optional) Check **Set this printer as the default printer**
7. Apply and close.

---

## Step 5: Configure Point and Print Restrictions

1. In the same GPO, navigate to:
   Computer Configuration > Policies > Administrative Templates > Printers
   

2. Configure the following policies:

- **Point and Print Restrictions**  
  - Set to **Disabled** (Disables security prompts for printer driver installs).

- **Package Point and Print - Approved Servers**  
  - Set to **Enabled**.  
  - Click **Show...** and add your print server UNC path(s), e.g.:  
    ```
    \\PR-PRN1-PRINTER02
    ```

3. Apply and close.

---

## Step 6: Test Printer Deployment

1. On a client machine joined to the domain, open Command Prompt as a user.
2. Run:
Run gpupdate /fore on client  device

3. Log off and log back in (or reboot).
4. Go to **Devices and Printers** on the client and verify the printer appears and installs automatically.
5. Print a test page to confirm functionality.

---

## Optional: Batch File Fallback

Provide a batch file as a manual fallback for users or helpdesk:


```batch@echo off
rundll32 printui.dll,PrintUIEntry /in /n"\\PR-PRN1-PRINTER02\Home Printer"'''

Place this batch file on a shared network location accessible by users, e.g.,

\\dc01\netlogon\install_home_printer.bat
Troubleshooting
Printer not appearing? Check GPO scope and OU membership of client/computer/user.

Group Policy not applying? Run gpresult /r on client to verify applied GPOs.

Driver prompts still showing? Ensure Point and Print Restrictions policy is correctly set and applied.

DNS issues? Verify client can resolve PR-PRN1-PRINTER02 hostname.

Further Enhancements
Use Item-Level Targeting in GPO to deploy different printers based on security groups or locations.

Configure Print Quotas or Monitoring tools like PaperCut.

Set up Printer Pooling or Failover Print Servers for high availability.

Automate printer driver updates via SCCM or other software deployment tools.
