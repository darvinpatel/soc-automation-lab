# Open Source Security Operations Center (SOC) | Wazuh | The Hive | Shuffle | Virus Total

### Summary
- The project involves creating a Security Operations Center (SOC) Automation system in a home lab environment.
- The aim is to set up a fully functional Wasuh instance with SOAR integration and implement case management using The Hive.
- The project consists of five labs, focusing on hands-on experience for day-to-day security operations

![](images/img1.png)
>FINAL RESULTS: Visual map displaying failed RDP login attempts to a honeypot vm sorted by the location using IP and the attack counter


### Technologies:
* Microsft Azure - a cloud computing service operated by Microsoft for application management via Microsoft-managed data centres
* Services within Azure: Log Analytics Workspace and Sentinel (Mircosoft's SIEM)
* Powershell 
* Remote desktop protocol

## Lab 1:
### Step 1: Lab Planning
- First, we will be creating a logical diagram to plan and map out the lab setup.
- The emphasis is on building a visual representation to understand the flow of data and identify the components necessary for the system to function.
- I recommend using draw.io for creating the diagram due to its accessibility and free usage.

### Step 2: Diagram Creation Steps
- The diagram includes components such as a Windows 10 client, router, internet (cloud), Wasuh manager, The Hive, Shuffle, and a SOC analyst workstation.
- Logical connections are established using arrows to represent the flow of events and actions between these components.
- Different arrow colours and types are used to signify various actions, such as sending events, receiving events, sending alerts, enriching IOCs (Indicators of Compromise), sending emails, and responding to actions.

### Step 3: Workflow Overview
- The logical workflow involves the Windows 10 client sending events to Wazuh manager, triggering alerts or responsive actions.
- Wazuh manager sends alerts to Shuffle, which then performs actions like enriching IOCs, sending alerts to The Hive, and sending emails to the SOC analyst.
- The SOC analyst may respond to alerts, initiating actions that are communicated back through the system to ultimately instruct the Windows 10 client.

### Step 4: Lab Progression

- The diagram serves as a reference for the subsequent parts of the series, where the focus will shift to the installation of virtual machines.
- The goal is to build a functional SOC Automation system with three hosts, including one PC and two servers (The Hive and Wazah) residing in the cloud.

## Lab 2:
> Lab 2 aims to install applications and virtual machines, targeting one Windows 10 machine with Sysmon, one Wazuh server, and one Hive server.
> Virtual machines for Wazuh and Hive will be set up in the cloud, while the Windows 10 client will be hosted locally on a server.

### Step 1: Wazuh Overview
- Wazuh is described as an open-source cybersecurity platform integrating SIEM and XDR capabilities, providing features like security analytics, intrusion detection, and incident response.
- The three main components of Wazuh are the indexer, server, and dashboard.

### Step 2: Virtual Machine setup

- Windows 10 and Ubuntu 20.04 are chosen as the client and server operating systems, respectively.
- Virtual Box is selected as the virtualization tool for installation. The process includes downloading, verifying, and installing Virtual Box on the host machine.

### Step 3: Sysmon Installation

- Sysmon, a system monitoring tool for Windows, is installed on the Windows 10 client to capture telemetry data and enhance security monitoring.
- The process involves downloading the Windows ISO, setting up a virtual machine in Virtual Box, and then installing and configuring Sysmon on the Windows client machine.

  
### Lab 3:
> The objective of this lab is to configure Hive and Wazuh servers in the SOC Automation Project Home Lab.

### Step 1: Hive Configuration

- Cassandra configuration files are edited to customize listen address, RPC address, and seed address.
- Cassandra service is stopped, old files are removed, and the service is started again.
- Elasticsearch configuration files are edited, and the service is started and enabled.
- Ownership of certain directories is changed to allow the Hive user access.
- The Hive configuration file is edited to set database and index configurations.

### Step 2: Hive Verification

- Hive services are started and enabled, and their status is verified to be active and running.
- The Hive dashboard is accessed via the public IP and Port 9000.

>Elasticsearch Troubleshooting: In case of login errors, Elasticsearch memory allocation is adjusted using a custom JVM option file.

### Step 3: Wazuh Configuration

- Administrative credentials are used to log into the Wasa dashboard.
- An agent is added for a Windows machine using its public IP, and the agent is installed using a provided command.
- The Wasa service is started, and the agent's status is verified on the dashboard.
> Note: The overall goal is to detect Mimikatz usage on the Windows 10 client machine, and Telemetry generation and alert creation for Mimikatz will be covered in the next episode.





## FINAL STEP: Deprovision resources 
- Once you are done with the lab delete the resources, otherwise they will eat away from your free credit (deprovisioning is also a good thing to keep in mind at the enterprise level)
- Search and click Resource group > honeypot-lab > Delete resource group
- Type the name  *honeypot-lab* to confirm deletion 



> And there you have it, you have successfully mapped out the location of your RDP attackers using a honey pot vm.

