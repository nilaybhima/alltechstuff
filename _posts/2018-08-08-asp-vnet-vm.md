---
title: "Simplest way to integrate App Service Plan with VNet and SQL Server Database"
excerpt: "This article takes you through the steps to create a App Service Plan (ASP) integrated with VNet. The web app then talks via Point-to-Site VPN to a backend SQL Database in a VM living in the virtual network."
tags: 
    - Azure
    - Web API
    - Virtual Machine
    - SQL Server
    - Virtual Network
    - Network Security Group
    - NSG
    - VPN
    - Point-to-Site
date:   2018-08-08 00:00:00 -0600
comments: true
#categories: ASP.NET
---

Recently, I was proposing a solution to one of our clients who made the strategic decision to migrate their on-prem system to Azure, part of the work is to migrate SQL Server and web applications in VMs to the cloud. This is a very common architecture, IIS application with a backend DB Server. And as predicted, I was not surprised to see the amount of work happening inside the DB Server, rather than just being a DB Server, like batch jobs, reporting etc. This also made me to think how lucky I was with my previous job, being a technical lead working a greenfield project that is cloud native or born in the cloud. We did not have to worry about any migration or rewrite anything to make it work in the cloud.

Given the nature of the DB Server setup, we decided to just lift and shift DB Server VM to Azure VM rather than migrating the data into Azure SQL Server Database (DBaaS) for now. So, we can get the system up and running in the cloud first, then enhance it later; always good to be agile. 

The customer has limited budget to be on App Service Environment (ASE), App Service Plan (ASP) also satisfies the requirements. However, apart from the SQL Authentication, having second layer protection for the DB Server is a must have.

## Component Architecture
Based on those requirements, my design is very simple. Having the DB Server VM inside a VNet, all the public access is restricted. So the VM and the database is completely sealed within a virtual network. Then, setup a Point-to-Site VPN from the application server (App Services) to the DB Server.
![Component Architecture]({{"/assets/images/asp-vnet-integration/asp-vnet.png"}})

## Implementation 
### 1. Develop the Web API application and Database 
If you do not have the application and database already, you can simply refer to my previous blog [Boilerplate for ASP.NET Web API]({{"http://williamwang.info/boilerplate-web-api/"}}){:target="_blank"}. 
Source code can be found from [https://github.com/bwwilliam/GenericRepositoryPattern](https://github.com/bwwilliam/GenericRepositoryPattern){:target="blank"}

### 2. Create the App Service in Azure
This is pretty straight forward, just follow the Web App deploy wizard within Azure Portal.
Please ensure the Pricing Tier for App Service Plan must be S1 or above. 
![App Service Plan]({{"/assets/images/asp-vnet-integration/asp-wizard.JPG"}})

After the Web App is provisioned, to setup VNet, go to `Networking` and `setup` to create a VNet
![Setup VNet]({{"/assets/images/asp-vnet-integration/VNet-setup.JPG"}})

### 3. Create the VM with SQL Server in Azure
`Create a resource`, search for `SQL Server`, there should be a list of options available. Depending on your requirements, you need to decide what you want to deploy for your DB Server, anything with `Windows Server` and `SQL Server` pretty much indicates a VM with SQL Server inside. For my case, I picked `SQL Server 2016 SP1 Enterprise on Windows Server 2016`.

* Ensure the location of your VM is same as your App Service.
* Choose the right machine size, use `B2s` if for PoC
* Pick the VNet that you created earlier from Step 2

![Setup VNet]({{"/assets/images/asp-vnet-integration/VM-VNet.JPG"}})
* SQL Server Authentication Mode must be enabled

![Setup SQL Authentication]({{"/assets/images/asp-vnet-integration/SQL-connectivity-Authentication-Mode.JPG"}})

I also strongly suggest you allow RDP inbound rule, so you can remote into your VM. This can be disabled later from NSG settings.

![Setup Inbound Rules]({{"/assets/images/asp-vnet-integration/inbound-rules.JPG"}})
* Once the VM is provisioned, deploy the database into it using the DB project from Step 1. 
* Create SQL Server Login per screenshot below, and under the `User Mapping`, ensure your new database is ticked and `Database Role` `db_owner` is selected. 

![Setup Inbound Rules]({{"/assets/images/asp-vnet-integration/DBLogins.JPG"}})

* Finally, go to the VM's `Networking` blade and copy out the private IP address and save it to your web application's database connection string

![Setup Inbound Rules]({{"/assets/images/asp-vnet-integration/NSG.JPG"}})

The connection should look like below, make sure the place holders are updated {PRIVATE IP ADDRESS}, {USERNAME} and {PASSWORD}

```xml
<add name="DBContext" connectionString="metadata=res://*/DBContext.csdl|res://*/DBContext.ssdl|res://*/DBContext.msl;provider=System.Data.SqlClient;provider connection string=&quot;Server=tcp:{PRIVATE IP ADDRESS},1433;Initial Catalog=Temp;Persist Security Info=False;User ID={USERNAME};Password={PASSWORD};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=True;Connection Timeout=30;App=EntityFramework&quot;" providerName="System.Data.EntityClient" />
```
## JOB DONE!!!
I know has lots of steps to set everything up. But if look back, there is not actually much if you know what you are doing.

So, I only budgeted few hours to wire up the components. However, it turned out to be much longer than I anticipated. I had everything built, deployed and hooked up within 1 hour, but my job was quite done when started testing the connectivity. 

I just kept getting network connectivity issue, access denied when connecting to the database via VNet. I ended up spending almost a day to just diagnose this issue. I tried several things including open NSG to public access to bypass VNet, it worked and proven that my database connection was fine. 

After trying many different odds things, I reached one of those moments that I almost forgot what I was trying to fix. I then delete the entire Resource Group and started all over again with a fresh deployment. And there was no surprise for me, still the same connectivity issue.

Finally, I deployed another App Service, left the existing App Service, VM, VNet untouched. I integrated the new App Service with existing VNet, and setup the DB Connection string, one of sudden, it worked. Just to prove it was a not human error, I redeployed the entire solution again exactly like what I had before, the same connectivity issue came back. This time, I just simply disconnected the App Service from VNet, integrated again and the problem went away. So, the trick here is that if the VNet was created as part of the App Service, then <mark>DISCONNECT & RECONNECT</mark>. 

![Disconnect VNet]({{"/assets/images/asp-vnet-integration/disconnect-vnet.JPG"}})

To me this sounds like a bug with Azure infrastructure. I will make contact with Microsoft to see what they say about this issue.



