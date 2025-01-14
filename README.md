# Failed-RDP-Sentinel

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/HoneyPot.png)

Demonstration of integrating and configuring Azure Sentinel as a SIEM solution with real-world use cases for monitoring, threat detection, and incident response

<h2>Description</h2>
SIEM, or Security Information and Event Management System, is a solution designed to help organizations identify, analyze, and address security threats before they disrupt business operations. It collects event log data from various network sources, including firewalls, IDS/IPS, and identity management systems. This data enables security professionals to monitor, prioritize, and mitigate potential threats in real-time. A honeypot is a security technique that creates a simulated, vulnerable system to attract and study attackers. It provides a controlled environment to observe how attackers operate, investigate different threat types, and refine security strategies. The goal of this lab is to learn how to gather honeypot attack log data, query it within a SIEM, and present it in an easily interpretable format—specifically, by visualizing event counts and geolocation data on a world map.

<h2>Learning Objectives:</h2>  

* Configuration & Deployment of Azure resources such as virtual machines, Log Analytics Workspaces, and Azure Sentinel
* Hands-on experience and working knowledge of a SIEM Log Management Tool (Microsoft's Azure Sentinel)
* Understand Windows Security Event logs
* Utilization of KQL to query logs
* Display attack data on a dashboard with Workbooks (World Map)

<h2>Technologies + Requirements</h2>

* Microsoft Azure + Account
* Azure Services: Sentinel, Log Analytics Workspace, Workbooks, Network Security Groups
* Powershell
* Remote Desktop Protocol (RDP)
* Third-party API: ipgeolocation.io
* Customized Powershell Script authored by Josh Madakor

<h2>Step 1: Create Microsoft Azure Account</h2>

![image](https://github.com/user-attachments/assets/7419e070-3074-4d0a-ad8b-ee7de816ab17)


<h2>Step 2: Configure Honeypot VM</h2>

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Create%20VM.png)

<h1>Basics</h1>

* After signing up, click "Go to the Azure Portal" , or visit portal.azure.com
* In the search bar type "virtual machines"
* Under Create tab click on Azure virtual machine

<b>Project Details</b>

* Create new resource group and name it "honeypotlab"

<b>Instance Details</b>

* Name your VM (honeypot-vm)
* Select a recommended region ((US) East US 2)
* Availability options: No infrastructure redundancy required
* Security type: Standard
* Image: Windows 10 Pro, version 21H2 - x62 Gen2
* VM Architecture: x64
* Size: Default is okay (Standard_D2s_v3 – 2vcpus, 8 GiB memory)

<b>Administrator Account</b>

* Create a username and password for virtual machine

<b>Import Port Rules</b>

* Public inbound ports: Allow RDP (3389)

<b>Licensing</b>

* Confirm licensing
* Select <b>Next : Disks ></b>

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Configure%20VM%20Settings%201.png)

<h1>Disks</h1>

* Leave all defaults
* Select <b>Next : Networking ></b>

<h1>Networking</h1>

<b>Network Interface</b>

* NIC network security group: Advanced > Create new
* Remove Inbound rules (1000: default-allow-rdp) by clicking three dots
* Add an inbound rule
* Destination port ranges: * (wildcard for anything)
* Protocol: Any
* Action: Allow
* Priority: 100 (low)
* Name: Anything (ALLOW_ALL_INBOUND)
* Select Review + create

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Configure%20VM%20Settings%202.png)

<h2>Step 3: Create Log Analytics Workspace</h2>

* Search for "Log analytics workspaces"
* Select Create Log Analytics workspace
* Put it in the same resource group as VM (honeypotlab)
* Give it a desired name (honeypot-log)
* Add to same region (East US 2)
* Select Review + create

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Create%20Log%20Analytics%20Workspace.png)

<h2>Step 4: Setup Microsoft Defender for Cloud</h2>

* Search for "Microsoft Defender for Cloud"
* Scroll down to "Environment settings" > subscription name > log analytics workspace name (log-honeypot)

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Microsoft%20Defender%20for%20Cloud%20Environment%20settings.png)

<b>Settings | Defender plans</b>

* Cloud Security Posture Management: ON
* Servers: ON
* SQL servers on machines: OFF
* Hit Save

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Defender%20Settings.png)
<b>Settings | Data Collection</b>

* Select "All Events"
* Hit <b>Save</b>

<h2>Step 5: Link VM to Log Analytics Workspace</h2>

* Search for "Log Analytics workspaces"
* Select workspace name (log-honeypot) > "Virtual machines" > virtual machine name (honeypot-vm)
* Click <b>Connect</b>

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Connect%20LAW%20to%20VM.png)

<h2>Step 6: Setup Microsoft Sentinel</h2>

* Search for "Microsoft Sentinel"
* Click <b>Create Microsoft Sentinel</b>
* Select Log Analytics workspace name (honeypot-log)
* Click <b>Add</b>

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Add%20Sentinel%20to%20workspace.png)

<h2>Step 7: Turn Virtual Machine's Firewall OFF</h2>

* Go to Virtual Machines and find the honeypot VM (honeypot-vm)
* By clicking on the VM copy the IP address
* Log into the VM via Remote Desktop Protocol (RDP) with credentials from step 2
* Accept Certificate warning
* Select NO for all <b>Choose privacy settings for your device</b>
* Click <b>Start</b> and search for "wf.msc" (Windows Defender Firewall)
* Click "Windows Defender Firewall Properties"
* Turn Firewall State OFF for <b>Domain Profile Private Profile</b> and <b>Public Profile</b>
* Hit <b>Apply</b> and <b>Ok</b>
* Ping VM via Host's command line to make sure it is reachable ping -t <VM IP>

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/defender_off.png)

<h2>Step 8: Automate Security Log Exporter</h2>

* In VM open Powershell ISE
* Set up Edge without signing in
* Copy [Powershell script](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1) into VM's Powershell (Written by Josh Madakor)
* Select <b>New Script</b> in Powershell ISE and paste script
* Save to Desktop and give it a name (Log_Exporter)

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/powershell_script.png)

* Make an account with [Free IP Geolocation API and Accurate IP Lookup Database](https://ipgeolocation.io/)
  > This account is free for 1000 API calls per day. Paying 15.00$ will allow 150,000 API calls per month.
* Copy API key once logged in and paste into script line 2: $API_KEY = "<API key>"
* Hit <b>Save</b>
* Run the PowerShell ISE script (Green play button) in the virtual machine to continuously produce log data

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/ipgeolocation.png)
  > The script will export data from the Windows Event Viewer to then import into the IP Geolocation service. It will then extract the latitude and longitude and then create a new log called failed_rdp.log in the following location: C:\ProgramData\failed_rdp.log

<h2>Step 9: Create Custom Log in Log Analytics Workspace</h2>

* To add the extra information from the IP Geolocation service to Azure Sentinel, create a custom log
* Search "Run" in VM and type "C:\ProgramData"
* Open file named "failed_rdp" hit <b>CTRL + A</b> to select all and <b>CTRL + C</b> to copy selection
* On the host PC, open notepad and paste the information
* Save to desktop as "failed_rdp.log" as (.txt) text file
* In Azure go to Log Analytics Workspaces -> Log Analytics workspace name (honeypot-law) -> Custom logs -> <b>Add custom log</b>

<h3>Sample</h3>

* Select Sample log saved to Desktop (failed_rdp.log) and click <b>Next</b>

<h3>Record Delimiter</h3>

* Look over sample logs and click <b>Next</b>

<h3>Collection Paths</h3>

* Type: Windows
* Path: "C:\ProgramData\failed_rdp.log

<h3>Details</h3>

* Name and describe the custom log (FAILED_RDP_WITH_GEO) before pressing the <b>Next</b> button
* Click Create

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/custom_log.png)

<h2>Step 10: Query + Extract Fields from Custom Log</h2>

* Navigate to the newly established workspace (honeypot-law) in Log Analytics Workspaces -> Logs
* We then can run a query and extract the different data filtering by different fields such as latitude, longitude, destinationhost, etc.
  > As of March 31st, 2023, Microsoft has disabled the creation of new custom fields and has migrated to KQL. You can learn more about it here
* Copy/Paste the following query into the query window and Run Query

```
FAILED_RDP_WITH_GEO_CL 
| extend username = extract(@"username:([^,]+)", 1, RawData),
         timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
         latitude = extract(@"latitude:([^,]+)", 1, RawData),
         longitude = extract(@"longitude:([^,]+)", 1, RawData),
         sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
         state = extract(@"state:([^,]+)", 1, RawData),
         label = extract(@"label:([^,]+)", 1, RawData),
         destination = extract(@"destinationhost:([^,]+)", 1, RawData),
         country = extract(@"country:([^,]+)", 1, RawData)
| where destination != "samplehost"
| where sourcehost != ""
| summarize event_count=count() by timestamp, label, country, state, sourcehost, username, destination, longitude, latitude
```
> Kusto Query Language (KQL) is used to query and extract logs from data stored in Azure Log Analytics or Azure Data Explorer. KQL is a powerful and expressive query language that allows you to perform advanced data analysis, filtering, aggregation, and visualization. With some practice composing questions and simple instructions, the language is meant to be simple to read and use.

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/LAW%20KQL%20script%20for%20log.png)

<h2>Step 11: Create World Attack Map in Microsoft Sentinel</h2>

* Access Microsoft Sentinel to view the Overview page and available events
* Click on Workbooks and <b>Add workbook</b> then click Edit
* Delete default widgets (three dots -> remove)
* Click <b>ADD</b>-><b>Add query</b>
* You can Copy/Paste the previous query or this one into the query window and Run Query

```
FAILED_RDP_WITH_GEO_CL 
| extend username = extract(@"username:([^,]+)", 1, RawData),
         timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
         latitude = extract(@"latitude:([^,]+)", 1, RawData),
         longitude = extract(@"longitude:([^,]+)", 1, RawData),
         sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
         state = extract(@"state:([^,]+)", 1, RawData),
         label = extract(@"label:([^,]+)", 1, RawData),
         destination = extract(@"destinationhost:([^,]+)", 1, RawData),
         country = extract(@"country:([^,]+)", 1, RawData)
| where destination != "samplehost"
| where sourcehost != ""
| summarize event_count=count() by latitude, longitude, sourcehost, label, destination, country
```
* When results appear, select <b>Map</b> from the <b>Visualization</b> drop-down box.
* Choose <b>Map Settings</b> to make additional adjustments
  > Most settings should be auto-configured from the script above
  
![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Sentinel%20KQL%20script.png)

<b>Layout Settings</b>

* <b>Location info using</b>: Latitude/Longitude
* <b>Latitude</b>: latitude
* <b>Longitude</b>: longitude
* <b>Size by</b>: event_count

<b>Color Settings</b>

* Coloring Type: Heatmap
* <bColor by</b>: event_count
* <b>Aggregation for color</b>: Sum of Values
* </b>Color palette</b>: Green to Red

<b>Metric Settings</b>

* <b>Metric Label</b>: label
* <b>Metric Value</b>: event_count
* Click <b>Apply</b> button and <b>Save and Close</b>
* Save as "Failed RDP International Map" in the same region and under the resource group (honeypot-lab)
* Keep refreshing the map to show more inbound failed RDP attacks
  > Note: Only unsuccessful RDP attempts will be shown on the map, not any additional attacks the VM might be facing.
  
  ![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Failed%20RDP%20Map.png)

  > Event Viewer showcasing failed RDP logon efforts. Event ID: 4625
  
  ![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Event%20Manager.png)

  > Data processing from a custom Poweshell script using a third party API
  
  ![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/ISE%20failed%20attempts.png)

<h2>Step 12: Shut Down Resources</h2>.

> CRUCIAL: DON'T SKIP !

* Look for "Resource groups" -> name of resource group
* Key in the name of the resource group (honeypot-lab) to verify removal of resources
* Select the <b>Apply force delete for selected Virtual machines and Virtual machine scale sets</b> box
* Click <b>Delete</b>

![image](https://github.com/WalterDotson02/Failed-RDP-Sentinel/blob/main/Images/Screenshot%202025-01-14%20092133.png)

> Resources will use free credits if they are not eliminated, and costs may start to accrue.



