# CST 89200 Lab  · DHCP, WDS, and PXE

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

## Lease an address and PXE boot the Windows client VM

**Before you start.** Open the Windows client through the VMware console. Record whether its IPv4 address and DNS settings are automatic or manual. Confirm the NAT VMnet is dedicated to the exercise and that the instructor has validated an interactive boot. Keep the existing VM firmware type, virtual disk and WDS security settings. Leave hands-free deployment disabled.

1. Record the NAT VMnet name, subnet mask, gateway, VMware DHCP settings, and the Windows client's IPv4/DNS configuration. Verify ServerDC1 and ServerDM2 have static addresses and that ServerDM2 can resolve the AD domain. On ServerDM2, verify DHCP and WDS are staged. Authorize the DHCP server in AD if required, but leave the Windows DHCP scope inactive for now.
2. In VMware Virtual Network Editor, select the NAT VMnet from Lab 1 and temporarily clear **Use local DHCP service to distribute IP address to VMs** on that VMnet. Keep the VMnet in NAT mode. Pause other DHCP-dependent VMs on that VMnet for the exercise. If other workloads need that VMware DHCP service, use the instructor's isolated VMware network and address plan instead. Enable only one DHCP service on the exercise network at a time.
3. On ServerDM2, create and activate the IPv4 scope with the actual NAT subnet mask. On a /24 network, use `192.168.X.100–192.168.X.149` after checking for conflicts with static addresses and NAT infrastructure. For a different mask, use the instructor-approved range in that subnet. Set option 006 to `192.168.X.10`, 015 to the existing AD DNS name and 003 to the recorded VMware NAT gateway.
4. On the Windows client, start Wireshark with the display filter `dhcp`. With IPv4 and DNS set to automatic, run `ipconfig /release` and `ipconfig /renew` from its console. Verify the lease and options in `ipconfig /all` and match the client ID in DHCP Manager. In DHCP Manager, open the lab scope’s **Address Leases**, right-click the active client lease, and select **Add to Reservation**. Verify it under **Reservations**, renew on the client, and confirm the same address. In `ipconfig /all`, verify that DHCP comes from ServerDM2 and DNS points to ServerDC1. If a different DHCP server appears, stop and check the selected VMnet and VMware DHCP setting before continuing. If the capture shows only Request and ACK, record that observation and use the initial exchange from the instructor's trace for the full sequence.
5. On ServerDM2, verify the staged x64 Windows Server 2019 boot image. Since DHCP and WDS are on the same server, check **Do not listen on DHCP ports** and **Configure DHCP options to indicate that this is also a PXE server** in WDS server properties. Verify the PXE response policy and use an interactive boot prompt. The instructor should confirm that the lab's existing Windows client firmware and NIC combination can boot the staged image.
6. Shut down the Windows client. In VMware Workstation, power it on to firmware and choose the virtual network adapter as a one-time boot device. Respond to the network boot prompt and select the staged image if prompted. Use the staged Windows Server 2019 boot image with either client OS. Stop at Windows PE or the first Windows Setup screen without starting an installation. Power off from VMware Workstation and boot normally from the virtual disk. If the existing VM cannot PXE boot, use the instructor's recorded boot trace for that portion and submit your own live DHCP evidence.
7. On ServerDM2, deactivate the lab Windows DHCP scope and confirm it is inactive. Stop the lab WDS service if you started it for this exercise. Then restore the VMware DHCP setting on the NAT VMnet. Restore the Windows client's original IPv4/DNS settings, renew its VMware lease if applicable, and verify AD DNS and client connectivity using the original lab configuration.

**Submit.** DHCP manager lease and reservation evidence, the Wireshark exchange, PXE/Windows PE evidence, and confirmation that VMware DHCP and the original Windows client DNS settings were restored.  
**Troubleshoot.** the Windows client receives a valid address in firmware but cannot download the boot program. Identify the completed phase and the next three checks.

## Submission checklist

Lease options, Wireshark exchange, reservation, PXE result or assigned trace, and confirmation that the original network settings were restored.

Label each item with the VM name. Include your answer to the lab’s troubleshooting question. Identify any instructor-provided trace or report and explain what it shows.
