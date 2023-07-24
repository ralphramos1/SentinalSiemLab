<h1>Azure Sentinal Siem Home Lab</h1>

<h2>Description</h2>
In this immersive experience, I have established Azure Sentinel, a SIEM solution, and successfully linked it to a live virtual machine, which I've designated as a honey pot. Through this setup, we will have the opportunity to witness real-time cyber attacks, specifically RDP Brute Force attempts originating from various global locations. To enhance our analysis, we will utilize a custom PowerShell script to retrieve the geolocation information of these attackers and dynamically plot it on the interactive Azure Sentinel Map, providing us with valuable insights into the geographical distribution of the threat landscape. Let the exploration of live cyber attacks and the power of Azure Sentinel's visualization capabilities begin!
<br />


<h2>Utilities Used</h2>

- <b>PowerShell</b> 

<h2>Environments Used </h2>

- <b>Windows 10</b> (21H2)

<h2>Program walk-through:</h2>

<p align="left">
Using Microsoft Azure, I was able to create a Windows 10 virtual machine. To establish an open environment for attackers, I enabled the network security group (NSG) to allow traffic from all ports and IP addresses.<br/>
<img src="https://i.imgur.com/MM6MdXd.png" height="100%" width="100%"><br>
<br />
<br />
To capture all events, you can navigate to the Security Center settings and access the "Environment settings" on the left blade. Once there, simply enable the Defender feature and set the data collection to capture all events.<br/>
<img src="https://i.imgur.com/x8uSMZf.png" height="100%" width="100%"><br>
<br>
Next step is to connect our log analytics workspace to the virtual machine.<br/>
<p align="center"><img src="https://i.imgur.com/RFPZt7Y.png" height="50%" width="50%"><br>
<br>
<p align="left">
After configuring the settings in Azure Defender to gather all Windows security events, I linked my Log Analytics workspace to the previously set up virtual machine. We then RDP into our virtual machine, where a tailor-made PowerShell script was employed, leveraging a third-party API service called "ipgeolocation," allowing the retrieval and logging of geo data pertaining to potential attackers. This data was then incorporated into a custom log, specifically designed within my Azure Log Analytics workspace, to effectively import the logged information from the virtual machine.  <br/>
<img src="https://i.imgur.com/I7Nl3s5.png" height="100%" width="100%">
<br />
<br />
Creating custom logs under log analytics workspace that would allow to use our custom logs with the geodata.
<br>
<br>
Select Tables on the left blade -> New Custom logs(MMA based)In the new custom log blade enter details like log name ,description and its source.Make sure the source is the same as the source where log file is stored in the virtual machine.In our case C:\ProgramData\failed_rdp.log.

Custom log is now created and can be used as a query in the logs section of log analytics workspace.<br/>
<img src="https://i.imgur.com/r6CZtWS.png" height="100%" width="100%">
<br />
<br />
Extracting custom fields from the query result. We use the extend option to extract data from the query result.The query used is given below:<br>
<br>
FAILED_RDP_WITH_GEO_CL
| extend username = extract(@”username:([^,]+)”, 1, RawData),<br>
timestamp = extract(@”timestamp:([^,]+)”, 1, RawData),<br>
latitude = extract(@”latitude:([^,]+)”, 1, RawData),<br>
longitude = extract(@”longitude:([^,]+)”, 1, RawData),<br>
sourcehost = extract(@”sourcehost:([^,]+)”, 1, RawData),<br>
state = extract(@”state:([^,]+)”, 1, RawData),<br>
label = extract(@”label:([^,]+)”, 1, RawData),<br>
destination = extract(@”destinationhost:([^,]+)”, 1, RawData),<br>
country = extract(@”country:([^,]+)”, 1, RawData)<br>
| where destination != “samplehost”<br>
| where sourcehost != “”<br>
| summarize event_count=count() by latitude, longitude, sourcehost, label, destination, country<br/>
<br>
Setting up a map in Sentinel. Creating a new workbook in Azure Sentinel and adding a query which is the same we used earlier to extract custom fields from the logs.
<img src="https://i.imgur.com/19cRIjK.png" height="100%" width="100%">
<br />
<br />
We utilize the map visualization type and customize the map settings to suit our preferences. With this, we can plot the map using longitude/latitude, country, and other relevant data. The specific map settings I used are listed below.<br/>
<img src="https://i.imgur.com/rgwaZuI.png" height="100%" width="100%">
<br />
<br />
We are nearing the completion of setting up Sentinel. Now, configure the auto-refresh option to occur every 5 minutes and patiently await various attackers to discover and attempt to breach our vulnerable virtual machine.<br/>
<br />
<br />
While setting this up, you can see we already have 2 attempts from India and Singapore(the 4 from US was from me for testing purposes)<br/>
<img src="https://i.imgur.com/NtHtq3h.png" height="100%" width="100%">
<br>
<br>
After leaving this on for 1 full day, here are the results.<br>
<img src="https://i.imgur.com/NtHtq3h.png" height="100%" width="100%">
<br>
<br>
<b>Conclusion</b><br>
A SIEM (Security Information and Event Management) enables us to investigate, record, and analyze information and events related to system security. It proves to be a valuable tool for detecting potential security threats. In this home-lab, we showcase the utilization of Microsoft Azure Sentinel to effectively capture and analyze these crucial security events.
<br>
<br>
<b>Key Concepts</b><br>
- Deploying and establishing a connection to a Virtual Machine using RDP (Remote Desktop Protocol).<br>
- Manipulating Inbound Rules to regulate data traffic, and to allow traffic from any port or IP address.<br>
- Leveraging Azure Sentinel to record and monitor security events on a Virtual Machine.<br>
- Utilizing Log Analytics Workspace to efficiently log and organize collected data.<br>
- Analyzing the data obtained from security logs in the Event Viewer.<br>
- Employing PowerShell ISE to extract valuable insights and information.<br>
</p>
