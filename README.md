# Internet-Access

## Network Topology

</br> The following lab shows how a python script can automate the creation of VLANs while giving the computers within them access to the internet.

### Devices Used:
- Cisco IOSv 15.7 router
- Cisco IOSvL2 15.2.1 switch
- GNS3 Software
- Ubuntu Container (running on VMware machine)


## Configurations

(management)<img width="1398" height="704" alt="image" src="https://github.com/user-attachments/assets/81443bbf-97c4-4bd7-a490-4209e3235cb9" />

(user2)<img width="1393" height="702" alt="image" src="https://github.com/user-attachments/assets/b56bc3c4-69ea-4cde-ba62-9d37163c457a" />

(user3)<img width="1394" height="707" alt="image" src="https://github.com/user-attachments/assets/69a60833-f45a-46d5-a055-35d7c9e88f08" />

switch configurations:
```
enable
conf t
host cisco-switch
vlan 99
name MANAGEMENT_VLAN
exit
int vlan 99
ip address 192.168.99.10 255.255.255.0 
no shut
end


conf t
int vlan 1
ip address 192.168.0.10 255.255.255.0 
no shut
end


enable
conf t
username msfadmin pass msfadmin
username msfadmin priv 15

line vty 0 4
login local
transport input all

ip domain-name example.com
crypto key generate rsa
1024

end


conf t
interface g0/0
switchport trunk encapsulation dot1q
switchport mode trunk
no shutdown
end


conf t
int g0/1
switchport mode access
switchport access vlan 99
no shutdown
end
wr

```

router configurations:
```
conf t
host cisco-router
int g0/1
ip address 192.168.0.1 255.255.255.0
no shut
end


conf t
username msfadmin pass msfadmin
username msfadmin priv 15

line vty 0 4
login local
transport input all

ip domain-name example.com
crypto key generate rsa
1024

end


conf t
interface g0/1.99
encapsulation dot1Q 99
ip address 192.168.99.1 255.255.255.0
no shut
end
wr

```

## Script in Action

## Python Script
</br>This script is an improvement from the [VLAN-Automation](https://github.com/Jeremiah-Rojas/VLAN-Automation) script.
</br>__It is recommended to ssh manually first.__

