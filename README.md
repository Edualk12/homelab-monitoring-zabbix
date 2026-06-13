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
- Virtual machine deployment and management using Proxmox
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



## Problems Encoutered and Troubleshooting

Some problems I've encountered by setting the systemd template by zabbix agent is the notification that is not compatible and it said that the it timeout before getting the systemd data.
The solution I found is to change the timeout value on the zabbix server itself from the web ui and in the .config file via the terminal.

![Pihole](https://github.com/Edualk12/homelab-monitoring-zabbix/blob/main/images/Zabbix-3.png)

Other minor details when using the systemd by Zabbix Agent 2 Template are :

- The Agent version running the device as it should be Agent 2
- Remove the old Agent config file to avoid confusion
 

### Future Improvements





