# TP2 Cybersécurité

- Nom: Vincent Jalbert
- Matricule: 6291397
- Groupe: 1040

## Attaque (exploit)

Voici les étapes détaillées pour pouvoir modifier le courriel / téléphone sur la page demandée:

### Étape 1
Vérifier la connexion réseau :

![IPCONFIG](./assets/ipconfig.png)

La commande ipconfig permet de vérifier que la machine (Client Windows) est bien connectée sur le réseau 192.168.0.0

Réponse : oui, car passerelle est 192.168.0.1 et adresse est 192.168.0.129.

### Étape 2
Ping du serveur visé :

![PING](./assets/ping.png)

On ping le serveur (192.168.0.15) afin de vérifier s'il est joignable sur le réseau pour la suite de l'attaque (à distance).

Réponse : oui, paquets reçus : 4 (envoyés : 4)

### Étape 3
Scan des ports (surtout ssh et http) avec nmap :

![NMAP](./assets/nmap.png)

La commande nmap avec l'ip du serveur permet de vérifier quels ports sont accessibles

Réponse : 22 (ssh) et 80 (http)

### Étape 4
Vérifier le site web :

Navigateur --> http://192.168.0.15 --> site web --> page contact (Coordonnées)

![SITE](./assets/site.png)

Cette étape permet de trouver la page visée que l'on souhaite modifier les données et connaître le nom du fichier exact

Réponse : page trouvée : contact.html (dans l'URL)

### Étape 5
Vérifier les données de la page Contact :

![CONTACT](./assets/contact.png)

Avec la commande curl.exe et l'URL vers la page contact, on vérifie que les données visées par l'attaque soient bien présentes

Réponse : oui, téléphone 555-555-1234 & mail ref to roger@smith.com

### Étape 6
Connexion par ssh au serveur :

Maintenant, on sait que le port 22 (SSH) est ouvert sur le serveur et que l'on devrait donc pouvoir se connecter par ssh via notre machine Windows, cependant, on doit trouver le nom d'utilisateur ainsi que le mot de passe de celui-ci.

Informations :

On sait que la personne qui a développé le site web s'appelle Bob Smith : donc le nom d'utilisateur pourrait être bob ou bobsmith

On sait qu'un de ses mdp est Kim2012! qui vient du nom d'un de ses enfants et leur naissance, proposant l'idée donc qu'il aurait peut-être une tendance à utiliser des mots de passe similaires constituant le nom de ses enfants, leur date de naissance et un point d'exclamation.

Donc, on pourrait tester avec ssh bob/bobsmith@192.168.0.15, les mots de passe suivants : Kim2012!, Luc2010! ou Sophie2014!

![SSH](./assets/ssh.png)

Réponse : après plusieurs essais-erreurs, connexion réussie avec bob@192.168.0.15 --> Sophie2014!

### Étape 7
Modification du site (page contact) :

On sait que les sites web hébergés par Apache (on sait que le site de Bob Smith est sur Apache grâce à nmap) sont habituellement dans /var/www/

Aller sur le fichier contact.html dans /var/www/html/ et trouver les lignes concernées, les modifier avec les nouvelles informations, et enregistrer.

![MODIF](./assets/modification.png)

### Étape 8
Confirmer l'attaque :

Fermer la connexion ssh et tester avec curl.exe après les modifications

![CONFIRM](./assets/confirmation.png)

Réponse : données bien changées à celles désirées (555-555-9876 & ouch.hacked@cem.ca)

### ....

## Correctif 1
Changer le mot de passe du compte bob (utilisé pour l'attaque) en un mdp plus robuste (long/complexe) et surtout qui ne comporte pas de liens envers ses enfants ou autre comme ceux d'avant

Exemple : bananebrocolipatate

Commandes utilisées :

sudo passwd bob (on connait déjà le nom d'utilisateur) --> nouveau mdp --> confirmer nouveau mdp

Ce correctif empêche l'attaquant de pouvoir se connecter par ssh au serveur, il n'a donc aucun autre moyen de modifier les fichiers du site puisqu'il ne peut plus s'y connecter.

![MDP](./assets/mdp.png)

![EXPLOIT1](./assets/exploit1.png)

Corrigé : tester connexion ssh comme avant avec Sophie2014! --> ne fonctionne plus (Permission denied)

![FONCTIONNEL](./assets/fonctionnel.png)

Image confirmant que le site fonctionne bel et bien comme prévu après le changement du mdp

## Correctif 2
Réduire permission de modification des autres (o) sur le site.

Commandes utilisées :
1. Vérifier permissions actuelles
sudo ls -l /var/www/html/contact.html
![LS](./assets/ls.png)
Les permissions sont à 777 (tout pour tous) pour la page contact, ce qui est extrêmement dangereux, il ne faut jamais faire cela. C'est en partie ce qui explique qu'on a pu modifier si facilement la page sur le site par le compte bob.

2. Changer permissions (on pourrait juste changer pour page contact.html et autres, mais on va faire le tout pour sécuriser le site entièrement tant qu'à y être)
sudo chmod 644 -R /var/www/html
sudo chmod 755 /var/www/html
![PERMISSIONS](./assets/permissions.png)

![EXPLOIT2](./assets/exploit2.png)

Ce correctif permet d'accorder les permissions optimales et sécurisées du site web afin d'éviter qu'un attaquant autre puisse porter modification à une page du site à partir du compte de bob ou autre que le compte de DaBoss (root)

Corrigé : lorsque l'on se connecte en ssh par le compte bob, on reçoit une erreur de Permission non accordée quand on tente de modifier la page contact.html du site.

![FONCTIONNEL](./assets/fonctionnel.png)

Image confirmant que le site fonctionne bel et bien comme prévu après le changement des permissions

## Correctif 3
Désactiver port 22 de SSH avec le pare-feu UFW

Objectif : désactiver le port de ssh pour enlever l'ouverture aux attaquants de s'y connecter et apporter des modifications directement sur le serveur.

Commandes utilisées :

sudo ufw default deny incoming

sudo ufw default allow outgoing

sudo ufw allow 80/tcp (il faut s'assurer d'activer le port 80 pour HTTP car sinon désactivé par défaut)

sudo ufw deny 22/tcp

sudo ufw enable

sudo ufw status numbered (vérification)

![UFW](./assets/ufw.png)

![EXPLOIT3](./assets/exploit3.png)

Corrigé : si on essaie de se connecter au / joindre le serveur via SSH (port 22), on reçoit une erreur Timed out puisque le port en question n'est plus ouvert sur le serveur du site.

![FONCTIONNEL](./assets/fonctionnel.png)

Image confirmant que le site fonctionne bel et bien comme prévu après le changement des règles de pare-feu
