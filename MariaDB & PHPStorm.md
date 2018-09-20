## MariaDB & PHPStorm  

Dans un premier temps, installer le Driver MariaDB sur votre machine hôte.  
Pour ce faire, télécharger le Driver MariaDB à [cette adresse](https://aur.archlinux.org/packages/mariadb-jdbc/).  
Rendez-vous ensuite dans le répertoire "*/usr/share/java/mariadb-jdbc*" et copier/coller le fichier "*.jar*" téléchargé.  

---

A présent, rendez-vous dans PHPStorm puis dans "*Data Sources and Drivers*".  
Créer un nouveau Driver.  
Changer le Dialect du nouveau Driver en MySQL.

---

Dans la boxe JDBC Driver, ajouter le fichier "*.jar*" précèdement téléchargé.  

---

Une fois fait, changer la Class pour mettre "*org.mariadb.jdbc.Driver*"

---

Dans la boxe URL Templates, ajouter une Template et completer la avec le code ci-dessous:  
```
jdbc:mariadb://{host::localhost}?[:{port::3306}][/{database}?][\?<&,user={user},password={password},{:identifier}={:identifier}>]
```  
---

Vous pouvez comparer votre configuration avec [cette image](http://i.imgur.com/pJZ6XmH.png).  

---
### Correction de bugs  
Il ce peut que la connexion soit refusé. Dans ce cas, éditer le fichier "*/etc/mysql/mariadb.conf.d/50-server.cnf*".  
Chercher la ligne :  
```
bind-adress       = 127.0.0.1
```  
Remplacer l'adresse IP par :  
```
bind-adress       = 0.0.0.0
```  
Puis relancer votre serveur de BDD à l'aide de la commande :  
```
service mariadb restart
```  

---  

Pour finir, lancer MariaDB :  
```
mariadb
```  

Une fois lancé, entrer la commande suivante :  

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
```  
> Modifier "TO '*root*'" par le nom d'utilisateur de votre BDD et "IDENTIFIED BY '*root*'" par le mot de passe de l'utilisateur de votre BDD.  

Enfin, executer la commande :  
```
FLUSH PRIVILEGES;
```  

Vous devriez à présent pouvoir vous connecter à votre BDD depuis PHPStorm.
