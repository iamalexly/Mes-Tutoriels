# Créer et sécuriser un serveur web Debian 9.5 (Stretch) sous Apache2

- Etape 01 - Créer une connexion SSH sécurisé
- Etape 02 - Créer et configurer un serveur LAMP
- Etape 03 - Configurer le certificat SSL (HTTPS) d'Apache2
- Etape 04 - Configurer les redirections de ports
- Etape 05 - Installer et configurer Fail2Ban
- Etape 06 - Configurer le Firewall

/!\ Ce tutoriel a pour but de montrer les étapes importantes d'une sécurisation (banale et minimale) d'un serveur web. Chaque partie ne sera donc pas détaillé. Si vous voulez en savoir plus, n'hésitez pas à vous référer à d'autres tutoriels, documentations... 

---

### Etape 01 - Créer une connexion SSH sécurisé

Dans un premier temps, sur la machine hôte, créer une clé SSH à l'aide de la commande :
```bash
ssh-keygen -b 4096
```
Une fois fait, éditer le fichier `/etc/ssh/sshd_config` sur le serveur. Chercher la ligne `PermitRootLogin` qui devrait contenir :
```apacheconfig
#PermitRootLogin prohibit-password
```
Décocher et remplacer cette ligne par :
```apacheconfig
PermitRootLogin yes
```
Redémarrer le service SSH :
```bash
service ssh restart
```
A présent, il va falloir envoyer la clé publique (généré dans la machine hôte) sur le serveur. Pour ce faire, executer la commande suivante sur la machine hôte :
```bash
ssh-copy-id -i ~/.ssh/id_rsa_NomDeLaCléGénéré.pub root@xxx.xxx.xxx.xxx
```
Retourner éditer le fichier `/etc/ssh/sshd_config` sur le serveur. Chercher la ligne `PermitRootLogin` qui devrait contenir :
```apacheconfig
PermitRootLogin yes
```
Remplacer cette ligne par :
```apacheconfig
PermitRootLogin prohibit-password
```
`prohibit-password` permet d'autoriser seulement les connexion SSH par clés.  
Redémarrer le service SSH :
```bash
service ssh restart
```
La connexion SSH et à présent sécurisé par une paire de clés SSH asymétriques codé sur 4096 bits. Ne pas oublier de faire une copie de la paire de clés SSH pour ne pas risquer de perdre son serveur.

---

### Etape 02 - Créer et configurer un serveur LAMP

Pour commencer, à l'heure ou j'écrit ce guide les versions des composants de mon serveur sont les suivantes :
- Apache2
- PHP 7.0.30
- MariaDB 10.01
```bash
apt install apache2 php libapache2-mod-php mariadb-server php-mysql
```
Une fois installé, nous allons par la même occasion installer les principaux modules de PHP (selon les besoins, vous pourrez être ammené à en installer d'autres) :
```bash
apt install php-curl php-gd php-intl php-json php-mbstring php-xml php-zip
```
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
Puis lui donner tous les droits (CRUD) sur la base de données précedemment créée à l'ai de la commande :
```mysql
GRANT ALL PRIVILEGES ON nomDeLaBDD.* TO 'nomDutilisateur'@'localhost' WITH GRANT OPTION;
```
Terminer l'opération avec la commande :
```mysql
FLUSH PRIVILEGES;
```
