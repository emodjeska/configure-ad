<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Video Demonstration</h2>

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Deployment and Configuration Steps</h2>

First, we are going to set up our resources in Azure. Create 2 virtual Machines. For the first VM, name it DC-1 and have it run Windows Server 2022. Take note that the vNet is automatically created when you create this resource. 

![image](https://github.com/emodjeska/configure-ad/assets/143763072/ca3b5b45-0c75-4ca4-8886-db9192de103d)

Be sure to go into DC-1's Virtual Network Interface Card (vNIC) and change DC-1's IP Address to "static". This ensures that DC-1's IP Address will not change.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/cc24b84e-751b-45af-b905-61a8d4cc6f0d)


The second Virtual Machine will be the Client. Make sure that it is running Windows 10 and be sure to use the same resource group as DC-1.

Now, we are going to ensure connectivity between the Client and Domain Controller.

Login to Client 1 using remote desktop. Search Command Prompt and open it. Then Ping DC-1's private IP Address.

You will have to login to DC-1's pc and go to

Start -> Windows Administative Tools -> Windows Defender Firewall with Advanced Security -> Inbound Rules -> Core Networking Diagnostics and ICMPv4 and enable these two inbound rules.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/4d690245-e09f-4437-93f9-74ba3d281b5f)

Now, You will be able to see that Client 1's pc will be able to ping DC-1, ensuring network connectivity.

To install active directory, Start by opening Server Manager. 

Add Roles and Features -> Foolow the Prompts -> check "Active Domain Services." -> Select Add Features -> select Next -> Complete the installation.

Select "Promote this Server to a Domain Controller".

![image](https://github.com/emodjeska/configure-ad/assets/143763072/230e99fb-23cb-4bea-9874-badbd31225cd)

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

![image](https://github.com/emodjeska/configure-ad/assets/143763072/a7be8140-9f09-492b-a28b-e9502e7d6aa5)

Go to _Admins OU -> Right click Jane Doe -> Select Properties -> Select Member of tab -> Select Add -> Type the name of your domain administrators -> Select Check Names -> OK -> Apply.

Then you are ready to log out of DC-1 as "labuser" and log back in as "mydomain.com/jane_admin"

![image](https://github.com/emodjeska/configure-ad/assets/143763072/2301c7d1-30da-435f-96f2-dc092bb282f7)

We will now Join Client-1 to your domain. 

Go to your Azure Virtual Machine -> Select Networking -> Select the link next to the NIC -> Select DNS Server -> Custom -> Type in DC-1's private IP address -> Click Save - Select Restart  and Select Yes. 

![image](https://github.com/emodjeska/configure-ad/assets/143763072/bd64608c-ff32-4525-9596-2592c44241b8)

Log back into Client 1 using Microsoft Remote Desktop as the original local admin -> Right click the start menu -> Select System -> Rename this PC (Advanced) -> Change -> Select Domain -> Type "mydomain.com" and select OK -> Username: mydomain.com/jane_admin -> Type in password and press OK -> Restart the computer.

![image](https://github.com/emodjeska/configure-ad/assets/143763072/81c85328-a965-4990-816e-09972a1cc277)

We have a User all set up, so we are going to setup Remote Desktop for Non-administative users on Client 1. 

Log back into Client 1 -> Use mydomain.com/jane_admin -> Right-click the start menu and select System -> Select Remote Desktop -> Under User Accounts, click "Select Users that can remotely access this PC -> Select Add -> Type in the name of your domain users ->Select check names -> OK -> OK

![image](https://github.com/emodjeska/configure-ad/assets/143763072/6d271cea-329a-4f93-b085-7d07fdafbb07)

Now, it is time to attempt to log into Client 1 with one of the users' profiles. 

Login to DC-1 as Jane _admin -> Search for "Powershell_ise" -> Right click on Powershell_ise and open it as an administrator -> Select New Script and paste the contents of the following script into it. The Script is here.

https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1 

![image](https://github.com/emodjeska/configure-ad/assets/143763072/33f7f37b-37db-4f41-917d-9b494ab82bac)

![image](https://github.com/emodjeska/configure-ad/assets/143763072/4e008a02-82be-4bdc-a159-4637175a7d28)

Run the script by clicking on the green arrow button. Once users are available, go back to Active Directory Users -> mydomain.com -> _Employees, and you will see all the accounts you created. You can now login into Client 1 with one of the user accounts.

Thank you for joining me today! Now we know how to run and use on campus Active Directory.
