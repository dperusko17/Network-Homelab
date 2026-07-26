# Network-Homelab
A homelab created to emulate a possible scenario when setting up a network.

Here is an architecture with three VLANs. VLANs 10 and 20 are dedicated to computers in the Sales and Engineering departments. VLAN 30 is for the Server farm. These computers and Servers are connected to three access switches, which then connect to two switches that also connect to two routers.

## Access Switches

### Access Switch 1 (DanAccess1)

**#First, let’s configure the hostname and the VLANs on the access switches.**

```ios
Hostname DanAccess1
Vlan 10
name Sales VLAN
Vlan 20
name Engineering VLAN
Vlan 30
name Server VLAN
```

**#For security, we will enable the secret password. This forces someone logging in on the router directly to enter an encrypted password if they want to raise their privilege level. We will also put a password on the ssh lines so that anyone who comes in via ssh will have to enter a password. SSHv2 will be enabled since that is more secure. We will enable the password encryption service so that the passwords are encrypted for anyone on the router who attempts to view them. All unused ports will be shut down to minimize the attack surface. Lastly, we will enable port security on our active access ports and instruct them to only allow the MAC address of the directly connected endpoints to connect.**

```ios
enable secret ########
line vty 0 15
login local
transport input ssh
ip domain-name DanNet
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret ##########
```

**#We will configure portfast on the switch, which allows it to skip the convergence time that typically goes along with spanning tree. Access mode is what allows the switches to communicate with the endpoint PCs.**

```ios
int fa0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast
switchport port-security
switchport port-security mac-address XXXX.XXXX.XXXX
switchport port-security violation shutdown

int fa0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast
switchport port-security
switchport port-security mac-address XXXX.XXXX.XXXX
switchport port-security violation shutdown
```

**#This will allow ports fa0/3 and fa0/4 to allow only the vlans 10, 20, and 30 to pass traffic through the link and also enable trunk mode to communicate between switches. The switches in this lab did not need the switchport trunk encapsulation dot1q.**

```ios
interface range fa0/3 – 4
switchport mode trunk
switchport trunk allowed vlan 10,20,30
```

**#The same configuration will be mirrored on switches DanAccess2 and DanAccess3.**

```ios
int range fa0/5 - 24
shut
```

### Access Switch 2 (DanAccess2)

```ios
Hostname DanAccess2
Vlan 10
name Sales_VLAN
Vlan 20
name Engineering VLAN
Vlan 30
name Server VLAN
```



```ios
enable secret ########
line vty 0 15
login local
transport input ssh
ip domain-name DanNet
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret ##########

int fa0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast
switchport port-security
switchport port-security mac-address XXXX.XXXX.XXXX
switchport port-security violation shutdown

int fa0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast
switchport port-security
switchport port-security mac-address XXXX.XXXX.XXXX
switchport port-security violation shutdown

interface range fa0/3 – 4
switchport mode trunk
switchport trunk allowed vlan 10,20,30

int range fa0/5 - 24
shut
```

### Access Switch 3 (Dan Access 3)

```ios
Hostname DanAccess3
Vlan 10
name Sales_VLAN
Vlan 20
name Engineering VLAN
Vlan 30
name Server VLAN

enable secret ########
line vty 0 15
login local
transport input ssh
ip domain-name DanNet
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret ##########

int fa0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast
switchport port-security
switchport port-security mac-address XXXX.XXXX.XXXX
switchport port-security violation shutdown

int fa0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast
switchport port-security
switchport port-security mac-address XXXX.XXXX.XXXX
switchport port-security violation shutdown

interface range fa0/3 – 4
switchport mode trunk
switchport trunk allowed vlan 10,20,30

int range fa0/5 - 24
shut
```

---

## Core Switches

### Core Switch 1 (DanCore1)

**#We will need to proceed to configure the Core switches, starting with Core1. The Ip routing command will enable the switch to operate at layer 3.**

```ios
Hostname DanCore1
Ip routing
```

**#This will put the VLANs on this switch.**

```ios
Vlan 10
Vlan 20
Vlan 30
```

```ios
enable secret ########
line vty 0 15
login local
transport input ssh
ip domain-name DanNet
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret ##########
```

**#Next we will designate the Core1 switch as the root bridge. This is necessary for the spanning tree protocol to know what ports to designate as root, designated, and blocked so that a loop does not occur in the switch environment.**

```ios
Spanning-tree vlan 10,20,30, root primary
```

**#Ports g1/0/1 – g1/0/3 will be configured as trunk ports since those are links between switches**

```ios
Int range g1/0/1 – 3
Switchport mode trunk
Switchport trunk allowed vlan 10,20,30
```

**#Now we will configure an ACL on the switch so that only specific types of traffic can go to the servers. HTTP and HTTPS for the web server. FTP and SSH for the file server. SMTP and POP3 for the mail server.**

```ios
Ip access-list extended SERVERACL
Permit tcp any host 192.168.30.10 eq 80
Permit tcp any host 192.168.30.20 eq 22
Permit tcp any host 192.168.30.30 eq 25
Permit tcp any host 192.168.30.30 eq 993
```

**#We are going to set up HSRP, which is a routing protocol that allows for redundancy. It creates a virtual IP that at least two routers use as their default gateway. If one of the routers goes down, the other will become the active router. Here we designate both the physical address on the link and also the virtual address that will be used as the default gateway by the PCs. Since there are three VLANs, we need to use three different standby groups with three virtual IP default gateways.**

```ios
Interface vlan10
Ip address 192.168.10.2 255.255.255.0
Standby 30 ip 192.168.30.1
Standby 30 priority 110
Standby 30 preempt
```

**#Below we will define the link between the Core1 Switch and the EdgeRouter1. We will need to disable switching on that port to route via IP.**

```ios
Int gi1/0/1
No switchport
Ip address 10.1.1.1 255.255.255.252
```

**#Finally, we implement OSPF and define our OSPF networks.**

```ios
Router ospf 1
Router-id 1.1.1.1
Network 192.168.20.0 0.0.0.255 area 0
Network 192.168.30.0 0.0.0.255 area 0
Network 10.1.1.0 0.0.0.3 area 0
```

### Core Switch 2 (DanCore2)

**#The Core2 switch will be a similar configuration, only differing on the IP addresses of the physical links and the spanning tree priority.**

```ios
Hostname DanCore2
Ip routing

Spanning-tree vlan 10,20,30, root secondary

Int range g1/0/1 – 3
Switchport mode trunk
Switchport trunk allowed vlan 10,20,30

enable secret ########
line vty 0 15
login local
transport input ssh
ip domain-name DanNet
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret ##########

Ip access-list extended SERVERACL
Permit tcp any host 192.168.30.10 eq 80
Permit tcp any host 192.168.30.10 eq 443

Router ospf 1
Router-id 2.2.2.2
Network 192.168.20.0 0.0.0.255 area 0
Network 192.168.30.0 0.0.0.255 area 0
Network 10.2.2.0 0.0.0.3 area 0
```

---

## Edge Routers

### Edge Router 1 (DanEdge1)

**#Lastly, we have the Edge Router configuration. This will just be typically IP address and route specification.**

```ios
Hostname DanEdge1
enable secret ########
line vty 0 15
login local
transport input ssh
ip domain-name DanNet
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret ##########

Int gi0/0
Ip address 10.1.1.2 255.255.255.252
No shut

Router ospf 1
Network 10.1.1.0 0.0.0.3 area 0
Default-information originate

Ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 192.168.10.0 255.255.255.0 10.1.1.1 120
ip route 192.168.20.0 255.255.255.0 10.1.1.1 120
ip route 192.168.30.0 255.255.255.0 10.1.1.1 120
```

### Edge Router 1 (DanEdge1)

**#Lastly, we have the Edge Router configuration. This will just be typically IP address and route specification.**

```ios
Hostname DanEdge2
enable secret ########
line vty 0 15
login local
transport input ssh
ip domain-name DanNet
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret ##########

Int gi0/0
Ip address 10.2.2.2 255.255.255.252
No shut

Router ospf 1
Network 10.2.2.0 0.0.0.3 area 0
Default-information originate

Ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 192.168.10.0 255.255.255.0 10.1.1.1 120
ip route 192.168.20.0 255.255.255.0 10.1.1.1 120
ip route 192.168.30.0 255.255.255.0 10.1.1.1 120
```
