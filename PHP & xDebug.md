## PHP & xDebug:

Dans un premier temps, récupérer la version de votre PHP. Pour cela:
```
php -v
```
---
En fonction de votre version de PHP, executer la commande suivante:
```
apt install php7.2-xdebug
```
> **Attention**, vous devez remplacer "php7.2" par votre version PHP.
---
Rendez-vous dans le repertoire */etc/php/7.2/apache2/* et éditer le fichier *php.ini*
Rechercher le ligne:
```
display_errors = Off
```
Et remplacer *Off* par *On*

Pour finir, relancer votre serveur Apache à l'aide de la commande:
```
service apache2 restart
```
