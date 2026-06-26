# Active-directory-lab
Active Directory home lab documenting a Windows Server 2022 domain controller, Windows 10 domain client, DNS/network configuration, domain user management, account lockout GPOs, department shared drives, mapped drives, and troubleshooting.
## Project Status

In progress. Core Active Directory setup, domain user creation, client domain join, account lockout policy, shared folders, and mapped drives have been configured. Screenshots and final documentation are still being added.

## Lab Environment

- VirtualBox
- Windows Server 2022
- Windows 10
- Active Directory Domain Services
- Group Policy Management
- Host-only networking
- NAT adapter for internet access

## What This Lab Demonstrates

- Creating and promoting a Windows Server 2022 domain controller
- Joining a Windows 10 client to an Active Directory domain
- Creating and managing domain users and groups
- Configuring account lockout policies with Group Policy
- Creating department-based shared folders
- Mapping network drives based on group membership
- Troubleshooting DNS, permissions, and drive mapping issues

1. Lab Preparation

**To prepare for this lab, I downloaded VirtualBox to use as my virtualization platform for the different roles in this environment. I went to Microsoft’s website and downloaded the ISO file for Windows Server 2022, and then used Microsoft’s media creation tool to create an ISO file for Windows 10.**

Screenshot placeholder: VirtualBox download, Windows Server 2022 ISO, and Windows 10 ISO/media creation tool.

2. Creating the Domain Controller VM

**To begin, I started by creating the domain controller for this lab. I specified hardware requirements that I believed would be sufficient for the domain controller, but also kept in mind that I needed to save resources for my actual desktop and the end-user machines that would eventually join the domain.**

Screenshot placeholder: VirtualBox VM creation and hardware/resource settings for the domain controller.

3. Installing Active Directory Domain Services

**Once the initial setup of Windows Server 2022 was completed and the server booted up, I began the process of installing Active Directory Domain Services.**

To do this, I went to:
Manage > Add Roles and Features > Server Roles > Active Directory Domain Services > select any desired features > Install

Once the installation completed, I selected:
Promote this server to a domain controller > Add a new forest > set the root domain name > set the Directory Services Restore Mode password > Install

For this lab, my domain is:
ad.lab.test

Screenshot placeholder: AD DS role installation and domain controller promotion.

4. Installing VirtualBox Guest Additions

**Next, I wanted to ensure I’d be able to have access to files stored on my PC - to do this I used a VirtualBox feature called Guest Additions. What it does is allows me to map a folder from my PC to the VMs I create so that I’m able to import/export files within my virtual environment.**

To install it, I selected:
Devices > Insert Guest Additions CD image > open File Explorer inside the VM > CD Drive (D:) VirtualBox Guest Additions > VBoxWindowsAdditions-amd64 > complete the installation process

Once the VM restarted, I went to:
Shared Folders > Shared Folders Settings > Add Share > map the desired folder

To verify the mapping was successful, I went back to File Explorer inside the VM, clicked This PC, and confirmed that the mapped location appeared beneath the local drives.

Screenshot placeholder: Guest Additions install and mapped shared folder verification.

5. Creating Domain Users

**Now that I had promoted my VM to a domain controller and installed Guest Additions, I created two users, John and Sarah Holmeleb, within Active Directory.**

First, from Server Manager, I clicked:
Tools > Active Directory Users and Computers

Then I selected my domain:
ad.lab.test

From there, I went to:
Users > right-click Users > New > User > fill out the required information

To verify the users were created correctly, I went back to the Users container and searched for the created users. Double-clicking a user account opens personalization and configuration options such as General, Address, Account, Profile, and other settings.

Right-clicking the user also provides commonly used options such as:
Add to a group
Disable account
Reset password
Move
Delete
Properties

Screenshot placeholder: Active Directory Users and Computers showing John and Sarah user accounts.

6. Creating the Windows 10 Client VM

For the client machines I decided to run Windows 10 primarily to reduce unnecessary variables/hardware requirements on my physical PC (TPM, Secure Boot, UEFI) - keeping the lab focused on Active Directory fundamentals.

To get everything set up, I went to the official Microsoft website and downloaded the media creation tool to create a Windows 10 ISO file. Then I created a new virtual machine in VirtualBox and completed the Windows 10 setup process.

Screenshot placeholder: Windows 10 VM creation and setup.

7. Configuring the Virtual Network

**To join the machines to my domain controller, I needed to ensure the network settings on both VMs were configured correctly; First I created a private network between my two VMs to ensure they wouldn’t be exposed to my home network. I did this because I felt it would make troubleshooting easier if anything went wrong.**

In VirtualBox, I went to:
Devices > Network > Network Settings > set Adapter 1 to Host-only Adapter (Host-only adapter creates a private network between my VMs and host machine - preventing network setting assignment from the DHCP server within my home’s router)

Screenshot placeholder: VirtualBox host-only adapter configuration.

8. Configuring the Domain Controller Network Settings

**Next, I assigned network settings to DC01.**
For DC01, I used:
Setting	Value
IPv4 Address	192.168.56.10
Subnet Mask	/24
Preferred DNS Server	127.0.0.1

**For this lab, 127.0.0.1 points DC01 back to itself for DNS because the domain controller is also running DNS.**

To change the IPv4 settings, I right-clicked the network icon in the bottom-right corner of the screen, then went to:
Network and Internet settings > Advanced network settings > right-click the network adapter > Properties > Internet Protocol Version 4 > configure as needed

After saving the settings, I ran:
ipconfig
This allowed me to verify that my changes were saved.

Screenshot placeholder: DC01 IPv4 settings and ipconfig output.

9. Configuring the Windows 10 Client Network Settings

**After ensuring DC01 had the appropriate network settings, I configured WIN10-01.**

For WIN10-01, I used:
Setting	Value
IPv4 Address	192.168.56.20
Subnet Mask	/24
Preferred DNS Server	192.168.56.10

**The client points to DC01 as its preferred DNS server because domain-joined machines need to use the domain controller’s DNS service to locate the domain.**

To verify the changes took place and that WIN10-01 could communicate with DC01, I first ran:
ipconfig

Then I pinged DC01:
ping 192.168.56.10
The ping returned 0% loss, confirming that the client machine could communicate with the domain controller.

Finally, I ran:
ipconfig /all
This allowed me to verify that the DNS server was assigned correctly.

Screenshot placeholder: WIN10-01 IPv4 settings, successful ping to DC01, and ipconfig /all output.

10. Joining the Windows 10 Client to the Domain

**At this point, I had verified basic network communication between WIN10-01 and DC01. Now it was time to actually join the client machine to the domain.**

There are a few ways to do this, but for this lab I opened File Explorer and went to:
This PC > right-click Properties > Advanced system settings > Computer Name > Change > select Domain > enter the domain name > sign in with domain credentials

For this lab, the domain name was:
ad.lab.test
If everything is configured correctly, Windows will display a welcome message confirming that the machine has joined the domain.

To verify the machine was joined to the domain, I checked Active Directory Users and Computers from DC01:
Tools > Active Directory Users and Computers > ad.lab.test > Computers

The client machine appeared in the Computers container, confirming that WIN10-01 had successfully joined the domain
Another verification method is to sign in as a domain user on the client machine and run:
whoami /fqdn

Screenshot placeholder: Domain join screen, welcome message, and WIN10-01 appearing in Active Directory Users and Computers.

11. Enabling Remote Desktop to DC01

**I wanted to allow WIN10-01 to remote into DC01 if needed, so I enabled Remote Desktop on DC01 and then connected to the domain controller from WIN10-01 using RDP.**

To enable Remote Desktop on DC01, I went to:
File Explorer > right-click This PC > Properties > Advanced system settings > Remote > Allow remote connections to this computer (If administrator credentials are not going to be used, another user can be added from this same Remote Desktop settings area)

Then on WIN10-01, I opened the Remote Desktop Connection app and connected to DC01 by entering:
192.168.56.10

After pressing Connect, I entered the appropriate credentials and signed into DC01 remotely.

Screenshot placeholder: Remote Desktop enabled on DC01 and successful RDP connection from WIN10-01.

12. Disabling a Domain User Account

**To demonstrate more Active Directory functionality, I simulated a scenario where an account is disabled and that user tries to sign in on a domain-joined machine.**

To disable John’s account, I went to:
Active Directory Users and Computers > Users > right-click John Holmeleb > Disable Account

**After disabling the account, the user should no longer be able to authenticate using those domain credentials. It is important to note the difference between a local account and a domain account - disabling a domain account prevents that user from authenticating to the domain, but if the user has a separate local account configured on that machine, they may still be able to sign in locally with that account.**

Screenshot placeholder: Disabled user account in Active Directory and failed sign-in attempt.

13. Creating an Account Lockout GPO

**Next, I decided to set up some GPOs for my lab user accounts to showcase their utility and learn more about their features. First, I created an account lockout policy to explore security enforcement through Group Policy. The goal was to lock a domain user’s account after multiple failed login attempts.**

To create the GPO, I went to:
Tools > Group Policy Management > Domains > ad.lab.test > right-click the domain > Create a GPO in this domain, and Link it here...

I named the policy:
LAB - Account Lockout Policy

Then I edited the GPO by going to:
Linked Group Policy Objects > right-click the GPO > Edit

From there, I went to
Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout Policy

I configured the policy with the following settings:
Policy Setting	Value
Account lockout threshold	4 invalid logon attempts
Account lockout duration	5 minutes
Reset account lockout counter after	5 minutes

Screenshot placeholder: Account lockout GPO settings.

14. Prioritizing and Testing the Account Lockout Policy

**After creating the account lockout policy, I went back to the ad.lab.test domain in Group Policy Management to make sure the LAB - Account Lockout Policy had higher precedence than the Default Domain Policy. This was to highlight the fact that if there are two domain-level policies concerning the same account settings, the one with the higher priority link takes precedence.**

To do this, I went back to:
Group Policy Management > ad.lab.test

Then I used the arrows to update the link order of the newly created lockout policy.
I wanted this new policy to go into effect immediately, so I ran:
gpupdate /force

Then I verified that the policy was active by running:
net accounts /domain
This command allowed me to view the password and logon policies at the domain level.

**To test the policy, I signed out of WIN10-01 and purposely entered the wrong password four times. On the next attempt, Windows displayed an error message showing that the account was locked.**

To unlock John’s account, I went to:
Server Manager > Tools > Active Directory Users and Computers > right-click John > Properties > Account > Unlock Account

Screenshot placeholder: GPO link order, gpupdate /force, net accounts /domain, account lockout test, and unlock account screen.

15. Creating Department Groups and Shared Folders

**Next, I simulated resource access using shared drives and department-based Active Directory groups.**

I created two groups, then added the appropriate users to each group:
HR
Sales

I did this from Server Manager by going to:
Tools > Active Directory Users and Computers > right-click Users > New > Group

Then I added users to the correct groups by right-clicking John and Sarah Holmeleb and selecting:
Add to a group

After that, within DC01, I created a folder on the C: drive called:
Shares

Inside that folder, I created two department folders (to mimic department resources):
Sales
HR

Screenshot placeholder: HR and Sales groups in ADUC, group membership, and the Shares folder structure on DC01.

16. Configuring Share and NTFS Permissions

**Now it was time to assign the folders to their appropriate departments and configure access.**

For each department folder, I went to:
Right-click folder > Properties > Sharing > Advanced Sharing > check Share this folder > confirm the share name > Permissions

**Then I assigned the appropriate permissions for this lab.**

After configuring the sharing permissions, I went back to the folder properties window and configured NTFS permissions by going to:
Security > Edit > Add > add the appropriate group > assign the appropriate permissions

This allowed me to control which department group had access to each department folder.

Screenshot placeholder: Share permissions and NTFS permissions for the Sales and HR folders.

17. Creating the Department Drive Mapping GPO

**Since the resource folders were created and the appropriate sharing options were configured, I needed to create the drive mapping through Group Policy.**

To do this, I went to:
Tools > Group Policy Management > right-click ad.lab.test > Create a GPO in this domain, and Link it here...

I named the policy:
LAB - Department Drive Mapping

Then I edited the GPO by going to:
Linked Group Policy Objects > right-click LAB - Department Drive Mapping > Edit

From there, I went to:
User Configuration > Preferences > Windows Settings > Drive Maps

Then I right-clicked the empty white space and selected:
New > Mapped Drive

For the Sales drive, I used the following settings:
Setting	Value
Action	Create
Location	\\TX-DC01\Sales
Label as	Sales Drive
Reconnect	Checked
Drive Letter	S:

Then I went to:
Common > Item-level targeting > Targeting... > New Item > Security Group

From there, I selected the appropriate Sales group so the drive would only map for members of that group.

I repeated this same process for the HR drive mapping.

Screenshot placeholder: Drive mapping GPO settings for Sales and HR, including item-level targeting.

18. Testing Mapped Drives and Troubleshooting

**Finally, it was time to test the functionality of the mapped drives. First, I logged in as John on WIN10-01 and confirmed that he had access to the Sales drive and only the Sales drive. Then I logged in as Sarah and ran into serious issues as I could not locate the HR drive from her account. This sent me on a long investigation. After double-checking everything and basically restarting from step one, I finally found the culprit: a single misplaced hyphen in the hostname of the domain controller!**

After fixing the typo, running:
gpupdate /force

and logging out and back in, the shared drive populated correctly.

**I’ve never been so happy to see a network share.**

Screenshot placeholder: John’s mapped Sales drive, Sarah’s troubleshooting issue, corrected path, and successful HR drive mapping.

Troubleshooting Notes

A few issues came up during the lab that helped reinforce the troubleshooting process:

DNS configuration matters because the client needs to point to the domain controller for domain-related name resolution.
Hostnames and UNC paths need to be exact. A small typo in \\TX-DC01\HR can prevent a mapped drive from appearing.
Share permissions and NTFS permissions both matter when controlling access to folders.
Group Policy changes may not apply immediately unless the policy refreshes or gpupdate /force is used.
Verifying with commands like ipconfig, ipconfig /all, ping, gpupdate /force, and net accounts /domain helps confirm whether the configuration is actually working.

Screenshot placeholder: Any troubleshooting screenshots, errors, command outputs, or before/after fixes.

Skills Demonstrated
VirtualBox lab setup
Windows Server 2022 installation
Windows 10 client setup
Active Directory Domain Services installation
Domain controller promotion
Domain user creation and management
Client domain join
DNS/client-domain communication
Remote Desktop access
Group Policy creation and linking
Account lockout policy configuration
Department-based Active Directory groups
Share and NTFS permissions
Mapped drives using Group Policy Preferences
Item-level targeting
Basic Windows Server troubleshooting
Command-line verification using ipconfig, ping, gpupdate, and net accounts
