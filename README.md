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

The following configurations can be performed on the "/etc/network/interfaces" configuration file.
</br>Management Computer: </br><img width="699" height="352" alt="image" src="https://github.com/user-attachments/assets/81443bbf-97c4-4bd7-a490-4209e3235cb9" />

User 2: </br><img width="699" height="352" alt="image" src="https://github.com/user-attachments/assets/b56bc3c4-69ea-4cde-ba62-9d37163c457a" />

User 3: </br><img width="699" height="352" alt="image" src="https://github.com/user-attachments/assets/69a60833-f45a-46d5-a055-35d7c9e88f08" />

The following configurations can either copied and pasted as a whole bewteen the break points or manually entered.
</br>switch configurations:
```
[BREAK]: These configurations set up the management VLAN which is VLAN 99
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

[BREAK]: These configurations set up the native VLAN where all untagged VLAN traffic is passed
conf t
int vlan 1
ip address 192.168.0.10 255.255.255.0 
no shut
end

[BREAK]: These configurations set up SSH
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

[BREAK]: These configurations set up the trunk port which passes traffic from multiple VLANs across one link
conf t
interface g0/1
switchport trunk encapsulation dot1q
switchport mode trunk
no shutdown
end

[BREAK]: These configurations allow the Management computer to communicate with the switch
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
[BREAK]: These configurations set up the native VLAN
conf t
host cisco-router
int g0/1
ip address 192.168.0.1 255.255.255.0
no shut
end

[BREAK]: These configurations set up SSH
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

[BREAK]: These configurations set up subinterfaces which are necessary for traffic from each VLAN to go out to the internet and to communicate with each other. This may not always be ideal.
conf t
interface g0/1.99
encapsulation dot1Q 99
ip address 192.168.99.1 255.255.255.0
no shut
end
wr

```

## Script in Action
__Note: the IP address assigned to the ```g0/1``` interface (which is link between the router and switch) must always be ```192.168.0.1``` which is the default gateway for native VLAN traffic; unless the native VLAN is not on the 192.168.0.0/24 network.__
</br>This is the script taking in input for the Sales VLAN which will be VLAN 2: </br><img width="709" height="369" alt="image" src="https://github.com/user-attachments/assets/53075d5a-7a0f-4a4f-896a-bc8864ba1678" />
</br>Now ```user 2``` can reach the internet from the Sales VLAN: </br><img width="461" height="232" alt="image" src="https://github.com/user-attachments/assets/769e1f70-574b-49ec-988a-d70d935b84ce" />


This is the script taking in input for the Marketing VLAN which will be VLAN 3: </br><img width="709" height="452" alt="image" src="https://github.com/user-attachments/assets/daa6c7e1-8e50-4f25-a0f2-1a0ed5be3723" />
</br>Now ```user 3``` can reach the internet from the Marketing VLAN: </br><img width="461" height="232" alt="image" src="https://github.com/user-attachments/assets/25c99df8-a874-4833-8bce-b49c337c2632" />

</br>You can see the newly created VLANs on the switch: </br><img width="699" height="352" alt="image" src="https://github.com/user-attachments/assets/aa1bb05d-e6b5-4fd4-999f-913b9345abdd" />

</br>These are the newly created subinterfaces on the router: </br><img width="699" height="352" alt="image" src="https://github.com/user-attachments/assets/2ea766a9-3763-4035-b205-387b9ce6dbc6" />


## Python Script
</br>This script is an improvement from the [VLAN-Automation](https://github.com/Jeremiah-Rojas/VLAN-Automation) script.
</br>__It is recommended to ssh manually first.__
SSH to the switch: <img width="699" height="352" alt="image" src="https://github.com/user-attachments/assets/2c3d963c-f9eb-4819-80ea-dba8dde96ce4" />
SSH to the router: <img width="699" height="352" alt="image" src="https://github.com/user-attachments/assets/0a0e5434-e735-409a-b79b-3ae62009cacf" />
</br>Afterwards you can exit the SSH sessions and then run the following script answering the questions accordingly:
```
# This is the improved version of the script
from netmiko import ConnectHandler
# ------------------------------ DEVICE DEFINITIONS ------------------------------
iosv_l2 = {
    'device_type': 'cisco_ios',
    'ip': '192.168.99.10',
    'username': 'msfadmin',
    'password': 'msfadmin'
}
iosv = {
    'device_type': 'cisco_ios',
    'ip': '192.168.99.1',
    'username': 'msfadmin',
    'password': 'msfadmin'
}

# ------------------------------ USER INPUT ------------------------------
config_decision = input("Has this script been run before? (y/n): ")
vlan_name = input("Enter name of new VLAN: ")
vlan_number = input("Enter VLAN number (Ex: 10): ")
vlan_int = input(
    f"Enter access interface(s) for {vlan_name}, (g0/0,g0/1,etc.): "
)
vlan_interfaces = vlan_int.replace(" ", "")
interfaces = vlan_interfaces.split(",")
trunk_port = input("Which switch interface is the trunk port?: ")
internet_link = input("Which router interface is connected to the internet?: ")
switch_link = input("Which router interface is connected to the switch?: ")
switch_link_ip = input(f"What IP should be assigned to {switch_link} (native VLAN): ")

# NAT ACL number (single, global)
NAT_ACL = 100

# ------------------------------ ROUTER CONFIGURATION ------------------------------
if config_decision.lower() == "n":
    net_connect = ConnectHandler(**iosv)

    router_commands = [
        # Internet-facing interface
        f"interface {internet_link}",
        "ip address dhcp",
        "ip nat outside",
        "no shutdown",

        # Native VLAN interface to switch
        f"interface {switch_link}",
        f"ip address {switch_link_ip} 255.255.255.0",
        "ip nat inside",
        "no shutdown",

        # NAT ACL (append-safe)
        f"access-list {NAT_ACL} permit ip 192.168.{vlan_number}.0 0.0.0.255 any",

        # NAT (defined ONCE)
        f"ip nat inside source list {NAT_ACL} interface {internet_link} overload",

        # Default route
        "ip route 0.0.0.0 0.0.0.0 192.168.122.1",

        # Optional router DNS (router-only)
        "no ip domain-lookup"
        ]

    router_commands.extend([
            f"access-list {NAT_ACL} permit ip 192.168.0.0 0.0.0.255 any"
        ])
        # VLAN subinterface (router-on-a-stick)
    router_vlan_commands = [
            f"interface {switch_link}.{vlan_number}",
            f"encapsulation dot1Q {vlan_number}",
            f"ip address 192.168.{vlan_number}.1 255.255.255.0",
            "ip nat inside",
            "no shutdown"
        ]

    print("Applying router configuration...")
    net_connect.send_config_set(router_commands)
    net_connect.send_config_set(router_vlan_commands)
    net_connect.save_config()
    net_connect.disconnect()
else:
    net_connect = ConnectHandler(**iosv)

    router_commands2 = [
        f"interface {switch_link}",
        "ip nat inside",
        "no shutdown",
        f"access-list {NAT_ACL} permit ip 192.168.{vlan_number}.0 0.0.0.255 any",
        f"ip nat inside source list {NAT_ACL} interface {internet_link} overload"
    ]
    
    router_vlan_commands2 = [
        f"interface {switch_link}.{vlan_number}",
        f"encapsulation dot1Q {vlan_number}",
        f"ip address 192.168.{vlan_number}.1 255.255.255.0",
        "ip nat inside",
        "no shutdown"
    ]
    print("Applying router configuration...")
    net_connect.send_config_set(router_commands2)
    net_connect.send_config_set(router_vlan_commands2)
    net_connect.save_config()
    net_connect.disconnect()

# ------------------------------ SWITCH CONFIGURATION ------------------------------
net_connect = ConnectHandler(**iosv_l2)

switch_commands = []

# VLAN creation
switch_commands.extend([
    f"vlan {vlan_number}",
    f"name {vlan_name}"
])

# Access ports
for intf in interfaces:
    switch_commands.extend([
        f"interface {intf}",
        "switchport mode access",
        f"switchport access vlan {vlan_number}",
        "no shutdown",
        "exit"
    ])

# Trunk port
switch_commands.extend([
    f"interface {trunk_port}",
    "switchport trunk encapsulation dot1q",
    "switchport mode trunk",
    f"switchport trunk allowed vlan add {vlan_number}",
    "no shutdown"
])

print("Applying switch configuration...")
net_connect.send_config_set(switch_commands)
net_connect.save_config()
net_connect.disconnect()
print("Switch configuration complete.\n")

print("Configuration complete. VLAN has internet access.")
```


