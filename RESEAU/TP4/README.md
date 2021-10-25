# TP4 : Vers un réseau d'entreprise

On va utiliser GNS3 dans ce TP pour se rapprocher d'un cas réel. On va focus sur l'aspect routing/switching, avec du matériel Cisco. On va aussi mettre en place des VLANs.

![best memes from cisco doc](./pics/the-best-memes-come-from-cisco-documentation.jpg)

# Sommaire

- [TP4 : Vers un réseau d'entreprise](#tp4--vers-un-réseau-dentreprise)
- [Sommaire](#sommaire)
- [I. Dumb switch](#i-dumb-switch)
  - [1. Topologie 1](#1-topologie-1)
  - [2. Adressage topologie 1](#2-adressage-topologie-1)
  - [3. Setup topologie 1](#3-setup-topologie-1)
- [II. VLAN](#ii-vlan)
  - [1. Topologie 2](#1-topologie-2)
  - [2. Adressage topologie 2](#2-adressage-topologie-2)
    - [3. Setup topologie 2](#3-setup-topologie-2)
- [III. Routing](#iii-routing)
  - [1. Topologie 3](#1-topologie-3)
  - [2. Adressage topologie 3](#2-adressage-topologie-3)
  - [3. Setup topologie 3](#3-setup-topologie-3)
- [IV. NAT](#iv-nat)
  - [1. Topologie 4](#1-topologie-4)
  - [2. Adressage topologie 4](#2-adressage-topologie-4)
  - [3. Setup topologie 4](#3-setup-topologie-4)
- [V. Add a building](#v-add-a-building)
  - [1. Topologie 5](#1-topologie-5)
  - [2. Adressage topologie 4](#2-adressage-topologie-5)
  - [3. Setup topologie 5](#3-setup-topologie-5)

# I. Dumb switch

## 1. Topologie 1

[...]

## 2. Adressage topologie 1

| Node  | IP            |
|-------|---------------|
| `pc1` | `10.1.1.1/24` |
| `pc2` | `10.1.1.2/24` |

## 3. Setup topologie 1

🌞 **Commençons simple**

- définissez les IPs statiques sur les deux VPCS
    ```
    PS C:\Users\Mathis> Telnet 192.168.65.3 5002

    PC1> ip 10.1.1.1/24
    Checking for duplicate address...
    PC1 : 10.1.1.1 255.255.255.0

    PC1> show ip

    NAME        : PC1[1]
    IP/MASK     : 10.1.1.1/24
    GATEWAY     : 0.0.0.0
    DNS         :
    MAC         : 00:50:79:66:68:00
    LPORT       : 20004
    RHOST:PORT  : 127.0.0.1:20005
    MTU         : 1500
    ```

    ```
    PS C:\Users\Mathis> Telnet 192.168.65.3 5004

    PC2> ip 10.1.1.2/24
    Checking for duplicate address...
    PC2 : 10.1.1.2 255.255.255.0
    
    PC2> show ip

    NAME        : PC2[1]
    IP/MASK     : 10.1.1.2/24
    GATEWAY     : 0.0.0.0
    DNS         :
    MAC         : 00:50:79:66:68:01
    LPORT       : 20006
    RHOST:PORT  : 127.0.0.1:20007
    MTU         : 1500
    ```

- `ping` un VPCS depuis l'autre
    ```
    PC1> ping 10.1.1.2 -c 4

    84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=5.340 ms
    84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=2.815 ms
    84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=4.055 ms
    84 bytes from 10.1.1.2 icmp_seq=4 ttl=64 time=5.839 ms
    ```

    ```
    PC2> ping 10.1.1.1 -c 4

    84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=7.056 ms
    84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=8.275 ms
    84 bytes from 10.1.1.1 icmp_seq=3 ttl=64 time=9.230 ms
    84 bytes from 10.1.1.1 icmp_seq=4 ttl=64 time=10.481 ms
    ```

> Jusque là, ça devrait aller. Noter qu'on a fait aucune conf sur le switch. Tant qu'on ne fait rien, c'est une bête multiprise.

# II. VLAN

## 1. Topologie 2

![Topologie 2](./pics/topo2.png)

## 2. Adressage topologie 2

| Node  | IP            | VLAN |
|-------|---------------|------|
| `pc1` | `10.1.1.1/24` | 10   |
| `pc2` | `10.1.1.2/24` | 10   |
| `pc3` | `10.1.1.3/24` | 20   |

### 3. Setup topologie 2

🌞 **Adressage**

- définissez les IPs statiques sur tous les VPCS
    ```
    PS C:\Users\Mathis> telnet 192.168.65.3 5006

    PC3> ip 10.1.1.3/24
    Checking for duplicate address...
    PC3 : 10.1.1.3 255.255.255.0
    PC3> show ip

    NAME        : PC3[1]
    IP/MASK     : 10.1.1.3/24
    GATEWAY     : 0.0.0.0
    DNS         :
    MAC         : 00:50:79:66:68:02
    LPORT       : 20042
    RHOST:PORT  : 127.0.0.1:20043
    MTU         : 1500
    ```
- vérifiez avec des `ping` que tout le monde se ping

    Pings depuis PC1:
    ```
    PC1> ping 10.1.1.2 -c 4

    84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=5.535 ms
    84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=10.400 ms
    84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=6.274 ms
    84 bytes from 10.1.1.2 icmp_seq=4 ttl=64 time=5.462 ms

    PC1> ping 10.1.1.3 -c 4

    84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=5.393 ms
    84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=5.222 ms
    84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=6.012 ms
    84 bytes from 10.1.1.3 icmp_seq=4 ttl=64 time=4.185 ms
    ```

    Pings depuis PC2:
    ```
    PC2> ping 10.1.1.1 -c 4

    84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=6.743 ms
    84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=8.260 ms
    84 bytes from 10.1.1.1 icmp_seq=3 ttl=64 time=5.406 ms
    84 bytes from 10.1.1.1 icmp_seq=4 ttl=64 time=13.815 ms

    PC2> ping 10.1.1.3 -c 4

    84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=7.160 ms
    84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=4.556 ms
    84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=5.905 ms
    84 bytes from 10.1.1.3 icmp_seq=4 ttl=64 time=3.174 ms
    ```

    Pings depuis  PC3:
    ```
    PC3> ping 10.1.1.1 -c 4

    84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=4.686 ms
    84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=6.754 ms
    84 bytes from 10.1.1.1 icmp_seq=3 ttl=64 time=9.384 ms
    84 bytes from 10.1.1.1 icmp_seq=4 ttl=64 time=15.242 ms

    PC3> ping 10.1.1.2 -c 4

    84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=10.347 ms
    84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=7.399 ms
    84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=8.865 ms
    84 bytes from 10.1.1.2 icmp_seq=4 ttl=64 time=9.725 ms
    ```

🌞 **Configuration des VLANs**

- référez-vous [à la section VLAN du mémo Cisco](../../cours/memo/memo_cisco.md#8-vlan)
- déclaration des VLANs sur le switch `sw1`
    ```
    Sw1>en
    Sw1#conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    Sw1(config)#vlan 10
    Sw1(config-vlan)#name admins
    Sw1(config)#vlan 20
    Sw1(config-vlan)#name guests

    Sw1#show vlan

    VLAN Name                             Status    Ports
    ---- -------------------------------- --------- -------------------------------
    1    default                          active    Gi0/0, Gi0/1, Gi0/2, Gi0/3
                                                    Gi1/0, Gi1/1, Gi1/2, Gi1/3
                                                    Gi2/0, Gi2/1, Gi2/2, Gi2/3
                                                    Gi3/0, Gi3/1, Gi3/2, Gi3/3
    10   admins                           active
    20   guests                           active
    1002 fddi-default                     act/unsup
    1003 token-ring-default               act/unsup
    1004 fddinet-default                  act/unsup
    1005 trnet-default                    act/unsup

    VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
    ---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
    1    enet  100001     1500  -      -      -        -    -        0      0
    10   enet  100010     1500  -      -      -        -    -        0      0
    20   enet  100020     1500  -      -      -        -    -        0      0
    1002 fddi  101002     1500  -      -      -        -    -        0      0
    1003 tr    101003     1500  -      -      -        -    -        0      0
    1004 fdnet 101004     1500  -      -      -        ieee -        0      0
    1005 trnet 101005     1500  -      -      -        ibm  -        0      0
    ```

- ajout des ports du switches dans le bon VLAN
  - ici, tous les ports sont en mode *access* : ils pointent vers des clients du réseau
    Pour le port relié au PC1
    ```
    Sw1(config)#interface Gi0/0
    Sw1(config-if)#switchport mode access
    Sw1(config-if)#switchport access vlan 10
    ```
    Pour le port relié au PC2
    ```
    Sw1(config)#interface Gi0/1
    Sw1(config-if)#switchport mode access
    Sw1(config-if)#switchport access vlan 10
    ```
    Pour le port relié au PC3
    ```
    Sw1(config)#interface Gi0/2
    Sw1(config-if)#switchport mode access
    Sw1(config-if)#switchport access vlan 20
    ```
    On vérifie que les confs soient bien appliquées:
    ```
    Sw1#show vlan br

    VLAN Name                             Status    Ports
    ---- -------------------------------- --------- -------------------------------
    1    default                          active    Gi0/3, Gi1/0, Gi1/1, Gi1/2
                                                    Gi1/3, Gi2/0, Gi2/1, Gi2/2
                                                    Gi2/3, Gi3/0, Gi3/1, Gi3/2
                                                    Gi3/3
    10   admins                           active    Gi0/0, Gi0/1
    20   guests                           active    Gi0/2
    ```

🌞 **Vérif**

- `pc1` et `pc2` doivent toujours pouvoir se ping
    Ping depuis PC1 vers PC2:
    ```
    PC1> ping 10.1.1.2 -c 4

    84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=2.575 ms
    84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=3.230 ms
    84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=4.723 ms
    84 bytes from 10.1.1.2 icmp_seq=4 ttl=64 time=5.805 ms
    ```
    Ping depuis PC2 vers PC1:
    ```
    PC2> ping 10.1.1.1 -c 4

    84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=10.594 ms
    84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=8.613 ms
    84 bytes from 10.1.1.1 icmp_seq=3 ttl=64 time=8.917 ms
    84 bytes from 10.1.1.1 icmp_seq=4 ttl=64 time=12.909 ms
    ```
- `pc3` ne ping plus personne
    PC3 ne peut plus ping les autres PCs car ils ne sont pas sur  le même VLAN.
    ```
    PC3> ping 10.1.1.1

    host (10.1.1.1) not reachable
    ```
    ```
    PC3> ping 10.1.1.2

    host (10.1.1.2) not reachable
    ```

# III. Routing

## 1. Topologie 3

## 2. Adressage topologie 3

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse       | VLAN associé |
|-----------|---------------|--------------|
| `clients` | `10.1.1.0/24` | 11           |
| `admins`  | `10.2.2.0/24` | 12           |
| `servers` | `10.3.3.0/24` | 13           |

L'adresse des machines au sein de ces réseaux :

| Node               | `clients`       | `admins`        | `servers`       |
|--------------------|-----------------|-----------------|-----------------|
| `pc1.clients.tp4`  | `10.1.1.1/24`   | x               | x               |
| `pc2.clients.tp4`  | `10.1.1.2/24`   | x               | x               |
| `adm1.admins.tp4`  | x               | `10.2.2.1/24`   | x               |
| `web1.servers.tp4` | x               | x               | `10.3.3.1/24`   |
| `r1`               | `10.1.1.254/24` | `10.2.2.254/24` | `10.3.3.254/24` |

## 3. Setup topologie 3

🖥️ VM `web1.servers.tp4`, déroulez la [Checklist VM Linux](#checklist-vm-linux) dessus

🌞 **Adressage**

- définissez les IPs statiques sur toutes les machines **sauf le *routeur***
    ```
    PC1> ip 10.1.1.1/24
    Checking for duplicate address...
    PC1 : 10.1.1.1 255.255.255.0
    ```
    ```
    PC2> ip 10.1.1.2/24
    Checking for duplicate address...
    PC2 : 10.1.1.2 255.255.255.0
    ```
    ```
    adm1> ip 10.2.2.1/24
    Checking for duplicate address...
    adm1 : 10.2.2.1 255.255.255.0
    ```
    ```
    [mathis@web1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 | grep -e "IPADDR" -e "NETMASK"
    IPADDR=10.3.3.1
    NETMASK=255.255.255.0
    ```

🌞 **Configuration des VLANs**

- déclaration des VLANs sur le switch `sw1`
    ```
    Sw1(config)#vlan 11
    Sw1(config-vlan)#name clients
    Sw1(config-vlan)#exit
    Sw1(config)#vlan 12
    Sw1(config-vlan)#name admins
    Sw1(config-vlan)#exit
    Sw1(config)#vlan 13
    Sw1(config-vlan)#name servers
    Sw1(config-vlan)#exit
    ```
- ajout des ports du switches dans le bon VLAN (voir [le tableau d'adressage de la topo 2 juste au dessus](#2-adressage-topologie-2))
    Pour le réseau clients:
    ```
    Sw1(config)#interface Gi0/0
    Sw1(config-if)#switchport mode access
    Sw1(config-if)#switchport access vlan 11
    % Access VLAN does not exist. Creating vlan 11
    Sw1(config)#interface Gi0/1
    Sw1(config-if)#switchport mode access
    Sw1(config-if)#switchport access vlan 11
    ```
    Pour le réseau admins:
    ```
    Sw1(config)#interface Gi0/2
    Sw1(config-if)#switchport mode access
    Sw1(config-if)#switchport access vlan 12
    % Access VLAN does not exist. Creating vlan 12
    ```
    Pour le réseau servers:
    ```
    Sw1(config)#interface Gi0/3
    Sw1(config-if)#switchport mode access
    Sw1(config-if)#switchport access vlan 13
    % Access VLAN does not exist. Creating vlan 13
    ```

- il faudra ajouter le port qui pointe vers le *routeur* comme un *trunk* : c'est un port entre deux équipements réseau (un *switch* et un *routeur*)
    ```
    Sw1(config)#interface Gi1/0
    Sw1(config-if)#switchport trunk encapsulation dot1q
    Sw1(config-if)# switchport mode trunk
    Sw1(config-if)#switchport trunk allowed vlan add 11,12,13
    ```
    ```
    Sw1#show interface trunk

    Port        Mode             Encapsulation  Status        Native vlan
    Gi1/0       on               802.1q         trunking      1

    Port        Vlans allowed on trunk
    Gi1/0       1-4094

    Port        Vlans allowed and active in management domain
    Gi1/0       1,11-13
    [...]
    ```
---

➜ **Pour le *routeur***

🌞 **Config du *routeur***

- attribuez ses IPs au *routeur*
  - 3 sous-interfaces, chacune avec son IP et un VLAN associé
    ```
    R1(config)#interface fastEth 0/0.11
    R1(config-subif)#encapsulation dot1Q 11
    R1(config-subif)#ip addr 10.1.1.254 255.255.255.0
    R1(config-subif)#exit
    R1(config)#interface fastEth 0/0.12
    R1(config-subif)#encapsulation dot1Q 12
    R1(config-subif)#ip addr 10.2.2.254 255.255.255.0
    R1(config-subif)#exit
    R1(config)#interface fastEth 0/0.13
    R1(config-subif)#encapsulation dot1Q 13
    R1(config-subif)#ip addr 10.3.3.254 255.255.255.0
    R1(config)#interface FastEthernet0/0
    R1(config-if)#no shut
    ```

🌞 **Vérif**

- tout le monde doit pouvoir ping le routeur sur l'IP qui est dans son réseau
  ```
  PC1> ping 10.1.1.254

  10.1.1.254 icmp_seq=1 timeout
  84 bytes from 10.1.1.254 icmp_seq=2 ttl=255 time=60.498 ms
  84 bytes from 10.1.1.254 icmp_seq=3 ttl=255 time=22.999 ms
  84 bytes from 10.1.1.254 icmp_seq=4 ttl=255 time=44.944 ms
  84 bytes from 10.1.1.254 icmp_seq=5 ttl=255 time=24.182 ms
  ```
  ```
  PC2> ping 10.1.1.254

  84 bytes from 10.1.1.254 icmp_seq=1 ttl=255 time=65.408 ms
  84 bytes from 10.1.1.254 icmp_seq=2 ttl=255 time=25.724 ms
  84 bytes from 10.1.1.254 icmp_seq=3 ttl=255 time=20.822 ms
  84 bytes from 10.1.1.254 icmp_seq=4 ttl=255 time=23.442 ms
  ```
  ```
  adm1> ping 10.2.2.254/24

  84 bytes from 10.2.2.254 icmp_seq=1 ttl=255 time=54.587 ms
  84 bytes from 10.2.2.254 icmp_seq=2 ttl=255 time=35.940 ms
  84 bytes from 10.2.2.254 icmp_seq=3 ttl=255 time=14.787 ms
  ```
  ```
  [mathis@web1 ~]$ ping 10.3.3.254 -c 3
  PING 10.3.3.254 (10.3.3.254) 56(84) bytes of data.
  64 bytes from 10.3.3.254: icmp_seq=1 ttl=255 time=18.7 ms
  64 bytes from 10.3.3.254: icmp_seq=2 ttl=255 time=35.4 ms
  64 bytes from 10.3.3.254: icmp_seq=3 ttl=255 time=22.3 ms

  --- 10.3.3.254 ping statistics ---
  3 packets transmitted, 3 received, 0% packet loss, time 2003ms
  rtt min/avg/max/mdev = 18.698/25.464/35.419/7.190 ms
  ```
- en ajoutant une route vers les réseaux, ils peuvent se ping entre eux
  - ajoutez une route par défaut sur les VPCS
  - ajoutez une route par défaut sur la machine virtuelle
    ```
    [mathis@web1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 | grep -e "IPADDR" -e "NETMASK" -e
    "GATEWAY"
    IPADDR=10.3.3.1
    NETMASK=255.255.255.0
    GATEWAY=10.3.3.254
    ```
  - testez des `ping` entre les réseaux 
    Depuis PC1:
    ```
    PC1> ping 10.3.3.1

    84 bytes from 10.3.3.1 icmp_seq=1 ttl=63 time=81.578 ms
    84 bytes from 10.3.3.1 icmp_seq=2 ttl=63 time=80.811 ms
    84 bytes from 10.3.3.1 icmp_seq=3 ttl=63 time=84.567 ms
    ```
    Depuis PC2:
    ```
    PC2> ping 10.3.3.1

    84 bytes from 10.3.3.1 icmp_seq=1 ttl=63 time=69.056 ms
    84 bytes from 10.3.3.1 icmp_seq=2 ttl=63 time=77.162 ms
    84 bytes from 10.3.3.1 icmp_seq=3 ttl=63 time=73.090 ms
    ```
    Depuis web1:
    ```
    [mathis@web1 ~]$ cat /etc/hosts
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    10.1.1.1    PC1
    10.1.1.2    PC2
    ```
    ```
    [mathis@web1 ~]$ ping PC1 -c 3
    PING PC1 (10.1.1.1) 56(84) bytes of data.
    64 bytes from PC1 (10.1.1.1): icmp_seq=1 ttl=63 time=78.0 ms
    64 bytes from PC1 (10.1.1.1): icmp_seq=2 ttl=63 time=65.5 ms
    64 bytes from PC1 (10.1.1.1): icmp_seq=3 ttl=63 time=48.1 ms

    --- PC1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 48.053/63.873/78.034/12.299 ms
    ```
    ```
    [mathis@web1 ~]$ ping PC2 -c 3
    PING PC2 (10.1.1.2) 56(84) bytes of data.
    64 bytes from PC2 (10.1.1.2): icmp_seq=1 ttl=63 time=36.1 ms
    64 bytes from PC2 (10.1.1.2): icmp_seq=2 ttl=63 time=59.10 ms
    64 bytes from PC2 (10.1.1.2): icmp_seq=3 ttl=63 time=57.1 ms

    --- PC2 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2001ms
    rtt min/avg/max/mdev = 36.077/51.072/59.993/10.670 ms
    ```

# IV. NAT

## 1. Topologie 4

## 2. Adressage topologie 4

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse       | VLAN associé |
|-----------|---------------|--------------|
| `clients` | `10.1.1.0/24` | 11           |
| `servers` | `10.2.2.0/24` | 12           |
| `routers` | `10.3.3.0/24` | 13           |

L'adresse des machines au sein de ces réseaux :

| Node               | `clients`       | `admins`        | `servers`       |
|--------------------|-----------------|-----------------|-----------------|
| `pc1.clients.tp4`  | `10.1.1.1/24`   | x               | x               |
| `pc2.clients.tp4`  | `10.1.1.2/24`   | x               | x               |
| `adm1.admins.tp4`  | x               | `10.2.2.1/24`   | x               |
| `web1.servers.tp4` | x               | x               | `10.3.3.1/24`   |
| `r1`               | `10.1.1.254/24` | `10.2.2.254/24` | `10.3.3.254/24` |

## 3. Setup topologie 4

🌞 **Ajoutez le noeud Cloud à la topo**

- branchez à `eth1` côté Cloud
- côté routeur, il faudra récupérer un IP en DHCP
  ```
  R1(config)#interface FastEthernet1/0
  R1(config-if)#ip address dhcp
  R1(config-if)#no shut
  R1(config-if)#exit
  R1(config)#exit
  R1#sh ip int br
  Interface                  IP-Address      OK? Method Status                Protocol
  FastEthernet0/0            unassigned      YES NVRAM  up                    up
  FastEthernet0/0.11         10.1.1.254      YES manual up                    up
  FastEthernet0/0.12         10.2.2.254      YES manual up                    up
  FastEthernet0/0.13         10.3.3.254      YES manual up                    up
  FastEthernet1/0            192.168.65.5    YES DHCP   up                    up
  [...]
  ```
- vous devriez pouvoir `ping 1.1.1.1`
  ```
  R1(config-if)#do ping 1.1.1.1

  Type escape sequence to abort.
  Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
  .!!!!
  Success rate is 80 percent (4/5), round-trip min/avg/max = 1/8/12 ms
  ```
  ```
  PC1> ping 1.1.1.1

  1.1.1.1 icmp_seq=1 timeout
  84 bytes from 1.1.1.1 icmp_seq=2 ttl=62 time=18.006 ms
  84 bytes from 1.1.1.1 icmp_seq=3 ttl=62 time=25.227 ms
  84 bytes from 1.1.1.1 icmp_seq=4 ttl=62 time=27.459 ms
  84 bytes from 1.1.1.1 icmp_seq=5 ttl=62 time=24.724 ms
  ```

🌞 **Configurez le NAT**

- référez-vous [à la section NAT du mémo Cisco](../../cours/memo/memo_cisco.md#7-configuration-dun-nat-simple)
```
R1(config-if)#interface fastEthernet 1/0
R1(config-if)#ip nat outside
R1(config-if)# access-list 1 permit any
R1(config-if)# ip nat inside source list 1 interface fastEthernet 1/0 overload
```

🌞 **Test**

- ajoutez une route par défaut (déjà fait)
  - sur les VPCS
  - sur la machine Linux
- configurez l'utilisation d'un DNS
  - sur les VPCS
    ```
    PC1> show ip

    NAME        : PC1[1]
    IP/MASK     : 10.1.1.1/24
    GATEWAY     : 10.1.1.254
    DNS         : 8.8.8.8
    [...]
    ```
    ```
    PC2> show ip

    NAME        : PC2[1]
    IP/MASK     : 10.1.1.2/24
    GATEWAY     : 10.1.1.254
    DNS         : 8.8.8.8
    ```
  - sur la machine Linux
    ```
    [mathis@web1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 | grep "DNS1"
    DNS1=8.8.8.8
    ```
- vérifiez un `ping` vers un nom de domaine
  ```
  PC1> ping google.com
  google.com resolved to 216.58.214.174

  84 bytes from 216.58.214.174 icmp_seq=1 ttl=114 time=27.398 ms
  84 bytes from 216.58.214.174 icmp_seq=2 ttl=114 time=30.329 ms
  84 bytes from 216.58.214.174 icmp_seq=3 ttl=114 time=34.021 ms
  84 bytes from 216.58.214.174 icmp_seq=4 ttl=114 time=35.060 ms
  ```

# V. Add a building

## 1. Topologie 5

## 2. Adressage topologie 5

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse       | VLAN associé |
|-----------|---------------|--------------|
| `clients` | `10.1.1.0/24` | 11           |
| `servers` | `10.2.2.0/24` | 12           |
| `routers` | `10.3.3.0/24` | 13           |

L'adresse des machines au sein de ces réseaux :

| Node                | `clients`       | `admins`        | `servers`       |
|---------------------|-----------------|-----------------|-----------------|
| `pc1.clients.tp4`   | `10.1.1.1/24`   | x               | x               |
| `pc2.clients.tp4`   | `10.1.1.2/24`   | x               | x               |
| `pc3.clients.tp4`   | DHCP            | x               | x               |
| `pc4.clients.tp4`   | DHCP            | x               | x               |
| `pc5.clients.tp4`   | DHCP            | x               | x               |
| `dhcp1.clients.tp4` | `10.1.1.253/24` | x               | x               |
| `adm1.admins.tp4`   | x               | `10.2.2.1/24`   | x               |
| `web1.servers.tp4`  | x               | x               | `10.3.3.1/24`   |
| `r1`                | `10.1.1.254/24` | `10.2.2.254/24` | `10.3.3.254/24` |

## 3. Setup topologie 5

🌞  **Vous devez me rendre le `show running-config` de tous les équipements**

- de tous les équipements réseau
  - le routeur
    ```
    R1#show running-config
    Building configuration...

    Current configuration : 1318 bytes
    !
    version 12.4
    service timestamps debug datetime msec
    service timestamps log datetime msec
    no service password-encryption
    !
    hostname R1
    !
    boot-start-marker
    boot-end-marker
    !
    !
    no aaa new-model
    memory-size iomem 5
    no ip icmp rate-limit unreachable
    !
    !
    ip cef
    no ip domain lookup
    !
    !
    ```
  - les 3 switches
    sw1:
    ```
    Sw1#show running-config
    Building configuration...

    Current configuration : 3499 bytes
    !
    ! Last configuration change at 20:41:44 UTC Sun Oct 24 2021
    !
    version 15.2
    service timestamps debug datetime msec
    service timestamps log datetime msec
    no service password-encryption
    service compress-config
    !
    hostname Sw1
    !
    boot-start-marker
    boot-end-marker
    !
    !
    !
    no aaa new-model
    !
    !
    !
    !
    ```
    sw2:
    ```
    Switch#show running-config
    Building configuration...

    Current configuration : 3954 bytes
    !
    ! Last configuration change at 14:32:36 UTC Sun Oct 24 2021
    !
    version 15.2
    service timestamps debug datetime msec
    service timestamps log datetime msec
    no service password-encryption
    service compress-config
    !
    hostname Switch
    !
    boot-start-marker
    boot-end-marker
    !
    !
    !
    no aaa new-model
    !
    !
    !
    !
    ```
    sw3
    ```
    Sw3#show running-config
    Building configuration...

    Current configuration : 3499 bytes
    !
    ! Last configuration change at 20:40:55 UTC Sun Oct 24 2021
    !
    version 15.2
    service timestamps debug datetime msec
    service timestamps log datetime msec
    no service password-encryption
    service compress-config
    !
    hostname Sw3
    !
    boot-start-marker
    boot-end-marker
    !
    !
    !
    no aaa new-model
    !
    !
    !
    !
    ```

> N'oubliez pas les VLANs sur tous les switches.

🖥️ **VM `dhcp1.client1.tp4`**, déroulez la [Checklist VM Linux](#checklist-vm-linux) dessus

🌞  **Mettre en place un serveur DHCP dans le nouveau bâtiment**

- il doit distribuer des IPs aux clients dans le réseau `clients` qui sont branchés au même switch que lui
- sans aucune action manuelle, les clients doivent...
  - avoir une IP dans le réseau `clients`
  - avoir un accès au réseau `servers`
  - avoir un accès WAN
  - avoir de la résolution DNS

> Réutiliser les serveurs DHCP qu'on a monté dans les autres TPs.

🌞  **Vérification**

- un client récupère une IP en DHCP
- il peut ping le serveur Web
- il peut ping `8.8.8.8`
- il peut ping `google.com`

> Faites ça sur n'importe quel VPCS que vous venez d'ajouter : `pc3` ou `pc4` ou `pc5`.

![i know cisco](./pics/i_know.jpeg)
