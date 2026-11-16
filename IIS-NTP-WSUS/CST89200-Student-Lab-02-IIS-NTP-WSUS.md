# CST 89200 Lab 2 · IIS, NTP, and WSUS

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

## Publish a site verify domain time and stage an update

**Before you start.** Verify that IIS and WSUS are staged on ServerDM1 and that the instructor has assigned an applicable update and a pilot computer policy. The Windows client must be domain joined and must resolve ServerDM1 using the lab AD DNS server. Check that the portal hostname and IIS binding are available before creating them.

1. On ServerDM1, verify the IIS Web Server role and management console. Create `C:\CST89200\Web` with an `index.html` containing a unique lab marker. Keep inherited permissions if the page is readable. If IIS returns 401.3, check the configured Anonymous Authentication identity and verify its read access to the folder and file. In IIS Manager, add site **CST89200**, HTTP port 80 and host name `portal.corp.example`. Use the folder as its physical path. Add an A record for `portal` on ServerDC1 pointing to `192.168.X.20`. Verify the IIS HTTP inbound rule permits the Windows client.
2. On the Windows client, run `Resolve-DnsName portal.corp.example` and request `http://portal.corp.example/index.html` and `http://portal.corp.example/missing.html`. On ServerDM1, find the IIS site ID in IIS Manager and view the matching W3C log under `%SystemDrive%\inetpub\logs\LogFiles\W3SVC<site ID>`. Allow time for the log to flush. Record the path and W3C status for each request. Expect 200 and your lab marker for index.html, and 404 for missing.html. If DNS fails, fix it first. If DNS succeeds and the connection fails, run `Test-NetConnection <portal-FQDN> -Port 80` from the client.
3. On the Windows client, run `w32tm /query /source`, `w32tm /query /status` and `w32tm /query /configuration`. Compare the source with ServerDC1 and run `w32tm /stripchart /computer:ServerDC1.corp.example /samples:5 /dataonly`. On ServerDC1, identify where the forest-root PDC emulator role is held using `(Get-ADDomain -Identity (Get-ADForest).RootDomain).PDCEmulator`. Explain the reported source and offset without changing system clocks.
4. In the preconfigured WSUS console on ServerDM1, verify that the Windows client is reporting to the server. Use the instructor’s OU-linked GPO and confirm on the Windows client using `gpresult /r /scope computer`. The intranet update and statistics URLs must match the lab WSUS endpoint. For the isolated HTTP lab endpoint, use `http://ServerDM1.corp.example:8530` when that is the configured binding. Create or select the **CST89200-Pilot** group and confirm server-side targeting is configured before assigning the Windows client in the console.
5. Approve one instructor-selected, applicable update for installation to the pilot group. Check that broader groups have no install approval for that update. Use the instructor-approved managed scan or observe the scheduled scan. Use the configured WSUS source for your evidence and avoid the separate Microsoft Update online scan. Record the approval and reporting states separately. If the next client report is pending when class ends, record the previous contact time and the checks completed.

**After the lab.** Leave the IIS site and DNS record available for Class 4. The instructor resets the pilot approval and assignment after collecting evidence.

**Submit.** DNS and IIS evidence for both requests, a comparison of time sources, and the update approval and client status.  
**Troubleshoot.** the Windows client appears in WSUS, but the selected update shows as not applicable. Which factors would you check before changing approvals?

## Submission checklist

DNS result and IIS statuses for both requests. Client and DC time sources with an offset interpretation. WSUS pilot membership, approval and reporting state.

Label each item with the VM name. Include your answer to the lab’s troubleshooting question. Identify any instructor-provided trace or report and explain what it shows.
