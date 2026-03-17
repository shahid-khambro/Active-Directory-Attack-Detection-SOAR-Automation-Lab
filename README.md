# Active-Directory-Attack-Detection-SOAR-Automation-Lab

# Project Overview

This project simulates a Security Operations Center (SOC) detection and response workflow in an Active Directory environment. The objective of this lab was to build a realistic enterprise environment where security telemetry is collected, analyzed, and automated responses are triggered for suspicious activities.

The lab environment consists of a Windows Active Directory domain, a Splunk SIEM server, an attacker machine (Kali Linux), and a Windows 10 target host. Suspicious login activities are detected using Splunk, and automated incident response is implemented using Shuffle SOAR integrated with Slack notifications.

The playbook automatically detects unauthorized login activity, alerts the SOC analyst, and provides an option to disable the compromised domain account through automation.

This project demonstrates practical experience in:
- Active Directory administration
- SIEM log ingestion and detection engineering
- SOC alert investigation
- Security automation using SOAR
- Incident response workflow


# Lab Architecture
The lab environment simulates a small enterprise network where different machines perform specific security roles.

**Components include:**
- Active Directory Domain Controller
- Splunk SIEM Server
- Windows Target Machine
- Attacker Machine (Kali Linux)
- SOAR Automation Platform (Shuffle)
- SOC Communication Platform (Slack)

When suspicious authentication activity is detected in Splunk logs, the system triggers an automated playbook that alerts the SOC team through Slack and optionally disables the compromised user account.
<img width="1271" height="488" alt="02 active directory project 2 (1)" src="https://github.com/user-attachments/assets/bd7b22a3-0988-4865-9e35-d4090e5dde69" />

<img width="1205" height="622" alt="shuffler playbook" src="https://github.com/user-attachments/assets/caec94d1-dabd-4d8f-95f0-e8a396d56fa0"/> 

# Components Used
Component&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	Purpose \
Kali Linux&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		Attacker machine used to simulate unauthorized access \
Windows Server&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Active Directory Domain Controller\
Windows 10&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		Domain-joined test machine\
Splunk Enterprise&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SIEM platform for log collection and analysis\
Splunk Universal Forwarder &nbsp;&nbsp;&nbsp;Sends logs from endpoints to Splunk \
Sysmon &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		Endpoint telemetry collection \
Slack	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	 SOC alert notification platform  \
Shuffle SOAR &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Security orchestration and automated incident response 
# Implementation Steps
The project was completed in multiple phases that simulate the workflow of a real SOC environment.

# Phase 1: Environment Setup
The first phase involved creating the infrastructure required for the lab environment.

**Infrastructure Setup**
Three virtual machines were deployed in a cloud environment:
<img width="891" height="219" alt="image" src="https://github.com/user-attachments/assets/d3de8cf2-3404-4551-b085-5574cf2d5b0b" />

**Network Configuration**
- A VPC network was created for internal communication.
- Firewall rules were configured to allow required ports for communication between machines.
- All machines were connected using private IP addresses for internal traffic.

**Active Directory Setup**
Active Directory was installed and configured on the domain controller.

**Domain configuration:**
Domain Name: MYDFIR.local
Administrator Password: SocAnalyst123@#$%

**A new user account was created**:
Full Name: Shahid Ali
Username: shahidali
Email: shahidali@MYDFIR.local
Password: Sockhambro123#%

The Windows 10 machine was then joined to the domain.
<img width="1108" height="557" alt="03" src="https://github.com/user-attachments/assets/c9e7125e-27a9-4881-9c77-63cb0c7e304e" />
<img width="711" height="550" alt="04 domain connect to test machine" src="https://github.com/user-attachments/assets/68b0e8ff-d61d-4bf2-8093-7bc8a19b9b51" />


# Phase 2: Attack Simulation & Detection
In this phase, simulated unauthorized access attempts were performed to generate security events.
The attacker machine (Kali Linux) was used to simulate login attempts against the Windows environment.
These actions generated authentication events in Windows security logs, which were then forwarded to Splunk for analysis.
Important Windows Event IDs generated during the attack simulation included:

<img width="889" height="151" alt="image" src="https://github.com/user-attachments/assets/e8b4d4f8-7fc7-4d7d-8726-f6ed71281b9c"/>\
 These events provided valuable data for building detection rules.


# Phase 3: Security Monitoring & Detection using Splunk  
Splunk was deployed as the central Security Information and Event Management (SIEM) platform.

**Splunk Installation**
Splunk was installed on the server using the following steps:
dpkg -i splunk_package_name
cd /opt/splunk/bin
./splunk start

Administrator credentials were configured for Splunk.

**Log Forwarding**
Splunk Universal Forwarder was installed on: 
- Active Directory Server
- Windows Target Host

This allowed Windows Event Logs to be sent to the Splunk server for analysis. 

**Detection Rule**
A detection rule was created in Splunk to identify suspicious login activity: \
index=mydfir-ad EventCode=4624 
(Logon_Type=7 OR Logon_Type=10)
Source_Network_Address=* 
Source_Network_Address!="-" 
Source_Network_Address!=40.*
| stats count by _time, ComputerName, Source_Network_Address, user


This query detects successful logins originating from suspicious external IP addresses.
<img width="1907" height="602" alt="07" src="https://github.com/user-attachments/assets/ce2d072f-b456-44fe-a26f-42574721739a" />

<img width="1613" height="320" alt="creating alert" src="https://github.com/user-attachments/assets/bbe369d3-ea7c-45ab-94d8-f5321167c9c7" />


**Converted the query into a Splunk alert:**
Alert triggers when a successful login occurs from an external or suspicious IP. \
Alert action: send email notification to SOC analyst.

<img width="1917" height="570" alt="slack and email notification" src="https://github.com/user-attachments/assets/c66d1bc3-c66f-4d52-8d3e-730193b8021e" />

**Testing the alert:** \
Performed unauthorized login attempts from the Kali Linux attacker machine to the Domain Controller. \
Splunk detected these events in real-time.
The alert was triggered, and an email notification was sent to the SOC analyst
<img width="1893" height="541" alt="mydfir unauthorized login success" src="https://github.com/user-attachments/assets/70c9f894-7f0a-41d1-b0d3-b1ad09861f0a" />

<img width="1847" height="796" alt="mydfir unauthorized login success 2" src="https://github.com/user-attachments/assets/035b0b04-0ffd-48f6-a7fc-01090d98c365" />

# Phase 4: Security Automation with SOAR
To simulate a real SOC environment, Shuffle SOAR automation was implemented.
The goal of the automation playbook was to respond to suspicious authentication events.

**Playbook Workflow** \
Splunk detects suspicious login activity \
Alert is sent to Shuffle SOAR \
Shuffle sends a notification to Slack \
SOC analyst receives the alert

The analyst is asked whether to disable the domain account 

**Decision workflow:**
- If YES → Disable the domain user account 
- If NO → No action taken


Additional functionality included:
- Enriching suspicious public IP addresses
- Sending notification messages to Slack
- Automated incident response actions


<img width="1401" height="578" alt="disabled user in active directory" src="https://github.com/user-attachments/assets/2a6c20f4-2754-4e4f-836d-18ab573009c5" />

<img width="1205" height="622" alt="shuffler playbook" src="https://github.com/user-attachments/assets/caec94d1-dabd-4d8f-95f0-e8a396d56fa0"/> 
  
<img width="1917" height="570" alt="slack and email notification" src="https://github.com/user-attachments/assets/3f511a74-858c-4f22-84eb-e1cb638c9692" />

<img width="1918" height="626" alt="account disabled " src="https://github.com/user-attachments/assets/fca1144d-723d-4e01-85b2-d704b003f712" />


# Phase 4: Key Findings & Observations
During the project, several key insights were observed:
- Unauthorized login attempts can be effectively detected using Windows authentication logs.
- SIEM platforms like Splunk provide powerful capabilities for log correlation and threat detection.
- Automation tools such as Shuffle SOAR significantly reduce incident response time.
- Integration with communication tools like Slack improves SOC team collaboration and alert management.
- Automated account disabling helps mitigate potential lateral movement in Active Directory environments.

# Skills Demonstrated
This project demonstrates several SOC analyst and blue team skills:
- Active Directory Administration
- Windows Security Log Analysis
- SIEM Log Ingestion and Detection Engineering
- Splunk Query Development (SPL)
- Security Alert Investigation
- Security Orchestration and Automation (SOAR)
- Incident Response Workflow
- Threat Detection using Authentication Logs
- Security Monitoring in Enterprise Networks

# Conclusion
This project demonstrates the design and implementation of a SOC detection and automated response environment using Active Directory, Splunk, and SOAR automation tools. \
By integrating SIEM detection with automated playbooks, the lab replicates a real-world security operations workflow where suspicious activity is detected, investigated, and remediated efficiently. \
The project highlights the importance of log monitoring, threat detection, and automated incident response in modern enterprise security environments and provides valuable hands-on experience for SOC analysts.
