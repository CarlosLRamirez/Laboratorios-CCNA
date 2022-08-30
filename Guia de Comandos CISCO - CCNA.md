# Comandos CISCO - Laboratorios de CCNA

## Laboratorio 1 - Comandos Básicos

#### Pasar a modo privilegiado
```text
Switch>enable
Switch#
```

#### Pasar a modo de configuración global
```text
Switch#configure terminal
Switch(config)#
```

#### Configurar hostname
```text
Switch(config)#hostname CCNA1
CCNA1(config)#
```

#### Configurar password consola
```text
CCNA1(config)#line console 0
CCNA1(config-line)#password cisco
CCNA1(config-line)#login
CCNA1(config-line)#exit
CCNA1(config)#
```

#### Configurar password de modo privilegiado
```text
CCNA1(config)#enable password class
CCNA1(config)#
```

#### Configurar password de modo privilegiado (mas seguro)
```text
CCNA1(config)#enable secret class
CCNA1(config)#
```

#### Para borrar el password de modo privilegiado (menos seguro)
```text
CCNA1(config)#no enable password
CCNA1(config)#
```

#### Encryptar contraseñas en el archivo de configuracion
```text
CCNA1(config)#service password-encryption 
CCNA1(config)#
```

#### Configurar IP en SVI (VLAN 1) para administracion
```text
CCNA1(config)#interface vlan 1
CCNA1(config-if)#ip address 192.168.0.5 255.255.255.0
CCNA1(config-if)#no shutdown
CCNA1(config-if)#exit
CCNA1(config)#
```

#### Configurar acceso por Telnet al switch
```text
CCNA1(config)#line vty 0 15
CCNA1(config-line)#password cisco
CCNA1(config-line)#login
CCNA1(config-line)#transport input telnet
CCNA1(config-line)#exit
CCNA1(config)#
```

#### Guardar configuración en la NVRAM
```text
CCNA1#copy running-config startup-config 
CCNA1#
```

### Otros comandos

#### habilitación de SSH
```text
R1(config)#hostname R1
R1(config)#ip domain-name cisco.gt
R1(config)#crypto key generate rsa
1024
R1(config)#username admin secret Abcd1234
R1(config)#line vty 0 15
R1(config-line)#transport input ssh
R1(config-line)#login local
R1(config-line)#exit
R1(config)#ip ssh version 2
```

#### cambiar el nivel de privilegios de un usuario (0-15)
```text
R1(config)#username admin privilege 15
```

#### exigir que las contraseñas tengan un minimo de 10 caracteres
```text
R1(config)#security passwords min-length 10
```

#### Bloquear un usuario por 120s si tiene 3 intentos fallidos en 60s
```text
R1(config)#login block-for 120 attempts 3 within 60
```

#### Desconectar una sesion de EXEC luego de 5min y 30seg (default 10min)
```text
R1(config-line)# exec-timeout 5 30 
```

#### Deshabilitar la busqueda de DNS
```text
Switch(config)#no ip domain-lookup
```

#### Configurar el nombre de dominio
```text
R1(config)#ip domain-name cisco.gt
```

