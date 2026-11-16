# Lab  IIS, NTP, and WSUS

## What you will build

- An IIS site on **ServerDM1** that the Windows client can open by DNS name.
- A Windows Time configuration in which the client gets time through the Active Directory domain hierarchy.
- A WSUS server on **ServerDM1** that synchronizes update metadata and offers one applicable update to a single pilot client.

## Before you start

Use three VMs on the same dedicated VMware Workstation NAT network. Open **Virtual Network Editor** to determine your subnet and gateway. Use your actual VM addresses and AD DNS name throughout. The names and addresses below describe the expected lab setup. Replace `X` with your NAT subnet's third octet and `corp.example` with the actual domain.

| VM | Address | Required starting state |
|---|---|---|
| ServerDC1 | `192.168.X.10` | Windows Server 2019 with AD DS, DNS and Group Policy Management |
| ServerDM1 | `192.168.X.20` | Windows Server 2019 member server, static IPv4 with ServerDC1 as DNS, and free NTFS disk space for WSUS |
| Windows client | Existing lab IP | Windows 10 or Windows 11 Education, Pro, or Enterprise, joined to the same domain and using ServerDC1 for DNS |

Use elevated Windows PowerShell for administrative commands. Use an account authorized to manage the lab DNS zone and create and link GPOs. On both domain members, confirm that your AD DNS name resolves and run `Test-ComputerSecureChannel -Verbose`. Expect `True`. On the client, record the computer name and run `winver` to identify the Windows release and build. ServerDM1 needs internet access to synchronize with Microsoft Update. Confirm DNS and outbound HTTPS connectivity and allow time for the first WSUS synchronization.

## Part A · Publish and validate an IIS site

1. On **ServerDM1**, install IIS and its management tools.

   ```powershell
   Install-WindowsFeature -Name Web-Server -IncludeManagementTools
   Get-WindowsFeature Web-Server
   ```

   Confirm the role shows **Installed**. If the installation reports a required restart, restart before proceeding.
2. On **ServerDM1**, create a new folder for the site and a test page. Choose a different path if this folder is already in use.

   ```powershell
   New-Item -ItemType Directory -Path 'C:\CST89200\Portal' -Force
   '<!doctype html><html lang="en"><title>CST 89200 Portal</title><h1>CST 89200 IIS portal on ServerDM1</h1></html>' |
       Set-Content -Path 'C:\CST89200\Portal\index.html' -Encoding UTF8
   ```

3. On **ServerDM1**, open **IIS Manager → Sites → Add Website**. Set site name to **CST89200-Portal**, physical path to the folder you created, type to **http**, port to **80**, and host name to `portal.corp.example` using your actual AD DNS suffix. Leave **Start Website immediately** checked. Confirm no existing site uses the same binding. If the client cannot connect to TCP 80, check the IIS HTTP inbound firewall rule in the active profile.
4. On **ServerDC1**, open **DNS Manager → Forward Lookup Zones → your AD DNS zone**. Create a new **Host (A)** record named **portal** with ServerDM1’s IPv4 address. If the name is already assigned to another service, use a unique hostname and update both the IIS binding and all client tests.
5. On the **Windows client**, run `Resolve-DnsName portal.corp.example` and `Test-NetConnection portal.corp.example -Port 80`. In the browser, open both `http://portal.corp.example/index.html` and `http://portal.corp.example/missing.html`. Expect a working page for the first URL and a 404 response for the second. On **ServerDM1**, select the site in IIS Manager and find its ID under **Advanced Settings**. Open the corresponding `%SystemDrive%\inetpub\logs\LogFiles\W3SVC<site ID>` folder. Allow time for IIS to flush the log, then save the two entries with status **200** and **404**.

## Part B · Configure and verify Windows Time

1. On **ServerDC1**, identify the forest-root PDC emulator and record the current time source. Run the PDC configuration step only on the VM that owns that role.

   ```powershell
   (Get-ADDomain -Identity (Get-ADForest).RootDomain).PDCEmulator
   w32tm /query /source
   w32tm /query /configuration
   ```

2. On the **lab PDC emulator**, configure the approved upstream NTP server. Use the example below if `time.windows.com` is permitted and reachable over UDP 123. Otherwise, use the lab's approved NTP server in its place. If `w32tm /query /configuration` shows Group Policy controls the settings, apply this change through the authorized lab policy instead of overriding it locally.

   ```powershell
   w32tm /config "/manualpeerlist:time.windows.com,0x8" /syncfromflags:manual /reliable:yes /update
   Restart-Service W32Time
   w32tm /resync /rediscover
   w32tm /query /source
   w32tm /query /status
   ```

   Expect the approved peer and a recent successful synchronization. If the source remains a local clock or VMware time provider, check the approved peer's DNS and UDP access, W32Time status and applicable policy before claiming upstream sync.
3. On the **Windows client**, configure the domain hierarchy and verify the time source. If a lab GPO controls W32Time, verify that it sets the client to **NT5DS** and apply the policy before testing.

   ```powershell
   w32tm /config /syncfromflags:domhier /update
   Restart-Service W32Time
   w32tm /resync /rediscover
   w32tm /query /source
   w32tm /query /status
   w32tm /query /configuration
   ```

4. From the **Windows client**, run `w32tm /stripchart /computer:ServerDC1.corp.example /samples:5 /dataonly`, substituting the actual server FQDN. Record the source and offset samples. The reported time source should be a domain controller. The stripchart measures the offset to the server you named, which can differ from the source selected by the client. If the command times out, record the error and check whether the specified server responds to NTP on the lab network.

## Part C · Build a WSUS pilot

### Install and synchronize WSUS

1. On **ServerDM1**, use **Server Manager → Manage → Add Roles and Features** to install **Windows Server Update Services**. Keep **WID Connectivity** and **WSUS Services** selected and accept the required features. Choose a local content path such as `C:\WSUS` on an NTFS volume with sufficient space. Complete installation and select **Launch Post-Installation tasks** from Server Manager notifications. Confirm post-installation succeeds.
2. Open **Server Manager → Tools → Windows Server Update Services**. In the Configuration Wizard, choose **Synchronize from Microsoft Update**. Select the client's language, the Windows product matching the release you recorded with `winver`, and **Security Updates** or **Critical Updates**. Start manual synchronization and wait for **Synchronizations** to show a successful result. Select only the required product and classifications to limit synchronization time.
3. In the WSUS console, under **Options → Computers**, confirm targeting uses the WSUS console. Under **Computers → All Computers**, create a group called **CST89200-Pilot**.

### Connect the client to your WSUS server

4. On **ServerDC1**, open **Active Directory Users and Computers**. Create a lab OU named **CST89200-WSUS-Pilot** and record the Windows client's current OU or container. Move only that lab client’s computer object into the pilot OU. In **Group Policy Management**, create and link a GPO named **CST89200-WSUS-Client** to the pilot OU.
5. Edit the GPO. Under **Computer Configuration → Policies → Administrative Templates → Windows Components → Windows Update**, enable **Configure Automatic Updates** and **Specify intranet Microsoft update service location**. The location setting may be under **Manage updates offered from Windows Server Update Service**. Enter `http://ServerDM1.corp.example:8530` for both the update detection and statistics endpoints, replacing the DNS suffix. If the installed policy templates expose **Specify source service for specific classes of Windows Updates**, set **Quality Updates** to **Windows Server Update Services**. Use the isolated lab HTTP endpoint.
6. On the **Windows client**, run the following after replacing the example FQDN. Check that the WSUS GPO appears under **Applied Group Policy Objects** and that TCP 8530 is reachable. Use **Windows Update → Check for updates** to start a managed scan after verifying policy, then allow time for the client to appear in WSUS under **Unassigned Computers**. Use the managed update scan rather than the separate option to check online with Microsoft Update.

   ```powershell
   gpupdate /force
   gpresult /r /scope computer
   Test-NetConnection ServerDM1.corp.example -Port 8530
   Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate' |
       Select-Object WUServer, WUStatusServer
   ```

### Approve and verify one pilot update

7. On **ServerDM1**, move the registered client into **CST89200-Pilot**. Select one synchronized update that the client reports as needed, then approve it for **Install** for the pilot group only. Verify other groups do not have an install approval for that update. Record the update title or KB number and the pilot group membership.
8. Trigger another managed scan or allow the next scheduled scan. In WSUS, record the status and last report time for the client. Record **Installed** only if the client reports installation. If reporting is pending, provide evidence of synchronization, policy, client registration and approval, then record the last report time and the current state. If the client reports every synchronized update as not applicable, check its edition, build and existing patch level and use an applicable update.

## What to submit

| Build | Evidence |
|---|---|
| IIS | DNS answer, successful portal page and IIS log lines for the 200 and 404 responses |
| NTP | Forest-root PDC owner, PDC upstream source, client domain time source and stripchart output or documented timeout |
| WSUS | Successful synchronization, client policy and WSUS URL, pilot membership, update approval, and the client’s actual update status and report time |

Label each result with the VM name. For any incomplete check, document the error, the checks you performed and the last successful step.
