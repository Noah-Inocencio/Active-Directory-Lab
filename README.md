# Active Directory Lab

## Requirements

- Virtualization tool (used here Oracle VirtualBox)
- Windows Server ISO (2019 or 2022)
- Windows 10 ISO

### Reference

- <https://www.youtube.com/watch?v=MHsI8hJmggI&t=805s>

## Overview

For this project, Active Directory will be used to support 1000 users by creating one domain to manage the clients. -- more content will be added overtime in regards to Active Directory in this repo --

To summarize the system design file, our home router connected to the host machine will be the way to access Internet for the windows server. The Server machine will have two NICs, one connected to the home router via NAT and receive its IP address via DHCP, and the other connected to VirtualBox's internal network which will be a static IP. The Windows Server will not only act as a domain, but it will provide services such as DHCP, DNS, RAS/NAT, etc.

Aside from the domain being connected to the internal network, our windows client will also be connected. The windows client will gain its IP address via DHCP when connecting to the internal network via windows DHCP server. Although the client's NIC is only connected to the internal network, because of RAS / NAT service configured on the windows server, the client users on this machine will gain access to the internet as well because traffic in the internal (private/local) network is routed through the domain controller to the internet (host's home router).

Finally, we will use a powershell script, one provided in the repo that was obtained by Josh Madakor on Youtube, that when executed on the windows server will create and add 1000 users into our domain. These users will be what is used to sign into the client machine when being a part of the domain.
