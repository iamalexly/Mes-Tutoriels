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
