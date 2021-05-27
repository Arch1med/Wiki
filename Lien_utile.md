# Abstract 

Cette page contient les ressources que j'utilise lorsque j'effectue des CTF.

# Liens 

Le "[guide du hacker](https://hacklab.yoloctf.org/yoloctf/toolbox/toolbox.php)", rédigé par [ZenZenvibe](https://github.com/jossets)

## Escalade de privilège (privilege escalation)

En général: le Github de All The Things Payload, par [swisskyrepo](https://github.com/swisskyrepo) à l'adresse suivante pour l'escalade de privilège : [lien](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)

### Enumération de la machine

* [LinPEAS.sh](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite), un outil opensource développé par [carlospolop](https://github.com/carlospolop) qui permet d'énumérer les "failles" avec pour objectif de trouver une manière d'effectuer l'escalade de privilège. 
* [lse.sh](https://github.com/diego-treitos/linux-smart-enumeration) (Linux Smart Enumeration), similaire à l'outil précédent, développé par [diego treitos](https://github.com/diego-treitos)

### Exploitation de binaire 

En général, lorsqu'on se trouve sur une machine, on regarde s'il y a des binaires avec des droits d'exécution root (on parle alors de sticky bit), ou alors des bianaire exécutables avec sudo, nous permettant de faire une escalade de privilège.

Pour cela: 
* Trouver les sticky bit: `find / -perm -4000 -exec ls -al {} \; 2>/dev/null`
* Regarder les binaire éxécutables avec sudo: `sudo -l`

On regarde ensuite s'il existe des méthodes d'escalade. Personnellement j'utilise le site suivant: [gtfobins](https://gtfobins.github.io/)
