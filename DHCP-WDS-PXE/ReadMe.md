# CST 8200 Lab 6 · DHCP, WDS, and PXE

## What you will build

- **DHCP.** Install and authorize a DHCP server, configure a scope with an exclusion and a client-specific policy, and prove that your Windows client receives the correct lease and retains its reserved address after renewal.
- **WDS and PXE.** Install and configure WDS, import a boot image and an install image, then deploy Windows Server 2019 over the network to a blank VMware VM. Verify the installed server boots from its virtual disk.

## Before you start

Use the existing **ServerDC1**, **ServerDM1** and **Windows 10 or Windows 11 client** on your dedicated VMware NAT VMnet. Create one blank VM for deployment named **WDS-Target**. Keep your existing client and servers available for the DHCP exercises. Use the VMware console throughout the lab.

Record the NAT VMnet name, actual subnet and mask, NAT gateway, VMware DHCP setting, your AD DNS domain and the Windows client's current IP and DNS configuration. Record whether IP assignment and DNS assignment are each automatic or manual. Verify ServerDC1 and ServerDM1 have static addresses and ServerDM1 is domain joined and uses ServerDC1 for DNS. Use a lab account authorized to install roles and authorize DHCP in AD.

Use an x64 Windows Server 2019 ISO with both `sources\boot.wim` and `sources\install.wim`. Mount it on **ServerDM1** and record its drive letter. Plan for sufficient free space for both WIM files and a blank target disk of at least 60 GB. Use the course-provided Windows Server 2019 edition and media.

## Part A · Install and test DHCP

1. On **ServerDM1**, open **Server Manager → Manage → Add Roles and Features**. Choose **Role-based or feature-based installation**, select ServerDM1, select **DHCP Server**, accept its management tools, and complete the installation. In Server Manager, select **Complete DHCP configuration** in the notification flag and authorize the server with your assigned lab credentials. Open **Tools → DHCP**, refresh, and confirm the server is authorized. If authorization is unavailable, check domain DNS and your assigned permissions before continuing.
2. In **DHCP Manager → ServerDM1 → IPv4**, create a scope named **Deployment Lab LAN**. Use the actual NAT subnet mask. For a /24 lab, use `192.168.X.100` through `192.168.X.149` and exclude `192.168.X.140–192.168.X.149`. Replace X with the actual third octet. Check the pool and exclusion against the existing VMnet configuration and static IPs. If your mask differs, use an instructor-approved range in the same subnet. Configure scope options **003 Router** with the VMware NAT gateway, **006 DNS Servers** with ServerDC1, and **015 DNS Domain Name** with your AD DNS domain. Leave the scope inactive until the next step.
3. On the VMware host, open **Virtual Network Editor**, select your dedicated NAT VMnet and clear **Use local DHCP service to distribute IP address to VMs**. Keep the network in NAT mode. If the VMnet is shared, use an instructor-assigned isolated VMnet instead. Then, on **ServerDM1**, right-click **Deployment Lab LAN** and select **Activate**.
4. On the **Windows client**, use automatic IPv4 and DNS. If Wireshark is available, start a capture on the active VM adapter before renewing. Run the commands below and verify the DHCP server is ServerDM1, the address is inside the available pool, and the gateway and DNS server match the scope. In DHCP Manager, match the client to its entry under **Address Leases**. A capture can show Discover, Offer, Request and Acknowledge. If it contains only Request and Acknowledge, label the exchange as a renewal.

   ```powershell
   ipconfig /release
   ipconfig /renew
   ipconfig /all
   ```

5. In **DHCP Manager → Deployment Lab LAN → Policies**, create a policy named **Client Suffix Test**. Match the Windows client's **MAC Address** using the Physical Address from `ipconfig /all` on the active adapter. Use the same scope pool and set policy option **015 DNS Domain Name** to `dhcp-test.example`. Leave router and DNS server options inherited. Release and renew on the client, then record the connection-specific DNS suffix from `ipconfig /all`. Disable the test policy, release and renew again, and verify the original lab AD suffix returns before continuing. Keep the client joined to its existing domain.
6. Under **Address Leases**, right-click the Windows client's active lease and select **Add to Reservation**. Confirm its IP address and client identifier under **Reservations**. On the Windows client, release and renew again. Verify that it receives the reserved address from ServerDM1. Leave the Windows DHCP scope active for Part B.

## Part B · Configure WDS and deploy a server

1. On **ServerDM1**, use **Add Roles and Features** to install **Windows Deployment Services** with both **Deployment Server** and **Transport Server**. Open **Server Manager → Tools → Windows Deployment Services**. Under **Servers**, right-click ServerDM1 and choose **Configure Server**.
2. Choose **Integrated with Active Directory**. Select a local NTFS path for **RemoteInstall** with room for the images. Follow the wizard's same-server DHCP settings. In **ServerDM1 → Properties → DHCP**, verify **Do not listen on DHCP ports** and **Configure DHCP options to indicate that this is also a PXE server**. These settings allow DHCP to listen on UDP 67 while advertising the WDS PXE service.
3. In the **PXE Response** settings, allow responses to the lab target on your isolated VMnet. If you select the option to require administrator approval for unknown clients, approve **WDS-Target** under **Pending Devices** when it network-boots. Use interactive deployment and keep WDS hands-free deployment disabled.
4. Under **Boot Images**, select **Add Boot Image**. Browse to `sources\boot.wim` on the mounted Windows Server 2019 ISO and name the image **Server 2019 Setup**. Under **Install Images**, create an image group named **Server 2019 Lab**. Use **Add Install Image** to import `sources\install.wim` and select the Windows Server 2019 edition specified for class. Verify that both images are listed, then start WDS from the server's **All Tasks** menu if it is stopped.
5. In VMware Workstation, create **WDS-Target** as an x64 Windows Server 2019 VM. Choose **I will install the operating system later**, create a new blank virtual disk and attach one virtual NIC to the dedicated lab VMnet. Use UEFI firmware and a NIC that supports PXE. Leave the target's virtual CD/DVD disconnected so that the deployment comes from the network.
6. Power **WDS-Target** on to firmware and choose its network adapter as the one-time boot device. Respond to the PXE prompt, approve the device if required, and choose **Server 2019 Setup**. When Windows Setup opens, follow the interactive prompts. If prompted to authenticate, use your assigned lab domain deployment account. Select the imported Windows Server 2019 edition and the **blank disk in WDS-Target**. Complete installation and let the VM boot from disk after the first restart. If it returns to PXE, choose its virtual disk from the firmware menu.
7. Complete first sign-in on **WDS-Target**. Run the commands below. Confirm it started from its virtual disk with the CD/DVD disconnected and that Windows reports the expected edition. Record the guest hostname shown by the command, which may differ from the VMware display name.

   ```powershell
   hostname
   Get-CimInstance Win32_OperatingSystem | Select-Object Caption, Version, BuildNumber
   ```

## Restore the lab network

1. Save your evidence and shut down **WDS-Target**.
2. On **ServerDM1**, stop the WDS service and deactivate **Deployment Lab LAN**. Confirm the Windows scope is inactive before restoring VMware DHCP.
3. Restore the original VMware DHCP setting on the NAT VMnet and the original IPv4 and DNS settings on the existing Windows client. Renew its lease if it normally uses DHCP. Verify the client reaches ServerDC1 and resolves the lab AD domain.

## What to submit

- **DHCP:** Scope range, exclusion and options, the lease from ServerDM1, evidence that the policy changed the DNS suffix and that disabling it restored the original value, and the reserved IP before and after renewal. Include the Wireshark exchange if captured.
- **WDS:** Boot and install images listed on ServerDM1, evidence of PXE or Windows Setup on WDS-Target, and the hostname and Windows edition after it boots from disk.
- **Cleanup:** Evidence of the deactivated Windows DHCP scope and restored VMware and client network settings.

If the target gets a DHCP address but never reaches Setup, record the last screen or error. Check WDS service state, the PXE response or approval policy, the NIC and firmware mode, and the boot image. If Setup opens but shows no install image, check the imported edition, image group, and deployment account. Report the phase reached and evidence collected if a blocker prevents completion.
