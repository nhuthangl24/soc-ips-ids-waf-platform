# SOC System - IPS / IDS / WAF

# Objectives

* Prevent external intrusions into the internal network
* Protect web applications from common attacks (SQLi, XSS, RCE)
* Monitor and analyze network traffic in real-time
* Centralize and analyze logs to support incident response
* Ensure high availability of critical services

# System Components

* **pfSense + IPS**: firewall, NAT, traffic monitoring, IPS rules
* **WAF (DMZ)**: web application protection, filter malicious HTTP/HTTPS requests
* **Web Server**: host internal websites and web applications
* **Database Server**: store application data
* **IDS Server**: monitor internal traffic, detect intrusions
* **ELK Stack**: collect, centralize, and analyze logs from all devices and services

# Data Flow

* Incoming traffic passes through **pfSense + IPS** for filtering and monitoring
* HTTP/HTTPS requests are NATed to the **WAF** in the DMZ
* WAF filters malicious requests and forwards clean traffic to the **Web Server**
* **IDS Server** monitors internal traffic for abnormal behavior
* All logs from firewall, WAF, Web Server, Database, and IDS are sent to **ELK Stack** for analysis

# Network Diagram

* Overall network diagram image
* Network configuration files (VMware / PFSENSE)
* Detailed description of subnets: WAN, LAN, DMZ, and static IPs for each VM

# PFSENSE Installation

* Create pfSense VM on VMware
* Configure interfaces: LAN, WAN, DMZ
* Set up basic **Firewall Rules** per interface
* Configure NAT / Port Forward for Web Server and DVWA services
* Configure IPS to detect and block intrusions

# Web Service Deployment

## Windows Server + IIS

* Set static IP in DMZ
* Install IIS 10 and deploy sample website
* Test access from LAN, DMZ, and WAN

## Ubuntu + DVWA / Docker

* Set static IP in DMZ
* Install Docker and run DVWA container
* Test web access from LAN, DMZ, WAN

# SOC Deployment

* Install SOC tools (ELK, Wazuh, Graylog...)
* Collect logs from pfSense, WAF, Web Server, Database, IDS
* Configure dashboards and alerts based on security rules
* Monitor network status and incidents in real-time

# IPS / IDS Setup

* Install Snort or Suricata
* Configure basic rules to detect network intrusions, malware, exploits
* Integrate with SOC for alerts and centralized logging

# WAF Deployment

* Install ModSecurity or compatible WAF for IIS / Web Server
* Configure policies to protect web endpoints
* Test effectiveness in filtering malicious requests

# High Availability (HA)

* Set up **pfSense HA** with CARP VIP for firewall failover
* Backup and failover for critical servers
* Test service availability in case of failure

# Testing & Evaluation

* Ping test from LAN/DMZ → WAN
* Access web services from WAN
* Verify alerts from SOC / IPS / IDS / WAF
* Evaluate HA failover and service availability

# Under Preparation

* Information and content are being updated and will be added as soon as possible
