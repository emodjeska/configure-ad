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

![image](https://github.com/emodjeska/configure-ad/assets/143763072/bb27d432-aee2-4fed-a6bd-13f1d73dd0e7)

The second Virtual Machine will be the Client. Make sure that it is running Windows 10 and be sure to use the same resource group as DC-1.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/0eda783b-e364-48ee-835a-5c7e617b507b)

Now, we are going to ensure connectivity between the Client and Domain Controller.

Login to Client 1 using remote desktop. Search Command Prompt and open it. Then Ping DC-1's private IP Address.

You will have to login to DC-1's pc and go to

Start -> Windows Administative Tools -> Windows Defender Firewall with Advanced Security -> Inbound Rules -> Core Networking Diagnostics and ICMPv4 and enable these two inbound rules.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/09af844b-e7bd-45d8-9891-e8c7158a8fc8)

Now, You will be able to see that Client 1's pc will be able to ping DC-1, ensuring network connectivity.

To install active directory, Start by opening Server Manager. 

Add Roles and Features -> Foolow the Prompts -> check "Active Domain Services." -> Select Add Features -> select Next -> Complete the installation.

Select "Promote this Server to a Domain Controller".

![image](https://github.com/emodjeska/configure-ad/assets/143763072/36d20a6b-8741-45c3-be75-b4e5f52c7af1)

Add a New Forest. 

select Next -> Create a Password -> Select Next and follow the prompts -> select install to complete the installation.

DC-1 will automatically restart. log back into DC-1.

We have installed Active Directory! Now, we are going to create an Admin and Normal User Account.

On DC-1, open Server Manager -> Click Tools -> select Active Directory Users and Computers.

Right- click the domain that your created -> New -> Select Orgonzational Unit (OU).

Create two OUs and name the first _Employees and the second _Admins.

Refresh the page to sort the new OUs to the top.

Go to _Admins OU -> Right click the name of the OU -> New -> User

First Name/ Last Name: Jane Doe -> User login name: jane_admin -> Select Next -> Create a password -> Uncheck all boxes -> Select Next and then select Finish.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/8383d883-926f-431f-987d-976f9463f4ba)

Go to _Admins OU -> Right click Jane Doe -> Select Properties -> Select Member of tab -> Select Add -> Type the name of your domain administrators -> Select Check Names -> OK -> Apply.

Then you are ready to log out of DC-1 as "labuser" and log back in as "mydomain.com/jane_admin"

![image](https://github.com/emodjeska/configure-ad/assets/143763072/b0a59ba0-9950-4308-ac9d-11a120a021f9)

We will now Join Client-1 to your domain. 

Go to your Azure Virtual Machine -> Select Networking -> Select the link next to the NIC -> Select DNS Server -> Custom -> Type in DC-1's private IP address -> Click Save - Select Restart  and Select Yes. 

![image](https://github.com/emodjeska/configure-ad/assets/143763072/64dd593b-94a1-4a3f-997e-0da306a5140f)

Log back into Client 1 using Microsoft Remote Desktop as the original local admin -> Right click the start menu -> Select System -> Rename this PC (Advanced) -> Change -> Select Domain -> Type "mydomain.com" and select OK -> Username: mydomain.com/jane_admin -> Type in password and press OK -> Restart the computer.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/fc399281-1b60-406f-b98e-42f4d0ecea34)

We have a User all set up, so we are going to setup Remote Desktop for Non-administative users on Client 1. 

Log back into Client 1 -> Use mydomain.com/jane_admin -> Right-click the start menu and select System -> Select Remote Desktop -> Under User Accounts, click "Select Users that can remotely access this PC -> Select Add -> Type in the name of your domain users ->Select check names -> OK -> OK

![image](https://github.com/emodjeska/configure-ad/assets/143763072/37799c90-349a-4d5a-a5a4-3f46cb40db53)

Now, it is time to attempt to log into Client 1 with one of the users' profiles. 

Login to DC-1 as Jane _admin -> Search for "Powershell_ise" -> Right click on Powershell_ise and open it as an administrator -> Select New Script and paste the contents of the following script into it. The Script is here.

https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1 

![image](https://github.com/emodjeska/configure-ad/assets/143763072/83f56e42-2436-4efe-b884-6675e8101dab)

Run the script by clicking on the green arrow button. Once users are available, go back to Active Directory Users -> mydomain.com -> _Employees, and you will see all the accounts you created. You can now login into Client 1 with one of the user accounts.

Thank you for joining me today! Now we know how to run and use On Campus Active Directory.
