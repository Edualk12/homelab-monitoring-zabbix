# HOMELAB MONITORING USING ZABBIX

![Proxmox](https://img.shields.io/badge/Proxmox-E57000?style=for-the-badge&logo=proxmox&logoColor=white)
![Cisco](https://img.shields.io/badge/Cisco-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Zabbix](https://img.shields.io/badge/Zabbix-CC0000?style=for-the-badge&logo=zabbix&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![EnGenius](https://img.shields.io/badge/EnGenius-003087?style=for-the-badge&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-0078D4?style=for-the-badge&logo=windows&logoColor=white)



## Overview

This project implements a centralized monitoring solution using Zabbix to improve the visibility, monitoring, and troubleshooting of my local area network (LAN). It was developed as a personal learning project to gain hands-on experience with network monitoring, system observability, and the use of industry-standard monitoring tools.










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



### Monitored Infrastructure



### Monitoring Configuration



### Dashboard Screenshots



## Alerting System (Custom Triggers)

Zabbix has the feature of creating custom alerts or triggers based on the device's status or a specific service running in the device being monitored. 

I've made triggers for detecting if my Pi-hole DNS server goes down

i used the readily made template from zabbix that scans for the status of the services running in the linux server which is where the Pihole is running

![Pihole]()

### Templates Used: 

- Systemd by Zabbix Agent 2
- Linux by Zabbix Agent

For the Trigger Alert itself I've configured for the trigger to activate if ever the pihole-FTL.service is not running and will send me an alert via Email.

![Pihole]()


Trigger Appearing in the Dashboard

![Pihole]()
  
Sample Email After Alert:

![Pihole]()


### Making a Custom Template



### Challenges & Solutions

### Future Improvements





