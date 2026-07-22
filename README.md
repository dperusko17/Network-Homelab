# Network-Homelab
A homelab created to emulate a possible scenario when setting up a network.

Here is an architecture with three VLANs. VLANs 10 and 20 are dedicated to computers in the Sales and Engineering departments. VLAN 30 is for the Server farm. These computers and Servers are connected to three access switches, which then connect to two switches that also connect to two routers.

#First, let’s configure the hostname and the VLANs on the access switches.

Hostname DanAccess1
Vlan 10
name Sales_VLAN
Vlan 20
name Engineering VLAN
Vlan 30
name Server VLAN

#This will configure portfast on the switch, which allows it to skip the convergence time that typically goes along with spanning tree. Access mode is what allows the switches to communicate with the endpoint PCs.

int fa0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast

int fa0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast

#This will allow ports fa0/3 and fa0/4 to allow only the vlans 10, 20, and 30 to pass traffic through the link and also enable trunk mode to communicate between switches. The switches in this lab did not need the switchport trunk encapsulation dot1q.

interface range fa0/3 – 4
switchport mode trunk
switchport trunk allowed vlan 10,20,30

#The same configuration will be mirrored on switches DanAccess2 and DanAccess3.
Hostname DanAccess1
Vlan 10
name Sales_VLAN
Vlan 20
name Engineering VLAN
Vlan 30
name Server VLAN

int fa0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast

int fa0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast

interface range fa0/3 – 4
switchport mode trunk
switchport trunk allowed vlan 10,20,30


Hostname DanAccess1
Vlan 10
name Sales_VLAN
Vlan 20
name Engineering VLAN
Vlan 30
name Server VLAN

int fa0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast

int fa0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast

interface range fa0/3 – 4
switchport mode trunk
switchport trunk allowed vlan 10,20,30


#We will need to proceed to configure the Core switches, starting with Core1. The Ip routing command will enable the switch to operate at layer 3.
Hostname DanCore1
Ip routing

#This will put the VLANs on this switch.

Vlan 10
Vlan 20
Vlan 30

#Next we will designate the Core1 switch as the root bridge. This is necessary for the spanning tree protocol to know what ports to designate as root, designated, and blocked so that a loop does not occur in the switch environment.

Spanning-tree vlan 10,20,30, root primary

#Ports g1/0/1 – g1/0/3 will be configured as trunk ports since those are links between switches

Int range g1/0/1 – 3
Switchport mode trunk
Switchport trunk allowed vlan 10,20,30

#Now we will configure an ACL on the switch so that only specific types of traffic can go to the servers. HTTP and HTTPS for the web server. FTP and SSH for the file server. SMTP and POP3 for the mail server.

Ip access-list extended SERVERACL
Permit tcp any host 192.168.30.10 eq 80
Permit tcp any host 192.168.30.10 eq 443
Permit tcp any host 192.168.30.20 eq 21
Permit tcp any host 192.168.30.20 eq 22
Permit tcp any host 192.168.30.30 eq 25
Permit tcp any host 192.168.30.30 eq 993

#We are going to set up HSRP, which is a routing protocol that allows for redundancy. It creates a virtual IP that at least two routers use as their default gateway. If one of the routers goes down, the other will become the active router. Here we designate both the physical address on the link and also the virtual address that will be used as the default gateway by the PCs. Since there are three VLANs, we need to use three different standby groups with three virtual IP default gateways.

Interface vlan10
Ip address 192.168.10.2 255.255.255.0
Standby 10 ip 192.168.10.1
Standby 10 priority 110
Standby 10 preempt

Interface vlan20
Ip address 192.168.20.2 255.255.255.0
Standby 20 ip 192.168.20.1
Standby 20 priority 110
Standby 20 preempt

Interface vlan30
Ip address 192.168.30.2 255.255.255.0
Standby 30 ip 192.168.30.1
Standby 30 priority 110
Standby 30 preempt

#Below we will define the link between the Core1 Switch and the EdgeRouter1. We will need to disable switching on that port to route via IP.

Int gi1/0/1
No switchport
Ip address 10.1.1.1 255.255.255.252

#Finally, we implement OSPF and define our OSPF networks.

Router ospf 1
Router-id 1.1.1.1
Network 192.168.10.0 0.0.0.255 area 0
Network 192.168.20.0 0.0.0.255 area 0
Network 192.168.30.0 0.0.0.255 area 0
Network 10.1.1.0 0.0.0.3 area 0

#The Core2 switch will be a similar configuration, only differing on the IP addresses of the physical links and the spanning tree priority.

Spanning-tree vlan 10,20,30, root secondary

#Ports g1/0/1 – g1/0/3 will be configured as trunk ports since those are links between switches

Int range g1/0/1 – 3
Switchport mode trunk
Switchport trunk allowed vlan 10,20,30

Ip access-list extended SERVERACL
Permit tcp any host 192.168.30.10 eq 80
Permit tcp any host 192.168.30.10 eq 443
Permit tcp any host 192.168.30.20 eq 21
Permit tcp any host 192.168.30.20 eq 22
Permit tcp any host 192.168.30.30 eq 25
Permit tcp any host 192.168.30.30 eq 993

Interface vlan10
Ip address 192.168.10.3 255.255.255.0
Standby 10 ip 192.168.10.1
Standby 10 priority 100
Standby 10 preempt

Interface vlan20
Ip address 192.168.20.3 255.255.255.0
Standby 20 ip 192.168.20.1
Standby 20 priority 100
Standby 20 preempt

Interface vlan30
Ip address 192.168.30.3 255.255.255.0
Standby 30 ip 192.168.30.1
Standby 30 priority 100
Standby 30 preempt

Int gi1/0/1
No switchport
Ip address 10.1.1.1 255.255.255.252

Router ospf 1
Router-id 2.2.2.2
Network 192.168.10.0 0.0.0.255 area 0
Network 192.168.20.0 0.0.0.255 area 0
Network 192.168.30.0 0.0.0.255 area 0
Network 10.2.2.0 0.0.0.3 area 0

#Lastly, we have the Edge Router configuration. This will just be typically IP address and route specification.

Hostname DanEdge1

Int gi0/0
Ip address 10.1.1.2 255.255.255.252
No shut

Int g0/1
Ip address 203.0.113.1 255.255.255.252
No shut

Router ospf 1 router-id 4.4.4.4
Network 10.1.1.0 0.0.0.3 area 0
Default-information originate

Ip route 0.0.0.0 0.0.0.0 203.0.113.1

#And a similar configuration on the 2nd Edge router.

Hostname DanEdge2

Int gi0/0
Ip address 10.2.2.2 255.255.255.252
No shut

Int g0/1
Ip address 203.0.113.2 255.255.255.252
No shut

Router ospf 1 router-id 5.5.5.5
Network 10.2.2.0 0.0.0.3 area 0
Default-information originate

Ip route 0.0.0.0 0.0.0.0 203.0.113.2




