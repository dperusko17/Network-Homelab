# Network-Homelab
A homelab created to emulate a possible scenario when setting up a network.

Here is an architecture with three VLANs. VLANs 10 and 20 are dedicated to computers in the Sales and Engineering departments. VLAN 30 is for the Server farm. These computers and Servers are connected to three access switches, which then connect to two switches that also connect to two routers.

## Access Switch 1 (DanAccess1)

**#First, let’s configure the hostname and the VLANs on the access switches.**

```ios
DanAccess1(config)# hostname DanAccess1
DanAccess1(config)# Vlan 10
DanAccess1(config-vlan) name Sales_VLAN
DanAccess1(config)# Vlan 20
DanAccess1(config-vlan) name Engineering VLAN
DanAccess1(config)# Vlan 30
DanAccess1(config-vlan) name Server VLAN
```

**#For security, we will enable the secret password. This forces someone logging in on the router directly to enter an encrypted password if they want to raise their privilege level. We will also put a password on the ssh lines so that anyone who comes in via ssh will have to enter a password. SSHv2 will be enabled since that is more secure. We will enable the password encryption service so that the passwords are encrypted for anyone on the router who attempts to view them. All unused ports will be shut down to minimize the attack surface. Lastly, we will enable port security on our active access ports and instruct them to only allow the MAC address of the directly connected endpoints to connect.**

```ios
DanAccess1(config)# enable secret ########
DanAccess1(config-line)# line vty 0 15
DanAccess1(config-line)# login local
DanAccess1(config-line)# transport input ssh
DanAccess1(config)# ip domain-name DanNet
DanAccess1(config)# crypto key generate rsa modulus 2048
DanAccess1(config)# ip ssh version 2
DanAccess1(config)# username admin privilege 15 secret ##########
```

**#We will configure portfast on the switch, which allows it to skip the convergence time that typically goes along with spanning tree. Access mode is what allows the switches to communicate with the endpoint PCs.**

```ios
DanAccess1(config)# int fa0/1
DanAccess1(config-if)#switchport mode access
DanAccess1(config-if)#switchport access vlan 10
DanAccess1(config-if)# spanning-tree portfast
DanAccess1(config-if)# switchport port-security
DanAccess1(config-if)# switchport port-security mac-address XXXX.XXXX.XXXX
DanAccess1(config-if)# switchport port-security violation shutdown
```

```ios
DanAccess1(config)#int fa0/2
DanAccess1(config-if)# switchport mode access
DanAccess1(config-if)# switchport access vlan 20
DanAccess1(config-if)# spanning-tree portfast
DanAccess1(config-if)# switchport port-security
DanAccess1(config-if)# switchport port-security mac-address XXXX.XXXX.XXXX
DanAccess1(config-if)# switchport port-security violation shutdown
```

**#This will allow ports fa0/3 and fa0/4 to allow only the vlans 10, 20, and 30 to pass traffic through the link and also enable trunk mode to communicate between switches. The switches in this lab did not need the switchport trunk encapsulation dot1q.**

```ios
DanAccess1(config)#interface range fa0/3 – 4
DanAccess1(config-if-range)# switchport mode trunk
DanAccess1(config-if-range)# switchport trunk allowed vlan 10,20,30

DanAccess1(config-if-range)# int range fa0/5 - 24 
DanAccess1(config-if-range)# shut
```

**#The same configuration will be mirrored on switches DanAccess2 and DanAccess3.**

## Access Switch 2 (DanAccess2)

```ios
DanAccess2(config)# Hostname DanAccess2
DanAccess2(config)# Vlan 10
DanAccess2(config-vlan)# name Sales_VLAN
DanAccess2(config)# Vlan 20
DanAccess2(config-vlan)# name Engineering VLAN
DanAccess2(config)# Vlan 30
DanAccess2(config-vlan)# name Server VLAN
```

**#For security, we will enable the secret password. This forces someone logging in on the router directly to enter an encrypted password if they want to raise their privilege level. We will also put a password on the ssh lines so that anyone who comes in via ssh will have to enter a password. SSHv2 will be enabled since that is more secure. We will enable the password encryption service so that the passwords are encrypted for anyone on the router who attempts to view them. All unused ports will be shut down to minimize the attack surface. Lastly, we will enable port security on our active access ports and instruct them to only allow the MAC address of the directly connected endpoints to connect.**

```ios
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
```

## Access Switch 3 (Dan Access 3)

```ios
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
```
