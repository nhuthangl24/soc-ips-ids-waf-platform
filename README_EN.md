# SOC - IPS / IDS / WAF System

> Updated architecture based on the latest network diagram

---

## Overview

This system follows a multi-layer defense architecture integrating firewall, IPS, WAF, IDS, and centralized logging with ELK.

## System Architecture

```text
[Attacker] --> [Internet] --> [pfSense + IPS]
                               WAN: .75.249
                               LAN: .10.10
                                      |
                         NAT 80/443 -> 192.168.30.40
                                      |
                                [DMZ - WAF]
                               192.168.30.40
                                      |
                     -------------------------------
                     |                             |
                     v                             v
            [DMZ-WEB Zone]                 [Security Zone]
          Web: 192.168.20.30             ELK: 192.168.40.60
          DB : 192.168.20.50             Log sources:
                                          - WAF -> ELK
                                          - IDS -> ELK
                                          - pfSense + IPS -> ELK

                IDS Server: 192.168.40.50
```

## Components

* pfSense + IPS
* WAF (DMZ)
* Web Server
* Database Server
* IDS Server
* ELK Stack

## Data Flow

* External traffic enters through pfSense + IPS
* HTTP/HTTPS requests are NATed to WAF
* WAF forwards clean traffic to Web Server
* IDS monitors internal traffic
* All logs are centralized into ELK

## Objectives

* Intrusion prevention
* Web application protection
* Traffic monitoring
* Centralized log analysis
* Incident response
