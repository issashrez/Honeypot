<h1>Honeypot hosted on Azure Cloud to monitor failed RDP logins</h1>

<h2>Description</h2>
This project was created to help understand the adversity of having a system's IP address public. The vulnerable system then gets exposed to everyone in the world, and the attackers' geolocation is then mapped.
<br />


<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>Azure Cloud</b>
- <b>Azure Log analytics workspace</b>
- <b>Microsoft Defender</b>
- <b>SIEM: Microsoft Sentinel</b>
- <b>Azure windows VM</b>
- <b>Remote Desktop Connection</b>
- <b>Kusto Query Language(KQL)</b>

<h2>Environments Used </h2>

- <b>Windows 10</b> (21H2)

<h2>Program walk-through:</h2>

<h3>1. Setting up Azure cloud and VM:</h3>
<p>We begin by creating a Microsoft Azure account. It is not necessary to pay for any subscriptions as 200$ will be provided if this is the first time setting up.

Once the account has been set up, go to the Azure portal page. Search for virtual machines. Now we have to create a Windows machine to become our honeypot. We are using a Virtual Machine to do this task because the project aims to gain attention and gather attacks from threat actors. Hence, a VM is <b>recommended</b>.

Follow along to create a VM:
<p align="center">
<img src="https://i.imgur.com/cUz336x.jpg" height="80%" width="80%"><br>
Fig 1.1<br>
<img src="https://i.imgur.com/6c99bca.jpg" height="80%" width="80%"><br>
Fig 1.2<br>
<img src="https://i.imgur.com/oBOH2Kl.jpg" height="80%" width="80%"><br>
Fig 1.3<br>
<img src="https://i.imgur.com/SoErxwq.jpeg" height="80%" width="80%"><br>
Fig 1.4<br>
<img src="https://i.imgur.com/frVVRsN.jpeg" height="80%" width="80%"><br>
Fig 1.5<br>
<img src="https://i.imgur.com/tvSpbvc.jpg" height="80%" width="80%"><br>
Fig 1.6<br>
</p>

It is not necessary to have the same names in the above-given fields of all the steps. However, other technical requirements should remain the same.
</p>

<h3>2. Setting the environment for the VM:</h3>
<p>In the search bar above, search for Log analytics workspace. We set this up to ensure that the VM's log data is ingested and stored in the workspace. Essentially, we are connecting the VM service to the log analytics service.</p><br>
<p align="center">
<img src="https://i.imgur.com/yZdSOLh.jpg" height="80%" width="80%"><br>
Fig 2.1<br></p>
<p>Next, search for Microsoft Defender. This a built-in program to defend the machine from any attacks. Similar to an antivirus, but host-based. To make our system vulnerable, it is required to tweak the settings. By default, all settings are set. However, from Fig 2.2, we set the SQL server off as we won't be using it in this project. We keep the servers ON because we need them for receiving and sending data across the network. </p><br>
<p align="center">
<img src="https://i.imgur.com/v5WyCpy.jpg" height="80%" width="80%"><br>
Fig 2.2<br></p>
<p>Now go back to the Log Analytics workspace and connect the created workspace.</p><br>
<p align="center">
<img src="https://i.imgur.com/Td2pxdd.jpg" height="80%" width="80%"><br>
Fig 2.3<br></p>
<p>Then, we connect the created workspace to sentinel. Microsoft Sentinel is a SIEM and it's designed to raise events or alerts when a rule is triggered while analyzing logs.</p><br>
<p align="center">
<img src="https://i.imgur.com/8C8vjSD.jpg" height="80%" width="80%"><br>
Fig 2.4<br></p>
<p>The VM and the backend environment have been successfully connected and set up. Now, copy the public IP address from your VM dashboard. We shall use Remote Desktop Connection (RDC) to connect to the VM and initiate it. </p><br>
<p align="center">
<img src="https://i.imgur.com/p0yTPBk.jpg" height="80%" width="80%"><br>
Fig 2.5<br></p>
<p align="center">
<img src="https://i.imgur.com/Logj3J0.jpg" height="80%" width="80%"><br>
Fig 2.6<br></p>

<h3>3. Getting familiar with the VM:</h3>
<p>Once the VM opens through RDC, search for Event Viewer->Security. This provides a summary of all the audit events that have happened on the machine. Go ahead, and open the RDC (<b>on your host machine</b>), Provide the actual IP address but give the wrong username and click on connect. This will trigger an event on the VM and it will be logged in the Event Viewer as "Audit Failure". You can double-click on the event to read more.</p><br>
<p align="center">
<img src="https://i.imgur.com/3kxcreJ.jpg" height="80%" width="80%"><br>
Fig 3.1<br></p>
<p>Now we have to check the network connection of the VM. To do this, open cmd(<b>on the host machine</b>) and run the command "ping {ip address of VM} -t". This is a normal ping command to check a system and initiate a connection with another system. But we can see that the output of the command indicates otherwise (Fig 3.2). This is due to Firewall configurations. Open the firewall(On the VM) using "wf.msc" in the run dialog box. Go to Defender Firewall properties and set the Firewall state as OFF in all the profiles. We have successfully deactivated the firewall, hence we are letting in ALL the inbound and outbound traffic without any restrictions. Try the ping command again. This time the VM would be reachable for the host. (Fig 3.4)</p><br>
<p align="center">
<img src="https://i.imgur.com/hYFuZ4y.jpg" height="80%" width="80%"><br>
Fig 3.2<br></p>
<p align="center">
<img src="https://i.imgur.com/UbLiumN.jpg" height="80%" width="80%"><br>
Fig 3.3<br></p>
<p align="center">
<img src="https://i.imgur.com/qsaHEk8.jpg" height="80%" width="80%"><br>
Fig 3.4<br></p>

<h3>4. Scripting and logging:</h3>
<p>It is time for some PowerShell scripting! Go ahead and download the Log_Exporter.ps1 file. This file essentially contains a script that connects to a geolocation mapper ie ipgeolocation.io - check Fig 4.1. It connects via an API key. It then extracts information from the Event Viewer, specifically Audit Failures with event ID 4625.

In the script, there is a function which is a sample log, and this log is of the same format as that we get from the API. We use this sample to train the Log Analytics workspace to then extract any new real log data. All this data is stored in failed_rdp.log in C:/ProgramData. New logs will also be added here. We shall use this log file later.
</p><br>
<p align="center">
<img src="https://i.imgur.com/I6NBUki.jpg" height="80%" width="80%"><br>
Fig 4.1<br></p>
<p>This is the major information that we would be extracting. This is then used to plot on a map. Also, the API key can be gathered from this website. Feel free to use your own API worth 1000 requests per day by creating an account.</p><br>
<p align="center">
<img src="https://i.imgur.com/uDVS4hq.jpg" height="80%" width="80%"><br>
Fig 4.2<br></p>
<p>We can see that we are already getting logs of other attackers trying to log into our system.</p><br>
<p>Next, on our host machine, go to the Log Analytics workspace(LAW) and create a custom log. Provide a sample log - Fig 4.3. This .log file is saved on the host machine. Then we provide a collection path. This leads to the C:\ProgramData\failed_rdp.log in our VM. Also, name the custom log. Here I have named it "FAILED_RDP_WITH_GEO". Great, we have linked the log in the VM to our workspace in the host.</p><br>
<p align="center">
<img src="https://i.imgur.com/fzWIv36.jpg" height="80%" width="80%"><br>
Fig 4.3<br></p>
<p>Now in LAW, go to the logs section and modify the created custom log. Why modify> The log gathered is scrambled and not in a presentable format. KQL queries are used here. </p><br>
<p align="center">
<img src="https://i.imgur.com/1HdIIp0.jpg" height="80%" width="80%"><br>
Fig 4.4<br></p>
<p>In this code, we extract info from the log and store it in individual variables. For example, username = extract(@"username:([^,]+)", 1, RawData):
This line extracts the value of the username from the RawData field. It uses a regular expression to match the pattern "username:[^,]+" and extracts the first capturing group (specified by 1). This captures everything after "username:" up to the first comma. Then we add a few WHERE clauses to ensure that the sample data (that we used for training) shouldn't appear on the map.</p><br>

<h3>5. Map the outputs: </h3>
<p>Head over to Microsoft Sentinel Workspace(MSW) and click on workbooks. Paste the same KQL queries here. Then customize the map settings.</p><br>
<p align="center">
<img src="https://i.imgur.com/8EQFthl.jpg" height="80%" width="80%"><br>
Fig 5.1<br></p>
<p>Now, open the workspace and observe the map. Ensure that the Powershell script is running in the background, only then new logs are entered. Check the initial output I observed below in Fig 5.2.</p><br>
<p align="center">
<img src="https://i.imgur.com/ZcPC6Xk.jpg" height="80%" width="80%"><br>
Fig 5.2<br></p>
<p>Observe the drastic difference in the count after 20 mins or so - Fig 5.3.</p><br>
<p align="center">
<img src="https://i.imgur.com/a47irhT.jpg" height="80%" width="80%"><br>
Fig 5.2<br></p>

<h2>Takeaways and best practices: </h2>
- <b>Use a unique username:</b><p> One of the important takeaways from the log outputs is the number of attempts by the attackers trying to gain access to our honeypot using the username "Administrator". An individual must follow best security practices for good secure username credentials.</p>
- <b>Careful configuration: </b><p>When you configure a machine or a system by tweaking the networking configurations or the adapters, it's necessary to be mindful of the changes being made. Any wrong setting would expose sensitive information to the public at large.</p>
- <b>Using strong passwords: </b><p>Follow safe and best password practices. Use a lengthy password with multiple unique characters including case letters, numbers, and special symbols.</p>
- <b>Use MFA: </b><p>Using a multifactor authentication system would ensure that there are multiple layers for logging in, making hacking difficult for the attackers.</p>
- <b>Setup firewall: </b><p>Use a firewall to control inbound and outbound traffic. Ensure that random RDP connections are not entertained. Additionally, have an antivirus software installed.</p>





