# Building a Honeynet in Azure with Live Traffic
Microsoft Azure Honeypot &amp; SIEM

## Introduction

The focus of this project is to build a small honeynet in Microsoft Azure consisting of two virtual machines (VMs) and collect the logs from those machines into the Azure Log Analytics Workspace.  These logs are then used by Microsoft Sentinel to build attack maps, trigger alerts, and create incidents to observe attacks on the network.  

A Microsoft Windows 10 OS is one of the VMs and the other is a Linux Ubuntu machine. Both machines security environments are initially configured to be insecure to allow for observation of attacks from the internet. The security environment for for both VMs are configured by deleting the inbound security rule in Network Security Guard in Azure for Remote Desktop Protocol (RDP) for Windows and Secure Shell Protocol (SSH) and replacing those rules with a rule that will allow any inbound connection to attempt to access the VMs through any port, any protocol, and any source or destination.  I also turned off Windows Defender Firewall with Advanced Security in the VM.  Lastly, I downloaded an instance of Microsoft SQL Server and configured the security properties to allow for both failed and successful logins to be recorded in Windows Event Viewer under application logs and forwarded to the Azure Security Log.

I then set up the network archeticutre of the network within Azure to monitor the unsecured envionment for 24 hours and recorded the results then applied security contorols to harden the envionment, measured the security metrics again for another 24 hours and report the results of both below. The metrics collected are as follows:

- Security Event (Windows Event Logs)
- Security Alert (Log analytics Alerts Triggered)
- Syslog (Linux Event Logs)
- Security Incident (Incidents Created by Sentinel)
- AzureNetworkAnalytics_CL (Malicious Flows allowed into our honeypot)

## Architecture Before Hardening / Security Contorls

![Architecture Before Hardening](https://drive.google.com/file/d/15ZwDLtrCle36rnuFf486mZ8VHRH2yLwi/view?usp=drive_link)
