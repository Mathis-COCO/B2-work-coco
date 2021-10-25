# TP2 pt. 2 : Maintien en condition opérationnelle

# Sommaire

- [TP2 pt. 2 : Maintien en condition opérationnelle](#tp2-pt-2--maintien-en-condition-opérationnelle)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist](#checklist)
- [I. Monitoring](#i-monitoring)
  - [1. Le concept](#1-le-concept)
  - [2. Setup](#2-setup)
- [II. Backup](#ii-backup)
  - [1. Intwo bwo](#1-intwo-bwo)
  - [2. Partage NFS](#2-partage-nfs)
  - [3. Backup de fichiers](#3-backup-de-fichiers)
  - [4. Unité de service](#4-unité-de-service)
    - [A. Unité de service](#a-unité-de-service)
    - [B. Timer](#b-timer)
    - [C. Contexte](#c-contexte)
  - [5. Backup de base de données](#5-backup-de-base-de-données)
  - [6. Petit point sur la backup](#6-petit-point-sur-la-backup)
- [III. Reverse Proxy](#iii-reverse-proxy)
  - [1. Introooooo](#1-introooooo)
  - [2. Setup simple](#2-setup-simple)
  - [3. Bonus HTTPS](#3-bonus-https)
- [IV. Firewalling](#iv-firewalling)
  - [1. Présentation de la syntaxe](#1-présentation-de-la-syntaxe)
  - [2. Mise en place](#2-mise-en-place)
    - [A. Base de données](#a-base-de-données)
    - [B. Serveur Web](#b-serveur-web)
    - [C. Serveur de backup](#c-serveur-de-backup)
    - [D. Reverse Proxy](#d-reverse-proxy)
    - [E. Tableau récap](#e-tableau-récap)

# I. Monitoring

On bouge pas pour le moment niveau machines :

| Machine         | IP            | Service                 | Port ouvert | IP autorisées |
|-----------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             | 80          | 10.102.1.0/24 |
| `db.tp2.linux`  | `10.102.1.12` | Serveur Base de Données | 3306        | 10.102.1.0/24 |

## 1. Le concept

[...]

## 2. Setup

🌞 **Setup Netdata**

- y'a plein de méthodes d'install pour Netdata
- on va aller au plus simple, exécutez, sur toutes les machines que vous souhaitez monitorer :

```
[mathis@web ~]$ sudo su -
[sudo] password for mathis:
[root@web ~]# bash <(curl -Ss https://my-netdata.io/kickstart-static64.sh)
 --- Downloading static netdata binary: https://storage.googleapis.com/netdata-nightlies/netdata-latest.gz.run ---
[/tmp/netdata-kickstart-RLBmT3sS0a]# curl -q -sSL --connect-timeout 10 --retry 3 --output /tmp/netdata-kickstart-RLBmT3sS0a/sha256sum.txt https://storage.googleapis.com/netdata-nightlies/sha256sums.txt
 OK

[/tmp/netdata-kickstart-RLBmT3sS0a]# curl -q -sSL --connect-timeout 10 --retry 3 --output /tmp/netdata-kickstart-RLBmT3sS0a/netdata-latest.gz.run https://storage.googleapis.com/netdata-nightlies/netdata-latest.gz.run
 OK

 --- Installing netdata ---
[/tmp/netdata-kickstart-RLBmT3sS0a]# sh /tmp/netdata-kickstart-RLBmT3sS0a/netdata-latest.gz.run -- --auto-update

[...]
```
🌞 **Manipulation du *service* Netdata**

- un *service* `netdata` a été créé
- la conf de netdata se trouve dans `/opt/netdata/etc/netdata/` (pas directement dans `/etc/netdata/`)
  - ceci est du à la méthode d'install peu orthodoxe que je vous fais utiliser
- déterminer s'il est actif, et s'il est paramétré pour démarrer au boot de la machine
  - si ce n'est pas le cas, faites en sorte qu'il démarre au boot de la machine
    ```
    [mathis@web ~]$ sudo systemctl is-enabled netdata
    enabled
    ```
    ```
    [mathis@db ~]$ sudo systemctl is-enabled netdata
    enabled
    ```
- déterminer à l'aide d'une commande `ss` sur quel port Netdata écoute
    ```
    [mathis@web ~]$ sudo ss -alpnt
    State     Recv-Q    Send-Q         Local Address:Port          Peer Address:Port    Process
    LISTEN    0         128                  0.0.0.0:19999              0.0.0.0:*        users:(("netdata",pid=2561,fd=5))
    LISTEN    0         128                  0.0.0.0:22                 0.0.0.0:*        users:(("sshd",pid=764,fd=5))
    LISTEN    0         128                127.0.0.1:8125               0.0.0.0:*        users:(("netdata",pid=2561,fd=35))
    LISTEN    0         128                     [::]:19999                 [::]:*        users:(("netdata",pid=2561,fd=6))
    LISTEN    0         128                        *:80                       *:*        users:(("httpd",pid=1962,fd=4),("httpd",pid=1746,fd=4),("httpd",pid=1745,fd=4),("httpd",pid=1744,fd=4),("httpd",pid=1741,fd=4))
    LISTEN    0         128                     [::]:22                    [::]:*        users:(("sshd",pid=764,fd=7))
    LISTEN    0         128                    [::1]:8125                  [::]:*        users:(("netdata",pid=2561,fd=34))
    ```
    ```
    [mathis@db ~]$ sudo ss -alpnt
    [sudo] password for mathis:
    State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port Process
    LISTEN  0       128            0.0.0.0:22           0.0.0.0:*     users:(("sshd",pid=754,fd=5))
    LISTEN  0       128          127.0.0.1:8125         0.0.0.0:*     users:(("netdata",pid=2029,fd=35))
    LISTEN  0       128            0.0.0.0:19999        0.0.0.0:*     users:(("netdata",pid=2029,fd=5))
    LISTEN  0       128               [::]:22              [::]:*     users:(("sshd",pid=754,fd=7))
    LISTEN  0       128              [::1]:8125            [::]:*     users:(("netdata",pid=2029,fd=34))
    LISTEN  0       128               [::]:19999           [::]:*     users:(("netdata",pid=2029,fd=6))
    LISTEN  0       80                   *:3306               *:*     users:(("mysqld",pid=841,fd=49))
    ```
- autoriser ce port dans le firewall
    ```
    [mathis@web ~]$ sudo firewall-cmd --add-port=19999/tcp
    success
    [mathis@web ~]$ sudo firewall-cmd --add-port=19999/tcp --permanent
    success
    [mathis@web ~]$ sudo firewall-cmd --add-port=8125/tcp
    success
    [mathis@web ~]$ sudo firewall-cmd --add-port=8125/tcp --permanent
    success
    [mathis@web ~]$ sudo firewall-cmd --reload
    success
    ```
    ```
    [mathis@db ~]$ sudo firewall-cmd --add-port=19999/tcp
    success
    [mathis@db ~]$ sudo firewall-cmd --add-port=19999/tcp --permanent
    success
    [mathis@db ~]$ sudo firewall-cmd --add-port=8125/tcp
    success
    [mathis@db ~]$ sudo firewall-cmd --add-port=8125/tcp --permanent
    success
    [mathis@db ~]$ sudo firewall-cmd --reload
    success
    ```

    ``` 
    [mathis@web ~]$ curl localhost:19999
    <!doctype html><html lang="en"><head><title>netdata dashboard</title><meta name="application-name" content="netdata"><meta http-equiv="Content-Type" content="text/html; charset=utf-8"/><meta charset="utf-8"><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"><meta name="viewport" content="width=device-width,initial-scale=1"><meta name="apple-mobile-web-app-capable" content="yes"><meta name="apple-mobile-web-app-status-bar-style" 
    ```

    [img](/LINUX/TP2_Part_2/img/img_netdata.png)

🌞 **Setup Alerting**

- ajustez la conf de Netdata pour mettre en place des alertes Discord
  - *ui ui c'est bien ça :* vous recevrez un message Discord quand un seul critique est atteint
- [c'est là que ça se passe dans la doc de Netdata](https://learn.netdata.cloud/docs/agent/health/notifications/discord)
  - noubliez pas que la conf se trouve pour nous dans `/opt/netdata/etc/netdata/`
    ```
    [mathis@web ~]$ cd /opt/netdata/etc/netdata/
    ```
    ```
    [mathis@web netdata]$ sudo ./edit-config health_alarm_notify.conf
    Copying '/opt/netdata/usr/lib/netdata/conf.d/health_alarm_notify.conf' to '/opt/netdata/etc/netdata/health_alarm_notify.conf' ...
    Editing '/opt/netdata/etc/netdata/health_alarm_notify.conf' ...
    ```
    ```
    # discord (discordapp.com) global notification options

    # multiple recipients can be given like this:
    #                  "CHANNEL1 CHANNEL2 ..."

    # enable/disable sending discord notifications
    SEND_DISCORD="YES"

    # Create a webhook by following the official documentation -
    # https://support.discordapp.com/hc/en-us/articles/228383668-Intro-to-Webhooks
    DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/897135613709545562/Wp8Gr9U9TUPYxwJ2NZLckteA8ybo_D7Yd_hKuo_zTBateSzL_itTmgjYQP2zusMZGsM3"

    # if a role's recipients are not configured, a notification will be send to
    # this discord channel (empty = do not send a notification for unconfigured
    # roles):
    DEFAULT_RECIPIENT_DISCORD="notifs-server"
    ```
- vérifiez le bon fonctionnement de l'alerting sur Discord
  - en suivant la doc
    ```
    bash-4.4$ /opt/netdata/usr/libexec/netdata/plugins.d/alarm-notify.sh test

    # SENDING TEST WARNING ALARM TO ROLE: sysadmin
    [...]
    # OK

    # SENDING TEST CLEAR ALARM TO ROLE: sysadmin
    [...]
    --- END curl command ---
    --- BEGIN received response ---
    ok
    --- END received response ---
    RECEIVED HTTP RESPONSE CODE: 200
    2021-10-11 19:05:46: alarm-notify.sh: INFO: sent discord notification for: db.tp2.linux test.chart.test_alarm is CLEAR to 'Général'
    # OK
    ```
  Image [img_notif_server](/LINUX/TP2_Part_2/img/img_notif_server.png)

🌞 **Config alerting**

- créez une nouvelle alerte pour recevoir une alerte à 50% de remplissage de la RAM
    On commence par créer le nouveau fichier dans le dossier health.d:
    ```
    [mathis@web ~]$ sudo touch health.d/ram-usage.conf
    ```
    ```
    [mathis@web ~]$ sudo ./edit-config health.d/ram-usage.conf
    Editing '/opt/netdata/etc/netdata/health.d/ram-usage.conf' ...
    ```
    Une fois dans le fichier ram-usage.conf, ajouter le contenu suivant:
    ```
     alarm: ram_usage
        on: system.ram
    lookup: average -1m percentage of used
     units: %
     every: 1m
      warn: $this > 50
      crit: $this > 90
      info: The percentage of RAM being used by the system.
    ```
- testez que votre alerte fonctionne
  - il faudra remplir artificiellement la RAM pour voir si l'alerte remonte correctement
  - sur Linux, on utilise la commande `stress` pour ça
  ```
  [mathis@web netdata]$ sudo killall -USR2 netdata
  [mathis@web netdata]$ stress --vm-bytes $(awk '/MemAvailable/{printf "%d\n", $2 * 0.98;}' < /proc/meminfo)k --vm-keep -m 1
  stress: info: [2319] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
  ```
  Une fois le stress test en cours, on reçoit une notif discord contenant les infos suivantes:
  ```
  web.tp2.linux is critical, mem.available (system), ram available = 1.67%
    ram available = 1.67%
    Percentage of an estimated amount of RAM available for userspace processes, without causing swapping. Low amount of available memory. It may affect the performance of applications. If there is no swap space available, OOM Killer can start killing processes. You might want to check per-process memory usage to find the top consumers.
    mem.available
    system
    Image

    web.tp2.linux•Aujourd’hui à 15:10
    web.tp2.linux needs attention, system.ram (ram), ram in use = 92.8%
    ram in use = 92.8%
    Percentage of used RAM. High RAM utilization. It may affect the performance of applications. If there is no swap space available, OOM Killer can start killing processes. You might want to check per-process memory usage to find the top consumers.
    system.ram
    ram
    Image

    web.tp2.linux•Aujourd’hui à 15:10
    ```
> Le terme *"stress test"* est employé de façon générique pour désigner le fait de générer artificiellement de la charge sur un système, afin de l'éprouver, tester comment il réagit. Vous allez donc ici effectuer un *stress test de la RAM*.


# II. Backup

| Machine            | IP            | Service                 | Port ouvert | IPs autorisées |
|--------------------|---------------|-------------------------|-------------|----------------|
| `web.tp2.linux`    | `10.102.1.11` | Serveur Web             | 19999       | 10.102.1.0/24  |
| `db.tp2.linux`     | `10.102.1.12` | Serveur Base de Données | 19999       | 10.102.1.0/24  |
| `backup.tp2.linux` | `10.102.1.13` | Serveur de Backup (NFS) | ?           | ?              |

🖥️ **VM `backup.tp2.linux`**

**Déroulez la [📝**checklist**📝](#checklist) sur cette VM.**

## 1. Intwo bwo

**La *backup* consiste à extraire des données de leur emplacement original afin de les stocker dans un endroit dédié.**  

**Cet endroit dédié est un endroit sûr** : le but est d'assurer la perennité des données sauvegardées, tout en maintenant leur niveau de sécurité.

Pour la sauvegarde, il existe plusieurs façon de procéder. Pour notre part, nous allons procéder comme suit :

- **création d'un serveur de stockage**
  - il hébergera les sauvegardes de tout le monde
  - ce sera notre "endroit sûr"
  - ce sera un partage NFS
  - ainsi, toutes les machines qui en ont besoin pourront accéder à un dossier qui leur est dédié sur ce serveur de stockage, afin d'y stocker leurs sauvegardes
- **développement d'un script de backup**
  - ce script s'exécutera en local sur les machines à sauvegarder
  - il s'exécute à intervalles de temps réguliers
  - il envoie les données à sauvegarder sur le serveur NFS
  - du point de vue du script, c'est un dossier local. Mais en réalité, ce dossier est monté en NFS.

![You're supposed to backup everything](./pics/backup_meme.jpg)

## 2. Partage NFS

🌞 **Setup environnement**

- créer un dossier `/srv/backup/`
- il contiendra un sous-dossier ppour chaque machine du parc
  - commencez donc par créer le dossier `/srv/backup/web.tp2.linux/`
    ```
    [mathis@backup ~]$ sudo mkdir /srv/backup/
    [sudo] password for mathis:
    [mathis@backup ~]$ sudo mkdir /srv/backup/web.tp2.linux/
    ```
- il existera un partage NFS pour chaque machine (principe du moindre privilège)

🌞 **Setup partage NFS**

- je crois que vous commencez à connaître la chanson... Google "nfs server rocky linux"
  - [ce lien me semble être particulièrement simple et concis](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=1)
    ```
    [mathis@backup ~]$ sudo dnf -y install nfs-utils
    Last metadata expiration check: 0:50:23 ago on Tue 12 Oct 2021 04:11:23 PM CEST.
    Dependencies resolved.
    ======================================================================================================
    Package                      Architecture      Version                       Repository         Size
    ======================================================================================================
    Installing:
    nfs-utils                    x86_64            1:2.3.3-41.el8_4.2            baseos            497 k
    Installing dependencies:
    gssproxy                     x86_64            0.8.0-19.el8                  baseos            118 k
    keyutils                     x86_64            1.5.10-6.el8                  baseos             62 k

    libevent                     x86_64            2.1.8-5.el8                   baseos            252 k

    libverto-libevent            x86_64            0.3.0-5.el8                   baseos             15 k

    /srv/backup/web.tp2.linux 10.102.1.11/24(rw,no_root_squash)
    rpcbind                      x86_64            1.2.5-8.el8                   baseos             69 k

    Transaction Summary
    ======================================================================================================
    Install  6 Packages
    ```
    ```
    [mathis@backup ~]$ sudo nano /etc/idmapd.conf
    ```
    modification de la ligne suivante:
    ```
    Domain = tp2.linux
    ```
    création du fichier exports ...
    ```
    [mathis@backup ~]$ sudo vim /etc/exports
    ```
    ajout de la ligne suivante dans exports:
    ```
    /srv/backup/web.tp2.linux 10.102.1.11/24(rw,no_root_squash)
    ```
    ```
    [mathis@backup ~]$ sudo systemctl start nfs-server.service
    [mathis@backup ~]$ sudo systemctl enable nfs-server.service
    Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
    [mathis@backup ~]$ sudo firewall-cmd --add-service=nfs
    success
    [mathis@backup ~]$ sudo firewall-cmd --add-service={nfs3,mountd,rpc-bind} --permanent
    [mathis@backup ~]$ sudo firewall-cmd --runtime-to-permanent
    success
    [mathis@backup ~]$ sudo firewall-cmd --reload
    ```
🌞 **Setup points de montage sur `web.tp2.linux`**

- monter le dossier `/srv/backups/web.tp2.linux` du serveur NFS dans le dossier `/srv/backup/` du serveur Web
  ```
  [mathis@backup ~]$ sudo mkdir /srv/backups
  ```
- vérifier...
  - avec une commande `mount` que la partition est bien montée
    ```
    [mathis@web ~]$ sudo mount /srv/backup/ -v
    mount.nfs: timeout set for Mon Oct 25 22:20:39 2021
    mount.nfs: trying text-based options 'vers=4.2,addr=10.102.1.13,clientaddr=10.102.1.11'
    ```
  - avec une commande `df -h` qu'il reste de la place
    ```
    [mathis@web ~]$ df -h | grep backup
    backup.tp2.linux:/srv/backup/web.tp2.linux  6.2G  2.4G  3.9G  38% /srv/backup
    ```
  - avec une commande `touch` que vous avez le droit d'écrire dans cette partition
    ```
    [mathis@web ~]$ touch /srv/backup/test_de_fou_furieux
    ```
    ```
    [mathis@backup backup]$ ls /srv/backup/web.tp2.linux/
    test_de_fou_furieux
    ```
- faites en sorte que cette partition se monte automatiquement grâce au fichier `/etc/fstab`
  ```
  [mathis@web ~]$ cat /etc/fstab

  #
  # /etc/fstab
  # Created by anaconda on Sun Oct 10 13:19:46 2021
  #
  # Accessible filesystems, by reference, are maintained under '/dev/disk/'.
  # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
  #
  # After editing this file, run 'systemctl daemon-reload' to update systemd
  # units generated from this file.
  #
  /dev/mapper/rl-root     /                       xfs     defaults        0 0
  UUID=e1670393-b397-447c-9e2b-a04e8566ccb2 /boot                   xfs     defaults        0 0
  /dev/mapper/rl-swap     none                    swap    defaults        0 0
  backup.tp2.linux:/srv/backup/web.tp2.linux /srv/backup    nfs     defaults        0 0
  ```
  ```
  [mathis@db ~]$ sudo systemctl start nfs-server.service
  [mathis@db ~]$ sudo systemctl enable nfs-server.service
  Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
  [mathis@db ~]$ sudo mkdir /srv/backup
  [mathis@db ~]$ sudo nano /etc/fstab
  [mathis@db ~]$ sudo nano /etc/hosts
  [mathis@db ~]$ sudo mount /srv/backup/
  ```
  On teste donc pour db.tp2.linux:
  ```
  [mathis@db ~]$ sudo touch /srv/backup/test_de_malade_2
  ```
  ```
  [mathis@backup backup]$ ls /srv/backup/db.tp2.linux/
  test_de_malade_2
  ```


🌟 **BONUS** : partitionnement avec LVM

- ajoutez un disque à la VM `backup.tp2.linux`
- utilisez LVM pour créer une nouvelle partition (5Go ça ira)
- monter automatiquement cette partition au démarrage du système à l'aide du fichier `/etc/fstab`
- cette nouvelle partition devra être montée sur le dossier `/srv/backup/`

## 3. Backup de fichiers

🌞 **Rédiger le script de backup `/srv/tp2_backup.sh`**

- le script crée une archive compressée `.tar.gz` du dossier ciblé
  - cela se fait avec la commande `tar`
- l'archive générée doit s'appeler `tp2_backup_YYMMDD_HHMMSS.tar.gz`
  - vous remplacerez évidemment `YY` par l'année (`21`), `MM` par le mois (`10`), etc.
  - ces infos sont déterminées dynamiquement au moment où le script s'exécute à l'aide de la commande `date`
- le script utilise la commande `rsync` afin d'envoyer la sauvegarde dans le dossier de destination
- il **DOIT** pouvoir être appelé de la sorte :

📁 **Fichier `/srv/tp2_backup.sh`**

🌞 **Tester le bon fonctionnement**

- exécuter le script sur le dossier de votre choix
- prouvez que la backup s'est bien exécutée
- **tester de restaurer les données**
  - récupérer l'archive générée, et vérifier son contenu
  ```
  [mathis@backup ~]$ sudo nano /srv/tp2_backup.sh
  [mathis@backup srv]$ sudo  mkdir  /srv/backup_test
  [mathis@backup srv]$ sudo mkdir -p test/test_de_fou
  [mathis@backup srv]$ sudo ./tp2_backup.sh backup_test/ test/
  [OK] Archive /srv/tp2_backup_211025_231447.tar.gz created.
  [OK] Archive /srv/tp2_backup_211025_231447.tar.gz synchronized to backup_test/.
  [OK] Directory backup_test/ cleaned to keep only the 5 most recent backups.
  [mathis@backup backup_test]$ cat tp2_backup_211025_231447.tar.gz
  Gwa��1
  �0
    ���70�F�#��4�_��8t�oH
  �b�B��{���cXs�������+��
                          �<,y�q
                                ��OB���-=��([mathis@backup backup_test]$
  ```

🌟 **BONUS**

- faites en sorte que votre script ne conserve que les 5 backups les plus récentes après le `rsync`
- faites en sorte qu'on puisse passer autant de dossier qu'on veut au script : `./tp2_backup.sh <DESTINATION> <DOSSIER1> <DOSSIER2> <DOSSIER3>...` et n'obtenir qu'une seule archive
- utiliser [Borg](https://borgbackup.readthedocs.io/en/stable/) plutôt que `rsync`

## 4. Unité de service

Lancer le script à la main c'est bien. **Le mettre dans une joulie *unité de service* et l'exécuter à intervalles réguliers, de manière automatisée, c'est mieux.**

Le but va être de créer un *service* systemd pour que vous puissiez interagir avec votre script de sauvegarde en faisant :

```bash
$ sudo systemctl start tp2_backup
$ sudo systemctl status tp2_backup
```

Ensuite on créera un *timer systemd* qui permettra de déclencher le lancement de ce *service* à intervalles réguliers.

**La classe nan ?**

![systemd can do that](./pics/suprised-cat.jpg)

---

### A. Unité de service

🌞 **Créer une *unité de service*** pour notre backup

- c'est juste un fichier texte hein
- doit se trouver dans le dossier `/etc/systemd/system/`
- doit s'appeler `tp2_backup.service`
  ```
  [mathis@web ~]$ cat /etc/systemd/system/tp2_backup.service
  [Unit]
  Description=Our own lil backup service (TP2)

  [Service]
  ExecStart=/srv/tp2_backup.sh /srv/backup /var/www/html
  Type=oneshot
  RemainAfterExit=no

  [Install]
  WantedBy=multi-user.target
  ```

🌞 **Tester le bon fonctionnement**

- n'oubliez pas d'exécuter `sudo systemctl daemon-reload` à chaque ajout/modification d'un *service*
- essayez d'effectuer une sauvegarde avec `sudo systemctl start backup`
  ```
  [mathis@web ~]$ sudo systemctl daemon-reload
  [mathis@web ~]$ sudo systemctl enable tp2_backup.service
  [mathis@web ~]$ sudo systemctl start tp2_backup.service
  ```
- prouvez que la backup s'est bien exécutée
  ```
  [mathis@web ~]$ sudo systemctl status tp2_backup.service
  ● tp2_backup.service - Our own lil backup service (TP2)
   Loaded: loaded (/etc/systemd/system/tp2_backup.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Mon 2021-10-25 23:45:16 CEST; 1s ago
  Process: 4093 ExecStart=/srv/tp2_backup.sh /srv/backup /var/www/html (code=exited, status=0/SUCCESS)
  Main PID: 4093 (code=exited, status=0/SUCCESS)

  Oct 25 23:45:16 web.tp2.linux systemd[1]: Starting Our own lil backup service (TP2)...
  Oct 25 23:45:16 web.tp2.linux tp2_backup.sh[4093]: [OK] Archive //tp2_backup_211025_234516.tar.gz created.
  Oct 25 23:45:16 web.tp2.linux tp2_backup.sh[4093]: [OK] Archive //tp2_backup_211025_234516.tar.gz synchronized to /srv/backup.
  Oct 25 23:45:16 web.tp2.linux tp2_backup.sh[4093]: [OK] Directory /srv/backup cleaned to keep only the 5 most recent backups
  Oct 25 23:45:16 web.tp2.linux systemd[1]: tp2_backup.service: Succeeded.
  Oct 25 23:45:16 web.tp2.linux systemd[1]: Started Our own lil backup service (TP2).
  ```
  - vérifiez la présence de la nouvelle archive
    ```
    [mathis@web ~]$ ls /srv/backup/
    test_de_fou_furieux  tp2_backup_211025_234453.tar.gz  tp2_backup_211025_234514.tar.gz  tp2_backup_211025_234516.tar.gz
    ```

---

### B. Timer

🌞 **Créer le *timer* associé à notre `tp2_backup.service`**

- toujours juste un fichier texte
- dans le dossier `/etc/systemd/system/` aussi
- fichier `tp2_backup.timer`
- contenu du fichier : 
  ```
  [mathis@web ~]$ sudo nano /etc/systemd/system/tp2_backup.timer
  [mathis@web ~]$ cat /etc/systemd/system/tp2_backup.timer
  [Unit]
  Description=Periodically run our TP2 backup script
  Requires=tp2_backup.service

  [Timer]
  Unit=tp2_backup.service
  OnCalendar=*-*-* *:*:00

  [Install]
  WantedBy=timers.target
  ```

> Le nom du *timer* doit être rigoureusement identique à celui du *service*. Seule l'extension change : de `.service` à `.timer`. C'est notamment grâce au nom identique que systemd sait que ce *timer* correspond à un *service* précis.

🌞 **Activez le timer**

- démarrer le *timer* : `sudo systemctl start tp2_backup.timer`
- activer le au démarrage avec une autre commande `systemctl`
  ```
  [mathis@web ~]$ sudo systemctl start tp2_backup.timer
  [sudo] password for mathis:
  [mathis@web ~]$ sudo systemctl enable tp2_backup.timer
  Created symlink /etc/systemd/system/timers.target.wants/tp2_backup.timer → /etc/systemd/system/tp2_backup.timer.
  ```
- prouver que...
  - le *timer* est actif actuellement
  - qu'il est paramétré pour être actif dès que le système boot
  ```
  [mathis@web ~]$ sudo systemctl status tp2_backup.timer
  ● tp2_backup.timer - Periodically run our TP2 backup script
    Loaded: loaded (/etc/systemd/system/tp2_backup.timer; enabled; vendor preset: disabled)
    Active: active (waiting) since Tue 2021-10-26 00:01:09 CEST; 3min 39s ago
    Trigger: Tue 2021-10-26 00:05:00 CEST; 10s left

  Oct 26 00:01:09 web.tp2.linux systemd[1]: Started Periodically run our TP2 backup script.
  ```
  autre façon moins détaillée:
  ```
  [mathis@web ~]$ sudo systemctl is-enabled tp2_backup.timer
  enabled
  [mathis@web ~]$ sudo systemctl is-active tp2_backup.timer
  active
  ```

🌞 **Tests !**

- avec la ligne `OnCalendar=*-*-* *:*:00`, le *timer* déclenche l'exécution du *service* toutes les minutes
- vérifiez que la backup s'exécute correctement
  ```
  [mathis@web srv]$ date
  Tue Oct 26 00:09:33 CEST 2021
  [mathis@web srv]$ ls /srv/backup
  tp2_backup_211026_000502.tar.gz  tp2_backup_211026_000602.tar.gz  tp2_backup_211026_000702.tar.gz  tp2_backup_211026_000802.tar.gz  tp2_backup_211026_000902.tar.gz
  ```
  1 min plus tard environ:
  ```
  [mathis@web srv]$ date
  Tue Oct 26 00:10:12 CEST 2021
  [mathis@web srv]$ ls /srv/backup
  tp2_backup_211026_000602.tar.gz  tp2_backup_211026_000702.tar.gz  tp2_backup_211026_000802.tar.gz  tp2_backup_211026_000902.tar.gz  tp2_backup_211026_001002.tar.gz
  ```

---

### C. Contexte

🌞 **Faites en sorte que...**

- votre backup s'exécute sur la machine `web.tp2.linux`
- le dossier sauvegardé est celui qui contient le site NextCloud (quelque part dans `/var/`)
- la destination est le dossier NFS monté depuis le serveur `backup.tp2.linux`
  ```
  [mathis@web srv]$ cat /etc/systemd/system/tp2_backup.service | grep Exec
  ExecStart=/srv/tp2_backup.sh /srv/backup /var/www/html
  ```
- la sauvegarde s'exécute tous les jours à 03h15 du matin
  ```
  [mathis@web srv]$ cat /etc/systemd/system/tp2_backup.timer | grep OnCalendar
  OnCalendar=*-*-* 3:15:00
  ```
- prouvez avec la commande `sudo systemctl list-timers` que votre *service* va bien s'exécuter la prochaine fois qu'il sera 03h15
vu qu'on vient de modifier la config du service, on doit reload le daemon
  ```
  [mathis@web srv]$ sudo systemctl daemon-reload
  [mathis@web srv]$ sudo systemctl list-timers
  NEXT                          LEFT          LAST                          PASSED       UNIT                         ACTIVATES
  Tue 2021-10-26 03:15:00 CEST  2h 56min left n/a                           n/a          tp2_backup.timer             tp2_backup.service
  Tue 2021-10-26 19:58:50 CEST  19h left      Mon 2021-10-25 19:58:50 CEST  4h 19min ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
  n/a                           n/a           Tue 2021-10-26 00:17:46 CEST  16s ago      dnf-makecache.timer          dnf-makecache.service

  3 timers listed.
  Pass --all to see loaded but inactive timers, too.
  ```

📁 **Fichier `/etc/systemd/system/tp2_backup.timer`**  
📁 **Fichier `/etc/systemd/system/tp2_backup.service`**

## 5. Backup de base de données

Sauvegarder des dossiers c'est bien. Mais sauvegarder aussi les bases de données c'est mieux.

🌞 **Création d'un script `/srv/tp2_backup_db.sh`**

- il utilise la commande `mysqldump` pour récupérer les données de la base de données
- cela génère un fichier `.sql` qui doit ensuite être compressé en `.tar.gz`
- il s'exécute sur la machine `db.tp2.linux`
- il s'utilise de la façon suivante :

```bash
$ ./tp2_backup_db.sh <DESTINATION> <DATABASE>
```

📁 **Fichier `/srv/tp2_backup_db.sh`**  

🌞 **Restauration**

- tester la restauration de données
- c'est à dire, une fois la sauvegarde effectuée, et le `tar.gz` en votre possession, tester que vous êtes capables de restaurer la base dans l'état au moment de la sauvegarde
  - il faut réinjecter le fichier `.sql` dans la base à l'aide d'une commmande `mysql`

🌞 ***Unité de service***

- pareil que pour la sauvegarde des fichiers ! On va faire de ce script une *unité de service*.
- votre script `/srv/tp2_backup_db.sh` doit pouvoir se lancer grâce à un *service* `tp2_backup_db.service`
- le *service* est exécuté tous les jours à 03h30 grâce au *timer* `tp2_backup_db.timer`
- prouvez le bon fonctionnement du *service* ET du *timer*

📁 **Fichier `/etc/systemd/system/tp2_backup_db.timer`**  
📁 **Fichier `/etc/systemd/system/tp2_backup_db.service`**

## 6. Petit point sur la backup

# III. Reverse Proxy

## 1. Introooooo

## 2. Setup simple

| Machine            | IP            | Service                 | Port ouvert | IPs autorisées |
|--------------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux`    | `10.102.1.11` | Serveur Web             | ?           | ?             |
| `db.tp2.linux`     | `10.102.1.12` | Serveur Base de Données | ?           | ?             |
| `backup.tp2.linux` | `10.102.1.13` | Serveur de Backup (NFS) | ?           | ?             |
| `front.tp2.linux`  | `10.102.1.14` | Reverse Proxy           | ?           | ?             |

🖥️ **VM `front.tp2.linu`x**

**Déroulez la [📝**checklist**📝](#checklist) sur cette VM.**

🌞 **Installer NGINX**

- vous devrez d'abord installer le paquet `epel-release` avant d'installer `nginx`
  - EPEL c'est des dépôts additionnels pour Rocky
  - NGINX n'est pas présent dans les dépôts par défaut que connaît Rocky
- le fichier de conf principal de NGINX est `/etc/nginx/nginx.conf`

🌞 **Tester !**

- lancer le *service* `nginx`
- le paramétrer pour qu'il démarre seul quand le système boot
- repérer le port qu'utilise NGINX par défaut, pour l'ouvrir dans le firewall
- vérifier que vous pouvez joindre NGINX avec une commande `curl` depuis votre PC

🌞 **Explorer la conf par défaut de NGINX**

- repérez l'utilisateur qu'utilise NGINX par défaut
- dans la conf NGINX, on utilise le mot-clé `server` pour ajouter un nouveau site
  - repérez le bloc `server {}` dans le fichier de conf principal
- par défaut, le fichier de conf principal inclut d'autres fichiers de conf
  - mettez en évidence ces lignes d'inclusion dans le fichier de conf principal

🌞 **Modifier la conf de NGINX**

- pour que ça fonctionne, le fichier `/etc/hosts` de la machine **DOIT** être rempli correctement, conformément à la **[📝**checklist**📝](#checklist)**
- supprimer le bloc `server {}` par défaut, pour ne plus présenter la page d'accueil NGINX
- créer un fichier `/etc/nginx/conf.d/web.tp2.linux.conf` avec le contenu suivant :
  - j'ai sur-commenté pour vous expliquer les lignes, n'hésitez pas à dégommer mes lignes de commentaires

```bash
[it4@localhost nginx]$ cat conf.d/web.tp2.linux.conf 
server {
    # on demande à NGINX d'écouter sur le port 80 pour notre NextCloud
    listen 80;

    # ici, c'est le nom de domaine utilisé pour joindre l'application
    # ce n'est pas le nom du reverse proxy, mais le nom que les clients devront saisir pour atteindre le site
    server_name web.tp2.linux; # ici, c'est le nom de domaine utilisé pour joindre l'application (pas forcéme

    # on définit un comportement quand la personne visite la racine du site (http://web.tp2.linux/)
    location / {
        # on renvoie tout le trafic vers la machine web.tp2.linux
        proxy_pass http://web.tp2.linux;
    }
}
```

## 3. Bonus HTTPS

**Etape bonus** : mettre en place du chiffrement pour que nos clients accèdent au site de façon plus sécurisée.

🌟 **Générer la clé et le certificat pour le chiffrement**

- il existe plein de façons de faire
- nous allons générer en une commande la clé et le certificat
- puis placer la clé et le cert dans les endroits standards pour la distribution Rocky Linux

```bash
# On se déplace dans un dossier où on peut écrire
$ cd ~

# Génération de la clé et du certificat
# Attention à bien saisir le nom du site pour le "Common Name"
$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt
[...]
Common Name (eg, your name or your server\'s hostname) []:web.tp2.linux
[...]

# On déplace la clé et le certificat dans les dossiers standards sur Rocky
# En le renommant
$ sudo mv server.key /etc/pki/tls/private/web.tp2.linux.key
$ sudo mv server.crt /etc/pki/tls/certs/web.tp2.linux.crt

# Setup des permissions restrictives
$ sudo chown root:root /etc/pki/tls/private/web.tp2.linux.key
$ sudo chown root:root /etc/pki/tls/certs/web.tp2.linux.crt
$ sudo chmod 400 /etc/pki/tls/private/web.tp2.linux.key
$ sudo chmod 644 /etc/pki/tls/certs/web.tp2.linux.crt
```

🌟 **Modifier la conf de NGINX**

- inspirez-vous de ce que vous trouvez sur internet
- il n'y a que deux lignes à ajouter
  - une ligne pour préciser le chemin du certificat
  - une ligne pour préciser le chemin de la clé
- et une ligne à modifier
  - préciser qu'on écoute sur le port 443, avec du chiffrement
- n'oubliez pas d'ouvrir le port 443/tcp dans le firewall

🌟 **TEST**

- connectez-vous sur `https://web.tp2.linux` depuis votre PC
- petite avertissement de sécu : normal, on a signé nous-mêmes le certificat
  - vous pouvez donc "Accepter le risque" (le nom du bouton va changer suivant votre navigateur)
  - avec `curl` il faut ajouter l'option `-k` pour désactiver cette vérification

# IV. Firewalling

**On va rendre nos firewalls un peu plus agressifs.**

Actuellement je vous ai juste demandé d'autoriser le trafic sur tel ou tel port. C'est bien.

**Maintenant on va restreindre le trafic niveau IP aussi.**

Par exemple : notre base de données `db.tp2.linux` n'est accédée que par le serveur Web `web.tp2.linux`, et par aucune autre machine.  
On va donc configurer le firewall de la base de données pour qu'elle n'accepte QUE le trafic qui vient du serveur Web.

**On va *harden* ("durcir" en français) la configuration de nos firewalls.**

## 1. Présentation de la syntaxe

> **N'oubliez pas d'ajouter `--permanent` sur toutes les commandes `firewall-cmd`** si vous souhaitez que le changement reste effectif après un rechargement de FirewallD.

**Première étape** : définir comme politique par défaut de TOUT DROP. On refuse tout, et on whiteliste après.

Il existe déjà une zone appelée `drop` qui permet de jeter tous les paquets. Il suffit d'ajouter nos interfaces dans cette zone.

```bash
$ sudo firewall-cmd --list-all # on voit qu'on est par défaut dans la zone "public"
$ sudo firewall-cmd --set-default-zone=drop # on configure la zone "drop" comme zone par défaut
$ sudo firewall-cmd --zone=drop --add-interface=enp0s8 # ajout explicite de l'interface host-only à la zone "drop"
```

**Ensuite**, on peut créer une nouvelle zone, qui autorisera le trafic lié à telle ou telle IP source :

```bash
$ sudo firewall-cmd --add-zone=ssh # le nom "ssh" est complètement arbitraire. C'est clean de faire une zone par service.
```

**Puis** on définit les règles visant à autoriser un trafic donné :

```bash
$ sudo firewall-cmd --zone=ssh --add-source=10.102.1.1/32 # 10.102.1.1 sera l'IP autorisée
$ sudo firewall-cmd --zone=ssh --add-port=22/tcp # uniquement le trafic qui vient 10.102.1.1, à destination du port 22/tcp, sera autorisé
```

**Le comportement de FirewallD sera alors le suivant :**

- si l'IP source d'un paquet est `10.102.1.1`, il traitera le paquet comme étant dans la zone `ssh`
- si l'IP source est une autre IP, et que le paquet arrive par l'interface `enp0s8` alors le paquet sera géré par la zone `drop` (le paquet sera donc *dropped* et ne sera jamais traité)

> *L'utilisation de la notation `IP/32` permet de cibler une IP spécifique. Si on met le vrai masque `10.102.1.1/24` par exemple, on autorise TOUT le réseau `10.102.1.0/24`, et non pas un seul hôte. Ce `/32` c'est un truc qu'on voit souvent en réseau, pour faire référence à une IP unique.*

![Cut here to activate firewall :D](./pics/cut-here-to-activate-firewall-best-label-for-lan-cable.jpg)

## 2. Mise en place

### A. Base de données

🌞 **Restreindre l'accès à la base de données `db.tp2.linux`**

- seul le serveur Web doit pouvoir joindre la base de données sur le port 3306/tcp
- vous devez aussi autoriser votre accès SSH
- n'hésitez pas à multiplier les zones (une zone `ssh` et une zone `db` par exemple)

> Quand vous faites une connexion SSH, vous la faites sur l'interface Host-Only des VMs. Cette interface est branchée à un Switch qui porte le nom du Host-Only. Pour rappel, votre PC a aussi une interface branchée à ce Switch Host-Only.  
C'est depuis cette IP que la VM voit votre connexion. C'est cette IP que vous devez autoriser dans le firewall de votre VM pour SSH.

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

- `sudo firewall-cmd --get-active-zones`
- `sudo firewall-cmd --get-default-zone`
- `sudo firewall-cmd --list-all --zone=?`

### B. Serveur Web

🌞 **Restreindre l'accès au serveur Web `web.tp2.linux`**

- seul le reverse proxy `front.tp2.linux` doit accéder au serveur web sur le port 80
- n'oubliez pas votre accès SSH

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

### C. Serveur de backup

🌞 **Restreindre l'accès au serveur de backup `backup.tp2.linux`**

- seules les machines qui effectuent des backups doivent être autorisées à contacter le serveur de backup *via* NFS
- n'oubliez pas votre accès SSH

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

### D. Reverse Proxy

🌞 **Restreindre l'accès au reverse proxy `front.tp2.linux`**

- seules les machines du réseau `10.102.1.0/24` doivent pouvoir joindre le proxy
- n'oubliez pas votre accès SSH

🌞 **Montrez le résultat de votre conf avec une ou plusieurs commandes `firewall-cmd`**

### E. Tableau récap

🌞 **Rendez-moi le tableau suivant, correctement rempli :**

| Machine            | IP            | Service                 | Port ouvert | IPs autorisées |
|--------------------|---------------|-------------------------|-------------|---------------|
| `web.tp2.linux`    | `10.102.1.11` | Serveur Web             | ?           | ?             |
| `db.tp2.linux`     | `10.102.1.12` | Serveur Base de Données | ?           | ?             |
| `backup.tp2.linux` | `10.102.1.13` | Serveur de Backup (NFS) | ?           | ?             |
| `front.tp2.linux`  | `10.102.1.14` | Reverse Proxy           | ?           | ?  