# Debian 8.6 (Jessie)
# Installer un serveur sécurisé LAMP (Linux Apache Mysql PHP)

<p><em>Dernière modification : 18/11/2016</em></p>

<pre>Avant de commancer
Les commandes à taper dans le terminal commencent par : '=>'
Un commentaire commence toujours par : '#'
</pre>

<pre>
Installation standard

Lors de l'installation il vous sera demandé :
* Un mot de passe pour le compte root
* Création d'un compte utilisateur
</pre>

<pre>
Ici, le nom de la machine est 'testing.labo.local'
L'adresse IP est '192.168.0.14'
</pre>

<pre>
Une fois la machine démarré, pour passer en mode terminal, pressez les touches (en simultanée)
Ctrl + Alt + F1
</pre>

Quel est le nom de la machine ?
```sh
=> hostname
```

Quel est mon adresse IP ?
```sh
=> ip a
```

Comment modifier le mot de passe ?
```sh
=> passwd nom_du_compte
```

Passer en mode 'root'
```sh
=> su
```

Comment modifier le nom de la machine ?
- modifier le fichier /etc/hostname
- modifier le fichier /etc/hosts
Par exemple, dans notre cas :
```sh
          127.0.0.1       localhost
          192.168.0.14    testing.labo.local      testing
```
```sh
=> invoke-rc.d hostname.sh start
=> invoke-rc.d networking force-reload
=> invoke-rc.d network-manager force-reload
```

Quelle version de Debian est utilisée ?
```sh
=> lsb_release -da
=> uname -a
```

Comment paramétrer la carte réseau en manuel ?
```sh
=> auto eth0
=> iface eth0 inet static
```
```sh
        address 192.0.2.7
        netmask 255.255.255.0
        gateway 192.0.2.254
```

Comment configurer les DNS ?
Modifier le fichier /etc/resolv.conf
Exemple de contenu :
```sh
nameserver 8.8.8.8 # DNS de Google
nameserver 8.8.4.4 # DNS de Google
```

Comment paramétrer la carte réseau en DHCP ?
```sh
=> auto eth0
=> allow-hotplug eth0
=> iface eth0 inet dhcp
```
Pour nos tests, l'adresse du site internet sera : www.debian.testing.local
Modifier le fichier /etc/hosts comme suits, nul besoins d'un serveur 'bind' :
```sh
          127.0.0.1       localhost
          192.168.0.14    testing.labo.local      testing
          192.168.0.14    www.debian.testing.local # Notre site !!!
```
Quelles ports sont actuellement ouverts ?
```sh
=> netstat -plnt
```

Par défaut le pare-feu est désactivé !!!!

Configurer le pare-feu très simplement à l'aide de UFW (Uncomplicated Firewall)
```sh
=> apt-get update
=> apt-get install ufw # on installe le framework ufw pour pouvoir administrer facilement le pare-feu
=> ufw enable # on active le pare-feu
=> ufw default deny incoming # on bloque tout en entrée (déjà activé par défaut)
=> ufw default allow outgoing # on autorise tout en sortie (déjà activé par défaut)
=> ufw status verbose # Vérifier l'état du pare-feu
=> ufw allow ssh # On ouvre le port SSH en entrée pour pouvoir administrer la machine à distance
=> ufw allow http/tcp # On ouvre le port 80 pour accéder à nos pages web à distance (http)
=> ufw allow https/tcp # On ouvre le port 443 pour accéder à nos pages web à distance (https)
=> ufw status verbose # Vérifier à nouveau l'état du pare-feu
```

### Installer Mysql
```sh
=> apt-get update
=> apt-get -y install mariadb-server mariadb-client # Lors de l'installation un mot de passe vous sera demandé pour configurer l'accès root de Mysql
```

Tester l'accès de Mysql
```sh
=> mysql -u root -p # Entrer le mot de passe que vous avez paramétré précédemment
```

### Installer Apache2
```sh
=> apt-get update
=> apt-get install apache2
=> apache2 -l # liste les modules disponibles pour Apache
=> a2dismod mpm_event # désactive le module mpm_event
=> a2enmod mpm_prefork # active le module mpm_prefork
=> service apache2 restart
```

Tester depuis un poste : http://IP
On doit tomber sur la page : Debian Logo Apache2 Debian Default Page

A SAVOIR : APACHE2 sera démarré automatiquement à chaque redémarrage, pratique !!!

### Installer Lynx (un navigateur pour terminal, très pratique)
apt-get install lynx
Tester à nouveau votre page dans le terminal!!!
```sh
=> lynx http://IP
```

Quitter lynx à l'aide de la touche 'q' (2x)

### Créer le site www.debian.testing.local (version HTML a des fins de test)
```sh
=> cd /etc/apache2/sites-available
=> cp 000-default.conf www.debian.testing.local.conf # ATTENTION NE PAS OUBLIER LE .CONF!!!
=> vi www.debian.testing.local.conf
```
Modifier le contenu comme suit:
```sh
        ServerName www.debian.testing.local
        ServerAdmin webmaster@labo.local
        DocumentRoot /var/www/www.debian.testing.local
```
```sh
=> mkdir -p /var/www/www.debian.testing.local
=> cd /var/www/www.debian.testing.local
=> touch index.html
=> echo "This site rocks!" >> index.html
=> a2ensite www.debian.testing.local
=> service apache2 reload
```
Tester !!!
```sh
=> lynx http://www.debian.testing.local
```
Vous devriez voir le message 'This site rocks!'

### Installer et configurer PHP 5
```sh
=> apt-get install php5 libapache2-mod-php5
=> apt-get install php5-mysqlnd php5-curl php5-gd php5-intl php-pear php5-imagick
=> apt-get install php5-mcrypt php5-memcache php5-pspell php5-xmlrpc php5-xsl php5-apcu
=> service apache2 restart
```
Tester PHP en créant un fichier index.php
```sh
=> cd /var/www/www.debian.testing.local
=> touch index.php
=> vi index.php
```

Exemple de contenu pour votre fichier php :
```sh
          <?php
          phpinfo();
          ?>
```

Tester PHP !!!
```sh
=> lynx http://IP/index.php
```

Mettre par défaut la page index.php au lieu de index.html
```sh
=> cd /var/www/www.debian.testing.local
=> vi index.php
```

Ajouter ceci :
```sh
          DirectoryIndex index.php
```

```sh
=> service apache2 reload
```

Tester !!!
=> lynx http://IP

<em>Enjoy!</em>

<pre>
Sources :
https://wiki.debian.org/Uncomplicated%20Firewall%20(ufw)
https://www.howtoforge.com/tutorial/install-apache-with-php-and-mysql-lamp-on-debian-jessie/
https://shiji.info/blog/2016/en/99
</pre>

