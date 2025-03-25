# Building a Honeynet in Azure with Live Traffic
![Cloud Honeynet & SOC](https://drive.google.com/uc?export=view&id=1xsHjvgYXt2fTmAh418b-N2TNprsHslci
)

## Introduction

The focus of this project is to build a small honeynet in Microsoft Azure consisting of two virtual machines (VMs) and collect the logs from those machines into the Azure Log Analytics Workspace.  These logs are then used by Microsoft Sentinel to build attack maps, trigger alerts, and create incidents to observe attacks on the network.  

A Microsoft Windows 10 OS is one of the VMs and the other is a Linux Ubuntu machine. Both machines security environments are initially configured to be insecure to allow for observation of attacks from the internet. The security environment for for both VMs are configured by deleting the inbound security rule in Network Security Guard in Azure for Remote Desktop Protocol (RDP) for Windows and Secure Shell Protocol (SSH) and replacing those rules with a rule that will allow any inbound connection to attempt to access the VMs through any port, any protocol, and any source or destination.  I also turned off Windows Defender Firewall with Advanced Security in the VM.  Lastly, I downloaded an instance of Microsoft SQL Server and configured the security properties to allow for both failed and successful logins to be recorded in Windows Event Viewer under application logs and forwarded to the Azure Security Log.

I then set up the network archeticutre of the network within Azure to monitor the unsecured envionment for 24 hours and recorded the results then applied security contorols to harden the envionment, measured the security metrics again for another 24 hours and report the results of both below. The metrics collected are as follows:

- Security Event (Windows Event Logs)
- Syslog (Linux Event Logs)
- Security Incident (Incidents Created by Sentinel)
- AzureNetworkAnalytics_CL (Malicious Flows allowed into our honeypot)

## Architecture Before Hardening / Security Contorls

![Architecture Before Hardening](https://drive.google.com/uc?export=view&id=15ZwDLtrCle36rnuFf486mZ8VHRH2yLwi)

## Architecture After Hardening / Security Contorls

![Architecture After Hardening](https://drive.google.com/uc?export=view&id=1qgAJaR-kV9ASVLi8C2p-8P2vZKr1Ywg4)

The architecture of the mini honeynet in Azure consists of the following components:

- Virtual Network (VNet)
- Network Security Group (NSG)
- Virtual Machines (1 Windows, 1 Linux)
- Log Analytics Workspace
- Azure Key Vault
- Azure Storage Account
- Microsoft Sentinel

Attack Maps for Unsecure Network

![NSGMaliciousAllowedIn](https://drive.google.com/uc?export=view&id=1rkViQ6a0VQ4SmT4s3GdVtB4uC8OcrOOe)
*Map of Network Security Guard Attacks Allowed in*

![SSH Authentication Failures](https://drive.google.com/uc?export=view&id=1pV2uyZq6F9L6wfLE3OfOB7OWkR9eW7mP)
*Linux SSH Authentication Failures*

![RDP Authentication Failures](https://drive.google.com/uc?export=view&id=1objsyzNTvQ0Svl45QpR9sS-33ZED8Q0L)
*Windows Remote Desktop Authentication Failurers*

![SQL Authentication Failures](https://drive.google.com/uc?export=view&id=1SCsl6rDNVFBLFCsMz2GIXLu-JBvL9l7z)
*Microsoft SQL Authentication Failures*

**NOTE:** ***DUE TO HARDENING OF THE NETWORK ARCHETICURE, I SELECTED TO NOT DISPLAY AFTER HARDENING MAPS***

## Metrics Before Hardening / Security Controls

The following table shows the metrics measured in unsecure network environment over a 24 hour period:

| Start Time: 2025-03-18 16:52:28<br>
| Stop Time: 2025-03-19 16:52:28

|Metric                    |Count   |
|--------------------------|--------|
|SecurityEvent             |7,925   |
|AzureNetworkAnalytics_CL  |1,483   |
|Syslog                    |27,294  |
|SecurityIncident          |239     |

## Metrics After Hardening / Security Controls

The following table shows the metrics measured in secure, hardened network environment over a 24 hour period:

| Start Time: 2025-03-20 15:05:17<br>
| Stop Time: 2025-03-21 15:05:17

|Metric                    |Count   |
|--------------------------|--------|
|SecurityEvent             |1,237   |
|AzureNetworkAnalytics_CL  |16      |
|Syslog                    |0       |
|SecurityIncident          |0       |

|RESULTS                                 |                                |
|:--------------------------------------:|:------------------------------:|
|**Metric**                              | **% Change After Hardening**   |
|SecurityEvents (Windows VM)             | -84.39%                        |
|AzureNetworkAnalytics_CL                | -98.92%                        |
|Syslog (Linux VM)                       | -100.00%                       |
|Security Incident (Sentinel Incidents)  | -100.00%                       |

## Conclusion
I was successful in creating a mini honeynet in Microsoft Azure and pushing the logs of both the Windows and Linux VMs in the network to Azure Log Analytics Workspace for analysis.  Microsoft Sentinel was also used to trigger alerts and create incidents based on the ingested logs.  Additionally, metrics were measured in both the unsecured and later in the secured environment for a period of 24 hours each and the results of those observations were reported.  It is very noticable that hardening systems drasticly improved the security posture of both machines, demonstrating their effectiveness in security.  It is also worth noting that if these measurements were conducted in a real-world environment with regular users, it is likely that more security events and alerts may have been generated within the 24 hour period following the hardening of the network and machines.  

## KQL Queries

Below are queries that were used to determine the results for the metrics reported above.

| Metric                                       | Query                                                                                                                                            |
|----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| Start/Stop Time                              | range x from 1 to 1 step 1<br>\| project StartTime = ago(24h), StopTime = now()                                                                  |
| Security Events (Windows VMs)                | SecurityEvent<br>\| where TimeGenerated>= ago(24h)<br>\| count                                                                                   |
| Syslog (Linux VMs)                           | Syslog<br>\| where TimeGenerated >= ago(24h)<br>\| count                                                                                         
| Security Incident (Sentinel Incidents)       | SecurityIncident<br>\| where TimeGenerated >= ago(24h)<br>\| count                                                                               |
| NSG Inbound Malicious Flows Allowed          | AzureNetworkAnalytics_CL<br>\| where FlowType_s == "MaliciousFlow" and AllowedInFlows_d > 0<br>\| where TimeGenerated >= ago(24h)<br>\| count    |