# Les Logs

## 1. Généralités
  * Adresse IP où se trouve le serveur Apache2 : `192.168.1.36`
  * Adresse IP de la machine local : `192.168.1.20`

## 2. Installation du serveur Web Apache 2

Pour cela, faites les commandes suivantes :
```bash
sudo apt update
sudo apt install apache2 -y
```

## 3. Configuration du logging
Par défaut, Apache2 est configuré pour enregistrer les accès et les erreurs dans des fichiers de log :
  * Fichier de log *accès* : `/var/log/apache2/access.log`
  * Fichier de log *erreurs* : `/var/log/apache2/error.log`

Il est tout de même possible de modifier la configuration d'enregistrement des logs via le fichier `/etc/apache2/apache2.conf` ou via les fichiers se trouvant dans le dossier `/etc/apache2/sites-available/`.  

## 4. Génération de trafic
Pour générer du trafic sur le serveur web, il existe deux façons de faire :
  1. Via la commande `curl`
     * Pour un log *accès* : `curl http://localhost`
     * Pour un log *erreurs* : `curl http://localhost/somepage`
  3. Via un navigateur :
     * Pour un log *accès* : `http://192.168.1.36`
     * Pour un log *erreurs* : `http://192.168.1.36/somepage`
    
## 5. Structure des logs
Le format de log par défaut d'Apache2 est généralement le format **combined**. Voici à quoi ressemble une ligne de log d'accès typique :  
`127.0.0.1 - - [16/Dec/2024:12:34:56 +0000] "GET /index.html HTTP/1.1" 200 1024 "http://referer" "User-Agent"`  
  * 127.0.0.1 : Adresse IP du client
  * '-' : Identité du client (non utilisée)
  * '-' : Nom d'utilisateur (non utilisée)
  * [16/Dec/2024:12:34:56 +0000] : Date et heure de la requête
  * "GET /index.html HTTP/1.1" : Requête HTTP
  * 200 : Code de statut HTTP (200 signifie succès)
  * 1024 : Taille de la réponse en octets
  * "http://referer" : URL de référence
  * "User-Agent" : Informations sur le navigateur du client

## 6. Analyse des logs
Pour rappel, pour savoir si une requête est réussie, le log aura un code 200, à contrario pour une erreur, le log aura un code 404.  
Pour pouvoir analyser les logs et extraire les informations requises, j'ai utilisé la commande `awk`.
**Requêtes réussies :**  
`awk '$9 == 200 {print $0}' /var/log/apache2/access.log`  
**Erreurs :**  
`awk '$9 == 404 {print $0}' /var/log/apache2/access.log`  
**Adresses IP les plus fréquentes :**  
`awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -nr`

## 7. Résultat des commandes
**Requêtes réussies :**  
```bash
192.168.1.20 - - [16/Dec/2024:11:05:02 +0100] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0"
192.168.1.20 - - [16/Dec/2024:11:05:03 +0100] "GET /icons/openlogo-75.png HTTP/1.1" 200 6040 "http://192.168.1.36/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0"
192.168.1.37 - - [16/Dec/2024:11:18:34 +0100] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:129.0) Gecko/20100101 Firefox/129.0"
192.168.1.37 - - [16/Dec/2024:11:18:34 +0100] "GET /icons/openlogo-75.png HTTP/1.1" 200 6040 "http://192.168.1.36/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:129.0) Gecko/20100101 Firefox/129.0"
192.168.1.20 - - [16/Dec/2024:14:25:05 +0100] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0"
127.0.0.1 - - [16/Dec/2024:14:26:26 +0100] "GET / HTTP/1.1" 200 10956 "-" "curl/7.88.1"
```
**Erreurs :**  
```bash
192.168.1.20 - - [16/Dec/2024:11:05:03 +0100] "GET /favicon.ico HTTP/1.1" 404 490 "http://192.168.1.36/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0"
192.168.1.37 - - [16/Dec/2024:11:18:34 +0100] "GET /favicon.ico HTTP/1.1" 404 490 "http://192.168.1.36/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:129.0) Gecko/20100101 Firefox/129.0"
127.0.0.1 - - [16/Dec/2024:14:26:42 +0100] "GET /nonexistentpage HTTP/1.1" 404 432 "-" "curl/7.88.1"
192.168.1.20 - - [16/Dec/2024:14:27:06 +0100] "GET /nonexistentpage HTTP/1.1" 404 491 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0"
192.168.1.20 - - [16/Dec/2024:16:15:02 +0100] "GET /favicon.ico HTTP/1.1" 404 491 "http://192.168.1.36/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0"
```
**Adresses IP les plus fréquentes :** 
```bash
     10 192.168.1.20
      3 192.168.1.37
      2 127.0.0.1
```
