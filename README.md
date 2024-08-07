![355522997-527441f2-3be6-4a12-9664-f01fbdc9e0c5](https://github.com/user-attachments/assets/aa20e0d5-689e-4079-8d7c-6c4ef28c12da)


<h1>Tutorial: Setting up Active Directory in the Cloud (Azure)</h1>
This tutorial outlines the implementation of Active Directory using Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used</h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Section I: Azure Setup
- Section II: Ensure Connectivity Between client (client1) and Domain Controller (DC1)
- Section III: Install Active Directory
- Section IV: Create Admin and Normal User Accounts in Active Directory
- Section V: Join Client PC to Domain
- Section VI: Setup Remote Desktop for Non-Administrative Users on Client PC (client1)
- Section VII: Create Additional Users and Attempt to Log Into client1
- Section VIII: Reset Password and Unlock Account

<h2>Section I: Azure Setup</h2>

<p>To start, we'll need to organize the resources we will be using in Azure. For this tutorial, we will be using 2 virtual machines under one resource group in Azure. For the sake of this tutorial, I've named this resource group ad_tutorial (active directory tutorial). 
  The first virtual machine we create will be acting as the Domain Controller, so we'll name it 'DC1', and create a new virtual machine that uses Windows Server 2022. 
  The second virtual machine will serve as a client or user PC within the domain, so we'll name it 'client1', and create a virtual machine that uses Windows 10.
  As you create both virtual machines, record your login information somewhere secure so you can access it later.
</p>
  
![Azure_Setup1](https://github.com/user-attachments/assets/de8061f9-f853-4b53-b55b-d0062dd9a0b3)

<p>As you create 'client1', ensure that both VMs are in the same virtual network on this page (NOTE: If the option is unavailable, make sure that 'DC1' is finished being created first):</p>

![Azure_Setup6](https://github.com/user-attachments/assets/61cef730-aa7b-4c19-af16-b480c4a68c4d)


<p>
Once both virtual machines are created, we'll need to navigate back to our 'DC1' domain controller VM in Azure and take a look at the Network Settings in order to set **DC1's NIC Private IP address to be static**. This can be found under Network Settings -> IP Configurations -> Edit IP Configurations -> Allocation: Static on the right hand side of the page (See image below). 

![Azure_Setup3](https://github.com/user-attachments/assets/7922b0e8-7938-4f0b-8b59-0fd2250b2efd)

This important step will ensure that our Domain Controller's private IP address doesn't change, which will be essential for any PCs that connect to the domain using DC1 as the domain controller as we'll see later in section V.</p>
<br />

<h2>Section II: Ensure Connectivity between client (client1) and domain controller (DC1)</h2>

<p>
In this section, we will be testing the connectivty between our client PC (client1), and our domain controller (DC1).
</p>
<p>
To do this, we'll login to both virtual machines in Azure by using Remote Desktop on our local PC. 
Once logged into our client PC(client1), we will try to test connectivty by pinging DC1's private IP address with the command 'ping -t <DC1's private ip address>'

![Azure_Setup10](https://github.com/user-attachments/assets/e066fd12-e599-4124-a4e5-895d2a462c57)

This perpetual ping command will have client1 keep attempting to connect to DC1. **Note that we are not able to connect to DC1 at this time.**

In order to allow client1 to connect to DC1, we'll need to adjust the firewall settings in our DC1 virtual machine. 

So back into DC1, we will open system settings and navigate to Windows Defender Firewall Settings, and filter for ICMPv4, and enable the option below:

![Azure_Setup11](https://github.com/user-attachments/assets/0a06315c-eb71-4f7e-9201-74f920bd898f)

Once enabled, we can navigate back to our 'client1' virtual machine, and note that the perpetual ping **is now resulting in a successful connection with a reply from DC1**.

![Azure_Setup12](https://github.com/user-attachments/assets/76ea93ac-ba97-47cf-96c4-733468c844af)

Now that we're connected, this will conclude Section II.
</p>
<br />

<h2>Section III: Install Active Directory</h2>
<p>
In this section, we are going to install Active Directory onto DC1 in order to turn it into a Domain Controller.
To do this, we'll log into DC1 and open up the Server Manager to select 'Add roles and features'.:

![Azure_Setup14](https://github.com/user-attachments/assets/77c22059-178e-4618-bf6d-5c1199349e55)

Under select server roles, click **Active Directory Domain Services** and then click 'next'.
![Azure_Setup15](https://github.com/user-attachments/assets/208d9586-127a-4769-a3d9-80290125cfd3)

Under Deployment Configuration, select **'add a new forest'**, and add a name for this domain. This can be anything, but for the sake of this tutorial, I am naming mine 'gardenstore.com'. Record this information somewhere secure as we will be using it later: 
![Azure_Setup17](https://github.com/user-attachments/assets/d540acaf-956f-430f-871d-e8a65c75930f)

You will also be prompted to setup a Directory Services Restore Mode (DSRM) password. Record this information somewhere secure as well, but we will not be using this during this tutorial.

![Azure_Setup18](https://github.com/user-attachments/assets/1fa6082b-bef1-4e2e-823e-04e7cba56f17)
![Azure_Setup19](https://github.com/user-attachments/assets/b67f980c-3a8d-4bee-a4b8-03a8d42c7fb2)

Select next, and proceed to install. DC1 should automatically restart, but if it doesn't, restart DC1.

Now that DC1 is a domain controller, the login will be a bit different, as we'll need to type our new domain name and a forward slash preceding the login information that we used earlier to connect again using Remote Desktop.

This will look like the image below: 

![Azure_Setup20](https://github.com/user-attachments/assets/80d17399-ac57-4020-a50c-9176c3106c22)

I am using gardenstore.com\DC1 (DC1 being the username I originally created for the 'DC1' virtual machine in this case).
</p>
<br />

<h2>Section IV: Create 'Admin' and 'Normal' User Accounts in Active Directory</h2>

<p>
After logging back into DC1, we'll navigate to Active Directory Users and Computers, and create two new Organizational Units. We'll name the first Organizational Unit '_Employees', and the second one '_Admins'. NOTE: Naming these Organizational Units with an underscore preceding the name can be helpful to designate which categories were user-created, and it will also keep them all together when ordered alphabetically.

![Azure_Setup21](https://github.com/user-attachments/assets/cf291a36-5e11-4026-b3c3-550697aa4e5f)

![Azure_Setup23](https://github.com/user-attachments/assets/06a09c10-9c36-447f-92c1-12693b438365)

![Azure_Setup24](https://github.com/user-attachments/assets/a47f9f79-a8bd-49ea-8c7f-3be553ec4fd6)

Now that we have both of these organizational units created, we'll create a new '_Admins' user and name them 'John Admin' for the sake of this tutorial. Record this new admin user's login information somewhere secure.

![Azure_Setup25](https://github.com/user-attachments/assets/55727583-70ca-446f-be1a-9177061f65e0)

![Azure_Setup26](https://github.com/user-attachments/assets/cd383b1f-1dd7-4c00-90e5-02c894a95868)

Next, even though this new 'John Admin' user is in our '_Admins' Organizational Unit, this doesn't mean that this user has 'Admin' privelages in our domain yet!

To change this, we'll right-click on John Admin's user, and select 'Properties':

![Azure_Setup27](https://github.com/user-attachments/assets/087f1b96-64b8-466b-8364-f5b1e7170091)

From here, we can type 'Domain Admins' in the 'Objects' box and select 'Check Names' to verify that this is a valid object. 

![Azure_Setup28](https://github.com/user-attachments/assets/b83a7065-4bf7-46d5-9237-b8d5bca48424)

Select OK, Apply, and verify that John Admin is a member of the Domain Admins group. Once you confirm, select OK again.

![Azure_Setup29](https://github.com/user-attachments/assets/835f5246-d2b5-4f53-a782-02725780583b)

Next, log out of DC1. You can now log back in using the admin user that you just created like the image below:

![Azure_Setup30](https://github.com/user-attachments/assets/3859def5-8d22-4bf6-bb2a-b7a5520bd877)

</p>

<h2>Section V: Join Client PC to Domain</h2>

<p>
In this section, we will be joining our client/user PC 'client1' to our domain (gardenstore.com).

To begin, in Azure, we will set client1's DNS settings to be our domain controller's private IP address (This can be found under Network Settings -> DNS Servers).

![Azure_Setup32](https://github.com/user-attachments/assets/f6b24e91-b84c-4050-9605-05db9e5da503)

After entering your domain controller's private IP address, select 'save', and then restart your 'client1' virtual machine in Azure.
</p>

<p>Next, we will log into client1 via Remote Desktop again using the original login information we created for this virtual machine.
  
![Azure_Setup33](https://github.com/user-attachments/assets/0ba954e8-d8a6-4dce-a321-d403a204d2fc)

Under system properties, navigate to 'Change Name/Domain' and add the domain name (gardenstore.com). You will be prompted to enter domain admin login information to confirm. 
Use the previous domain admin user that we created in the last section, and select OK. 
![Azure_Setup35](https://github.com/user-attachments/assets/1464fa3e-5c03-4d78-a9aa-7b278f72a77e)

![Azure_Setup36](https://github.com/user-attachments/assets/97b35779-cac4-43dc-a78f-6a3fa37120d0)

A pop-up will appear to confirm that you have joined client1 to the domain.

Finally, log back into the domain controller 'DC1', and confirm that 'client1' shows up in the Active Directory Users and Computers inside the 'computers' container on the root of the domain:

![Azure_Setup39](https://github.com/user-attachments/assets/7cd6858d-f4f3-417b-b56c-06ca18f5dad0)

</p>

<h2>Section VI: Setup Remote Desktop for Non-Administrative Users on Client PC (client1)</h2>

<p>In this section, we will configure our client/user PC, 'client1', so that any domain users can access the virtual machine.
  
To do this, we'll open up our Remote Desktop settings (Remote Desktop Users -> Select Users or Groups) and add the 'domain users' group object and select 'OK' (see image below):
![Azure_Setup38](https://github.com/user-attachments/assets/2d06ade8-54d6-4e92-81d0-7dcdb1a464eb)

Now, any non-administrative user in the gardenstore.com domain can log into our client/user machine, client1, using their login credentials.
</p>

<h2>Section VII: Create Additional Users and Attempt to Log Into client1.</h2>

<p>In this section, we will be utilizing a Powershell script by Josh Madakor to create a number of regular user accounts so that we can test logging into our client/user machine, 'client1', using one of the non-administrative users created by the powershell script.

To begin, log into your Domain Controller virtual machine using the Administrator User that you created. 

Open Powershell_ise _as an administrator_, and paste the contents of the script into it: (https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1). 

Note: I adjusted this script so that it only creates 100 users for this tutorial, I adjusted the default password to be 'Welcome123!', and I also adjusted the Organizational Unit name to be '_Employees' so that the case matches my system. You can adjust these variables to suit your needs.

![Azure_Setup41](https://github.com/user-attachments/assets/202b4143-fac5-4002-9953-883a84463625)

![Azure_Setup42](https://github.com/user-attachments/assets/6d3ee99c-8fb6-42fe-a0e3-0051d93cd761)

Run the script, and the specified number of accounts will be created using randomized vowel+consonant names. When the script is finished running, you can verify that the listed names appear in your Active Directory Users and Computers under your '_Employees' Organizational unit.

Finally, attempt to log into your 'client1' virtual machine using one of the user accounts that was created:

![Azure_Setup43](https://github.com/user-attachments/assets/c1d1848c-dc5d-491d-887e-9ed42f052419)

![Azure_Setup44](https://github.com/user-attachments/assets/886c95e7-3c02-4a95-b5e4-a1e5d44639b6)

![Azure_Setup45](https://github.com/user-attachments/assets/1156a23c-e326-4a6d-80dc-48ae704765ca)

Success!

</p>

<h2>Section VIII: Reset Password and Unlock Account</h2>

If you forgot your default password for your example user, or if a real-life employee in your organization forgot their password or locked their account, you can reset their password and unlock their account by navigating to their user in your Active Directory:

![Azure_Setup46](https://github.com/user-attachments/assets/a482e12d-d1ba-4f46-9c48-e78f78a1e461)

![Azure_Setup47](https://github.com/user-attachments/assets/5d9d265e-9511-4c0c-bd92-92bb10baf47c)
