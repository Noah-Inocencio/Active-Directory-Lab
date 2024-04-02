# Active Directory Lab

## Requirements

- Virtualization tool (used here Oracle VirtualBox)
- Windows Server ISO (2019 or 2022)
- Windows 10 ISO

### Reference

- video reference - <https://www.youtube.com/watch?v=MHsI8hJmggI&t=805s>

## Overview

For this project, Active Directory will be used to support 1000 users by creating one domain to manage the clients. -- more content will be added overtime in regards to Active Directory in this repo --

To summarize the system design file, our home router connected to the host machine will be the way to access Internet for the windows server. The Server machine will have two NICs, one connected to the home router via NAT and receive its IP address via DHCP, and the other connected to VirtualBox's internal network which will be a static IP. The Windows Server will not only act as a domain, but it will provide services such as DHCP, DNS, RAS/NAT, etc.

Aside from the domain being connected to the internal network, our windows client will also be connected. The windows client will gain its IP address via DHCP when connecting to the internal network via windows DHCP server. Although the client's NIC is only connected to the internal network, because of RAS / NAT service configured on the windows server, the client users on this machine will gain access to the internet as well because traffic in the internal (private/local) network is routed through the domain controller to the internet (host's home router).

Finally, we will use a powershell script, one provided in the repo that was obtained by Josh Madakor on Youtube, that when executed on the windows server will create and add 1000 users into our domain. These users will be what is used to sign into the client machine when being a part of the domain.

## Begin Setup

Once you have downloaded the requirements mentioned above, now we must create our first virtual machine in Oracle VirtualBox. Let's start with the windows server machine

### Windows Server

- open up VirtualBox and at the top tabs go to Machine -> New
  - fill in name field and the Type of machine, and click Next
  - hardware recommended 2 Gb of RAM and as much vCPU as you can give given your own hosts machines specs (more vCPU, faster and better experience. 1 is enough though), and click Next
  - Allocate an amount of space on virtual hard disk (preferably on an SSD for better performance), and click Next -> Finish
- now the virtual machine is created. Go to Settings -> Network -> have one adapter be NAT (connected to your home router) -> activate another adapter and configure it to be Internal (connected to VirtualBox Internal Network)
- Open the Virtual Machine and you will be prompted to give the location of your ISO image. Open it and choose the Standard Evaluation (Desktop Experience)
- create an administator login
- once logged into the machine, click the network icon, scroll down and click the network icon -> Network and Internet Settings
  - check out both NICs and find out which has an IP obtained by your router, should look something like 10.0.x.x
  - the NIC with this configuration, rename it as INTERNET to show that this is the adapter connected to the host home router connecting to the internet
  - name the other NIC to something that will let you know it is connected to the internal network such as "X_INTERNAL_X"
    - go to properties of this NIC and double click TCP/IP
    - configure a static IP address. If you want to use the same as mine, configure it with -- IP: 172.16.0.1  Subnet: 255.255.255.0  Preferred DNS: 127.0.0.1

### Active Directory Domain Controller

- time to create active directory domain
  - open up Server Manager from the search bar in the bottom left
  - click Manage -> Add Roles and Features -> Next -> Next -> Next -> check Active Directory Domain Services -> Next -> Next -> Install
  - when finished installing there should be a flag by Manage, click it to finish Post Deployment
  - click add new forest, add a name for root domain (example: mydomain.com) -> next
  - input password, can use the same password you have been using -> Next
  - unselect DNS delegation -> click next until install and install
  - once finished, restart machine
- when signing in you can notice you're signing into the domain now
- once signed in, open up Server Manager -> Tools -> Active Directory Users and Computers
  - right click on your domain and hover New -> click Organization Unit. This will create an OU in which you can place users, groups, service accounts, computers, etc.
  - Let's create an admins group, call it what you may (_ADMINS)
  - uncheck the box for "protect container from accidental deletions" this is just to make it easier when playing around with deleting it
  - right click on the created OU -> New -> User
    - set up a new user (yourself) input fields to your liking. click Next then input your password and Finish
    - now right click on the user and select "Add to group" and type in Domain Admins (a group created upon creating the domain) which will give admin privileges to users within this group
- restart machine and try logging in with the new user created

### Active Directory NAS / RAT

- open up Server Manager -> Manage -> Add roles and features -> Next -> Next -> Next -> check the Remote Access -> Next -> install Routing -> Next until Install
- now go to Tools -> Remote Access -> right click the domain controller and configure
- check the NAT box, this will allow internal users to use the internet. Click Next
- check Use this public interface to connect to the internet and use the network adapter we labeled as INTERNET. This will perform NAT translation for clients on the internal network to have their traffic to the internet translated to that public IP

### Active Directory DHCP Server

- open up Server Manager -> Manage -> Add roles and features -> Next -> Next -> Next -> check the DCHP -> Next -> Next until Install
- open up Tools -> DCHP -> open domain -> right click IPv4 -> New Scope
- here you will create your DCHP pool, create your own pool or follow along with mine start ip: 172.16.0.100 end ip: 172.16.0.200 subnet mask: 255.255.255.0
- lease duration is how long the server will lease an ip address, configure this to your own liking. click Next
- add router should be the IP address of the DHCP Server, which we configured to be 172.16.0.1 if you're following along my configurations. click Next until Finish
- right click domain controller and select Refresh and DHCP server will be up and running

### Running Powershell Script to Add 1000 Users to the Domain

- bring the scripts in this repo into your windows server virtual machine
- open up Powershell ISE from the search. input the following command -> Set-ExecutionPolicy Unrestricted
- this command will allow for execution of the script. Navigate to the create user script and run -> ./1_CREATE_USERS.ps1

### Windows 10 Machine

- follow the same process as creating the Windows Server, instead using the ISO file for Windows 10
- when setting up, click every option with the least setting up option to go through the process quicker and sign in as a guest
- now to connect to the internet. Click the network icon -> Network and Internet Settings -> Ethernet -> Change Adapter Options -> right click NIC -> Properties -> double click TCP/IP
- check box automatic IP address (DHCP), at the bottom configure preferred DNS server as 172.16.0.1 or whatever the windows server IP address is on the Internal Network
- go to command prompt and input ipconfig to check if we have an ip address which should be the first address the in the DHCP pool we created on the DHCP server. try pinging 8.8.8.8 for google.com to chekc internet connectivity

### Joining the Domain Controller

- click the windows icon on the bottom left and click System -> About -> scroll down and select Rename this PC (advanced)
- click Change -> name the pc a new host name, also on the bottom checkbox Domain and enter the name of the domain controller to join
- a pop up of signing into a user with permissions to the domain will pop up, just sign in with the admin account created earlier.
- you will be prompted to restart. Now you may log into the machine being a part of the domain using any of the new users generated info
