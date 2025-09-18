Solución Repaso General CCNA 2

https://drive.google.com/file/d/1p9Eb3tbSiZEHdGaDxfS48iNiM5oa5SUJ/view?usp=drive_link

# Laboratorio de Integración CCNA 2

## Información de Direccionamiento IP

#### Data Center

| VLAN ID | NAME        | NETWORK     | Netmask       | Ports    |
|---------|-------------|-------------|---------------|----------|
|     100 | DATA CENTER | 10.1.0.0/20 | 255.255.240.0 | Fa0/1-20 |

#### Red Headquarters

| VLAN ID | Name       | Network     | Netmask         | Access Ports | Default Gateway |
| ------- | ---------- | ----------- | --------------- | ------------ | --------------- |
|     100 | OPERATIONS | 10.24.0.0   | 255.255.254.0   | F0/1-9       | 10.24.1.254     |
|     101 | ACCOUNTING | 10.24.2.0   | 255.255.255.0   | F0/10-15     | 10.24.2.254     |
|     102 | MARKETING  | 10.24.3.0   | 255.255.255.128 | F0/16-22     | 10.24.3.126     |
|      99 | IT-MGMT    | 10.24.3.128 | 255.255.255.192 | VLAN 99      | 10.24.3.190     |
|      98 | NATIVA     |             |                 | F0/23-24     |                 |

#### Branch Office
| VLAN ID | Name        | Network       | Netmask       | Access Ports | Default Gateway |
| ------- | ----------- | ------------- | ------------- | ------------ | --------------- |
| 10      | GENERAL     | 10.16.10.0/24 | 255.255.255.0 | F0/1-20      | 10.16.10.1      |
| 11      | CONFERENCIA | 10.16.11.0/24 | 255.255.255.0 | N/A (WIFI)   | 10.16.11.1      |
| 12      | INVITADOS   | 10.16.12.0    | 255.255.255.0 | N/A (WIFI)   | 10.16.12.1      |
| 99      | MGMT-AP     | 10.16.99.0    | 255.255.255.0 | F0/21-24     | 10.16.99.1      |
| 100     | NATIVE      |               |               |              |                 |




## Instrucciones

### Port Security

### Spanning Tree

- Configura el Spanning Tree Protocol (STP) para distribuir la carga entre los switches de distribución en HQ de la siguiente manera:
  - SW1 (SW-D-HQ01) debe ser configurado como el Root Bridge para las VLANs 100 (OPERATIONS) y 102 (MARKETING).
  - SW2 (SW-D-HQ02) debe ser configurado como el Root Bridge para las VLANs 101 (ACCOUTING) y 99 (IT-MGMT).

- El modo de STP debe ser Rapid-PVST

- Excluir las primeras 10 IPs de cada red

## Configuración Paso a Paso

### Data Center

#### Router (RO-DC01)

```
hostname RO-DC01

interface g0/0/1
description VLAN 100
ip address 10.1.0.1 255.255.240.0
no shutdown
exit

interface s0/1/0
description WAN to HQ
ip address 10.99.99.10 255.255.255.252
no shutdown
exit

interface s0/1/1
description WAN to Branch
ip address 10.99.99.1 255.255.255.252
no shutdown
exit

//ruta para poder llegar a las redes de HQ
ip route 10.24.0.0 255.255.252.0 10.99.99.9 

```

#### Switch (SW-DC01)

```
hostname SW-DC01

vlan 100
name data-center
exit

int vlan 100
description MGMT
//ultima ip de la red
ip address 10.1.15.254 255.255.240.0
no shutdown
exit

ip default-gateway 10.1.0.1

//sacamos todos los puertos de la vlan por defecto (1)
int range f0/1-24,g0/1-2
switchport mode access
switchport access vlan 100
exit


///configuración de dhcp server
//excluir las primeras 10 ips de cada red

ip dhcp excluded-address 10.24.0.1 10.24.0.10
ip dhcp pool hq-vlan100-operations
network 10.24.0.0 255.255.254.0
default-router 10.24.1.254
dns-server 10.1.0.4
domain-name ccna2.done

ip dhcp excluded-address 10.24.2.1 10.24.2.10
ip dhcp pool hq-vlan101-accounting
network 10.24.2.0 255.255.255.0
default-router 10.24.2.254
dns-server 10.1.0.4
domain-name ccna2.done

ip dhcp excluded-address 10.24.3.1 10.24.3.10
ip dhcp pool hq-vlan102-marketing
network 10.24.3.0 255.255.255.128
default-router 10.24.3.126
dns-server 10.1.0.4
domain-name ccna2.done

ip dhcp excluded-address 10.24.3.129 10.24.3.139
ip dhcp pool hq-vlan99-it-mgmt
network 10.24.3.128 255.255.255.192
default-router 10.24.3.190
dns-server 10.1.0.4
domain-name ccna2.done

!Redes Branch-office
ip dhcp excluded-address 10.16.10.1 10.16.10.10
ip dhcp pool broff-vlan10-general
network 10.16.10.0 255.255.255.0
default-router 10.16.10.1
dns-server 10.1.0.4
domain-name ccna2.done

ip dhcp excluded-address 10.16.11.1 10.16.11.10
ip dhcp pool broff-vlan11-conferencia
network 10.16.11.0 255.255.255.0
default-router 10.16.11.1
dns-server 10.1.0.4
domain-name ccna2.done

ip dhcp excluded-address 10.16.12.1 10.16.12.10
ip dhcp pool broff-vlan12-invitados
network 10.16.12.0 255.255.255.0
default-router 10.16.12.1
dns-server 10.1.0.4
domain-name ccna2.done
exit





```

### Red de Transito

#### Router (RO-WAN1)

```
hostname RO-WAN1

interface s0/2/0
description WAN to Data Center
ip address 10.99.99.9 255.255.255.252
no shutdown
exit

interface S0/1/1
description WAN to Branch
ip address 10.99.99.6 255.255.255.252
no shutdown
exit

interface S0/1/0
description Transit to HQ
ip address 10.99.99.14 255.255.255.252
no shutdown
exit

//ruta para llegar a las redes del HQ
ip route 10.24.0.0 255.255.252.0 10.99.99.13

//ruta para llegar al data-center
ip route 10.1.0.0 255.255.240.0 10.99.99.10

```
#### Router (RO-WAN2)

```
hostname RO-WAN2

interface s0/1/1
description WAN to Data Center
ip address 10.99.99.2 255.255.255.252
no shutdown
exit

interface S0/1/0
description WAN to HQ
ip address 10.99.99.5 255.255.255.252
no shutdown
exit

interface g0/0/0
description Transit to Branch Office
ip address 10.99.99.25 255.255.255.252
no shutdown
exit

!ruta estatica hacia el DC
ip route 10.1.0.0 255.255.240.0 10.99.99.1

!rutas estaticas hacia Branch Office
ip route 10.16.10.0 255.255.255.0 10.99.99.26
ip route 10.16.11.0 255.255.255.0 10.99.99.26
ip route 10.16.12.0 255.255.255.0 10.99.99.26
ip route 10.16.99.0 255.255.255.0 10.99.99.26


```

### Headquarters

#### SWITCH (SW-A-HQ01)
	
```
hostname SW-A-HQ01

vlan 100
name OPERATIONS
vlan 101 
name ACCOUTING
vlan 102
name MARKETING
vlan 99
name IT-MGMT
exit

//puertos de acceso
interface range f0/1-9
description Operations
switchport mode access
switchport access vlan 100
exit

interface range f0/10-15
description Accouting
switchport mode access
switchport access vlan 101
exit

interface range f0/16-22
description Marketing
switchport mode access
switchport access vlan 102
exit

interface range f0/23-24
switchport mode access
switchport acces vlan 98
shutdown
exit


interface range g0/1-2
description trunk to dist
switchport mode trunk
switchport trunk allowed vlan 99-102
switchport trunk native vlan 98
exit

spanning-tree mode rapid-pvst

```



#### SWITCH (SW-A-HQ02)

```
hostname SW-A-HQ02

vlan 100
name OPERATIONS
vlan 101 
name ACCOUTING
vlan 102
name MARKETING
vlan 99
name IT-MGMT
exit

interface range f0/1-9
description Operations
switchport mode access
switchport access vlan 100
exit

interface range f0/10-15
description Accouting
switchport mode access
switchport access vlan 101
exit

interface range f0/16-22
description Marketing
switchport mode access
switchport access vlan 102
exit

interface range f0/23-24
switchport mode access
switchport acces vlan 98
shutdown
exit


interface range g0/1-2
description trunk to dist
switchport mode trunk
switchport trunk allowed vlan 99-102
switchport trunk native vlan 98
exit

spanning-tree mode rapid-pvst

```

#### SWITCH (SW-D-HQ01)

```
hostname SW-D-HQ01

vlan 100
name OPERATIONS
vlan 101 
name ACCOUTING
vlan 102
name MARKETING
vlan 99
name IT-MGMT
exit

interface range g0/1-2
description trunk downlink
switchport mode trunk
switchport trunk allowed vlan 99-102
switchport trunk native vlan 98
exit

interface f0/24
description trunk to RO-HQ1
switchport mode trunk
switchport trunk allowed vlan 99-102
switchport trunk native vlan 98
exit


interface range f0/1-4
shutdown
description Etherchannel to SW-D-HQ02
channel-group 1 mode auto
exit
 
interface po1
description PortChannel to SW-D-HQ02
switchport mode trunk
switchport trunk allowed vlan 99-102
switchport trunk native vlan 98
exit

interface range f0/1-4
no shutdown
exit

spanning-tree mode rapid-pvst
//root bridge para las vlans 100 y 102
spanning-tree vlan 100,102 priority 4096

ip route 0.0.0.0 0.0.0.0 10.99.99.21




```

#### SWITCH (SW-D-HQ02)

```
hostname SW-D-HQ02

vlan 100
name OPERATIONS
vlan 101 
name ACCOUTING
vlan 102
name MARKETING
vlan 99
name IT-MGMT
exit

interface range g0/1-2
description trunk downlink
switchport mode trunk
switchport trunk allowed vlan 99-102
switchport trunk native vlan 98
exit

interface f0/24
description trunk to RO-HQ1
switchport mode trunk
switchport trunk allowed vlan 99-102
switchport trunk native vlan 98
exit


interface range f0/1-4
shutdown
description Etherchannel to SW-D-HQ01
channel-group 1 mode desirable
exit

interface po1
description PortChannel to SW-D-HQ01
switchport mode trunk
switchport trunk allowed vlan 99-102
switchport trunk native vlan 98
exit

interface range f0/1-4
no shutdown
exit

spanning-tree mode rapid-pvst
//root bridge para las vlan 101 y 99
spanning-tree vlan 101,99 priority 4096

ip route 0.0.0.0 0.0.0.0 10.99.99.17


```

#### Router (RO-HQ1)

```
hostname RO-HQ1

interface G0/0/1
description to RO-TRAN1
ip address 10.99.99.22 255.255.255.252
no shutdown
exit

interface G0/0/0.100
encapsulation dot1q 100
description vlan-operations
//una ip menos que la ip virtual
ip address 10.24.1.253 255.255.254.0 
standby 100 ip 10.24.1.254
standby 100 priority 200
standby 100 preempt
ip helper-address 10.1.15.254
exit

interface G0/0/0.101
encapsulation dot1q 101
description vlan-accounting
//una ip menos que la ip virtual
ip address 10.24.2.253 255.255.255.0
standby 101 ip 10.24.2.254
ip helper-address 10.1.15.254
exit

interface G0/0/0.102
encapsulation dot1q 102
description vlan-marketing
//una ip menos que la ip virtual
ip address 10.24.3.125 255.255.255.128
standby 102 ip 10.24.3.126
standby 102 priority 200
standby 102 preempt4
ip helper-address 10.1.15.254
exit

interface G0/0/0.99
encapsulation dot1q 99
description vlan-it-mgmt
//una ip menos que la ip virtuals
ip address 10.24.3.189 255.255.255.192
standby 99 ip 10.24.3.190
ip helper-address 10.1.15.254
exit

interface G0/0/0
description sub-interfaces
no shutdown
exit

ip route 0.0.0.0 0.0.0.0 10.99.99.21 
```

#### Router (RO-HQ2)
```
hostname RO-HQ2

interface G0/0/1
description to RO-TRAN1
ip address 10.99.99.18 255.255.255.252
no shutdown
exit

interface G0/0/0.100
encapsulation dot1q 100
description vlan-operations
ip address 10.24.1.252 255.255.254.0 
standby 100 ip 10.24.1.254
ip helper-address 10.1.15.254
exit

interface G0/0/0.101
encapsulation dot1q 101
description vlan-accounting
ip address 10.24.2.252 255.255.255.0
standby 101 ip 10.24.2.254
standby 101 priority 200
standby 101 preempt
ip helper-address 10.1.15.254
exit

interface G0/0/0.102
encapsulation dot1q 102
description vlan-marketing
ip address 10.24.3.124 255.255.255.128
standby 102 ip 10.24.3.126
ip helper-address 10.1.15.254
exit

interface G0/0/0.99
encapsulation dot1q 99
description vlan-it-mgmt
ip address 10.24.3.188 255.255.255.192
standby 99 ip 10.24.3.190
standby 99 priority 200
standby 99 preempt
ip helper-address 10.1.15.254
exit

interface G0/0/0
description sub-interfaces
no shutdown
exit

ip route 0.0.0.0 0.0.0.0 10.99.99.17 
```

#### Router (RO-TRAN1)

```
hostname RO-TRAN1

interface g0/0/0
description to RO-HQ1
ip address 10.99.99.21 255.255.255.252
no shutdown
exit

interface g0/0/1
description to RO-HQ2
ip address 10.99.99.17 255.255.255.252
no shutdown
exit

interface s0/1/0
description transit to RO-WAN1
ip address 10.99.99.13 255.255.255.252
no shutdown
exit

ip route 0.0.0.0 0.0.0.0 10.99.99.14
ip route 10.24.0.0 255.255.252.0 10.99.99.22
ip route 10.24.0.0 255.255.252.0 10.99.99.18
```

### Branch Office
#### Switch L3 (SW-D-BO01)

```cisco

hostname SW-D-BR01

vlan 10
name GENERAL
vlan 11
name CONFERENCIA
vlan 12
name INVITADOS
vlan 99
name MGMT-AP
vlan 100
name NATIVE
exit

interface range F0/21-22
shutdown
channel-group 1 mode desirable
exit
interface Port-channel1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10-12,99
switchport trunk native vlan 99
exit
interface range F0/21-22
no shutdown
exit

interface range F0/23-24
shutdown
channel-group 2 mode desirable
exit
interface Port-channel2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10-12,99
switchport trunk native vlan 99
exit
interface range F0/23-24
no shutdown
exit

ip routing

interface Vlan10
ip address 10.16.10.1 255.255.255.0
ip helper-address 10.1.15.254
exit

interface Vlan11
ip address 10.16.11.1 255.255.255.0
ip helper-address 10.1.15.254
exit

interface Vlan12
ip address 10.16.12.1 255.255.255.0
ip helper-address 10.1.15.254
exit

interface Vlan99
ip address 10.16.99.1 255.255.255.0
exit

interface GigabitEthernet0/1
no switchport
ip address 10.99.99.26 255.255.255.252
exit

ip route 0.0.0.0 0.0.0.0 10.99.99.25

```


#### Switch (SW-A-BO01)

```cisco
hostname SW-A-BO01

vlan 10
name GENERAL
vlan 11
name CONFERENCIA
vlan 12
name INVITADOS
vlan 99
name MGMT-AP
vlan 100
name NATIVE
exit

interface range f0/1-18
description GENERAL
switchport mode access
switchport access vlan 10
exit

interface range f0/19-22
description MGMT-AP
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10-12,99
exit


interface range f0/23-24
description PagP uplink
shutdown
channel-group 1 mode auto
exit
interface po1
description Portchannel trunk
switchport mode trunk
switchport trunk allowed vlan 10-12,99
switchport trunk native vlan 99
exit
interface range f0/23-24
no shutdown
exit
```

#### Switch (SW-A-BO02)

```cisco
hostname SW-A-BO02

vlan 10
name GENERAL
vlan 11
name CONFERENCIA
vlan 12
name INVITADOS
vlan 99
name MGMT-AP
vlan 100
name NATIVE
exit

interface range f0/1-18
description GENERAL
switchport mode access
switchport access vlan 10
exit

interface range f0/19-22
description MGMT-AP
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10-12,99
exit


interface GigabitEthernet0/1
description WLC
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10-12,00
exit

interface range f0/23-24
description PagP uplink
shutdown
channel-group 1 mode auto
exit
interface po1
description Portchannel trunk
switchport mode trunk
switchport trunk allowed vlan 10-12,99
switchport trunk native vlan 99
exit
interface range f0/23-24
no shutdown
exit
```




