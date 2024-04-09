# Active Directory Home Lab | Add Users With PowerShell

## Description
In this lab, I'll walk you through creating an Active Directory (AD) home lab environment using Oracle VirtualBox and how to add users with PowerShell. Two virtual machines will be made - the first is the DC or Domain controller which will have 2 NICs or network interface controllers, the first connects to our home internet and the second is connected to a private network that the second virtual machine, which is our client, can connect to. This lab will also walk through installing Active Directory, setting up a domain server, configuring internet settings, installing RAS/NAT and DHCP, and lastly using PowerShell to add users. This is a great way to get hands-on experience and understand how Active Directory and Windows networking works. 

## Languages and Utilities Used

- Oracle VirtualBox
- PowerShell

## Environments Used

- Windows 10
- Server 2022

## Program Walk-Through

### Creating the Domain Controller (DC)

1. Download and install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) for your respective platform. Then install the [Extension Pack](https://download.virtualbox.org/virtualbox/7.0.14/Oracle_VM_VirtualBox_Extension_Pack-7.0.14.vbox-extpack).
2. Download [Windows Server ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022) and [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10).
3. Open up VirtualBox and click **New** starburst icon to create our first virtual machine, the domain controller.
   + Name: DC
   + ISO Image: Windows Server ISO.
   + Edition: Windows Server 2022 Standard Evaluation (Desktop Experience)
   + Uncheck Skip Unattended Installation.
>  If you select one of the non-Desktop Experience (Evaluation), after it installs, this version will have a command line with no GUI, making it more difficult to navigate.

![New Settings Server](https://i.imgur.com/dZtepoL.png)

4. Click Next, Base Memory: 4096 MB and Processors: 2
> If your computer has enough memory and cores, it's recommended to increase it from the default settings to speed up using the virtual machine.

![Hardware](https://i.imgur.com/GJGxuWz.png)

5. Keep Virtual hard disk settings, click Next, then Finish.
6. Click on **Settings** cog wheel icon, General, Advanced, change Shared Clipboard and Drag'n'Drop to **Bidirectional**
> Bidirectional setting lets you copy and paste and drag and drop between your actual computer and virtual machine.

![General Advanced Settings](https://i.imgur.com/cJrU8hE.png)

7. Click on Network, and keep Adapter 1 - NAT and change Adapter 2 - Internal Network. click Ok.
> Adapter 1 connects to our home internet and automatically gets an IP address. Adapter 2 is the internal network that we'll need to setup the IP address manually in a bit.
8. Open the DC VM by either clicking the **Start** green arrow icon or double-clicking on the DC VM in the left pane. Click Next to start Server installation and choose Windows Server 2022 Standard Evaluation (Desktop Experience).

![Desktop Experience](https://i.imgur.com/FJBBfYi.png))

9. Check to accept the Microsoft Software License Terms, Next, Custom, Next, and then let it install. This will take a few minutes.
10. After it boots up, go to Input, Keyboard, Insert Ctrl+Alt+Del to login and then type in your Administrator password.

![Unlock Screen](https://i.imgur.com/2NCMY6M.png)

11. If the blue Network pane on the right appears, click **Yes** to discoverable by other computers.
12. Click Devices, Insert Guest Additions CD image... Then navigate to **This PC** by clicking the **File Explorer** folder icon in the taskbar. Double-click CD Drive (D:) VirtualBox Guest Additions then double-click VBoxWindowsAdditions-amd64 to install. Click Next, Next, Install and then let it reboot.
> Installing Guest Additions gives a better experience and speeds up everything.

![Insert Guest](https://i.imgur.com/pEp9muN.png)
![CD Drive](https://i.imgur.com/SUML8gU.png) 

### Internet Settings

1. Once the DC reboots, unlock the screen and log in. Click the Network icon in the lower right taskbar, Network Connected, Change Adapter Options.
2. Two Ethernet Networks will appear. Right-click on one of them, **Status**, **Details...**
> The IP looks ok here. This is connected to our home internet.

![Ethernet](https://i.imgur.com/bbLNjLC.png)

3. Close this window and the Ethernet Status one. Right-click, **Rename**, and type **INTERNET**.
4. Right-click on Ethernet 2, **Status**, **Details...**
> Autoconfiguration IPv4: 169.254.192.140, means that this adapter was looking for a DHCP server to try to get an IP address from somewhere but it was unable to find one, so this one was automatically assigned to it. This is our Internal network.

![Ethernet 2](https://i.imgur.com/sIxSGvR.png)

5. Close this window and the Ethernet Status one. Right-click, **Rename**, and type **INTERNAL**.
6. Right-click on **INTERNAL**, **Properties**, and double-click **Internet Protocol Version 4 (TCP/IPv4).

![INTERNAL Properties](https://i.imgur.com/NmvBEOn.png)

7. Check **Use the following IP address:** and enter the settings as seen below:
> We won't use a Default gateway because the Domain Controller itself will serve as the Default gateway. For the DNS server, when we installed Active Directory, it automatically installed DNS so this will use itself as the DNS server. To do this, either enter its own IP address or use the loopback address of 127.0.0.1, which is a generic address that refers to itself. Whenever a computer pings this address, they're essentially pinging themself automatically.

![Internet Protocol Version 4 Properties](https://i.imgur.com/SuimBXD.png)

8. Click **Ok**, then **Ok** again to close out the Network windows.
9. Next, we'll rename the PC. Right-click **Start Menu**, **System**, **Rename this PC**. Rename it **DC**, **Next**, **Restart now**, **Continue**.

![Rename PC](https://i.imgur.com/wm1e9FT.png)

### Install Active Directory and Create Domain Server

1. Log back in after it restarts. In **Server Manager**, **Add roles and features**.

![Add roles and features](https://i.imgur.com/4rC9LPp.png)

2. Click **Next**, **Next**, **Next**, then check **Active Directory Domain Services** and click **Add Features**.

![Active Directory Domain Services](https://i.imgur.com/MAXcMGy.png)

3. Click **Next**, **Next**, **Next**, **Install**. After it finishes installing, click **Close** 
4. Click the **flag** icon with the yellow warning symbol. Notice the **Post-deployment Configuration**. Click **Promote this server to a domain controller**.
> This warning means that the software for Active Directory domain services has been installed, but the actual domain hasn't been created yet.

![Post Warning](https://i.imgur.com/XTOilyg.png)

5. Select **Add a new forest** and choose a **Root domain name**. For this lab, I'll use **mydomain.com** and then click **Next**.

![Forest](https://i.imgur.com/WDByvwl.png)

6. Enter in a password, although you probably will never have to use it. Keep clicking **Next** until you can click **Install**. It'll automatically restart your computer. Unlock your screen and log back in.
> Notice it now says MYDOMAIN\Administrator account since you're part of the MYDOMAIN server.

![MYDOMAIN Login](https://i.imgur.com/RMPWi6U.png)

8. Now, we'll create our own dedicated Domain Admin account instead of using the built-in Administrator account. Access this either by clicking:
   + **Start** menu, **Windows Administrative Tools** folder, **Active Directory Users and Computers**
   + **Server Manager**, **Tools**, **Active Directory Users and Computers**

![StartUsers](https://i.imgur.com/iXc1UsD.png)

![ServerUsers](https://i.imgur.com/bpjyfVK.png)

9. Expand the newly created mydomain.com in the left pane, then right-click on it, **New**, **Organization Unit**.
> Organizational Units are like folders in the Active Directory.

![New OU](https://i.imgur.com/ucZKw6x.png)

10. Let's name it **_ADMINS** and uncheck **Protect container from accidental deletion**, then click **OK**.

![OU Name](https://i.imgur.com/1yR5YfL.png)

11. Right-click the newly created **_ADMINS** OU, **NEW**, **USER** and enter in **First name, Last name**, and **User Logon name**. Click **Next**. 
> For the **User logon name**, a common naming convention in many organizations is a- to represent it’s an admin account, and whatever naming convention follows. So in this case, first name initial followed by last name.

![New User](https://i.imgur.com/h3sO71i.png)

12. Enter in a password and uncheck **User must change password at next logon** and check **Password never expires**. Click **Next** then **Finish**.
> Since this is a lab environment, it's safe to enable **Password never expires**, but in the real world, you'll want to leave **User must change password at next logon** checked on so the new user can create their own password that only they know. Not even administrators should have access to another user's password.

13. See your new account in the right pane, but it’s not an admin yet. To make it a domain admin, right-click **Properties, Add to a group...**.
![Add to a group](https://i.imgur.com/rrr7DjV.png)

14. In **Enter the object names to select**, type: domain admins, then click **Check Names** so that it resolves to Domain Admins, indicating it's an existing group. Click **Apply, Ok, Ok**.
![Domain Admins](https://i.imgur.com/UXTUMWB.png)

15. Now you have your very own domain admin account. To use this, click **Start** menu, **Profile** icon, **Sign out**.

![Sign out](https://i.imgur.com/AKl49Yi.png)

16. Instead of logging into the **Administrator** account, click **Other user** and use your newly created domain admin account. In my case, that's **a-cnguyen**.

### Install RAS/NAT
+ RAS = remote access server
+ NAT = network address translation
+ By setting up RAS/NAT, it allows client machines to be on a private virtual network, but still be able to access the internet through the Domain Controller.


1. In **Server Manager**, click **Add roles and features**, **Next**, **Next**, **Next**, then select **Remote Access**. 

![Remote Access](https://i.imgur.com/IFM8Vp3.png)

2. Click **Next**, **Next**, **Next**, select **Routing**, then click **Add Features**. DirectAccess and VPN (RAS) will automatically be selected for you.

![Routing](https://i.imgur.com/c12yapg.png)

3. Click **Next**, **Next**, **Next**, **Install**. Once it's done installing, click **Close** then **Tools, **Routing and Remote Access**.

![Routing and Remote Access](https://i.imgur.com/51TPHXK.png)

4. Right-click **DC (local), Configure and Enable Routing and Remote Access, Next**, then select **Network address translation (NAT), Next**
> If **Network address translation (NAT)** is grayed out, click **Cancel**, close the **Routing and Remote Access** window and reopen it again from **Tools**. 

5. Select **INTERNET**, then click **Next** and **Finish**.

![INTERNET](https://i.imgur.com/kBlcp6u.png)

6. Now you’ll see the DC icon with a green arrow pointing, which means **Routing and Remote Access Is Configured on This Server**.

![DC Configured](https://i.imgur.com/jGF4C2J.png)

### Install DHCP
+ The whole purpose of DHCP is to allow client computers on the network to automatically get their IP addresses instead of manually inputting them in.
+ We'll set up the DHCP server on the Domain Controller with scope information. This allows Windows 10 clients to get IP addresses (within a range we specify) and lets them browse on the Internet even though they’re on a private Internal network, just like in an office or school.

1. Back in **Server Manager**, click **Add roles and features, Next, Next, Next**, select **DHCP Server, Add Features, Next, Next, Next, Install**.

![DHCP Server](https://i.imgur.com/lyqdp8G.png)

2. After it installs, let's set up our scope by clicking **Close, Tools, DHCP**. 

![Tools DHCP](https://i.imgur.com/mggcPdZ.png)

3. In the left pane, expand **dc.mydomain.com** and **IPv4**. Right-click on **dc.mydomain.com** and click **Authorize**. 
4. Right-click on **IPv4** and select **New Scope...**

![New Scope](https://i.imgur.com/qWxfDcL.png)

5. Click **Next** and **Name** it the scope or desired IP range and leave the description blank. Click **Next**.

![Scope Name](https://i.imgur.com/DRwoTG7.png)

6. Enter in **Start IP address, End IP address, Length, and Subnet mask** as seen below:

![IP Address Range](https://i.imgur.com/yREJw00.png)

7. Click **Next, Next, Next** then enter in the **IP address: 172.16.0.1 ** on the **Router (Default Gateway)** page and click **Add**:
> 172.16.0.1 is the Domain Controller's IP address that has NAT configured on it.

8. Click **Next, Next, Next, Finish**
> If the arrows on both IPv4 and IPv6 aren't green, right-click **dc.mydomain.com, Authorize** then right-click **Refresh**.

9. If you expand **IPv4, Scope, Address Leases**, there's no leases yet because client computers haven't been created. DNS is now set up!

![Address Leases](https://i.imgur.com/QWpLPH5.png)

### Add Users Using PowerShell
+ Before creating a client computer and joining it to the domain, let's first use a PowerShell script to create a whole bunch of users in Active Directory created for instead of doing it manually.

1. Download the file from my GitHub here. 
2. 
3. 
4. 

### Create Client Machine

1. Go back to VirtualBox and click **New** to create a new virtual machine. This will be our client machine.
2.  
<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
