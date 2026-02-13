# Internet-Access

## Network Topology

<img width="599" height="574" alt="image" src="https://github.com/user-attachments/assets/1391af5d-9710-4475-949f-6319ca63a517" />


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
interface g0/1
switchport trunk encapsulation dot1q
switchport mode trunk
no shutdown
end


conf t
int g0/2
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
This is the script taking in input for the Sales VLAN which will be VLAN 2: <img width="1296" height="785" alt="image" src="https://github.com/user-attachments/assets/53075d5a-7a0f-4a4f-896a-bc8864ba1678" />
Now ```user 2``` can reach the internet from the Sales VLAN: <img width="877" height="412" alt="image" src="https://github.com/user-attachments/assets/769e1f70-574b-49ec-988a-d70d935b84ce" />


This is the script taking in input for the Marketing VLAN which will be VLAN 3: <img width="1044" height="805" alt="image" src="https://github.com/user-attachments/assets/daa6c7e1-8e50-4f25-a0f2-1a0ed5be3723" />
Now ```user 3``` can reach the internet from the Marketing VLAN: <img width="902" height="381" alt="image" src="https://github.com/user-attachments/assets/25c99df8-a874-4833-8bce-b49c337c2632" />




## Python Script
</br>This script is an improvement from the [VLAN-Automation](https://github.com/Jeremiah-Rojas/VLAN-Automation) script.
</br>__It is recommended to ssh manually first.__
SSH to the switch: <img width="1348" height="736" alt="image" src="https://github.com/user-attachments/assets/2c3d963c-f9eb-4819-80ea-dba8dde96ce4" />
SSH to the router: <img width="1355" height="747" alt="image" src="https://github.com/user-attachments/assets/0a0e5434-e735-409a-b79b-3ae62009cacf" />
</br>Afterwards you can exit the SSH sessions and then run the following script answering the questions accordingly.



