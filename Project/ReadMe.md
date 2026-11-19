# CST 8200 Final Project
# From Startup to Managed Windows Infrastructure

**Course** · Windows Domain Administration  
**Professor** · Islam Gomaa  
**Work format** · Individual  
**Implementation period** · Two weeks from the published release date  
**Assessment** · Working environment, submitted demonstration video, technical handover and five-minute individual presentation  

## Your assignment

Northstar Services has hired you to organize and commission its Windows infrastructure. You must translate the company requirements into a working virtual environment, test the result and demonstrate it to management.

Build the environment yourself and be prepared to explain your configurations, scripts and troubleshooting decisions. Submit evidence from your own implementation. You may consult course materials and technical documentation. Acknowledge external sources and any adapted scripts.

The assignment defines the required outcomes. You select the OU structure, security groups, GPO links, server role placement and implementation methods, and explain your choices.

## The company

Northstar Services began as a small startup. Employees configured their computers individually, application installation required manual work, and shared folders accumulated inconsistent permissions.

The company now employs 40 people.

| Department | Employees | Business needs |
|---|---:|---|
| Human Resources | 6 | Confidential personnel documents and standard workstation settings |
| Finance | 8 | Restricted financial documents and controlled user settings |
| Sales | 20 | Shared sales documents and access to company information |
| IT | 6 | Administrative tools, documented configurations and repeatable deployment |
| **Total** | **40** | An organized and supportable Windows environment |

Management has identified these problems.

- Employee accounts and computers need a clear organizational structure.
- Staff should access departmental information according to their roles.
- Standard users need appropriate restrictions and the ability to install Notepad++ from Windows Control Panel.
- New devices need correct network settings and internal name resolution.
- IT needs to deploy Windows 11 to a blank workstation through the network.
- Employees need an internal website for company announcements and IT support information.
- IT needs evidence that the environment works and documentation another administrator can use.

Represent the company with **12 employee accounts, three per department**. Explain how your design accommodates all 40 employees and future growth.

## Environment and planning

Use VMware Workstation and Windows Server 2019 for server roles. You may reuse your existing server VMs and domain. Use the Windows 11 workstation you deploy through PXE for the employee acceptance tests.

Create a blank x64 VM with a new virtual disk, UEFI firmware, Secure Boot capability and a virtual TPM 2.0. Give it a clear VMware display name such as **NS-DeployTarget**. Record the deployment from the blank target and preserve the completed Windows 11 VM.

| Function | What your environment must provide |
|---|---|
| Identity and policy | AD DS, AD-integrated DNS and Group Policy management |
| File services | Departmental shares and an application distribution share |
| Networking and deployment | Authorized Windows DHCP, WDS and PXE |
| Intranet | IIS and a DNS hostname |
| Employee testing | The domain-joined Windows 11 workstation deployed through PXE |
| Deployment testing | A separate VM with a blank disk |

You may combine appropriate roles on existing servers. Explain the arrangement and any limitations. Document the VMs that must run together during testing and the presentation.

Use your dedicated virtual network for DHCP and PXE. Identify its subnet, mask and gateway from VMware Virtual Network Editor. Document how you prevent VMware DHCP and Windows DHCP from issuing competing leases on that network. Keep the exercise confined to your own virtual environment.

Use your existing AD DNS domain or choose a unique project domain such as `ns1234.example`. Use actual values consistently in your configurations and documentation. Record server names, IP addresses, DNS servers, role placement and test account names.

## Required builds and acceptance tests

### 1. Design the Active Directory structure

Design an OU hierarchy that separates employee accounts, workstations, servers and administrative accounts where appropriate. Organize users by department and choose GPO links that match the intended scope.

Create the 12 employee accounts and suitable security groups. Standard employee accounts must operate with standard user permissions. Use a separate administrative account for administration.

**Acceptance tests**

- Present your OU structure and explain the purpose of each major OU.
- Identify one representative employee and the groups that grant access to their resources.
- Explain where a new employee and a new workstation belong.
- Demonstrate that a representative employee account has standard user privileges.

**Evidence** · OU structure, account and group inventory, representative group membership and administrative account separation.

### 2. Implement departmental file access

Create one SMB share or shared departmental folder for HR, Finance, Sales and IT, plus a shared company information area.

Use security groups to assign access. Department members need the ability to create and modify files in their own departmental folder. Standard employees must be unable to open any other department’s folder. Test IT access using a standard IT employee account. All employees must be able to read company information. Assign responsibility for maintaining that information to an appropriate group.

Explain how SMB share permissions and NTFS permissions combine in your design.

**Acceptance tests**

- Sign in with one standard employee account from each department. For each account, create or modify a test file in its own department’s folder and demonstrate access denied when opening each of the other three department folders. Record the four successful tests and all twelve denied tests.
- An employee reads a file in the company information area.
- A user without publishing permission cannot modify the company information file.

**Evidence** · Permission matrix, share and NTFS configuration, and successful and denied access tests with the test identities identified.

### 3. Apply targeted Group Policy

Implement policies that meet all of the following requirements.

| Requirement | Expected result |
|---|---|
| User settings restriction | HR, Finance and Sales employees can open **Programs and Features** and reach **Install a program from the network**. Restrict access to other Control Panel items and demonstrate at least one blocked settings area. |
| Registry tools restriction | HR, Finance and Sales employees cannot use Registry Editor |
| IT exception | The preceding employee restrictions do not apply to the designated IT administrative account |
| Computer configuration | Employee workstations display a company logon notice |
| User convenience | A departmental drive mapping points the employee to their own departmental share |

Use OU placement, policy links and security filtering as appropriate. Record which settings are under User Configuration and which are under Computer Configuration. Explain the intended scope and any inheritance decisions.

**Acceptance tests**

- Sign in as a standard employee and demonstrate a blocked settings area, the Registry Editor restriction and access to the network program catalog in Control Panel.
- Show the departmental drive mapping and open the intended share.
- Show the workstation logon notice.
- Demonstrate the IT exception with the designated administrative account.
- Generate an applied Group Policy report and explain which GPO supplies each demonstrated setting.

**Evidence** · GPO inventory with scope, selected settings, the applied policy report and the user tests.

### 4. Publish Notepad++ for self-service installation

Publish **Notepad++** so a standard employee can install it from Control Panel on the Windows 11 workstation deployed in Section 6. Download the official x64 MSI from the [Notepad++ releases page](https://github.com/notepad-plus-plus/notepad-plus-plus/releases). Record the version, installer filename and release URL.

Place the MSI on a UNC distribution share and grant the targeted employees read access through both SMB and NTFS permissions. Use the full UNC path to add the MSI under **User Configuration → Policies → Software Settings → Software installation**. Choose **Published** and link the GPO to the intended employee user OU or OUs. Keep the **Get Programs** task available to those employees when applying the Control Panel restrictions in Section 3.

**Acceptance tests**

- Sign in to the PXE-deployed Windows 11 workstation as a targeted standard employee. Verify that Notepad++ is absent and show that the publication GPO applies to the employee.
- Open **Control Panel → Programs → Programs and Features → Install a program from the network**. Select **Notepad++** and install it using the standard employee account without entering administrator credentials.
- Launch Notepad++ and confirm the installed version.

**Evidence** · Installer source and version, UNC path, share and NTFS permissions, publication settings and user scope, Control Panel listing, and before-and-after installation evidence.

### 5. Configure DHCP and DNS

Install and authorize Windows DHCP. Configure a scope appropriate to your virtual subnet, including an address range, an exclusion, a lease duration and options for the gateway, DNS server and AD DNS suffix.

Create one reservation for a test client. Keep static infrastructure addresses outside the dynamic allocation pool or protect them with exclusions.

Provide DNS records for the servers and intranet portal. Configure a reverse lookup zone and demonstrate a matching PTR record for at least one server. Verify AD service discovery through an SRV query.

**Acceptance tests**

- A client receives an address from Windows DHCP and the expected gateway, DNS server and suffix.
- Its DHCP server address identifies your Windows DHCP server.
- The reserved client receives its reserved address after release and renewal.
- The client resolves the intranet hostname to the intended server.
- A reverse lookup returns the documented server name.
- An AD SRV lookup returns domain controller service information.

**Evidence** · Scope, exclusion, options, lease and reservation, plus forward, reverse and SRV query results.

### 6. Deploy a Windows 11 workstation through WDS and PXE

Configure WDS to PXE-boot a custom x64 Windows PE image built with the Windows ADK and Windows PE add-on. Stage a licensed x64 Windows 11 Pro, Education or Enterprise installation image on a protected deployment share. The Windows 11 ISO's standard WDS setup workflow is unsupported. Use WDS to deliver Windows PE, then use DISM in Windows PE to apply the Windows 11 image from the share and BCDBoot to make the VM bootable from its virtual disk.

Document the boot image, Windows edition and image index. Confirm the DHCP and WDS co-hosting settings if the services share a server. Verify that Windows PE can access the VMware network and virtual disk. Document how deployment targets receive a PXE response or approval.

**Acceptance tests**

- Start the target with a blank disk, its virtual CD/DVD disconnected and the network adapter selected for boot.
- Show that the VM receives a DHCP lease and boots the custom Windows PE image from WDS over PXE.
- In Windows PE, connect to the deployment share, prepare the virtual disk for UEFI boot, apply Windows 11 and create the boot files.
- Restart from the virtual disk, complete Windows setup and verify the installed Windows 11 edition.
- Rename the workstation, join it to the project domain and place its computer account in the intended workstation OU.
- Sign in as a standard employee. Use this workstation to install Notepad++ from Control Panel and open the IIS intranet by hostname.

**Evidence** · WDS configuration, Windows PE boot image, Windows 11 image and selected index, blank target and network boot, image application, local disk boot, installed OS details and domain membership.

Record the deployment in the submitted video, including the initial network boot from a blank disk and the final Windows 11 boot from disk. You may trim installation wait time while keeping both stages and the image application visible.

### 7. Publish an IIS employee intranet

Install IIS and publish a company intranet using a named site and a DNS hostname such as `intranet.ns1234.example`, adjusted to your domain.

The site must contain a company welcome message, department information and IT support instructions. Use fictional information. A simple static site is sufficient.

Configure the site binding, content location and required read permissions. Permit access through the appropriate Windows Firewall rule and keep a record of the rule's scope. Enable IIS logging.

**Acceptance tests**

- A standard employee on the PXE-deployed, domain-joined Windows 11 workstation resolves the intranet hostname and opens the IIS site using that name.
- The displayed page contains your company information.
- A request for an existing page produces HTTP 200 in the IIS log.
- A request for a deliberately missing page produces HTTP 404 in the IIS log.
- You identify the client address and requested path in the relevant log entries.

**Evidence** · DNS record, IIS binding and site configuration, firewall rule, browser result and the matching log entries.

### 8. Produce a PowerShell handover tool

Write a PowerShell script that assists administration of your environment. Choose one of these purposes.

- Create representative employee accounts and group memberships from a CSV input.
- Check the infrastructure and produce a readable status report covering at least three relevant services or functions, such as AD DNS resolution, DHCP service state and the IIS endpoint.

Make the inputs and output understandable. Explain the commands and report errors clearly. If your script changes objects, account for running it again without creating duplicate objects.

**Acceptance tests**

- Run the script and explain its inputs and output.
- Show that its result matches the environment.
- Demonstrate or explain its handling of an unavailable service or an existing object, according to the script's purpose.

**Evidence** · Script, any input file, sample output and a short explanation of its behavior.

## Cumulative evidence and technical handover

Maintain **one cumulative evidence record** throughout the two weeks. Use the acceptance test table in your report as the index. Include dated screenshots, command output, reports or logs in the PDF and reference them by figure or appendix number. Use the same test ID for each attempt and keep the earlier entries when you retest. Record what changed and link the new result to the original failure or configuration.

| Test ID | Date | Requirement | Machine and test identity | Expected result | Actual result | Status | Evidence reference |
|---|---|---|---|---|---|---|---|

- **Baseline** · Record which VMs and services already exist, what works and what still needs configuration or testing.
- **Build and retest** · Add evidence as each service is configured and tested. Include the required allowed and denied access tests. For a failed test, record the symptom, fix and retest result under the same test ID.
- **Final infrastructure status** · Add a dated summary near the beginning of your report. Give each area a row with its status (**Passed**, **Failed** or **Not tested**), a short result and references to the latest relevant tests. If some required tests are incomplete, list them in that row.

The final summary must cover AD DS, departmental file access, Group Policy, DHCP, DNS, WDS/PXE and Windows 11 domain join, Notepad++ self-service installation, the IIS intranet and the PowerShell handover tool. Reference the four required video demonstrations in the cumulative test table. Include the start time for each demonstration so the assessor can find it from the final status summary.

Document at least **two issues encountered during the project**. For each, describe the symptom, evidence, cause, corrective action and verification. If your build completes without two faults, perform and document two reversible tests, such as an incorrect test DNS record or an intentionally incorrect test-group membership, then restore the correct configuration.

Create a handover another administrator can follow. Include the network diagram, VM and IP inventory, AD and group design, GPO scope, access matrix, application package details, deployment image details and service verification results. Explain how to onboard an employee, deploy a new machine and restore the normal DHCP arrangement after demonstration.

## What to submit

Submit two files through the course submission location.

| File | Required content |
|---|---|
| **Final project report (.pdf)** | Write the report in Microsoft Word as a regular `.docx` document, then export it to PDF for submission. Include your name and student ID, company name, project overview, final infrastructure status, design decisions, network diagram, inventories, access matrix, GPO scope, implementation summary, cumulative test evidence and troubleshooting record. Include the PowerShell script, any sample input and sample output in an appendix. Include the start time for each video demonstration. |
| **Demonstration video (.mp4)** | Submit one screen recording with the four demonstrations listed below. |

Keep credentials out of the PDF and video. Preserve your working VMs and image configuration until the individual assessment is complete.

## Required demonstration video

Submit one screen recording with the four demonstrations below. In your PDF report, list the start times for each demonstration. Keep account names, folder paths and results readable. Hide passwords and other credentials.

| Demonstration | What the recording must show |
|---|---|
| Windows 11 deployment via PXE | Start a VM with a blank disk and its virtual CD/DVD disconnected. Show the WDS network boot into Windows PE, Windows 11 image application, first boot from the virtual disk and domain membership. |
| Notepad++ installation from Control Panel | Sign in to the deployed Windows 11 workstation as a standard employee. Show Notepad++ is absent, then open **Control Panel → Programs → Programs and Features → Install a program from the network**. Select Notepad++, install it without entering administrator credentials, and launch it. |
| Department folder permissions | Show the account name for one standard user from each of HR, Finance, Sales and IT. For each account, show a successful write to its own folder and **Access denied** when attempting to open each of the other three folders. |
| IIS intranet access via DNS | As a standard employee on the deployed Windows 11 workstation, show the intranet hostname resolving to the IIS server. Open the site in a browser using that hostname and show the page loading. |

## Five-minute individual presentation

You have **five minutes** to show your work. Briefly explain your design, use the final infrastructure status summary to show what works, and highlight a result from your implementation. The submitted video contains the detailed demonstrations. Have your timestamped evidence available for reference during the presentation.

## Marking rubric

| Category | Points | Full-credit evidence |
|---|---:|---|
| Design and Active Directory organization | 15 | Coherent topology and role placement, purposeful OUs, 12 employee accounts, suitable groups and separate administration |
| File access and targeted GPOs | 20 | Correct permissions, video showing own-folder write access and all cross-department denials, both employee restrictions, IT exception, logon notice and drive mapping |
| Group Policy software deployment | 10 | Official Notepad++ x64 MSI, readable UNC distribution point, correct user scope, video showing installation from Control Panel without administrator credentials and successful launch |
| DHCP and DNS | 10 | Authorized DHCP, appropriate scope and options, working reservation, and successful forward, reverse and SRV tests |
| WDS and PXE deployment | 15 | Video showing PXE boot into custom Windows PE, Windows 11 image application, local disk boot and domain membership |
| IIS intranet | 10 | Functional named site on IIS, DNS and firewall configuration, video showing hostname resolution and site access from the PXE-deployed workstation, and matching HTTP 200 and 404 log evidence |
| PowerShell tool and technical handover | 10 | Understandable script, useful verified output, cumulative test evidence, final status for each area and troubleshooting documentation |
| Individual presentation | 10 | Clear five-minute overview, accurate explanations and ability to interpret the environment's behavior |
| **Total** | **100** | |

Each category receives credit for the portion you implement, verify and explain. Record incomplete requirements clearly so the assessor can distinguish a configuration issue from missing evidence.

## Completion checklist

- [ ] The OU, account and group structure supports the company's departments.
- [ ] Departmental permissions pass both allowed and denied access tests.
- [ ] The required user restrictions, IT exception, logon notice and drive mapping work.
- [ ] A standard employee installs the Group Policy-published Notepad++ from Control Panel on the PXE-deployed Windows 11 workstation and launches it.
- [ ] DHCP supplies the intended settings and the reservation survives renewal.
- [ ] DNS forward, reverse and AD SRV queries work.
- [ ] A blank target PXE-boots into Windows PE through WDS, receives Windows 11, boots from disk and joins the domain.
- [ ] The IIS intranet opens by hostname on the PXE-deployed Windows 11 workstation and produces the required log entries.
- [ ] The PowerShell tool works and you can explain it.
- [ ] The PDF report contains a dated final infrastructure status summary and references to the cumulative evidence.
- [ ] Your submitted video covers all four demonstrations, its timestamps are in the PDF report and your individual presentation fits within five minutes.

## Technical reference links

- [Group Policy software installation](https://learn.microsoft.com/en-us/troubleshoot/windows-server/group-policy/use-group-policy-to-install-software)
- [WDS boot image compatibility](https://learn.microsoft.com/en-us/windows/deployment/wds-boot-support)
- [IIS site bindings](https://learn.microsoft.com/en-us/iis/configuration/system.applicationhost/sites/site/bindings/)

- [Customize Windows PE boot images](https://learn.microsoft.com/en-us/windows/deployment/customize-boot-image)
