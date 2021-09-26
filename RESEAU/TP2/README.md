## - Mathis COCO B2B -

---

# TP2 : On va router des trucs
# 1 Echange ARP

 Générer des requêtes ARP
 
 ```
 [mathis@node1 ~]$ ping 10.2.1.12
PING 10.2.1.12 (10.2.1.12) 56(84) bytes of data.
64 bytes from 10.2.1.12: icmp_seq=1 ttl=64 time=0.657 ms
64 bytes from 10.2.1.12: icmp_seq=2 ttl=64 time=0.685 ms
64 bytes from 10.2.1.12: icmp_seq=3 ttl=64 time=0.591 ms
64 bytes from 10.2.1.12: icmp_seq=4 ttl=64 time=0.735 ms

--- 10.2.1.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3063ms
rtt min/avg/max/mdev = 0.591/0.667/0.735/0.052 ms
 ```
 
 ```
 [mathis@node2 ~]$ ping 10.2.1.11
PING 10.2.1.11 (10.2.1.11) 56(84) bytes of data.
64 bytes from 10.2.1.11: icmp_seq=1 ttl=64 time=1.64 ms
64 bytes from 10.2.1.11: icmp_seq=2 ttl=64 time=1.55 ms
64 bytes from 10.2.1.11: icmp_seq=3 ttl=64 time=1.71 ms
64 bytes from 10.2.1.11: icmp_seq=4 ttl=64 time=1.55 ms

--- 10.2.1.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 1.545/1.610/1.711/0.069 ms
 ```
 
 ip n s sur node1:
 ```
[mathis@node1 ~]$ ip n s
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:4d REACHABLE
10.2.1.12 dev enp0s8 lladdr 08:00:27:37:0c:d3 REACHABLE
```
 ip n s sur node2:
 ```
[mathis@node2 ~]$ ip n s
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:4d REACHABLE
10.2.1.11 dev enp0s8 lladdr 08:00:27:71:b4:d6 REACHABLE
```

Suite à la commmande ip n s, on peut constater que l'adresse MAC de la carte réseau de node1 est 08:00:27:71:b4:d6  et celle de node2 est 08:00:27:37:0c:d3. On vérifie ça avec un ip a:
```
[mathis@node1 ~]$ ip a
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:71:b4:d6 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.11/24 brd 10.2.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe71:b4d6/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```
```
[mathis@node2 ~]$ ip a
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:37:0c:d3 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.12/24 brd 10.2.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe37:cd3/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

# PARTIE PRECEDENTE A REVERIFIER PARTIE PRECEDENTE A REVERIFIER

# 2. Analyse de trames

Analyse de trames

clear de la table ARP sur node1:
```
[mathis@node1 ~]$ sudo ip -s -s neigh flush all
10.2.1.254 dev enp0s8 lladdr 08:00:27:0a:50:bb used 1620/1620/1593 probes 1 STALE
10.2.1.12 dev enp0s8 lladdr 08:00:27:37:0c:d3 used 265/261/244 probes 1 STALE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:4d ref 1 used 0/0/0 probes 1 REACHABLE

*** Round 1, deleting 3 entries ***
*** Flush is complete after 1 round ***
```

tcpdump sur la machine node1:

```
[mathis@node1 ~]$ sudo tcpdump -w tp2_arp.pcap -i enp0s8 -c 10 not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
11 packets received by filter
0 packets dropped by kernel
```
clear de la table ARP depuis node2:
```
[mathis@node2 ~]$ sudo ip -s -s neigh flush all
[sudo] password for mathis:
10.2.1.11 dev enp0s8 lladdr 08:00:27:71:b4:d6 used 306/306/264 probes 4 STALE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:4d ref 1 used 0/0/3 probes 4 DELAY

*** Round 1, deleting 2 entries ***
*** Flush is complete after 1 round ***
```
ping de node2 vers node1:
```
[mathis@node2 ~]$ ping node1
PING node1 (10.2.1.11) 56(84) bytes of data.
64 bytes from node1 (10.2.1.11): icmp_seq=1 ttl=64 time=2.28 ms
64 bytes from node1 (10.2.1.11): icmp_seq=2 ttl=64 time=29.10 ms
64 bytes from node1 (10.2.1.11): icmp_seq=3 ttl=64 time=3.16 ms
64 bytes from node1 (10.2.1.11): icmp_seq=4 ttl=64 time=2.74 ms

--- node1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3009ms
rtt min/avg/max/mdev = 2.276/9.541/29.989/11.809 ms
```


| ordre | type trame  | source                      | destination                |
|-------|-------------|-----------------------------|----------------------------|
| 9     | Requête ARP | `node1` `08:00:27:37:0c:d3` | Broadcast `FF:FF:FF:FF:FF` |
| 10    | Réponse ARP | `node2` `08:00:27:71:b4:d6` | `node1` `08:00:27:37:0c:d3`|

# II. Routage

Suite à la création de la nouvelle machine nommée marcel, je modifie le fichier config de enp0s8:

```
  GNU nano 2.9.8                                ifcfg-enp0s8                                          
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s8
UUID=14934e42-19f2-48e0-8daf-be9662adaca7
DEVICE=enp0s8
ONBOOT=no
IPADDR=10.2.2.12
NETMASK=255.255.255.0
GATEWAY=10.2.2.254
```

activer le routage sur le noeud router.net2.tp2:

```
[mathis@marcel ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
[sudo] password for mathis:
success
```

Ajouter les routes statiques nécessaires pour que node1.net1.tp2 et marcel.net2.tp2 puissent se ping:

ajout de marcel dans le fichier hosts de node1 avec sudo nano /etc/hosts:
``` 10.2.2.12   marcel marcel.net2.tp2 ```
ajout de node1 dans le fichier hosts de marcel:
``` 10.2.1.11   node1 node1.net1.tp2 ```



On vérifie si la route fonctione bien avec un ping :

```
[mathis@node1 etc]$ ping marcel.net2.tp2
PING marcel (10.2.2.12) 56(84) bytes of data.
64 bytes from marcel (10.2.2.12): icmp_seq=1 ttl=63 time=3.11 ms
64 bytes from marcel (10.2.2.12): icmp_seq=2 ttl=63 time=4.54 ms
64 bytes from marcel (10.2.2.12): icmp_seq=3 ttl=63 time=3.54 ms
64 bytes from marcel (10.2.2.12): icmp_seq=4 ttl=63 time=3.79 ms

--- marcel ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 3.106/3.742/4.536/0.522 ms
```

```
[mathis@marcel ~]$ ping node1.net1.tp2
PING node1 (10.2.1.11) 56(84) bytes of data.
64 bytes from node1 (10.2.1.11): icmp_seq=1 ttl=63 time=2.69 ms
64 bytes from node1 (10.2.1.11): icmp_seq=2 ttl=63 time=4.23 ms
64 bytes from node1 (10.2.1.11): icmp_seq=3 ttl=63 time=1.98 ms
64 bytes from node1 (10.2.1.11): icmp_seq=4 ttl=63 time=3.84 ms

--- node1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 1.980/3.185/4.229/0.898 ms
```

# 2 Analyse de trames

Analyse des échanges ARP:

Vider les tables ARP des 3 noeuds:

```
[mathis@node1 ~]$ sudo ip -s -s neigh flush all
[sudo] password for mathis:
10.2.1.254 dev enp0s8 lladdr 08:00:27:0a:50:bb used 107/105/89 probes 1 STALE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:4d ref 1 used 0/0/2 probes 1 DELAY

*** Round 1, deleting 2 entries ***
*** Flush is complete after 1 round ***
```
```
[mathis@marcel ~]$ sudo ip -s -s neigh flush all
10.2.2.254 dev enp0s8 lladdr 08:00:27:04:92:22 used 101/98/54 probes 1 STALE
10.2.2.11 dev enp0s8  used 120/210/118 probes 6 FAILED
10.2.2.1 dev enp0s8 lladdr 0a:00:27:00:00:52 ref 1 used 0/0/4 probes 4 DELAY

*** Round 1, deleting 3 entries ***
*** Flush is complete after 1 round ***
```
```
[mathis@router ~]$ sudo ip -s -s neigh flush all
[sudo] password for mathis:
10.2.2.12 dev enp0s9 lladdr 08:00:27:83:69:12 used 111/109/62 probes 1 STALE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:4d ref 1 used 0/0/2 probes 0 DELAY
10.2.1.11 dev enp0s8 lladdr 08:00:27:71:b4:d6 used 111/109/70 probes 1 STALE
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 used 1161/1156/1132 probes 1 STALE

*** Round 1, deleting 4 entries ***
*** Flush is complete after 1 round ***
```

Ping de node1 vers marcel:
```
[mathis@node1 ~]$ ping marcel
PING marcel (10.2.2.12) 56(84) bytes of data.
64 bytes from marcel (10.2.2.12): icmp_seq=1 ttl=63 time=4.83 ms
64 bytes from marcel (10.2.2.12): icmp_seq=2 ttl=63 time=3.82 ms
64 bytes from marcel (10.2.2.12): icmp_seq=3 ttl=63 time=4.18 ms
64 bytes from marcel (10.2.2.12): icmp_seq=4 ttl=63 time=4.26 ms
64 bytes from marcel (10.2.2.12): icmp_seq=5 ttl=63 time=4.01 ms
64 bytes from marcel (10.2.2.12): icmp_seq=6 ttl=63 time=6.44 ms
^C
--- marcel ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5012ms
rtt min/avg/max/mdev = 3.818/4.589/6.442/0.886 ms
```

Tables ARP des 3 noeuds:
```
[mathis@node1 ~]$ ip n s
10.2.1.254 dev enp0s8 lladdr 08:00:27:0a:50:bb REACHABLE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:4d REACHABLE
```
```
[mathis@marcel ~]$ ip n s
10.2.2.254 dev enp0s8 lladdr 08:00:27:04:92:22 REACHABLE
10.2.2.1 dev enp0s8 lladdr 0a:00:27:00:00:52 REACHABLE
```
```
[mathis@router ~]$ ip n s
10.2.2.12 dev enp0s9 lladdr 08:00:27:83:69:12 STALE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:4d DELAY
10.2.1.11 dev enp0s8 lladdr 08:00:27:71:b4:d6 STALE
```

Après analyse des tables ARP, on peut déduire que les paquets envoyés par node1 ont transités en passant par le routeur pour ensuite arriver vers marcel.

tcpdump depuis node1:
```
[mathis@node1 ~]$ sudo tcpdump -w tp2_routage_node1.pcap -i enp0s8 -c 10 not port 22
[sudo] password for mathis:
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
11 packets received by filter
0 packets dropped by kernel
```
```
[mathis@marcel ~]$ ping node1
PING node1 (10.2.1.11) 56(84) bytes of data.
64 bytes from node1 (10.2.1.11): icmp_seq=1 ttl=63 time=5.17 ms
64 bytes from node1 (10.2.1.11): icmp_seq=2 ttl=63 time=4.14 ms
64 bytes from node1 (10.2.1.11): icmp_seq=3 ttl=63 time=3.74 ms
64 bytes from node1 (10.2.1.11): icmp_seq=4 ttl=63 time=3.76 ms
64 bytes from node1 (10.2.1.11): icmp_seq=5 ttl=63 time=4.31 ms
64 bytes from node1 (10.2.1.11): icmp_seq=6 ttl=63 time=2.04 ms

--- node1 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5013ms
rtt min/avg/max/mdev = 2.041/3.859/5.174/0.945 ms
```
tcpdump depuis marcel:

```
[mathis@marcel ~]$ sudo tcpdump -w tp2_routage_marcel.pcap -i enp0s8 -c 100 not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
100 packets captured
102 packets received by filter
0 packets dropped by kernel
```

```
[mathis@node1 ~]$ sudo ip -s -s neigh flush all
[sudo] password for mathis:
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:1f ref 1 used 0/0/4 probes 0 DELAY

*** Round 1, deleting 1 entries ***
*** Flush is complete after 1 round ***
[mathis@node1 ~]$ ping marcel
PING marcel (10.2.2.12) 56(84) bytes of data.
64 bytes from marcel (10.2.2.12): icmp_seq=1 ttl=63 time=2.12 ms
64 bytes from marcel (10.2.2.12): icmp_seq=2 ttl=63 time=1.21 ms
64 bytes from marcel (10.2.2.12): icmp_seq=3 ttl=63 time=1.52 ms
...
...
--- marcel ping statistics ---
30 packets transmitted, 30 received, 0% packet loss, time 29078ms
rtt min/avg/max/mdev = 0.746/1.363/2.123/0.324 ms
```

| ordre | type trame  | IP source | MAC source                   | IP destination | MAC destination             |
|-------|-------------|-----------|------------------------------|----------------|-----------------------------|
| 1     | Requête ARP | x         | `node1` `08:00:27:0a:50:bb`  | x              | Broadcast `FF:FF:FF:FF:FF`  |
| 2     | Réponse ARP | x         | `router` `08:00:27:71:b4:d6` | x              | `node1` `08:00:27:0a:50:bb` |
| ...   | ...         | ...       | ...                          |                |                             |
| ?     | Ping        | ?         | `node1` `08:00:27:0a:50:bb`  | ?              | `router` `08:00:27:71:b4:d6`|
| ?     | Pong        | ?         | `router` `08:00:27:71:b4:d6` | ?              | ?                           |

# 3. Accès Internet

Donnez un accès internet à vos machines:

Ajout de la route vers le routeur avec la commande sudo nano ifcfg-enp0s8 dans le dossier /etc/sysconfig/network-scripts/ :
Ligne " GATEWAY=10.2.1.254" sur node1
et "GATEWAY=10.2.2.254" sur marcel.

Ping vers 8.8.8.8:

```
[mathis@marcel etc]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=20.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=18.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=20.1 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=112 time=25.3 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 18.099/21.087/25.261/2.612 ms
```

```
[mathis@node1 etc]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=18.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=18.5 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=20.2 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=112 time=18.7 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 18.339/18.938/20.166/0.727 ms
```

Pour ping un nom de domaine, ajouter dans le même fichier (ifcfg-enp0s8) la ligne: 
"DNS1=8.8.8.8"

```
[mathis@node1 network-scripts]$ dig google.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42306
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             89      IN      A       216.58.198.206

;; Query time: 21 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Sun Sep 26 18:03:55 CEST 2021
;; MSG SIZE  rcvd: 55
```

```
[mathis@node1 network-scripts]$ ping google.com
PING google.com (172.217.19.238) 56(84) bytes of data.
64 bytes from par21s11-in-f14.1e100.net (172.217.19.238): icmp_seq=1 ttl=115 time=45.5 ms
64 bytes from par21s11-in-f14.1e100.net (172.217.19.238): icmp_seq=2 ttl=115 time=19.3 ms
64 bytes from par21s11-in-f14.1e100.net (172.217.19.238): icmp_seq=3 ttl=115 time=31.8 ms
64 bytes from par21s11-in-f14.1e100.net (172.217.19.238): icmp_seq=4 ttl=115 time=19.5 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 19.272/29.003/45.465/10.770 ms
```

```
[mathis@marcel network-scripts]$ dig google.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15609
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             103     IN      A       216.58.214.78

;; Query time: 23 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sun Sep 26 18:02:14 CEST 2021
;; MSG SIZE  rcvd: 55

```

```
[mathis@marcel network-scripts]$ ping google.com
PING google.com (142.250.179.78) 56(84) bytes of data.
64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=1 ttl=112 time=18.8 ms
64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=2 ttl=112 time=18.6 ms
64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=3 ttl=112 time=20.9 ms
64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=4 ttl=112 time=18.7 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 18.563/19.212/20.857/0.952 ms
```

```
[mathis@node1 network-scripts]$ sudo tcpdump -w tp2_routage_internet.pcap -i enp0s8 -c 10 not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

```
[mathis@node1 ~]$ ping -c 1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=18.5 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 18.501/18.501/18.501/0.000 ms
```

| ordre | type trame | IP source           | MAC source                  | IP destination | MAC destination       |
|-------|------------|---------------------|-----------------------------|----------------|-----------------------|
| 1     | ping       | `node1` `10.2.1.11` | `node1` `08:00:27:71:b4:d6` | `8.8.8.8`      | `08:00:27:0a:50:bb`   |
| 2     | pong       | `8.8.8.8`           | `08:00:27:0a:50:bb`         | `10.2.1.11`    | `08:00:27:71:b4:d6`   |

# III. DHCP

Installation du serveur sur node1.net1.tp2:

`[mathis@node1 ~]$ sudo dnf -y install dhcp-server`

`[mathis@node1 ~]$ sudo nano /etc/dhcp/dhcpd.conf`

dans le fichier dhcpd.conf:

```
default-lease-time 900;
max-lease-time 10800;
authoritative;
subnet 10.2.1.0 netmask 255.255.255.0 {
    range 10.2.1.10 10.2.1.200;
    option subnet-mask 255.255.255.0;
    option routers 10.2.1.254;
    option domain-name-servers 1.1.1.1;
}
```

```
[mathis@node1 ~]$ sudo firewall-cmd --runtime-to-permanent
```

```
[mathis@node1 ~]$ sudo firewall-cmd --runtime-to-permanent
success
```

```
sudo systemctl start dhcpd
```

```
sudo systemctl enable dhcpd
```

```
[mathis@node1 ~]$ sudo systemctl enable --now dhcpd
```

```
[mathis@node1 ~]$ sudo systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2021-09-26 19:22:37 CEST; 2h 21min ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 2474 (dhcpd)
   Status: "Dispatching packets..."
    Tasks: 1 (limit: 11385)
   Memory: 4.9M
   CGroup: /system.slice/dhcpd.service
           └─2474 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Sep 26 21:28:53 node1.net1.tp2 dhcpd[2474]: DHCPACK on 10.2.1.10 to 08:00:27:37:0c:d3 (node2) via enp>
Sep 26 21:35:03 node1.net1.tp2 dhcpd[2474]: DHCPREQUEST for 10.2.1.10 from 08:00:27:37:0c:d3 (node2) >
Sep 26 21:35:03 node1.net1.tp2 dhcpd[2474]: DHCPACK on 10.2.1.10 to 08:00:27:37:0c:d3 (node2) via enp>
Sep 26 21:35:11 node1.net1.tp2 dhcpd[2474]: reuse_lease: lease age 8 (secs) under 25% threshold, repl>
Sep 26 21:35:11 node1.net1.tp2 dhcpd[2474]: DHCPREQUEST for 10.2.1.10 from 08:00:27:37:0c:d3 (node2) >
Sep 26 21:35:11 node1.net1.tp2 dhcpd[2474]: DHCPACK on 10.2.1.10 to 08:00:27:37:0c:d3 (node2) via enp>
Sep 26 21:40:57 node1.net1.tp2 dhcpd[2474]: DHCPREQUEST for 10.2.1.10 from 08:00:27:37:0c:d3 (node2) >
Sep 26 21:40:57 node1.net1.tp2 dhcpd[2474]: DHCPACK on 10.2.1.10 to 08:00:27:37:0c:d3 via enp0s8
Sep 26 21:42:33 node1.net1.tp2 dhcpd[2474]: DHCPREQUEST for 10.2.1.10 from 08:00:27:37:0c:d3 via enp0>
Sep 26 21:42:33 node1.net1.tp2 dhcpd[2474]: DHCPACK on 10.2.1.10 to 08:00:27:37:0c:d3 (node2) via enp>
```

```
[mathis@node2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:37:0c:d3 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.10/24 brd 10.2.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 749sec preferred_lft 749sec
    inet6 fe80::a00:27ff:fe37:cd3/64 scope link
       valid_lft forever preferred_lft forever
```

ping vers sa passerelle:

```
[mathis@node2 ~]$ ping 10.2.1.254
PING 10.2.1.254 (10.2.1.254) 56(84) bytes of data.
64 bytes from 10.2.1.254: icmp_seq=1 ttl=64 time=1.41 ms
64 bytes from 10.2.1.254: icmp_seq=2 ttl=64 time=0.821 ms
64 bytes from 10.2.1.254: icmp_seq=3 ttl=64 time=0.683 ms

--- 10.2.1.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.683/0.970/1.406/0.313 ms
```

présence de la route:
```
[mathis@node2 ~]$ ip r s
default via 10.2.1.254 dev enp0s8 proto dhcp metric 100
10.2.1.0/24 dev enp0s8 proto kernel scope link src 10.2.1.10 metric 100
```

ping 8.8.8.8:

```
[mathis@node2 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=18.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=18.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=18.8 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=112 time=19.8 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 18.583/19.007/19.806/0.490 ms
```

dig de google.com:

```
[mathis@node2 ~]$ dig google.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53342
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             105     IN      A       216.58.209.238

;; Query time: 62 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Sun Sep 26 22:20:52 CEST 2021
;; MSG SIZE  rcvd: 55
```

ping vers un nom de domaine:

```
[mathis@node2 ~]$ ping google.com
PING google.com (142.250.75.238) 56(84) bytes of data.
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=1 ttl=114 time=23.9 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=2 ttl=114 time=20.6 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=3 ttl=114 time=19.1 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=4 ttl=114 time=19.9 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 19.135/20.882/23.908/1.827 ms
```

tcpdump sur le serveur dhcp:
```
[mathis@node1 ~]$ sudo tcpdump -w tp2_dhcp.pcap -i enp0s8 -c 100 not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
^C59 packets captured
60 packets received by filter
0 packets dropped by kernel
```

sur node1:
suppression de l'adresse IP actuelle:

```
[mathis@node2 ~]$ sudo dhclient -r
[sudo] password for mathis:
Killed old client process
```

requête d'une nouvelle IP:

```
[mathis@node2 ~]$ sudo dhclient
```

