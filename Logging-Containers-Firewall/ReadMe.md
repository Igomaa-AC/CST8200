# Logging, Windows Containers, and Firewall

## Shared lab environment

Use the five VMware Workstation VMs and the NAT VMnet recorded in **Lab 1 – Preparing the Lab Environment**. Find the actual subnet and mask in Virtual Network Editor. Replace `X` with the third octet for your NAT subnet wherever it appears, including in PowerShell examples. Verify the computer names and IP addresses before changing settings, and document any differences.

| Existing VM | Lab 1 IP | Use in these exercises |
|---|---|---|
| ServerDC1 | 192.168.X.10 | AD DS and DNS in the existing course domain |
| ServerDC2 | 192.168.X.11 | Second DC in the same domain for Class 4, if available for that role |
| ServerDM1 | 192.168.X.20 | IIS and WSUS |
| ServerDM2 | 192.168.X.21 | Windows containers, firewall, DHCP, WDS and partner DNS |
| Windows 10 or Windows 11 client | VMware DHCP at baseline | Client tests, Wireshark and interactive PXE boot |

**Before class.** Lab 1 installs Windows and VMware Tools and captures snapshots. Check whether AD DS, domain membership and any required server roles are already configured. For Classes 2–4, use your existing AD DNS name wherever `corp.example` appears. If the environment is still at the original Lab 1 baseline, the instructor must first configure ServerDC1 as the lab DC and DNS server, join ServerDM1, ServerDM2 and the Windows client where required, and verify DNS and time. ServerDC2 is only added to that domain when appropriate for Class 4. Use elevated PowerShell unless directed otherwise. Check free storage and memory on the existing VMs before staging images and update content.

**Network baseline.** Use the NAT VMnet for Classes 1, 2 and 4. Keep your normal Windows client IP settings and verify the current lease before using a source-restricted firewall rule. Use the existing Windows 10 or Windows 11 Education, Pro, or Enterprise client VM and keep its current computer name. Identify it by that name in DHCP, WSUS, and submitted evidence. Domain members must query the lab AD DNS server. For Class 3, record the VMware and guest settings before temporarily switching the DHCP service on the NAT VMnet. Use VM consoles for address or firmware changes. Where snapshots are suitable, take a new checkpoint of the current disposable member/client VM state. Use the instructor's AD backup and recovery procedure for domain controllers.

### Student worksheet

Record your VMware NAT VMnet, actual IPv4 subnet and mask, AD DNS name, and Windows client computer name before starting. Substitute your own values for every `192.168.X.*` and `corp.example` example. In Classes 2–4, confirm the Windows client can resolve the AD domain through the lab DNS server.

## Trace an event and test access to a Windows container host

**Before you start.** Open elevated PowerShell on ServerDM2 and on the Windows client. Confirm both VMs are on the Lab 1 NAT VMnet. On the client, run `ipconfig` and record its current IPv4 address for the firewall rule. On ServerDM2, run `Get-NetTCPConnection -LocalPort 8080 -State Listen -ErrorAction SilentlyContinue`. If there is already a listener, have the instructor assign an alternate port and use it throughout.

1. On ServerDM2, produce a controlled Application event. Verify its timestamp, provider and message in Event Viewer and PowerShell.

```powershell
eventcreate /T INFORMATION /ID 100 /L APPLICATION /SO CST89200 /D "Lab 1 baseline"
Get-WinEvent -FilterHashtable @{
  LogName='Application'; ProviderName='CST89200'; Id=100
  StartTime=(Get-Date).AddMinutes(-15)
} -MaxEvents 5 | Select-Object TimeCreated, ProviderName, Id, Message
```

2. On **ServerDM2**, install Docker Engine and run your first Windows container.

   **Enable the Windows Containers feature.** On ServerDM2, open Windows PowerShell as Administrator and run:

   ```powershell
   Install-WindowsFeature -Name Containers
   ```

   Check that the command reports success. If **Restart Needed** is **Yes**, restart ServerDM2 from its VMware console. Sign back in and reopen Windows PowerShell as Administrator. Run `Get-WindowsFeature -Name Containers` and confirm the feature is installed before continuing.

   **Install Docker Engine (Moby).** ServerDM2 needs internet access to download the installer, engine and image. Download and run the Microsoft Windows Containers setup script:

   ```powershell
   Set-Location $env:TEMP
   $installerUrl = 'https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1'
   Invoke-WebRequest -UseBasicParsing -Uri $installerUrl -OutFile '.\install-docker-ce.ps1' -ErrorAction Stop
   .\install-docker-ce.ps1 -NoRestart
   ```

   After setup completes, open a fresh elevated PowerShell window on ServerDM2. Verify the Docker service and engine:

   ```powershell
   Get-Service -Name docker
   docker version
   ```

   Expect the service status to be **Running** and `docker version` to show both **Client** and **Server**. If either check fails, record the installer output or service error and resolve it before pulling the image.

   **Pull and run the Windows Server Core 2019 image.** Run these commands on ServerDM2:

   ```powershell
   hostname
   docker pull mcr.microsoft.com/windows/servercore:ltsc2019
   docker image ls mcr.microsoft.com/windows/servercore
   docker run --rm --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd /c hostname
   ```

   Confirm the image list shows the `ltsc2019` tag. The first `hostname` prints the host name. The last command prints the container name and exits. Record both outputs. If the pull fails, check ServerDM2’s DNS and internet access to `mcr.microsoft.com`.

3. Use the following host-side listener to isolate the firewall test from container networking quirks. Keep this PowerShell window open while testing. It accepts TCP handshakes while running.

```powershell
$listener = [System.Net.Sockets.TcpListener]::new([System.Net.IPAddress]::Any,8080)
$listener.Start()
```

4. From the Windows client, run `Test-NetConnection 192.168.X.21 -Port 8080` before the lab allow rule, after adding it, and after removing it. Keep the listener running for all three tests. In a second elevated ServerDM2 window, run the first two lines below. Test from the client, run the removal command on ServerDM2, then test again from the client. Replace the example value of $clientIp with the Windows client’s current IPv4 address from `ipconfig`. If the baseline already succeeds, inspect existing active rules and the current firewall profile with the instructor.

```powershell
$clientIp = '<the Windows client IPv4 address>'
New-NetFirewallRule -Name CST89200-8080 -DisplayName "CST 89200 TCP 8080" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8080 -RemoteAddress $clientIp -Profile Any
# After the successful client test:
Remove-NetFirewallRule -Name CST89200-8080
```

5. Stop the listener with `$listener.Stop()` in its original window.

**Submit.** Filtered event output, Docker service and version output, the pulled image tag, host and container output, and the three client test results with an explanation. Expected on the prepared baseline is False → True → False.  
**Troubleshoot.** The listener works locally, but the client test fails with the allow rule enabled. What two observations would distinguish a rule scope issue from a network path issue?

## Submission checklist

Event provider, ID and timestamp. Host and container command output. Three client TCP results labelled before, during and after the allow rule.

Label each item with the VM name. Include your answer to the lab’s troubleshooting question. Identify any instructor-provided trace or report and explain what it shows.
