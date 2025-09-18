# Laboratorio de Rutas Estaticas

https://drive.google.com/file/d/1-NcKjySMRQeY4X2aqS-fL2AEyjwPov2l/view?usp=drive_link



## Red Celeste 

### Switch:S1

```
hostname S1

enable secret cisco
line vty 0 15
password cisco
login

ip default-gateway 192.168.3.1

vlan 10 
name datos
vlan 11
name invitados
vlan 12
name printers
vlan 80
name mgmt

interface f0/1
description datos
switchport mode access
switchport access vlan 10

interface f0/2
description invitados
switchport mode access
switchport access vlan 11

interface f0/3
description printers
switchport mode access
switchport access vlan 12

interface g0/1
description trunk to R1
switchport mode trunk
switchport trunk allowed vlan 10,11,12,80

interface vlan 80
description mgmt
ip address 192.168.3.2 255.255.255.0
no shutdown
```

### Router:R1

```
hostname R1

enable secret cisco
line vty 0 15
password cisco
login

interface G0/0/0.10
encapsulation dot1q 10
ip address 192.168.0.1 255.255.255.0
ip helper-address 10.0.101.11
description vlan datos
no shutdown

interface G0/0/0.11
encapsulation dot1q 11
ip address 192.168.1.1 255.255.255.0
ip helper-address 10.0.101.11
description vlan invitados
no shutdown

interface G0/0/0.12
encapsulation dot1q 12
ip address 192.168.2.1 255.255.255.0
ip helper-address 10.0.101.11
description vlan printers
no shutdown

interface G0/0/0.80
encapsulation dot1q 80
ip address 192.168.80.1 255.255.255.0
ip helper-address 10.0.101.11
description vlan mgmt
no shutdown

interface G0/0/0
no shutdown

interface S0/1/0
description WAN to R3
ip address 10.1.1.1 255.255.255.252
no shutdown

interface G0/0/1
description WAN to SWL3
ip address 10.1.1.9 255.255.255.252
no shutdown
exit



```

#### Opcion 1x: Rutas estaticas individuales directamente conectadas

```
ip route 192.168.4.0 255.255.255.128 s0/1/0
ip route 192.168.4.128 255.255.255.128 s0/1/0
ip route 192.168.5.0 255.255.255.128 s0/1/0
ip route 172.16.0.0 255.255.255.0 s0/1/0
```

#### Opcion 2: Rutas estaticas individuales por ip de siguiente salto
```
ip route 192.168.4.0 255.255.255.128 10.1.1.2
ip route 192.168.4.128 255.255.255.128 10.1.1.2
ip route 192.168.5.0 255.255.255.128 10.1.1.2
ip route 172.16.0.0 255.255.255.0 10.1.1.2
```

#### Opcion 3a: Rutas estaticas por defecto directamente conectada
```
ip route 0.0.0.0 0.0.0.0 s0/1/0
```

#### Opcion 3b: Rutas estaticas por defecto por ip de siguiente salto
```
ip route 0.0.0.0 0.0.0.0 10.1.1.2
```

#### Opcion 4a: Rutas estaticas sumarizadas con interfaz de salida

```
ip route 192.168.4.0 255.255.254.0 s0/1/0
ip route 172.16.0.0 255.255.255.0 s0/1/0
```

### Opcion 4b: Rutas estaticas sumarizadas con ip de siguiente salto ****

```
ip route 192.168.4.0 255.255.254.0 10.1.1.2
ip route 172.16.0.0 255.255.255.0 10.1.1.2
```

### Ruta estatica sumarizada al Data Center

Segmento 10.0.0.0/16 reservado para el Data Center:

´´´
ip route 10.0.0.0 255.255.0.0 10.1.1.2
´´´

### Ejemplo de Ruta de host

´´´
ip route 10.0.100.10 255.255.255.255 10.1.1.2
ip route 10.0.101.10 255.255.255.255 10.1.1.2
ip route 10.0.101.11 255.255.255.255 10.1.1.2
´´´

### Configuracion de rutas flotantes para el enlace redudante

---- red amarilla - ruta principal --
ip route 192.168.4.0 255.255.254.0 10.1.1.10 
ip route 172.16.0.0 255.255.255.0 10.1.1.10

--- red amarilla - ruta secundaria (flotante) ---
ip route 192.168.4.0 255.255.254.0 10.1.1.2 2
ip route 172.16.0.0 255.255.255.0 10.1.1.2 2

--- red data center - ruta principal --
ip route 10.0.100.10 255.255.255.255 10.1.1.2 
ip route 10.0.101.10 255.255.255.255 10.1.1.2 
ip route 10.0.101.11 255.255.255.255 10.1.1.2 

-- red data center - ruta secundaria (flotante) --
ip route 10.0.100.10 255.255.255.255 10.1.1.10 2
ip route 10.0.101.10 255.255.255.255 10.1.1.10 2
ip route 10.0.101.11 255.255.255.255 10.1.1.10 2
´´´



## Red Amarilla

### Switch:S2

```
hostname S2

enable secret cisco
line vty 0 15
password cisco
login

ip default-gateway 172.16.0.1

vlan 10 
name datos
vlan 11
name invitados
vlan 12
name printers
vlan 80
name mgmt

interface f0/1
description datos
switchport mode access
switchport access vlan 10

interface f0/2
description invitados
switchport mode access
switchport access vlan 11

interface f0/3
description printers
switchport mode access
switchport access vlan 12

interface g0/1
description trunk to R1
switchport mode trunk
switchport trunk allowed vlan 10,11,12,80

interface vlan 80
description mgmt
ip address 172.16.0.2 255.255.255.0
no shutdown

```
### Switch: SWL3
```
hostname SWL3

enable secret cisco
line vty 0 15
password cisco
login

vlan 10 
name datos
vlan 11
name invitados
vlan 12
name printers
vlan 80
name mgmt

ip routing

interface G0/2
description trunk to S2
switchport trunk encapsulation dot1q 
switchport mode trunk
switchport trunk allowed vlan 10,11,12,80

interface vlan 10
description datos
ip address 192.168.4.1 255.255.255.128
ip helper-address 10.0.101.11
no shutdown

interface vlan 11
description invitados
ip address 192.168.4.129 255.255.255.128
ip helper-address 10.0.101.11
no shutdown

interface vlan 12
description printers
ip address 192.168.5.1 255.255.255.0
ip helper-address 10.0.101.11
no shutdown

interface vlan 80
description mgmt
ip address 172.16.0.1 255.255.255.0
ip helper-address 10.0.101.11
no shutdown


interface g0/1
description WAN to R3
no switchport
ip address 10.1.1.6 255.255.255.252
no shutdown

interface f0/24
description WAN to R1
no switchport
ip address 10.1.1.10 255.255.255.252
no shutdown

ip route 0.0.0.0 0.0.0.0 10.1.1.5




```

## Red Data Center

### Switch Data Center

´´´
enable
configure terminal
hostname SWDC

enable secret cisco
line vty 0 15
password cisco
login

vlan 100
name web
vlan 101
name servicios_red
vlan 80
name admin
exit

interface g0/1
switchport mode trunk
switchport trunk allowed vlan 100,101,80
description to_R3
exit

interface f0/1
switchport mode access
switchport access vlan 100
description web
exit

interface range f0/2-3
switchport mode access
switchport access vlan 101
description servicios_red
exit


interface vlan 80
description admin
ip address 10.0.80.2 255.255.255.0
no shutdown

ip default-gateway 10.0.80.1
´´´

### Router R3

```
hostname R3

enable secret cisco
line vty 0 15
password cisco
login

interface S0/1/1
description WAN to Celeste
ip address 10.1.1.2 255.255.255.252
no shutdown
exit

interface G0/0/0
description WAN to Amarilla
ip address 10.1.1.5 255.255.255.252
no shutdown	
exit

interface G0/0/1.100
encapsulation dot1q 100
ip address 10.0.100.1 255.255.255.0
description vlan web
no shutdown
exit

interface G0/0/1.101
encapsulation dot1q 101
ip address 10.0.101.1 255.255.255.0
description vlan servicios_red
no shutdown
exit

interface G0/0/1.80
encapsulation dot1q 80
ip address 10.0.80.1 255.255.255.0
description vlan admin
no shutdown
exit

interface g0/0/1
no shutdown
exit
```


##### Rutas a las red Celeste

```
ip route 192.168.0.0 255.255.252.0 10.1.1.1
```


##### Rutas a las red Amarilla

```
ip route 192.168.4.0 255.255.254.0 10.1.1.6
ip route 172.16.0.0 255.255.255.0 10.1.1.6
```

### Router DHCP

hostname DHCP

interface g0/0
ip address 10.0.101.11 255.255.255.0
no shutdown

ip route 0.0.0.0 0.0.0.0 10.0.101.1


#### Red Celeste - Vlan 10
ip dhcp excluded-address 192.168.0.1 192.168.0.10
ip dhcp pool C_10
network 192.168.0.0 255.255.255.0
default-router 192.168.0.1
dns-server 10.0.101.10
domain-name puravida.cr
exit

#### Red Celeste - Vlan 11
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp pool C_11
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 10.0.101.10
domain-name puravida.cr
exit

#### Red Celeste - Vlan 12
ip dhcp excluded-address 192.168.2.1 192.168.2.10
ip dhcp pool C_12
network 192.168.2.0 255.255.255.0
default-router 192.168.2.1
dns-server 10.0.101.10
domain-name puravida.cr
exit

#### Red Amarilla - Vlan 10
ip dhcp excluded-address 192.168.4.1 192.168.4.10
ip dhcp pool A_10
network 192.168.4.0 255.255.255.128
default-router 192.168.4.1
dns-server 10.0.101.10
domain-name puravida.cr
exit

#### Red Amarilla - Vlan 11
ip dhcp excluded-address 192.168.4.129 192.168.4.138
ip dhcp pool A_11
network 192.168.4.128 255.255.255.128
default-router 192.168.4.129
dns-server 10.0.101.10
domain-name puravida.cr
exit

#### Red Amarilla  - Vlan 12
ip dhcp excluded-address 192.168.5.1 192.168.5.10
ip dhcp pool A_12
network 192.168.5.0 255.255.255.0
default-router 192.168.5.1
dns-server 10.0.101.10
domain-name puravida.cr
exit













