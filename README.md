# HOMELAB MONITORING USING ZABBIX

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
My Zabbix Dashboard that ive set to monitor crucial metrics such as the bits incoming and outgoing of my PC,and also the CPU and Memory utilization of the Zabbix Server itself and the Proxmox Hypervisor
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

Zabbix has the feature of creating custom alerts or triggers based on the device's status or a specific service running in the device being monitored. I've made triggers for detecting if my Pi-hole DNS server goes down. I used the readily made template from zabbix that scans for the status of the services running in the linux server which is where the Pihole is running

### Templates Used: 

- Systemd by Zabbix Agent 2
- Linux by Zabbix Agent

For the Trigger Alert itself I've configured for the trigger to activate if ever the pihole-FTL.service is not running and will send me an alert via Email.

### Trigger Action:
This just shows when the status of the service running is equal, it will trigger the operation

![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-4.png)

### Trigger Operation:
The operation is just send a message to the designated users, where the users profile is linked to their email, in this case my email is the recever of the alert.

![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-9.png)


### Trigger Appearing in the Dashboard
This shows when the trigger is activated it has successfully sent the email alert to the designated users.
![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-6.png)
  
### Sample Email After Alert:

Using the Gmail option is the fastest and easiest way out of all the other options like Generic SNMP, etc.
Ive made another email just sending alerts from zabbix and another important detail to consider is to set a key for the email to ensure security of the email being used.

![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-7.png)



## Making a Custom Template

One of my network devices (Engenius EWS7928P) Switch does not have a readily available template so I need to make my own through getting the private oid through snmp.

For my Template the main metrics I need to have for each interface of the switch:

- Uptime
- Rx Traffic
- Tx Traffic
- Interface Status
- Sytem Description
- Hostname
- Location
- Contact Information

### Step 1
I enabled snmp v2c in my switch and set the snmp community:

```
Set snmp commmunity public ro
Set snmp community private rw
```


### Step 2 
There are common Object Identifier (OID) used widely by every company which are:

| Metric | OID |
|--------|-----|
| Contact information | .1.3.6.1.2.1.1.4.0 | 
| Uptime | .1.3.6.1.2.1.1.3.0 |
| Hostname | .1.3.6.1.2.1.1.5.0 |
| Location | .1.3.6.1.2.1.1.6.0 | 
| System Description | 1.3.6.1.2.1.1.1.0 |

I need to use SNMPwalk or other ways to get the private OID which is vendor specific.

The first method I used in getting the OID for the template is using snmpwalk command which gave me this result:

Command:
  - snmpwalk -v2c -c public <ip> <1.3.6.1.2.1.1>
    
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/snmpwalk.png)

Due to the time-consuming process of using snmpwalk of finding the specific OID I need, I used MIB broswers to be more efficient in finding the oid I need which are Rx, Tx and, Status:

### OID for interface description: [.1.3.6.1.2.1.2.2.1.2]
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%201.png)

### OID for interface type : [.1.3.6.1.2.1.2.2.1.3]

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%202.png)

Finding the RX and TX Traffic is quite tedious as I searched for what it is called which is "ifhcOctets" which means High Capacity Octet (Octet = 8 bitS = 1 Byte) and I also what number it is designated based from the RFC 2863 — "The Interfaces Group MIB" (published June 2000, supersedes RFC 1573).

So I found out that the OID is 1.3.6.1.2.1.31.1.1.1.6.10  which means:

| OID | MIB |
|--------|-----|
| 1.3.6.1.2.1.31.1.1.1.6 | IF-MIB::ifHCInOctets | 
| 1.3.6.1.2.1.31.1.1.1.10 | IF-MIB::ifHCOutOctets |


- 1.3.6.1.2.1.31 = IF-MIB (ifMIB) 
- .1.1.1 = ifXTable > ifXEntry
- .6 = ifHCInOctets 
- .10 = ifHCOutOctets

### IMPORTANT: This metric actually store the total bytes though a specific interface after its enabled or boot so later I will need to configure it to turn this into a data rate or throughput instead of total accumulated bytes.

### NOTE: The initial interface table (ifTable) provides 32-bit traffic counters (ifInOctets and ifOutOctets). But the template uses the extended ifXTable high-capacity 64-bit counters (ifHCInOctets and ifHCOutOctets) to avoid counter overflow on modern high-speed interfaces. 

### ifinOctet (Rx Octet) 
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%203.png)
This is the ifInOctect but looking closely it is counter32 which is limited to only 4gb that is why I need Counter 64 as it is signafically more capable than counter 32 in terms of capacity.

### ifhcOcters RX 

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/RX.png)


### ifhcOcters TX

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/TX.png)

### Configuring Zabbix Template

Due to the number of interfaces in the switch I used (Low Level Discovery) LLD to automate the process of getting the Rx Traffic, Tx Traffic, Status of each interface, so I will be using LDD macros for this process. 

MACROS USED:

| LDD Macro | Meaning |
|--------|-----|
| {#IFNAME} | Interface Name | 
| {#SNMPINDEX} | SNMP Table Index |

I will be showing only one same screenshot as all the metrics are the same in configuration.

Sample Item for the Template:

### RX Traffic
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/item%20prototype%201.png)

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/item%20prototype%202.png)

This metric shows that I need to convert it into bits and add a change for second preprocessing step for it to work in a graph and multiply to convert the bytes into bits to match the standard way of measurement.

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%204.png)

### I also configured one LLD trigger: 
![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%206.png)

The expression last(/Engenius Switch/if.s tatus[{#SNMPINDEX}]=2
This means it gets the information from Engenius Switch host and looks at the if.status (interface status) if the value matches with 2 which is equal to down which I’ve also included in the value mapping to make it easier to interpret.

### 1 -> up
### 2 -> down
### 3 -> testing

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/oid%205.png)

### The remaining metrics have similar process of configuration. The Completed YAML file is in the link below:

### YAML FILE: [config/ews7928p.yaml](config/ews7928p.yaml)


### Dashboard of Completed Template

![snmp](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/graph.png)








## Problems Encoutered and Troubleshooting


Some problems I've encountered by setting the systemd template by zabbix agent is the notification that is not compatible and it said that the it timeout before getting the systemd data.
The solution I found is to change the timeout value on the zabbix server itself from the web ui and in the .config file via the terminal.

![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-3.png)

Other minor details when using the systemd by Zabbix Agent 2 Template are :

- The Agent version running the device as it should be Agent 2
- Remove the old Agent config file to avoid confusion
 

### Future Improvements

- Improve Template making Process
- Adding more trigger for different possible failure
- Adding more metrics in the template for more options in monitoring.
  
### Things Learned

- Public and Private OIDs
- In depth configuration of SNMP
- Using MIB manager to navigate easier
- How SNMP works
- How to configure Zabbix agents and SNMP
- How to create a custom Template
- Troubleshooting Zabbix
- Zabbix Navigation and Administration
- OID vs MIB
- Configuring Triggers
  


