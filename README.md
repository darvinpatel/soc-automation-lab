![](images/Azurelogo.png)

# Open Source Security Operations Center (SOC) | Wazuh | The Hive | Shuffle | Virus Total

### Summary
- The project involves creating a Security Operations Center (SOC) Automation system in a home lab environment.
- The aim is to set up a fully functional Wasuh instance with SOAR integration and implement case management using The Hive.
- The project is of five parts, focusing on hands-on experience for day-to-day security operations

![](images/img1.png)
>FINAL RESULTS: Visual map displaying failed RDP login attempts to a honeypot vm sorted by the location using IP and the attack counter


### Technologies:
* Microsft Azure - a cloud computing service operated by Microsoft for application management via Microsoft-managed data centers
* Services within Azure: Log Analytics Workspace and Sentinel (Mircosoft's SIEM)
* Powershell 
* Remote desktop protocol
 
### Overview:
![](images/azure.png)

> Step-by-step overview of lab:
1. Create Azure subscription (FREE $200 credits)
2. Create virtual machine in Azure (honeypot-vm) > turn firewalls off (making it vulnerable to brute force attacks) 
3. Use a Powershell script to extract  IP of attackers > feed IP into third party API and return back to honeypot-vm specific location information.
4.  Create log repository in Azure (Log Analytics Workspace) - this will ingest our logs from honeypot-vm
5. Set up Sentinel - Microsoft’s cloud native SIEM
6. Use data from SIEM to map out attacker information and magnitude 

## Step 1: Create FREE Azure account: [Azure](https://azure.microsoft.com/en-us/free/ "Azure")
- Click on “Go to the Azure Portal” or go to `portal.azure.com` once you create your account.

![](images/img2.PNG)

## Step 2: Create our honey pot virtual machine
- In the search bar, search and click virtual machine 
- This will be the honey pot virtual machine made to entice attackers from all over the world

![](images/img3.PNG)

## Step 3: On the “virtual machines” page click Create > Azure virtual machine 
- Edit the virtual machine as follows:
- Click create new under resource group and name it (this resource group is a logical grouping of similar resources)
- Name the virtual machine
- Under region select
- Under Image select
- Availability zone
- Size
- Create a username and password - **don’t forget credentials**
- Finally, check confirm box - leaving the rest in their default options  

![](images/img10.PNG)

## Step 4: Click > Next: Disk but leave it as is, click to continue to Networking
-  Under *NIC network security group* select > Advance and under *Configure network security group* select Create new
- You should see a default rule (something like 1000: default-allow-rdp), click the three dots to the right of it and **remove** it.
- Select *Add an inbound rule* 
- Match the settings of the new rule as follows: 
- Set *Destination port ranges*: * 
- Priority: 100
- Name: Danger_All_Traffic_Inbound_Allowed
- Leave the rest of the settings as default
- Click Add > OK > Review + create - wait a bit to load and click Create

![](images/img11.PNG)
![](images/img12.PNG)

> The point of this new firewall rule is to allow any traffic from anywhere.  This will make our virtual machine very discoverable. 

## Step 5: Create Log Analytics workspace
- As we wait for our vm to deploy, go back to the search bar and search and click *Log Analytics workspaces*

![](images/img13.PNG)
![](images/img14.PNG)

> The purpose  of this workspace is to ingest logs from our vm. Additionally, we will create our own custom logs that will contain geographic information on who is attacking us. Later, our MS SIEM will feed logs into here.

- Select the blue Create log analytics workspace button
- Under the Basics tab
- Resource source group
- Name
- Region
- Click Review + Create and click Create

## Step 6A: Enable log collection from vm to log workspace
- Back in the search bar search and click *Microsoft Defender for Cloud*
- Once on the dashboard click > Environment Settings > (through the drop down menus) > log-honeypot-1

![](images/img15.PNG)
![](images/img16.PNG)

## Step 6B: Under log-honeypot-1 select *Defender Plans* and enable *Servers* ON and *SQL servers on machines* OFF. With *Cloud Security Posture Management* ON. Hit save.
- Under *Data Collection* tab select *All Events*. Hit save.

![](images/img17.PNG)
![](images/img18.PNG)


## Step 7: connect Log Analytics workspace to our vm
- On the search bar select Log Analytics workspace
- Select log-honeypot-11 > Virtual Machines > honeypot-vm
- Click **connect**, after clicking honeypot-vm
- It will take some time to successfully connect; you should get a message confirming connection.

![](images/img19.PNG)
![](images/img20.PNG)

## Step 8: Add Microsoft Sentinel to our workspace 
- In search bar find **Microsoft Sentinel**
- Click Create Microsoft Sentinel > select log-honeypot-11 > Add
- This will also take some time

![](images/img21.PNG)


## Step 9A: Log into vm through host machine
- Through the search bar, find our honeypot-vm > copy the Public IP address
![](images/img22.PNG)


## Step 9B: RDP from host Windows machine
- On your Windows machine (Windows vm will also work) search and open *Remote Desktop Connection*
- Paste your Azure IP into *Computer*
- Before connecting, click Display and scale down display configuration for easier viewing
- Click connect
- In the *Enter your credentials* window click more choices > Use a different account 
- Enter invalid credentials tin order to generate a log for later viewing.
- Then, enter your credentials we created for our Azure vm in Step 3, click OK.
- Accept the certificate warning
- You should be logged into the vm when you see “Remote Desktop Connection” at the top of the screen.

![](images/img23.PNG)
![](images/img24.PNG)
![](images/img25.PNG)

> Please note that I have used Microsoft Remote Desktop to RDP into my VM as my host machine is Mac OS X

## Step 10A: Set up vm and explore 
- Click NO to all privacy settings and Accept
- Set up Edge
- Search and click *Event Viewer*
- Click Windows Logs > Security and find the Audit Failure log (our failed login attempt; if you don’t see it at first filter current log by “Audit Failure” found to the left)

![](images/img26.PNG)
![](images/img27.PNG)

> The Source Network Address will represent the attacker’s IPs and eventually where on Earth they are attacking us!
> But in order to do this we need to send this network address to a third party API… but more on that later.

## Step 10B: Turn off firewall to make vm more susceptible to attack 
- Open command prompt on your **host** machine and try to ping the Azure vm - it shouldn’t work!
- Search and open wf.msc on Azure vm - *remember* to keep an eye on vm IP at the very top to confirm you’re in the vm and NOT in on your host to avoid confusion.
- Click Windows Defender Firewall Properties near the middle of the page
- Under the Domain Profile > Firewall state: OFF
- Under Private Profile > Firewall state: OFF
- Under Public Profile > Firewall state: OFF
- Try to ping vm again from your **host** machine - this should now work!

![](images/img29.PNG)

> Before when Firewalls were in the default state of ON

![](images/img30.PNG)

> After when Firewalls were turned off intentionally

## Step 11A: Retrieve Powershell script: [Script](https://github.com/darvinpatel/sentinelMap/blob/main/Custom_Security_Log_Exporter.ps1 "Script")
- Open Powershell ISE
- For convenience you can copy/paste the code into a new ps1 file and save it to the desktop of the **honeypot-vm** (remember to see vm IP at the top)
- You will also need an API key, get here: [API key](https://ipgeolocation.io/ "API key")
- Create an account and log in
- Copy and paste *your* API key in your Powershell script `$API_KEY = “_your API key_”`
- Save file.

![](images/img28.PNG)
![](images/img31.PNG)

> Quick explanation of script: the script will parse through the security event logs (Audit Failure/failed login logs we looked at earlier) and grab IP information. The script then **passes** the IP thorough the API and correlates the info into longitude and latitude, giving us specific geographical information. 

## Step 11B: Powershell script (cont.)
- Test and run the script pass pressing green play button at top of window 
- You should receive purple logs indicating latitude / latitude of failed logins (some sample logs and some log when we failed to log in)
> NOTE: Keep Powershell script running in the backgroup. We need to continously feed our log repository information.

![](images/img32.PNG)

> ProgramData directory is where we will locate our log file after execution

![](images/img33.PNG)

> Intentional login failed attempts by myself to check whether logs are being displayed by the script

![](images/img35.PNG)

> failed_rdp filed found in the directory created by our custom Powershell script

![](images/img36.PNG)

> Logs being created by ipgeolocation.io API to our designated file

![](images/img37.PNG)

> Manually checking the logs that are created by my failed login attempts

![](images/img38.PNG)

> Final attempt to login using username darwinLive so that we can track it in Powershell ISE and log file

![](images/img39.PNG)

> logs for darwinLive login attempt successfully captured by the script

![](images/img40.PNG)

> Confirming the log files whether darwinLive failed login is available for further investigation


## Step 12A: Create custom geolocation log in Log Analytics Workspace.
- This log will use IP information to give us specific geolocation to our create map down the line.
- Search and click Log Analytics Workspace > log-honeypot-1 > custom logs > + Add custom log
- We need to upload a sample log to “train” log analytics on what to look for.

![](images/img41.PNG)
![](images/img42.PNG)

## Step 12B: Custom geolocation log (cont.)
- Our sample logs are in our **honeypot-vm**.
- **In our honeypot-vm**, search RUN > search C:\ProgramData\ and open the failed_rdp file.
- Our failed RDP logins are sent to this txt file, open and copy all the sample logs. 
- Back on our **host machine**, open notes and paste our sample logs.
- Save the file in a log or txt format and upload it in the *Create a custom log* page. Click next and you should see the sample logs.

![](images/img43.PNG)

## Step 12C: Click next and under Collection Paths > under Type > Windows, under Path write C:\ProgramData\failed_rdp.log

![](images/img44.PNG)

## Step 12D: Click next > under Details > Custom log name write FAILED_RDP_WITH_GEOLOCATION (CL will be added to the end)
- Click next > Create >Review + Create
- Let’s go back to log analytics and check if Azure is connected and listening to our vm.

![](images/img45.PNG)

## Step 12E: Secure connection between honeypot-vm and log analytics 
- Under law-honeypot1 > General > Logs > search SecurityEvent and click blue Run button.
- Give it a moment, and voila! It returns the same security logs window from our honeypot-vm’s Event Viewer.
- Give it some time and search our custom: `FAILED_RDP_WITH_GEOLOCATION_CL will` it will return our sample logs.

![](images/img46.PNG)

 
## Step 13A: Overview: Extract geo-data from the RawData of our sample logs.
- Take a look at our sample logs in our FAILED_RDP_WITH_GEOLOCATION_CL.
- In the RawData columns we find information like longitude, latitude, destination host, etc.
- We need to categorize longitude, latitude, destination host, etc. **values** from the raw data before we can obtain  geolocation data.
- It sounds a bit abstract now, but bare with me.
> NOTE: If you step away and come back to this lab after a day or two make sure to change the *Time range* accordingly. 

![](images/img47.PNG)

> Querying the logs using EventID == 4625 which signals failed logins

![](images/img48.PNG)

> Now querying the logs using the custom logs we created above


## Step 13B: Extract and categorize data from the sample log
- Use the KQL custom query to extract fields that we will use to send it to the map

![](images/img49.PNG)

> This is an important step because we are ‘training’ our SIEM what to look out for. 

## Step 13C: Check if the KQL query can capture the most recent entry
- Have another failed login to your VM and check the PowerShell results
![](images/img51.PNG)

> darwinFinalFAIL captured

## Step 13D: Check LAW for the login attempt
- After succesfully observing the failed log we check the entry in our LAW > logs
- Run the KQL again

![](images/img50.PNG)

> darwinFinalFAIL captured and categorised appoproately

> A couple of notes before moving on: **make sure our Powershell script log_exporter.log is running**. The script will continue to feed our SIEM with fresh new logs. 

> After extracting the data from our logs, you may or may not already see people trying to RDP into our vm (!). Give it some time.

## Step 14A : Set up our map within Microsoft Sentinel 
- Next, we will map out our logs within Sentinel with the extracted data - to see where in the world is our vm is being attacked from.
- Search and click Microsoft Sentinel > choose law-honeypot1 and under Threat management choose *Workbooks* > click + Add workbook
- Click edit > click the “ … “ on the right side on the screen and remove the two widgets.
- Click Add > Add query and paste the following into the query

![](images/img53.PNG)
![](images/img54.PNG)


> This will parse through the failed RDP’s logs and return to us location information through our custom fields we created.

## Step 14B: Threat visualization
- Click Run Query
- From the visualization drop box select *Map*
- In map settings > layout setting (on the right of screen):
- Configure the map according to your requirements
- On the map you see where you’re being attacked from!
- You might only see the failed logins you made, but after some time refresh and look again.
- Tip - **if you take a look at the actual logs you can see source IP, time, country, user name and other details!**
- Remember too, these logs are only reporting back failed RDP attempts… who knows what other attacks are being attempted.  

![](images/img55.PNG)


## Step 14C: Finish/save threat visualization 
- Hit > save and close
- And we’re done! - by now people should be attacking your vm, congrats!
- You can hit the refresh icon near the top of the map (**make sure Powershell script is running**) to load more logs into the map
- Also, you can click Auto refresh ON to refresh every so often.

![](images/img56.PNG)

## FINAL STEP: Deprovision resources 
- Once you are done with the lab delete the resources, otherwise they will eat away from your free credit (deprovisioning is also a good thing to keep in mind at the enterprise level)
- Search and click Resource group > honeypot-lab > Delete resource group
- Type the name  *honeypot-lab* to confirm deletion 



> And there you have it, you have successfully mapped out the location of your RDP attackers using a honey pot vm.

