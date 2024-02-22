               SIEM and Honeypot | Microsoft Azure Sentinel Attack Map

Summary
SIEM, short for Security Information and Event Management System, is a crucial solution aiding organizations in identifying, analyzing, and addressing security threats before they disrupt business operations. By aggregating event log data from various network sources like Firewalls, IDS/IPS, and Identity solutions, SIEM enables security professionals to actively monitor, prioritize, and mitigate potential threats in real-time. On the other hand, a honeypot serves as a strategic security measure, simulating a vulnerable system to entice attackers into a controlled environment. This allows for the observation of attacker behavior, analysis of threats, and refinement of security protocols. The objective of this lab is to explore the process of gathering honeypot attack data, integrating it into a SIEM platform, and presenting it in a visually accessible format, such as a world map showcasing event counts and geolocations.
Learning Objectives
Provisioning and deprovisioning virtual environments within Azure.
Third-party API calls.
PowerShell Extracts
Security Information and Event Management - log analysis and visualization.
Technologies and Protocols: 
Microsft Azure - a cloud computing service operated by Microsoft for application management via Microsoft-managed data centers
Services within Azure: Log Analytics Workspace and Sentinel (Mircosoft's SIEM)
Powershell
Remote desktop protocol

Step 1: Create a Microsoft Azure account. https://azure.microsoft.com/en-us/free/

 ![azurewebsite](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/4339332b-9652-440c-8c68-3d1b5a12ecdc)


Step 2: Create a honeypot VM
Go to portal.azure.com

![AccountCreation](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/32f7ee5a-09f3-4759-9b81-08385e19045c)

Search for "virtual machines"
Create > Azure virtual machine 
Name your VM (honeypot-vm) (I recently made a honeypot so I named it honeybaby-vm to not get confused)
Select a recommended region ((US) West US 2)
Availability options: No infrastructure redundancy required
Security type: Standard
Image: Windows 10 Pro, version 21H2 - x62 Gen2
VM Architecture: x64
Size: Default is okay (Standard_D2s_v3 – 2vcpus, 8 GiB memory)
Create credentials for the machine
Note these will be used to log into the virtual machine so keep that in mind. 
Everything else can be left as default. 

NETWORKING 
We want our honeypot to be as vulnerable as possible so we need to disable any security measures

![allowinbound](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/d63e1706-c9cf-4d66-9794-8378b81ea849)
 
Remove Inbound rules (1000: default-allow-rdp) by clicking three dots
Add an inbound rule
Destination port ranges: * (wildcard for anything)
Protocol: Any
Action: Allow
Priority: 100 (low)
Name: Anything (ALLOW_ALL_INBOUND)
Select Review + create

Step 3: Create a Log Analytics Workspace
 
 ![Loganalytics](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/ffa7f81a-6aae-49f5-b581-62fb529ac9dc)

Creating a resource group is a great way to keep your project organized. 

Step 4 Microsoft Defender for cloud
Search for "Microsoft Defender for Cloud"
Scroll down to "Environment settings" > subscription name > log analytics workspace name (log-honeypot)
Settings | Defender plans
Cloud Security Posture Management: ON
Servers: ON
SQL servers on machines: OFF

![defender_plans](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/a09ce1e0-3273-4cc6-a201-5a1361f89827)


 Save
Step 5: Connect Log Analytics Workspace to Virtual Machine
Search for "Log Analytics workspaces"
Select workspace name (log-honeypot) > "Virtual machines" > virtual machine name (honeypot-vm)
Click Connect 
![ConnectingVMtoLAW](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/0e2d9588-5852-4ab8-8a2f-e46c2a154327)


Step 6 Sentinel
Search for "Microsoft Sentinel"
Click Create Microsoft Sentinel
Select Log Analytics workspace name (honeypot-log)
Click Add  

![sentiel](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/74a0dc92-f791-44c4-a758-d95dc52e7a88)

I failed a logon on attempt to make finding failed logon attempts easier before leaving my machine vulnterable 
 
![FailTOshoeexample](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/c41748cc-55dd-4f07-b7de-b0d07c422f3f)

Step 7: Disable the Firewall in Virtual Machine
Go to Virtual Machines and find the honeypot VM (honeypot-vm)
By clicking on the VM copy the IP address
Log into the VM via Remote Desktop Protocol (RDP) with credentials from step 2
Accept Certificate warning
Select NO for all Choose privacy settings for your device
Click Start and search for "wf.msc" (Windows Defender Firewall)
Click "Windows Defender Firewall Properties"
Turn Firewall State OFF for Domain Profile Private Profile and Public Profile
Hit Apply and Ok
Ping VM via Host's command line to make sure it is reachable ping -t <VM IP>
 ![firewall](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/fce7dba1-8586-4abc-b220-205d0668415e)



Step 8: Scripting the Security Log Exporter
In VM open Powershell ISE
Set up Edge without signing in
Copy Powershell script into VM's Powershell (Written by Josh Madakor)
Select New Script in Powershell ISE and paste script
Save to Desktop and give it a name (Log_Exporter)

 ![scriptrunning](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/d472d641-7959-4d4d-9674-99270478714f)


Make an account with Free IP Geolocation API and Accurate IP Lookup Database
This account is free for 1000 API calls per day. Paying 15.00$ will allow 150,000 API calls per month.

Copy API key once logged in and paste into script line 2: $API_KEY = "<API key>"
Hit Save
Run the PowerShell ISE script (Green play button) in the virtual machine to continuously produce log data 

![apikey](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/de834c6f-00f2-4bc0-b5eb-b074a17c9aa0)


Step 9: Create Custom Log in Log Analytics Workspace
Create a custom log to import the additional data from the IP Geolocation service into Azure Sentinel
Search "Run" in VM and type "C:\ProgramData"
Open file named "failed_rdp" hit CTRL + A to select all and CTRL + C to copy selection
Open notepad on Host PC and paste contents
Save to desktop as "failed_rdp.log"
In Azure go to Log Analytics Workspaces > Log Analytics workspace name (honeypot-log) > Custom logs > Add custom log
Sample
Select Sample log saved to Desktop (failed_rdp.log) and hit Next
Record delimiter
Review sample logs in Record delimiter and hit Next
Collection paths
Type > Windows
Path > "C:\ProgramData\failed_rdp.log"
Details
Give the custom log a name and provide description (FAILED_RDP_WITH_GEO) and hit Next
Hit Create

![ConnectLAWtoVM](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/23a12867-d98d-4c76-aa9b-b7fa55ec96e9)

 
Step 10: Query the Custom Log
In Log Analytics Workspaces go to the created workspace (honeypot-log) > Logs
Run a query to see the available data (FAILED_RDP_WITH_GEO_CL)
May take some time for Azure to sync VM and Log Analytics
 
![Workaroundforextraction](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/d415626f-42fa-4cd6-a16e-bc04f4f93b21)

Step 11: Extract Fields from Custom Log
The RawData within a log contains information such as latitude, longitude, destinationhost, etc. Data will have to be extracted to create separate fields for the different types of data
Azure has stopped using extract so theres a way to work around extracting the data

Run FAILED_RDP_WITH_GEO_CL
| extend username = extract(@"username:([^,]+)", 1, RawData),
         timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
         latitude = extract(@"latitude:([^,]+)", 1, RawData),
         longitude = extract(@"longitude:([^,]+)", 1, RawData),
         sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
         state = extract(@"state:([^,]+)", 1, RawData),
         label = extract(@"label:([^,]+)", 1, RawData),
         destination = extract(@"destinationhost:([^,]+)", 1, RawData),
         country = extract(@"country:([^,]+)", 1, RawData)
| project username, timestamp, latitude, longitude, sourcehost, state, label, destination, country

And it will extract organized data
Step 12 Map data 
Layout Settings
Location info using > Latitude/Longitude
Latitude > latitude
Longitude > longitude
Size by > lattitude
Color Settings
Coloring Type: Heatmap
Color by > lattitude
Aggregation for color > Sum of values
Color palette > Green to Red
Metric Settings
Metric Label > label
Metric Value > country
Select Apply button and Save and Close
Save as "Failed RDP World Map" in the same region and under the resource group  

![mapsettings](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/d23e0429-bb65-45d0-8095-182e03207baa)


Continue to refresh map to display additional incoming failed RDP attacks
Results after 24 hours

![failedupdate](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/b670ea6f-79af-4245-9437-30783b1a64a9)

 
Event viewer 4625
 showing that the events were propperly pulled from the vm
![eventviewer4625](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/7c1b89fb-e4f7-444e-b9d2-b4430423f036)


![delete](https://github.com/JamesSoria/azure-sentinel-attack-map/assets/106378237/2ac7aa7e-f1e7-458b-b7c5-40853d53b501)

Make sure to delete 
Step 13: Deprovision Resources
VERY IMPORTANT - Do NOT skip!

Search for "Resource groups" > name of resource group (honeypotlab) > Delete resource group
Type the name of the resource group ("honeypotlab") to confirm deletion
Check the Apply force delete for selected Virtual machines and Virtual machine scale sets box
Select Delete
