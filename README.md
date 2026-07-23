# Network-Homelab
A homelab created to emulate a possible scenario when setting up a network.

Here is an architecture with three VLANs. VLANs 10 and 20 are dedicated to computers in the Sales and Engineering departments. VLAN 30 is for the Server farm. These computers and Servers are connected to three access switches, which then connect to two switches that also connect to two routers.

Access Switch 1 (DanAccess1)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**#First, let’s configure the hostname and the VLANs on the access switches.**

---------------------------------------------------------

DanAccess1(config)# hostname DanAccess1
DanAccess1(config)# Vlan 10
DanAccess1(config-vlan) name Sales_VLAN
DanAccess1(config)# Vlan 20
DanAccess1(config-vlan) name Engineering VLAN
DanAccess1(config)# Vlan 30
DanAccess1(config-vlan) name Server VLAN

---------------------------------------------------------

**#For security, we will enable the secret password. This forces someone logging in on the router directly to enter an encrypted password if they want to raise their privilege level. We will also put a password on the ssh lines so that anyone who comes in via ssh will have to enter a password. SSHv2 will be enabled since that is more secure. We will enable the password encryption service so that the passwords are encrypted for anyone on the router who attempts to view them. All unused ports will be shut down to minimize the attack surface. Lastly, we will enable port security on our active access ports and instruct them to only allow the MAC address of the directly connected endpoints to connect.**


---------------------------------------------------------


DanAccess1(config)# enable secret ########
DanAccess1(config-line)# line vty 0 15
DanAccess1(config-line)# login local
DanAccess1(config-line)# transport input ssh
DanAccess1(config)# ip domain-name DanNet
DanAccess1(config)# crypto key generate rsa modulus 2048
DanAccess1(config)# ip ssh version 2
DanAccess1(config)# username admin privilege 15 secret ##########


---------------------------------------------------------


**#We will configure portfast on the switch, which allows it to skip the convergence time that typically goes along with spanning tree. Access mode is what allows the switches to communicate with the endpoint PCs.**


---------------------------------------------------------


DanAccess1(config)# int fa0/1
DanAccess1(config-if)#switchport mode access
DanAccess1(config-if)#switchport access vlan 10
DanAccess1(config-if)# spanning-tree portfast
DanAccess1(config-if)# switchport port-security
DanAccess1(config-if)# switchport port-security mac-address XXXX.XXXX.XXXX
DanAccess1(config-if)# switchport port-security violation shutdown


---------------------------------------------------------


DanAccess1(config)#int fa0/2
DanAccess1(config-if)# switchport mode access
DanAccess1(config-if)# switchport access vlan 20
DanAccess1(config-if)# spanning-tree portfast
DanAccess1(config-if)# switchport port-security
DanAccess1(config-if)# switchport port-security mac-address XXXX.XXXX.XXXX
DanAccess1(config-if)# switchport port-security violation shutdown


---------------------------------------------------------


**#This will allow ports fa0/3 and fa0/4 to allow only the vlans 10, 20, and 30 to pass traffic through the link and also enable trunk mode to communicate between switches. The switches in this lab did not need the switchport trunk encapsulation dot1q.**


---------------------------------------------------------


DanAccess1(config)#interface range fa0/3 – 4
DanAccess1(config-if-range)# switchport mode trunk
DanAccess1(config-if-range)# switchport trunk allowed vlan 10,20,30

DanAccess1(config-if-range)# int range fa0/5 - 24 
DanAccess1(config-if-range)# shut


---------------------------------------------------------


**#The same configuration will be mirrored on switches DanAccess2 and DanAccess3.**

Access Switch 2 (DanAccess2)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

DanAccess2(config)# Hostname DanAccess2
DanAccess2(config)# Vlan 10
DanAccess2(config-vlan)# name Sales_VLAN
DanAccess2(config)# Vlan 20
DanAccess2(config-vlan)# name Engineering VLAN
DanAccess2(config)# Vlan 30
DanAccess2(config-vlan)# name Server VLAN


---------------------------------------------------------


**#For security, we will enable the secret password. This forces someone logging in on the router directly to enter an encrypted password if they want to raise their privilege level. We will also put a password on the ssh lines so that anyone who comes in via ssh will have to enter a password. SSHv2 will be enabled since that is more secure. We will enable the password encryption service so that the passwords are encrypted for anyone on the router who attempts to view them. All unused ports will be shut down to minimize the attack surface. Lastly, we will enable port security on our active access ports and instruct them to only allow the MAC address of the directly connected endpoints to connect.**


---------------------------------------------------------


DanAccess2(config)# enable secret ########
DanAccess2(config)# line vty 0 15
DanAccess2(config)# login local
DanAccess2(config)# transport input ssh
DanAccess2(config)# ip domain-name DanNet
DanAccess2(config)# crypto key generate rsa modulus 2048
DanAccess2(config)# ip ssh version 2
DanAccess2(config)# username admin privilege 15 secret ##########

DanAccess2(config)#int fa0/1
DanAccess2(config-if)# switchport mode access
DanAccess2(config-if)# switchport access vlan 10
DanAccess2(config-if)# spanning-tree portfast
DanAccess2(config-if)# switchport port-security
DanAccess2(config-if)# switchport port-security mac-address XXXX.XXXX.XXXX
DanAccess2(config-if)# switchport port-security violation shutdown

DanAccess2(config-if)# int fa0/2
DanAccess2(config-if)# switchport mode access
DanAccess2(config-if)# switchport access vlan 20
DanAccess2(config-if)# spanning-tree portfast
DanAccess2(config-if)# switchport port-security
DanAccess2(config-if)# switchport port-security mac-address XXXX.XXXX.XXXX
DanAccess2(config-if)# switchport port-security violation shutdown

DanAccess2(config-if)# interface range fa0/3 – 4
DanAccess2(config-if-range)# switchport mode trunk
DanAccess2(config-if-range)# switchport trunk allowed vlan 10,20,30

DanAccess2(config-if-range)# int range fa0/5 - 24 
DanAccess2(config-if-range)# shut

Access Switch 3 (Dan Access 3)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

DanAccess3(config)# Hostname DanAccess3
DanAccess3(config)# Vlan 10
DanAccess3(config-vlan)# name Sales_VLAN
DanAccess3(config)# Vlan 20
DanAccess3(config-vlan)# name Engineering VLAN
DanAccess3(config)# Vlan 30
DanAccess3(config-vlan)# name Server VLAN

DanAccess3(config)# enable secret ########
DanAccess3(config)# line vty 0 15
DanAccess3(config)# login local
DanAccess3(config)# transport input ssh
DanAccess3(config)# ip domain-name DanNet
DanAccess3(config)# crypto key generate rsa modulus 2048
DanAccess3(config)# ip ssh version 2
DanAccess3(config)# username admin privilege 15 secret ##########

DanAccess3(config)# int fa0/1
DanAccess3(config-if)# switchport mode access
DanAccess3(config-if)# switchport access vlan 10
DanAccess3(config-if)# spanning-tree portfast
DanAccess3(config-if)# switchport port-security
DanAccess3(config-if)# switchport port-security mac-address XXXX.XXXX.XXXX
DanAccess3(config-if)# switchport port-security violation shutdown

DanAccess3(config)# int fa0/2
DanAccess3(config-if)# switchport mode access
DanAccess3(config-if)# switchport access vlan 20
DanAccess3(config-if)# spanning-tree portfast
DanAccess3(config-if)# switchport port-security
DanAccess3(config-if)# switchport port-security mac-address XXXX.XXXX.XXXX
DanAccess3(config-if)# switchport port-security violation shutdown

DanAccess3(config)# interface range fa0/3 – 4
DanAccess3(config-if-range)# switchport mode trunk
DanAccess3(config-if-range)# switchport trunk allowed vlan 10,20,30

DanAccess3(config)# int range fa0/5 - 24
DanAccess3(config-if-range)#shut



**#We will need to proceed to configure the Core switches, starting with Core1. The Ip routing command will enable the switch to operate at layer 3.**

Core Switch 1 (DanCore1)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

DanCore1(config)# Hostname DanCore1
DanCore1(config)# Ip routing


---------------------------------------------------------


**#This will put the VLANs on this switch.**

DanCore1(config)# Vlan 10
DanCore1(config)# Vlan 20
DanCore1(config)# Vlan 30

DanCore1(config)# enable secret ########
DanCore1(config)# line vty 0 15
DanCore1(config)# login local
DanCore1(config)# transport input ssh
DanCore1(config)# ip domain-name DanNet
DanCore1(config)# crypto key generate rsa modulus 2048
DanCore1(config)# ip ssh version 2
DanCore1(config)# username admin privilege 15 secret ##########


---------------------------------------------------------


**#Next we will designate the Core1 switch as the root bridge. This is necessary for the spanning tree protocol to know what ports to designate as root, designated, and blocked so that a loop does not occur in the switch environment.**


---------------------------------------------------------


DanCore1(config)# Spanning-tree vlan 10,20,30, root primary


---------------------------------------------------------


**#Ports g1/0/1 – g1/0/3 will be configured as trunk ports since those are links between switches**


---------------------------------------------------------


DanCore1(config)# Int range g1/0/1 – 3
DanCore1(config-if-range)#Switchport mode trunk
DanCore1(config-if-range))#Switchport trunk allowed vlan 10,20,30


---------------------------------------------------------


**#Now we will configure an ACL on the switch so that only specific types of traffic can go to the servers. HTTP and HTTPS for the web server. FTP and SSH for the file server. SMTP and POP3 for the mail server.**


---------------------------------------------------------


DanCore1(config)# Ip access-list extended SERVERACL
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.10 eq 80
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.10 eq 443
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.20 eq 21
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.20 eq 22
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.30 eq 25
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.30 eq 993


---------------------------------------------------------


**#We are going to set up HSRP, which is a routing protocol that allows for redundancy. It creates a virtual IP that at least two routers use as their default gateway. If one of the routers goes down, the other will become the active router. Here we designate both the physical address on the link and also the virtual address that will be used as the default gateway by the PCs. Since there are three VLANs, we need to use three different standby groups with three virtual IP default gateways.**


---------------------------------------------------------


DanCore1(config)# Interface vlan10
DanCore1(config-if)# Ip address 192.168.10.2 255.255.255.0
DanCore1(config-if)# Standby 10 ip 192.168.10.1
DanCore1(config-if)# Standby 10 priority 110
DanCore1(config-if)# Standby 10 preempt

DanCore1(config)# Interface vlan20
DanCore1(config-if)# Ip address 192.168.20.2 255.255.255.0
DanCore1(config-if)# Standby 20 ip 192.168.20.1
DanCore1(config-if)# Standby 20 priority 110
DanCore1(config-if)# Standby 20 preempt

DanCore1(config)# Interface vlan30
DanCore1(config-if)# Ip address 192.168.30.2 255.255.255.0
DanCore1(config-if)# Standby 30 ip 192.168.30.1
DanCore1(config-if)# Standby 30 priority 110
DanCore1(config-if)# Standby 30 preempt


---------------------------------------------------------


**#Below we will define the link between the Core1 Switch and the EdgeRouter1. We will need to disable switching on that port to route via IP.**


---------------------------------------------------------


DanCore1(config)# Int gi1/0/1
DanCore1(config-if)# No switchport
DanCore1(config-if)# Ip address 10.1.1.1 255.255.255.252


---------------------------------------------------------


**#Finally, we implement OSPF and define our OSPF networks.**


---------------------------------------------------------


DanCore1(config)#Router ospf 1
DanCore1(config-router)# Router-id 1.1.1.1
DanCore1(config-router)# Network 192.168.10.0 0.0.0.255 area 0
DanCore1(config-router)# Network 192.168.20.0 0.0.0.255 area 0
DanCore1(config-router)# Network 192.168.30.0 0.0.0.255 area 0
DanCore1(config-router)# Network 10.1.1.0 0.0.0.3 area 0


---------------------------------------------------------

**#The Core2 switch will be a similar configuration, only differing on the IP addresses of the physical links and the spanning tree priority.**

Core Switch 2 (DanCore2)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

DanCore1(config)# hostname DanCore2
DanCore1(config)# ip routing
DanCore1(config)# Spanning-tree vlan 10,20,30, root secondary

#Ports g1/0/1 – g1/0/3 will be configured as trunk ports since those are links between switches

DanCore1(config)# Int range g1/0/1 – 3
DanCore1(config-if-range)# Switchport mode trunk
DanCore1(config-if-range)# Switchport trunk allowed vlan 10,20,30

DanCore1(config)# enable secret ########
DanCore1(config)# line vty 0 15
DanCore1(config)# login local
DanCore1(config)# transport input ssh
DanCore1(config)# ip domain-name DanNet
DanCore1(config)# crypto key generate rsa modulus 2048
DanCore1(config)# ip ssh version 2
DanCore1(config)# username admin privilege 15 secret ##########

DanCore1(config)# Ip access-list extended SERVERACL
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.10 eq 80
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.10 eq 443
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.20 eq 21
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.20 eq 22
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.30 eq 25
DanCore1(config-ext-nacl)# Permit tcp any host 192.168.30.30 eq 993

DanCore1(config)# Interface vlan10
DanCore1(config-if)# Ip address 192.168.10.3 255.255.255.0
DanCore1(config-if)# Standby 10 ip 192.168.10.1
DanCore1(config-if)# Standby 10 priority 100
DanCore1(config-if)# Standby 10 preempt

DanCore1(config)# Interface vlan20
DanCore1(config-if)# Ip address 192.168.20.3 255.255.255.0
DanCore1(config-if)# Standby 20 ip 192.168.20.1
DanCore1(config-if)# Standby 20 priority 100
DanCore1(config-if)# Standby 20 preempt

DanCore1(config)# Interface vlan30
DanCore1(config-if)# Ip address 192.168.30.3 255.255.255.0
DanCore1(config-if)# Standby 30 ip 192.168.30.1
DanCore1(config-if)# Standby 30 priority 100
DanCore1(config-if)# Standby 30 preempt

DanCore1(config)# Int gi1/0/1
DanCore1(config-if)# No switchport
DanCore1(config-if)# Ip address 10.1.1.1 255.255.255.252

DanCore1(config)# Router ospf 1
DanCore1(config-router)# Router-id 2.2.2.2
DanCore1(config-router)# Network 192.168.10.0 0.0.0.255 area 0
DanCore1(config-router)# Network 192.168.20.0 0.0.0.255 area 0
DanCore1(config-router)# Network 192.168.30.0 0.0.0.255 area 0
DanCore1(config-router)# Network 10.2.2.0 0.0.0.3 area 0


**#Lastly, we have the Edge Router configuration. This will just be typically IP address and route specification.**

Edge Router 1 (DanEdge1)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

DanCore1(config)# Hostname DanEdge1

DanCore1(config)# enable secret ########
DanCore1(config)# line vty 0 15
DanCore1(config)# login local
DanCore1(config)# transport input ssh
DanCore1(config)# ip domain-name DanNet
DanCore1(config)# crypto key generate rsa modulus 2048
DanCore1(config)# ip ssh version 2
DanCore1(config)# username admin privilege 15 secret ##########

DanCore1(config)# Int gi0/0
DanCore1(config)-if)# Ip address 10.1.1.2 255.255.255.252
DanCore1(config)-if)# No shut

DanCore1(config)# Int g0/1
DanCore1(config-if)# Ip address 203.0.113.1 255.255.255.252
DanCore1(config-if)# No shut

DanCore1(config)# Router ospf 1 router-id 4.4.4.4
DanCore1(config-router)# Network 10.1.1.0 0.0.0.3 area 0
DanCore1(config-router)# Default-information originate

DanCore1(config)# Ip route 0.0.0.0 0.0.0.0 203.0.113.1
DanCore1(config)# ip route 192.168.10.0 255.255.255.0 10.1.1.1 120
DanCore1(config)# ip route 192.168.20.0 255.255.255.0 10.1.1.1 120
DanCore1(config)# ip route 192.168.30.0 255.255.255.0 10.1.1.1 120

#And a similar configuration on the 2nd Edge router.

Edge Router 2 (DanEdge2)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

DanCore1(config)# Hostname DanEdge2

DanCore1(config)# enable secret ########
DanCore1(config)# line vty 0 15
DanCore1(config)# login local
DanCore1(config)# transport input ssh
DanCore1(config)# ip domain-name DanNet
DanCore1(config)# crypto key generate rsa modulus 2048
DanCore1(config)# ip ssh version 2
DanCore1(config)# username admin privilege 15 secret ##########

DanCore1(config)# Int gi0/0
DanCore1(config-if)# Ip address 10.2.2.2 255.255.255.252
DanCore1(config-if)# No shut

DanCore1(config)# Int g0/1
DanCore1(config-if)# Ip address 203.0.113.2 255.255.255.252
DanCore1(config-if)# No shut

DanCore1(config-router)# Router ospf 1 router-id 5.5.5.5
DanCore1(config-router)# Network 10.2.2.0 0.0.0.3 area 0
DanCore1(config-router)# Default-information originate

DanCore1(config)# Ip route 0.0.0.0 0.0.0.0 203.0.113.2
DanCore1(config)# ip route 192.168.10.0 255.255.255.0 10.1.1.1 120
DanCore1(config)# ip route 192.168.20.0 255.255.255.0 10.1.1.1 120
DanCore1(config)# ip route 192.168.30.0 255.255.255.0 10.1.1.1 120

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

With this configuration, we have created a secure and fault-tolerant network.
