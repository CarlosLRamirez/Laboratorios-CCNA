# Laboratorio CCNA: Enrutamiento Inter-VLAN, DHCP, HSRP, Voice Vlan

Update: version 2 en progreso

## Topologia

![07-CCNALab-Topologia](images/07-CCNA%20Lab-Topology.png)

## Archivo PT

Configuración verson 2 [final] (labs/Lab07-ver2-inicial.pkt)

## Configuración paso a paso

### Configuracion básica

En todos los dispostivos de red:

```
hostname CORE2
ip domain-name edutek.edu
enable secret cisco
no ip domain-lookup 
service password-encryption
line console 0
password cisco
login
logging synchronous 
exit
crypto key generate rsa
1024
ip ssh version 2
username admin secret cisco
username admin privilege 15
line vty 0 15
login local
transport input ssh 
exit
```


### Creacion de VLANS


En TODOS los switches:
```
vlan 2
name datos
vlan 3
name voip
vlan 4
name iot
vlan 10
name it
exit
```


### Puertos de Acceso

En los switches de ACCESO:

```
interface range fa0/1-20
description pc_telefono
switchport mode access
switchport access vlan 2
switchport voice vlan 3
interface range fa0/21-24
description iot
switchport mode access
switchport access vlan 4
exit
```

### Puertos de Uplink

En los switches de ACCESO:


```
interface range gi0/1-2
description uplink
switchport mode trunk
switchport trunk allowed vlan 2,3,4,10
switchport trunk native vlan 10
exit
```

### Trunks en los switches de distribucion

En los switches de distribucion:

```
interface range fa0/19-24
switchport mode trunk
switchport trunk allowed vlan 2,3,4,10
switchport trunk native vlan 10
exit
interface range fa0/20-21
description etherchannel
shutdown
channel-group 1 mode active
interface po1
switchport mode trunk
switchport trunk allowed vlan 2,3,4,10
switchport trunk native vlan 10
interface range fa0/20-21
no shutdown
exit
```

### Intervlan Routing y HSRP

#### CORE1

```
interface gi0/0/0.2
encapsulation dot1q 2
ip address 192.168.2.2 255.255.255.0
standby 2 ip 192.168.2.1
standby 2 priority 200
standby 2 preempt
exit

interface gi0/0/0.3
encapsulation dot1q 3
ip address 192.168.3.2 255.255.255.0
standby 3 ip 192.168.3.1
standby 3 priority 200
standby 3 preempt
exit

interface gi0/0/0.4
encapsulation dot1q 4
ip address 192.168.4.2 255.255.255.0
standby 4 ip 192.168.4.1
standby 4 priority 200
standby 4 preempt
exit

interface gi0/0/0.10
encapsulation dot1q 10 native
ip address 192.168.10.2 255.255.255.0
standby 10 ip 192.168.10.1
standby 10 priority 200
standby 10 preempt
exit

interface gi0/0/0
no shutdown
exit
```

#### CORE2

```
interface gi0/0/0.2
encapsulation dot1q 2
ip address 192.168.2.3 255.255.255.0
standby 2 ip 192.168.2.1
exit

interface gi0/0/0.3
encapsulation dot1q 3
ip address 192.168.3.3 255.255.255.0
standby 3 ip 192.168.3.1
exit

interface gi0/0/0.4
encapsulation dot1q 4
ip address 192.168.4.3 255.255.255.0
standby 4 ip 192.168.4.1
exit

interface gi0/0/0.10
encapsulation dot1q 10 native
ip address 192.168.10.3 255.255.255.0
standby 10 ip 192.168.10.1
exit

interface gi0/0/0
no shutdown
exit
```

### Servidor DHCP (Router)

```
hostname DHCP
interface gi0/0
ip address 192.168.2.99 255.255.255.0
no shutdown
exit
ip route 0.0.0.0 0.0.0.0 192.168.2.1

ip dhcp excluded-address 192.168.2.1 192.168.2.99
ip dhcp excluded-address 192.168.3.1 192.168.3.99
ip dhcp excluded-address 192.168.4.1 192.168.4.99
ip dhcp excluded-address 192.168.10.1 192.168.10.99

ip dhcp pool datos
network 192.168.2.0 255.255.255.0
default-router 192.168.2.1
dns-server 10.10.10.10
domain-name edutek.edu

ip dhcp pool voip
network 192.168.3.0 255.255.255.0
default-router 192.168.3.1
dns-server 10.10.10.10
domain-name edutek.edu

ip dhcp pool iot
network 192.168.4.0 255.255.255.0
default-router 192.168.4.1
dns-server 10.10.10.10
domain-name edutek.edu

ip dhcp pool it
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 10.10.10.10
domain-name edutek.edu
exit
```

### DHCP Relay (helper address) - ambos Core 
```
interface gi0/0/0.3
ip helper-address 192.168.2.99
interface gi0/0/0.4
ip helper-address 192.168.2.99
interface gi0/0/0.10
ip helper-address 192.168.2.99
exit
```

### Enlaces WAN y OSPF

En CORE1

```
interface Se0/1/0
description to CORE2
ip address 10.0.0.10 255.255.255.252
no shutdown
interface Se0/1/1
description to ISP
ip address 10.0.0.1 255.255.255.252
no shutdown
exit
```

En CORE1

```
interface Se0/1/0
description to CORE1
ip address 10.0.0.9 255.255.255.252
no shutdown
interface Se0/1/1
description to ISP
ip address 10.0.0.6 255.255.255.252
no shutdown
exit
```

En ISP

```
interface Se0/1/0
description to CORE1
ip address 10.0.0.2 255.255.255.252
no shutdown
interface Se0/1/1
description to CORE2
ip address 10.0.0.5 255.255.255.252
no shutdown
exit
interface G0/0
description INTERNET
ip address 200.10.10.2 255.255.255.252
no shutdown
exit
```






## Pruebas 
- Debe existir conectividad entre PCs, Telefono y dispositivo IoT.
- PCs, Telefonos e IoT deben obtener IP por DHCP según el segmento de red que le corresponde.
- Las PCs deben poder alcanzar por el explorador http://web.cisco.lab




