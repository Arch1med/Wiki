# Hashcat 

## Présentation

Hashcat est un outil Linux open-source et développer par le MIT qui permet de cracker des hash. Il est réputé pour être le plus rapide et le plus avancé des outils actuellement disponible pour ce genre d'usage.

### C'est quoi un hash ?

"Le terme hash fait référence à un type de fichier utilisé dans le monde de l'informatique et celui de la cryptographie. Il est associé à la fonction de hashage, un algorithme mathématique qui consiste à convertir une chaîne de caractères en une valeur inférieure."

Source: https://www.journaldunet.fr/web-tech/dictionnaire-du-webmastering/1203455-hash-definition-traduction/

On utilise notamment dans les bases de données pour "cacher" les mots de passes et éviter qu'il se retrouve en clair dans ces dernières. C'est d'ailleurs l'une des préconisations de la CNIL concernant les applications, les sites Web et les Bases de données. (cf: https://www.cnil.fr/fr/securiser-vos-sites-web-vos-applications-et-vos-serveurs)

## Installation 

Sous Kali l'outil est déjà installé. Sinon simplement via la commande: 

	sudo apt install hashcat
	
Ou directement sur le [site officiel de Hashcat](https://hashcat.net/hashcat/).


## Usage

### Mode de hashage

Lors d'une tentative de crack d'un hash il est primordial de connaître la méthode de hashage. Pour se faire il existe de nombreux outil, tels que `hash-identifier` ou bien des sites internet qui se charge de faire le travaille (ex : [hash-analyzer](https://www.tunnelsup.com/hash-analyzer/))

Une fois la méthode de hashage connu il faut trouver le "mode" qui y est associé. En effet, lors de l'usage de hashcat pour cracker un hash, il faut préciser le mode que l'on souhaite utiliser pour casser notre hash. La commande `hashcat --help` nous donne tout les modes possibles, pour trier, il est recommander d'utiliser grep `hashcat --help | grep <hashage>`. Par exemple pour du bcrypt:

`hashcat --help | grep bcrypt` 
 
nous donne:

`3200 | bcrypt $2*$, Blowfish (Unix)                     | Operating Systems`

Il faudrat donc utiliser le mode **3200**

### Les différents mode d'attaques

- 0 : Straight, il correspont à l'**attaque par dictionnaire** (en testant tout une liste de mots).
- 1 : Combinaison, il correspond à l'**attaque combinée** qui consiste à concaténer les mots de plusieurs dictionnaires.
- 3 : Brute-force, il correspond à l'**attaque par force brute** qui consiste à essayer toute les combinaisons de caractères possibles une à une (très fastidieux)
- 6 : Hybrid Wordlist + Mask, qui correspond à l'**attaque hybride** qui combine des mots avec des masques.
- 7 : Mask + Hybrid Wordlist ,  qui correspond à l'**attaque hybride** qui combine des mots avec des masques (mais inversé par rapport au 6).

#### Attaque par Dictionnaire 

Dans le cadre d'une attaque par dictionnaire il est conseillé d'utiliser le dictionnaire **rockyou.txt**, facilement trouvable sur Internet. On peut aussi choisir d'utiliser nos propres dictionnaires.

```
hashcat -a 0 -m 0 hash.txt rockyou.txt -o cracked.txt

- -a 0 : indique que c'est une attaque par dictionnaire
- -m 0 : indique qu'il s'agit d'un hash de type md5
- hash.txt : Notre fichier texte contenant le hash que l'on souhaite cracker
- rockyou.txt : Notre dictionnaire de mot pour l'attaque
- -o cracked.txt : Le fichier de sorti dans lequel écrire le résultat
```

#### Attaque par dictionnaire avec l'usage de règle

L'usage de règle permet de décliner une liste de mot selon des intructions. Par exemple prenons le dictionnaire suivant contenant 3 mots:

```txt
alice
bob
password
```

Si je souhaite tester ces combinaisons, ainsi que ces même combinaisons avec: 
- une masjuscule en début de mot, 
- le mot en majuscule, 
- les e remplacé par des 3
- ...

il faudrait que je modifie mon dictionnaire de la façon suivante: 

```txt
alice
Alice
ALICE
alic3
bob
Bob
BOB
password
Password
PASSWORD
```

Cela peut-être très fastidieux lorsque nous souhaitons effectuer ce traitement sur un dictionnaire contenant des centaines et des centaines de mots. 
Pour effectuer ces déclinaisons, il existe donc des règles que nous pouvons appliquer à un dictionnaire en précisant le fichier de règle avec l'option `-r <emplacement fichier de règle>`.

Il est possible de créer nous même notre fichier de règle, ou d'en utiliser des déjà existant se trouvant dans le répertoire `/usr/share/hashcat/rules/`. 

L'objectif de cette page n'est pas de vous apprendre à créer des fichiers de règles, vous pouvez cependant trouver des tutos très facilement sur Internet à l'aide d'un moteur de recherche ou bien aux adresses suivantes:
- [hashcat site officiel](https://hashcat.net/wiki/doku.php?id=rule_based_attack)
- [4armed.com](https://www.4armed.com/blog/hashcat-rule-based-attack/) site détaillant la procédurepour créer des règles.

Ainsi nous la syntaxe devient: 



```
hashcat -m 3200 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/d3ad0ne.rule -o crack.txt --show

- -a 0 : indique que c'est une attaque par dictionnaire
- -m 3200 : indique qu'il s'agit d'un hash de type bcrypt
- hash.txt : Notre fichier texte contenant le hash que l'on souhaite cracker
- rockyou.txt : Notre dictionnaire de mot pour l'attaque
- -r /usr/share/hashcat/rules/d3ad0ne.rule : Notre fichier de règle
- -o cracked.txt : Le fichier de sorti dans lequel écrire le résultat
```


## CheatSheet

 ```
- Perform a brute-force attack (mode 3) with the default hashcat mask:

   hashcat -m {{hash_type_id}} -a {{3}} {{hash_value}}


 - Perform a brute-force attack (mode 3) with a known pattern of 4 digits:
 
   hashcat -m {{hash_type_id}} -a {{3}} {{hash_value}} "{{?d?d?d?d}}"


 - Perform a brute-force attack (mode 3) using at most 8 of all printable ASCII characters:
 
   hashcat -m {{hash_type_id}} -a {{3}} --increment {{hash_value}} "{{?a?a?a?a?a?a?a?a}}"


 - Perform a dictionary attack (mode 0) using the RockYou wordlist of a Kali Linux box:
 
   hashcat -m {{hash_type_id}} -a {{0}} {{hash_value}} {{/usr/share/wordlists/rockyou.txt}}
   

 - Perform a rule based dictionary attack (mode 0) using the RockYou wordlist mutated with common password variations:
 
   hashcat -m {{hash_type_id}} -a {{0}} --rules-file {{/usr/share/hashcat/rules/best64.rule}} {{hash_value}} {{/usr/share/wordlists/rockyou.txt}}


 - Perform a combination attack (mode 1) using the concatenation of words from two different custom dictionaries:
 
   hashcat -m {{hash_type_id}} -a {{1}} {{hash_value}} {{/path/to/dictionary1.txt}} {{/path/to/dictionary2.txt}}


 - Show result of an already cracked hash:
 
   hashcat --show {{hash_value}}
```
From tldr page.
