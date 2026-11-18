#  Advanced DNS and Domain Migration

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

## Validate DNS and rehearse a domain controller cutover

**Before you start.** Confirm with the instructor whether ServerDC2 serves the same AD domain as ServerDC1. If it serves a separate domain or is unavailable, use instructor-provided outputs for the same-domain replication, cutover and rollback steps. Complete the DNS forwarding task and cross-domain plan.

1. On ServerDC1, record the current domain controllers, FSMO owners and DNS baseline. Run `dcdiag /test:DNS` and investigate any unexpected failures before proceeding. Use `Get-ADDomainController -Filter *`, `netdom query fsmo`, and the following DNS query.

```powershell
Resolve-DnsName _ldap._tcp.dc._msdcs.corp.example -Type SRV -Server 192.168.X.10
```

2. On ServerDC1, use DNS Manager to add a conditional forwarder for `partner.example` to ServerDM2 (`192.168.X.21`), where the instructor has staged an authoritative test zone and an A record for `app.partner.example`. Run `Resolve-DnsName app.partner.example -Type A -Server 192.168.X.21`, then repeat with `-Server 192.168.X.10`. Expect the same A-record address. If only the direct query succeeds, verify the conditional forwarder’s suffix, target, and connectivity. If the test DNS zone on ServerDM2 is unavailable, use its captured output as the forwarding exercise and record the substitution.
3. Verify whether ServerDC2 is already a DNS server and global catalog in the existing domain. If it is unassigned, the instructor can prepare it for that role before class. If ServerDC2 already hosts a separate lab domain, keep it in that domain and use instructor-provided replication evidence for this same-domain exercise. Do not redirect the Windows client’s AD DNS to a DC from a different domain for the pilot. When both controllers serve the existing domain, verify them in AD, run `repadmin /replsummary`, `repadmin /showrepl ServerDC2` and `dcdiag /s:ServerDC2 /test:DNS`, and check that ServerDC2 exposes SYSVOL and NETLOGON. Add a temporary A record `migration-test.corp.example` on ServerDC1 and verify it arrives on ServerDC2 without manually recreating it. Resolve the AD SRV set directly against ServerDC2 and confirm ServerDC2 registers as a target.
4. If both DCs serve the existing domain, record the Windows client’s current DNS configuration. With the separate-domain fallback, use instructor-provided cutover output to complete steps 4 and 5 and document the expected rollback. For the live exercise, change the Windows client’s active adapter DNS server list to ServerDC2 for the pilot and clear the DNS client cache with `Clear-DnsClientCache`. Run `Resolve-DnsName portal.corp.example`, `nltest /dsgetdc:corp.example /force`, `Test-ComputerSecureChannel -Verbose` and access the IIS lab page using its DNS name. Record the DC returned by discovery. DC Locator can still select ServerDC1 while both controllers are online. This step verifies DNS cutover and domain connectivity; the instructor separately demonstrates domain logon with ServerDC1 unavailable in a resettable copy after checking dependencies.
5. For the live exercise, restore the Windows client’s original DNS mode and server list. Clear its DNS cache and confirm the IIS hostname still resolves. Remove the temporary A record and any conditional forwarder you created solely for the exercise. For the production-style exercise, write the retirement sequence covering FSMO transfer where required, DNS client and DHCP option updates, dependent service checks, time source verification if the PDC role moves, and graceful demotion after the acceptance window.
6. In pairs, write a plan to move ten pilot users and two workstations from `corp.example` into a separate AD domain. Include naming and UPN mapping, DNS and trust requirements, tooling support validation, file access and service account checks, user profiles, a pilot acceptance checklist, and the decision point to pause or roll back before retiring source access.

**Submit.** DNS forwarding and replication evidence, pilot DNS change and rollback results, and the cross-domain pilot plan.  
**Troubleshoot.** ServerDC2 resolves the IIS record, but clients fail to discover the domain through it. Which records and replication and service checks would you prioritize?

## Submission checklist

Direct and forwarded DNS answers, replication and client DNS cutover and rollback results or assigned outputs, and the cross-domain pilot plan.

Label each item with the VM name. Include your answer to the lab’s troubleshooting question. Identify any instructor-provided trace or report and explain what it shows.
