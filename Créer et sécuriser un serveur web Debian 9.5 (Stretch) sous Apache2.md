# Créer et sécuriser un serveur web Debian 9.5 (Stretch) sous Apache2

Avant de commencer, si vous trouvez des incohérences/erreurs ou si vous avez des remarques à propos de ce guide, n'hésitez pas à m'en faire part en me contactant à l'adresse `alexlebaillypro@gmail.com` ce tutoriel a pour but de montrer les étapes importantes d'une sécurisation (banale et minimale) d'un serveur. Le but de ce cours et de montrer la marche à suivre. Chaque sujet évoqué ne sera donc pas forcement développé en long et en large. En revanche, vous pourrez trouver des liens vers des sources externes développant chaque partie à la fin de ce guide.  

### Sommaire :
- [Etape 01 - Créer une connexion SSH sécurisé]()
- [Etape 02 - Créer un serveur LAMP]()
- [Etape 03 - Configurer MariaDB]()
- [Etape 04 - Configurer le pare-feu]()
- [Etape 05 - Installer et configurer Fail2Ban]()
- [Etape 06 - Configurer le certificat SSL (HTTPS) d'Apache2]()  
- [Sources complémentaires]()

---

## Etape 01 - Créer une connexion SSH sécurisé
Dans cette première étape, nous allons voir comment créer une clé SSH codé sur 4096 bits pour établir une connexion sur un serveur via ce procole. Sachez avant toutes choses qu'il est conseillé de privilégier l'utisation d'un compte utilisateur classique plutôt que le compte `root`.  
Pour commencer, sur la machine hôte, créer une clé SSH à l'aide de la commande :
```bash
ssh-keygen -b 4096
```
Une fois fait, éditer le fichier `/etc/ssh/sshd_config` présent sur le serveur. Chercher la ligne `PermitRootLogin` qui devrait contenir :
```apacheconfig
#PermitRootLogin prohibit-password
```  
Ou bien :
```apacheconfig
PermitRootLogin yes
```  
> Cette ligne désigne les permissions de connexion au compte `root` via le protocole SSH.  

Dans les deux cas, remplacer cette ligne par :
```apacheconfig
PermitRootLogin yes
```
Redémarrer le service SSH :
```bash
service ssh restart
```
A présent, il va falloir envoyer la clé publique (généré dans la machine hôte) sur le serveur. Pour ce faire, exécuter la commande suivante sur la machine hôte :
```bash
ssh-copy-id -i ~/.ssh/id_rsa_NomDeLaCléGénéré.pub root@xxx.xxx.xxx.xxx
```
> Attention, la clé est dans ce cas uniquement envoyé sur le compte `root`. Si vous voulez envoyer la clé sur un autre compte, vous devrez réitérer l'opération en changeant le compte utilisateur `root` par un autre utilisateur. 

Retourner éditer le fichier `/etc/ssh/sshd_config` sur le serveur. Chercher la ligne `PermitRootLogin` qui devrait contenir :
```apacheconfig
PermitRootLogin yes
```
Remplacer cette ligne par :
```apacheconfig
PermitRootLogin prohibit-password
```
`prohibit-password` permet d'autoriser seulement les connexions SSH par clés. Vous pouvez aussi choisir de désactiver la connexion au compte root via SSH en inscrivant `no` à la place de `prohibit-password`  
Redémarrer le service SSH :
```bash
service ssh restart
```
La connexion SSH et à présent sécurisé par une paire de clés SSH asymétriques codé sur 4096 bits. Ne pas oublier de faire une copie de la paire de clés SSH ou d'en générer d'autres sur d'autres postes et de les envoyer sur le serveur pour ne pas risquer de le perdre.

---

## Etape 02 - Créer un serveur LAMP

Pour commencer, à l'heure où j'écrit ce guide les versions des composants de mon serveur LAMP sont les suivantes :
- Debian 9.5 (Stretch)
- Apache2
- MariaDB 10.01  
- PHP 7.0.30
  
Pour les installer, lancer la commande :
```bash
apt install apache2 php libapache2-mod-php mariadb-server php-mysql
```
Une fois installés, nous allons par la même occasion installer les principaux modules de PHP (selon les besoins, vous pourrez être amené à en installer d'autres) :
```bash
apt install php-curl php-gd php-intl php-json php-mbstring php-xml php-zip
```
Par soucis de précautions, nous allons aussi voir comment sécuriser le répertoire ou ce trouvera votre application.  
Sous Apache2, ce répertoire ce trouve généralement à l'emplacement `/var/www/html`  
Comme il est conseillé de ne pas ce connecter en `root` sur votre serveur, le but de cette méthode sera de donner des droits de lecture, d'éxecution et de création à un utilisateur classique (non super utilisateur) sur le dossier qui contiendra votre application.  
Pour ce faire, créer un nouveau groupe d'utilisateur :
```bash
addgroup nom_du_groupe
```
Ajouter l'utilisateur classique au nouveau groupe :
```bash
adduser nom_utilisateur nom_du_groupe
```
A présent, changer groupe propriétaire du dossier `html` :
```bash
chgrp nom_du_groupe /var/www/html/
```
> Il faut reboot le serveur pour que les changements soient comptabilisés.
Pour finir, le but est d'enlever les droits de lecture et d'écriture aux utilisateurs ne faisant pas partie du groupe créé précedemment et de donner tous les droits au nouveau groupe créé. Pour ce faire, éxecuter la commande :
```bash
chmod g+rwx,o-rw /var/www/html/
```
Voilà, c'est terminé (pour cette partie :p).  
Par la suite, si vous n'avez plus besoin d'effectuer d'opération sur le répertoire contenant votre application, vous pouvez rechanger le groupe propriétaire du répertoire en éxecutant la commande :
```bash
chgrp root /var/www/html/
```
L'utilisateur classique n'aura alors plus aucun droits sur le dossier.
> N'oubliez pas de reboot votre serveur pour que les changements soient pris en compte.
---

## Etape 03 - Configurer MariaDB
Par défaut, l'utilisateur `root` de MariaDB n'a aucun mot de passe. Nous allons donc voir comment configurer MariaDB et le compte `root` du serveur de base de données.  
A présent, ouvrir MariaDB en ligne de commande :
```bash
mariadb
```
Créer une nouvelle base de données :
```mysql
CREATE DATABASE nomDeLaBDD;
```
Créer un utilisateur :
```mysql
CREATE USER 'nomDutilisateur'@'localhost' IDENTIFIED BY 'mdp';
```
Puis lui donner tous les droits sur la base de données précedemment créée à l'aide de la commande :
```mysql
GRANT ALL PRIVILEGES ON nomDeLaBDD.* TO 'nomDutilisateur'@'localhost' WITH GRANT OPTION;
```
Terminer l'opération avec la commande :
```mysql
FLUSH PRIVILEGES;
```
Il est à présent temps de sécuriser le compte utilisateur `root` des bases de données.  
Ce compte a notamment accés à la base de données `mysql` qui contient la configuration du serveur mais aussi la configuration des utilisateurs des bases de données (mot de passe, privilèges...).  
Tout d'abord, ce connecter en `root` à la base de données :
```bash
mariadb -u root -p
```
A présent, se placer dans la base de données `mysql` :
```mysql
USE mysql
```
Ensuite, entrer la requête suivante :
```mysql
UPDATE user SET plugin = NULL WHERE User = 'root';
```
Cette requête a pour but de supprimer les plugins de l'utilisateur `root` qui peuvent bloquer la connexion ou la (re)définition du mot de passe du compte `root`.
Executer ensuite la requête suivante :
```mysql
SET PASSWORD FOR root@'localhost' = PASSWORD('nouveauMDP');
```
Puis appliquer les requêtes précédentes à l'aide de la commande MySQL :
```mysql
FLUSH PRIVILEGES
```
Dorénavant, pour se connecter au compte `root` MySQL il faudra utiliser la commande :
```bash
mariadb -u root -p
```
Puis entrer le mot de passe définie quelques instants plus tôt.
Votre base de données aura maintenant un utilisateur ayant tous les droits sur une base de données, et un utilisateur `root` ayant un mot de passe qui aura quand à lui des droits sur toutes les bases de données.  
> Le fait de définir un mot de passe pour le compte `root` et de lier un utilisateur à la base de donnée de votre application est une pratique à mettre en place pour plusieurs raisons. Par exemple, en utilisant cette méthode, si une personnes mal intentionné venait à extraire les identifiants de l'utilisateur classique de votre base de données, il pourrait ce connecter dessus, faire des opérations (CRUD) sur vos données mais il ne pourra par exemple pas supprimer entièrement la base de données (DROP)... 
Pensez donc bien à définir les privilèges de l'utilisateur classique correctement.

---

## Etape 04 : Configurer le pare-feu
Dans cette partie nous allons voir comment configurer le firewall de votre serveur.  
Je tiens avant toutes choses à préciser que cette configuration et très simpliste.  
Le domaine des firewall sous Linux est un sujet complexe et vaste. Si vous voulez comprendre et créer des firewall plus spécifiques, rendez-vous dans les sources complémentaires de ce guide ou des liens explicatifs sur les firewall seront présent.  
Si vous souhaitez simplement comprendre le script ci-dessous sans aller plus loin, retenez simplement que :
- `-A` permet d'ajouter une règle
- `INPUT` désigne les connexions entrantes (utilisateur vers machine)
- `OUTPUT` désigne les connexions sortantes (machine vers autre machine)
- `dport` permet d'indiquer le port sur laquelle s'appliquera la règle
- `ACCEPT` autorise la connexion
- `DROP` n'autorise pas la connexion

Dans un premier temps, installer `iptables` :
```bash
apt install iptables
```
Créer ensuite un fichier `parefeu` dans le repertoire `/etc/init.d/`.  
```bash
nano parefeu /etc/init.d/
```
A présent copier/coller le code ci-dessous dans le fichier.
```bash
#!/bin/sh

#########
# FLUSH #
#########
iptables -t filter -F

##############
# POLITIQUES #
##############
iptables -t filter -P INPUT DROP
iptables -t filter -P OUTPUT DROP
iptables -t filter -P FORWARD DROP

#######################
# CONNEXIONS ETABLIES #
#######################
iptables -A INPUT -m state --state  ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

############
# LOOPBACK #
############
iptables -t filter -A INPUT -i lo -j ACCEPT
iptables -t filter -A OUTPUT -o lo -j ACCEPT

########
# PING #
########
iptables -t filter -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 22 -j ACCEPT

#######
# SSH #
#######
iptables -t filter -A INPUT -p tcp --dport 6464 -j ACCEPT

#################
# HTTP / HTTPS #
#################
iptables -t filter -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 443 -j ACCEPT

```
Quelques explications s'imposent.  
Les lignes :
```bash
##############
# POLITIQUES #
##############
iptables -t filter -P INPUT DROP
iptables -t filter -P OUTPUT DROP
iptables -t filter -P FORWARD DROP
```
Permettent de désactiver toutes les connexions entrantes et sortantes de la machine.  
Ensuite, les lignes :
```bash
#######################
# CONNEXIONS ETABLIES #
#######################
iptables -A INPUT -m state --state  ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```
Permettent de ne pas désactiver les connexions entrantes et sortantes déjà établient.  
Les lignes :
```bash
############
# LOOPBACK #
############
iptables -t filter -A INPUT -i lo -j ACCEPT
iptables -t filter -A OUTPUT -o lo -j ACCEPT
```
Permettent quand à elles d'autoriser le loopback avec la machine (`lo` = `localhost`).  
La partie suivante :
```bash
########
# PING #
########
iptables -t filter -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 22 -j ACCEPT
```
Permet d'autoriser le IMCP (Ping).  
Et enfin, les lignes : 
```bash
#######
# SSH #
#######
iptables -t filter -A INPUT -p tcp --dport 6464 -j ACCEPT

#################
# HTTP / HTTPS #
#################
iptables -t filter -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -t filter -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 443 -j ACCEPT
```
Permettent d'accepter les connexions SSH entrantes (et non sortantes) mais aussi d'accepter les connexion HTTP et HTTPS entrantes et sortantes. En ce qui concerne les connexions sortantes pour le HTTP/HTTPS, elle ne sont utiles que si vous souhaiter executer des requêtes vers d'autres serveurs (commande `CURL`...).  
Pour finir, il faut pouvoir executer ce script à chaque démarrage du serveur. Pour ce faire :
```bash
chmod +x /etc/init.d/parefeu
```
Puis :
```bash
update-rc.d parefeu defaults
```
Ce script permet donc de n'autoriser que le `Loopback`, le `ping`, la connexion `SSH` et les connexions `HTTP` et `HTTPS`.

---

## Etape 05 - Installer et configurer Fail2Ban
> fail2ban est une application qui analyse les logs de divers services (SSH, Apache, FTP…) en cherchant des correspondances entre des motifs définis dans ses filtres et les entrées des logs. Lorsqu'une correspondance est trouvée une ou plusieurs actions sont exécutées. Typiquement, fail2ban cherche des tentatives répétées de connexions infructueuses dans les fichiers journaux et procède à un bannissement en ajoutant une règle au pare-feu iptables pour bannir l'adresse IP de la source.

(Source : [https://doc.ubuntu-fr.org/fail2ban](https://doc.ubuntu-fr.org/fail2ban))

Nous allons dans cette partie configurer `Fail2Ban` pour bloquer les tentatives de connexions répétés au service SSH.  
Pour ce faire, installer `Fail2Ban` :
```bash
apt install fail2ban
```
Une fois installé, rendez-vous dans le fichier `/etc/fail2ban/jail.d/defaults-debian.conf`, puis ajoutez le code ci-dessous :
```bash
[DEFAULT]
findtime = 3600
bantime = 18000
maxretry = 3

[sshd]
enabled = true
```
- `findtime` correspond au temps en secondes depuis lequel une anomalie est recherchée dans les logs
- `bantime` correspond au temps de banissement en seconde d'une IP
- `maxretry` correspond au nombre de tentative maximum  

> Si vous avez changer le port de `SSH`, ajoutez la ligne suivante sous `enabled = true` :
```bash
port = le_port_ssh
```
> Si vous voulez recevoir des notifications par mail lorsque Fail2Ban bloque une IP, éditer le fichier `/etc/fail2ban/jail.conf` et chercher la ligne `destemail` puis remplacer l'adresse email présente par la votre.  

Puis, redémarrer le service `Fail2Ban` pour appliquer les modifications :
```bash
systemctl restart fail2ban
```
Vous pouvez voir si le service est bien fonctionnel à l'aide de la commande :
```bash
fail2ban-client status
```
> Le bloquage est fonctionnel si `SSHD` apparait dans la `Jail List`.

Vous pouvez voir les bloquages SSH en cours avec la commande :
```bash
fail2ban-client status sshd
```
Vous pouvez aussi arrêter ou démarrer un bloquage sur un service spécifique à l'aide de la commande :
```bash
# Arrêter
fail2ban-client stop sshd
# Démarrer
fail2ban-client stop sshd
```



---

## Sources complémentaires :  

A propos des connexions SSH et des clés asymétriques :
- [Documentation SSH Debian (https://wiki.debian.org/fr/SSH)](https://wiki.debian.org/fr/SSH)
- [Authentification SSH par clés (https://www.it-connect.fr/chapitres/authentification-ssh-par-cles/)](https://www.it-connect.fr/chapitres/authentification-ssh-par-cles/)
- [Fonctionnement des clés asymétriques (https://www.it-connect.fr/les-cles-asymetriques/)](https://www.it-connect.fr/les-cles-asymetriques/)  
  
A propos des serveurs LAMP :
- [Qu'est ce qu'un serveur LAMP (http://fr.open-lamp.com/quest-ce-quun-serveur-web-lamp/)](http://fr.open-lamp.com/quest-ce-quun-serveur-web-lamp/)
- [Documentation Ubuntu sur l'installation et la configuration d'un serveur LAMP (https://doc.ubuntu-fr.org/lamp)](https://doc.ubuntu-fr.org/lamp)
- [Documentation Ubuntu sur les permissions (droits) (https://doc.ubuntu-fr.org/permissions)](https://doc.ubuntu-fr.org/permissions)

A propos de MariaDB (et MySQL) :
- [Documentation Ubuntu sur Mysql (https://doc.ubuntu-fr.org/mysql)](https://doc.ubuntu-fr.org/mysql)
- [Documentation Ubuntu sur MariaDB (https://doc.ubuntu-fr.org/mariadb)](https://doc.ubuntu-fr.org/mariadb)
- [Tutoriel sur les privilèges utilisateur (https://www.hostinger.fr/tutoriels/creer-un-utilisateur-mysql/)](https://www.hostinger.fr/tutoriels/creer-un-utilisateur-mysql/)

A propos des parefeu :
- [Sécuriser son serveur Linux (https://openclassrooms.com/fr/courses/1197906-securiser-son-serveur-linux)](https://openclassrooms.com/fr/courses/1197906-securiser-son-serveur-linux)
- [Tutoriel vidéo (FR) iptables (https://www.grafikart.fr/formations/serveur-linux/iptables)](https://www.grafikart.fr/formations/serveur-linux/iptables)
- [Guide complet iptables (https://www.inetdoc.net/guides/iptables-tutorial/)](https://www.inetdoc.net/guides/iptables-tutorial/)
- [Tutoriel iptables (https://www.malekal.com/tutoriel-iptables/)](https://www.malekal.com/tutoriel-iptables/)  

A propos de Fail2Ban :
- [Documentation Ubuntu Fail2Ban (https://doc.ubuntu-fr.org/fail2ban)](https://doc.ubuntu-fr.org/fail2ban)
- [Protéger son serveur en utilisant Fail2Ban (https://blog.nicolargo.com/2012/02/proteger-son-serveur-en-utilisant-fail2ban.html)](https://blog.nicolargo.com/2012/02/proteger-son-serveur-en-utilisant-fail2ban.html)
- [Tutoriel Développé.com Fail2Ban (https://reseau.developpez.com/tutoriels/fail2ban/)](https://reseau.developpez.com/tutoriels/fail2ban/#L3-4)
