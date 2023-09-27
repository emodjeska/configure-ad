<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Deployment and Configuration Steps</h2>

In today's tutorial, we are going to implement on-premises Active Directory. Active Directory is a Mircosoft based software that allows you to centally manage thousdands of user accounts in one place. Lets get into it.

First, we are going to set up our resources in Azure. Create two Virtual Machines. For the first VM, name it DC-1 and have it run Windows Server 2022. Take note that the vNet is automatically created when you create this resource. 

![image](https://github.com/emodjeska/configure-ad/assets/143763072/4e4da571-09b9-4364-ac45-b0839988d5e6)

Be sure to go into DC-1's Virtual Network Interface Card (vNIC) and change DC-1's IP Address to "static". This ensures that DC-1's IP Address will not change. We do this because, our other resources are going to run off of this server, so it is important that the IP address does not change. 

![image](https://github.com/emodjeska/configure-ad/assets/143763072/c411b1bf-93a3-45ee-bf2f-98b15a9d4ae5)

The second Virtual Machine will be the Client. Make sure that it is running Windows 10 and be sure to use the same resource group as DC-1.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/0eda783b-e364-48ee-835a-5c7e617b507b)

Now, we are going to ensure connectivity between the Client and Domain Controller.

Login to Client 1 using remote desktop. Search Command Prompt and open it. Then Ping DC-1's private IP Address.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/31babb9c-0962-45b3-9a11-26ae04894bf2)

Then, we will have to login to DC-1's PC.

Click start -> Windows Administative Tools -> Windows Defender Firewall with Advanced Security -> Inbound Rules -> Core Networking Diagnostics and ICMPv4 and enable these two inbound rules.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/2d20006b-88cb-447e-9ad6-aae46262ceaf)

Now, you will be able to see that Client 1's PC will be able to ping DC-1, ensuring network connectivity.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/2d3edfe6-edd3-49ce-9a7d-9150a73cf124)

To install active directory, Start by opening Server Manager. 

Add Roles and Features -> Follow the Prompts -> Check "Active Domain Services" -> Select Next -> Complete the Installation.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/c7031112-091c-4c8f-b31e-06b49e4e714d)

Select "Promote this Server to a Domain Controller".

Add a "New Forest" -> Name it "mydomain.com" -> Select Next -> Create a Password -> Select Next and follow the prompts -> Select install to complete the installation.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/f42c121a-5594-4bd8-96e3-cb635ca990cb)

DC-1 will automatically restart. log back into DC-1.

We have installed Active Directory! Now, we are going to create an Admin and Normal User Account.

On DC-1, open Server Manager -> Click Tools -> Select Active Directory Users and Computers -> Right-click the domain that your created -> New -> Select Orgonzational Unit (OU).

Create two OUs and name the first "_EMPLOYEES" and the second "_ADMINS".

![image](https://github.com/emodjeska/configure-ad/assets/143763072/e2dcb9d7-6c84-4ba9-935c-fb370d82f360)

![image](https://github.com/emodjeska/configure-ad/assets/143763072/ba65596e-a063-4dd3-9a03-55e70774e6b1)

Refresh the page to sort the new OUs to the top.

We are now going to create a new Admin account to simulate a real world senario.

Go to _Admins OU -> Right click the name of the OU -> New -> User -> First Name/ Last Name: Jane Doe -> User login name: jane_admin -> Select Next -> Create a password -> Uncheck all boxes -> Select Next-> Select Finish.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/311f28bc-9381-4d38-8907-4833d96db04d)


Go to _Admins OU -> Right click Jane Doe -> Select Properties -> Select Member of tab -> Select Add -> -> Select Check Names -> Type the name of your Domain Administrators -> Select Check Names -> OK.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/5589ba39-4eec-49c6-926d-d50c836c6c95)

Now Jane is set to be an Admin account. We are ready to log out of DC-1 as "labuser" and log back in as "jane_admin@mydomain.com".

We will now Join Client-1 to your domain. 

Go to your Azure Virtual Machine -> Select Networking -> Select the link next to the NIC -> Select DNS Server -> Custom -> Type in DC-1's private IP address -> Click save - Select Restart  and Select Yes. This will join the DNS systems of both of our VMs.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/910c3661-7d1d-467f-92fd-361421e8d959)

Log back into Client 1 using Microsoft Remote Desktop as the original local admin -> Right click the start menu -> Select System -> Rename this PC (Advanced) -> Change -> Select Domain -> Type "mydomain.com" -> Select OK -> Username: mydomain.com/jane_admin -> Type in password and press OK -> Restart the computer.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/daae9b6e-f2d9-47bd-9645-68b2542eb8d7)

We have a User set up, so we are going to setup Remote Desktop for Non-administative users on Client 1. This will allow use to login to Client-1 with any user that we create.

Log back into Client 1 -> Use mydomain.com/jane_admin -> Right-click the start menu and select System -> Select Remote Desktop -> Under User Accounts, click "Select Users that can remotely access this PC -> Select Add -> Type in the name of your domain users ->Select check names -> OK -> OK

![image](https://github.com/emodjeska/configure-ad/assets/143763072/d36fd37b-ba4b-4f1c-90d1-f8ee1302d4a0)

Now, it is time to attempt to log into Client 1 with one of the users' profiles. 

Login to DC-1 as Jane _admin -> Search for "Powershell_ise" -> Right click on Powershell_ise and open it as an administrator -> Select New Script and paste the contents of the following script into it. The Script is here.

https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1 

![image](https://github.com/emodjeska/configure-ad/assets/143763072/8b7c3e7c-a2cb-4a6c-8578-7cb13faf7329)

Run the script by clicking on the green arrow button. Once users are available, go back to Active Directory Users -> mydomain.com -> _Employees, and you will see all the accounts you created. You can now login into Client 1 with one of the user accounts.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/53c9e0cf-21a7-4dbe-b75e-886090cd2e4d)

![image](https://github.com/emodjeska/configure-ad/assets/143763072/60112c4b-b673-47d8-b461-c3c741237c69)

From here you are able to unlock passwords by going to "Active Directory Users and Computers" -> Click on  _EMPLOYEES -> Click on the user account -> Check the box that says "Unlock Account". 

Now we know how to implement and use On Campus Active Directory.

Thank you for joining me today.
