# Zabbix Homelab: SNMP Monitoring, Custom Templates, and Alerting

![Proxmox](https://img.shields.io/badge/Proxmox-E57000?style=for-the-badge&logo=proxmox&logoColor=white)
![Cisco](https://img.shields.io/badge/Cisco-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Zabbix](https://img.shields.io/badge/Zabbix-CC0000?style=for-the-badge&logo=zabbix&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![EnGenius](https://img.shields.io/badge/EnGenius-003087?style=for-the-badge&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-0078D4?style=for-the-badge&logo=windows&logoColor=white)



## Overview

This project implements a centralized monitoring solution using Zabbix to improve the visibility, monitoring, and troubleshooting of my local area network (LAN). It was developed as a personal learning project to gain hands-on experience with network monitoring, system observability, and the use of industry-standard monitoring tools.


## Monitoring Configuration

![Zabbix](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/ZABBIX.png)


### Dashboard 
My Zabbix dashboard is configured to monitor crucial metrics such as incoming and outgoing network traffic on my PC, as well as CPU and memory utilization of the Zabbix server and Proxmox hypervisor.
![Zabbix](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images%20/Dashboard.png)


### Skills Demonstrated
- Network monitoring 
- Zabbix configuration and administration
- SNMP and agent-based monitoring
- Network troubleshooting and fault detection
- Performance monitoring and resource analysis
- Alert and trigger configuration
- Linux server administration
- Documentation and network diagram creation


## Alerting System (Custom Triggers)

Zabbix provides the ability to create custom alerts and triggers based on a device's status or the state of specific services being monitored. I created triggers to detect when my Pi-hole DNS server becomes unavailable. I used the pre-built template from Zabbix that monitors the status of services running on the Linux server hosting Pi-hole.

### Templates Used: 

- Systemd by Zabbix Agent 2
- Linux by Zabbix Agent

I configured the trigger to activate whenever pihole-FTL.service is not running. and will send me an alert via email.

### Trigger Action:
This shows that when the service status matches the defined condition, the trigger will execute the configured operation.

![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-4.png)

### Trigger Operation:
The operation sends a message to the designated users whose profiles are linked to their email addresses. In this case, my email receives the alert.

![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-9.png)


### Trigger Appearing in the Dashboard
This shows when the trigger is activated it has successfully sent the email alert to the designated users.
![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-6.png)
  
### Sample Email After Alert:

Using Gmail SMTP is the simplest and fastest option compared to other email providers.
I created a dedicated email account used solely for sending Zabbix alerts. Another important consideration is to configure an application-specific password key from google's 2 Factor Authentication to improve the security of the email account.
![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-7.png)



## Making a Custom Template

One of my network devices, the EnGenius EWS7928P switch, does not have a readily available template, so I created my own by discovering its private OIDs through SNMP.
For my template the main metrics I need to have for each interface of the switch:

- Uptime
- Rx Traffic
- Tx Traffic
- Interface Status
- System Description
- Hostname
- Location
- Contact Information

### Step 1
I enabled snmp v2c in my switch and set the snmp community:

```
set snmp community public ro
set snmp community private rw
```


### Step 2 
There are common Object Identifiers (OIDs) standardized and used by many vendors:

| Metric | OID |
|--------|-----|
| Contact information | .1.3.6.1.2.1.1.4.0 | 
| Uptime | .1.3.6.1.2.1.1.3.0 |
| Hostname | .1.3.6.1.2.1.1.5.0 |
| Location | .1.3.6.1.2.1.1.6.0 | 
| System Description | 1.3.6.1.2.1.1.1.0 |

I used SNMPwalk and other tools to discover private OIDs that are vendor-specific.

The first method I used in getting the OID for the template is using snmpwalk command which gave me this result:

Command:
  - snmpwalk -v2c -c public <ip> <1.3.6.1.2.1.1>
    
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/snmpwalk.png)

Because finding specific OIDs using SNMPwalk was time-consuming, I used MIB browsers to more efficiently locate the required OIDs for RX, TX, and interface status.

### OID for interface description: [.1.3.6.1.2.1.2.2.1.2]
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%201.png)

### OID for interface type : [.1.3.6.1.2.1.2.2.1.3]

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%202.png)

Finding RX and TX traffic was quite tedious, as I first needed to identify the corresponding SNMP object, ifHCInOctets and ifHCOutOctets which means High Capacity Octets (1 octet = 8 bits = 1 byte) and I identified its assigned OID number based on RFC 2863. — "The Interfaces Group MIB" (published June 2000, supersedes RFC 1573).

So I found out that the OID is 1.3.6.1.2.1.31.1.1.1.6.10  which means:

| OID | MIB |
|--------|-----|
| 1.3.6.1.2.1.31.1.1.1.6 | IF-MIB::ifHCInOctets | 
| 1.3.6.1.2.1.31.1.1.1.10 | IF-MIB::ifHCOutOctets |


- 1.3.6.1.2.1.31 = IF-MIB (ifMIB) 
- .1.1.1 = ifXTable > ifXEntry
- .6 = ifHCInOctets 
- .10 = ifHCOutOctets

### IMPORTANT: This metric stores the total number of bytes that pass through a specific interface.This metric stores the total number of bytes that pass through a specific interface since the device was last rebooted. Therefore, it must be converted into a data rate to represent actual throughput.

### NOTE: The initial interface table (ifTable) provides 32-bit traffic counters (ifInOctets and ifOutOctets). But the template uses the extended ifXTable high-capacity 64-bit counters (ifHCInOctets and ifHCOutOctets) to avoid counter overflow on modern high-speed interfaces. 

### ifinOctet (Rx Octet) 
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%203.png)
This is the ifInOctet but looking closely it is a Counter32 value, which has a maximum value of approximately 4 GB. 
That is why I need Counter 64 as it is significantly more capable than counter 32 in terms of capacity.

### IFHCOCTET RX 

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/RX.png)


### IFHCOCTET TX

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/TX.png)

### Configuring Zabbix Template

Due to the large number of interfaces on the switch, I used Low-Level Discovery (LLD) to automatically create monitoring items for RX traffic, TX traffic, and interface status.

MACROS USED:

| LLD Macro | Meaning |
|--------|-----|
| {#IFNAME} | Interface Name | 
| {#SNMPINDEX} | SNMP Table Index |

I will only show one sample configuration because the remaining metrics follow a similar configuration process.

Sample Item for the Template:

### RX Traffic
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/item%20prototype%201.png)

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/item%20prototype%202.png)

This metric shows that I need to convert it into bits and add a "change per second" preprocessing step to convert the cumulative byte counter into a throughput value, then multiply it by 8 to convert bytes to bits.

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%204.png)

### I also configured one LLD trigger: 
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%206.png)

last(/EnGenius Switch/if.status[{#SNMPINDEX}])=2
This expression retrieves the interface status from the EnGenius switch host. If the value equals 2, the interface is considered down based on the configured value mapping.

### 1 -> up
### 2 -> down
### 3 -> testing

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%205.png)

### The remaining metrics have similar process of configuration. The Completed YAML file is in the link below:

### YAML FILE: [config/ews7928p.yaml](config/ews7928p.yaml)


### Dashboard of Completed Template

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/graph.png)








## Problems Encountered and Troubleshooting


One issue I encountered while configuring the Systemd by Zabbix Agent 2 template was a timeout error when retrieving systemd data. The solution was to increase the timeout value in the Zabbix server configuration.

![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-3.png)

Other minor details when using the systemd by Zabbix Agent 2 Template are :

- Ensure that the agent running on the device is Zabbix Agent 2.
- Remove the old Agent config file to avoid confusion
 

### Future Improvements

- Improve the template creation process.
- Add more triggers for different failure scenarios.
- Include additional metrics for more comprehensive monitoring.
  
### Things Learned

- Public and Private OIDs
- In-depth SNMP configuration
- Using MIB browsers to navigate OIDs more efficiently
- Understanding how SNMP works
- How to configure Zabbix agents and SNMP
- How to create a custom Template
- Troubleshooting Zabbix
- Zabbix Navigation and Administration
- OID vs MIB
- Configuring Triggers
  


