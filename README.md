# Arbeit für das Fach V-BA
Diese Arbeit wurde erstellt von:
- Giuseppe Sonetto
- Javier Lopez
- Mattia Gallelli

## Inhaltsverzeichnis
- [Einleitung](#einleitung)
- [Installation 1. VM (Cacti DB-Server)](#installation-1-vm-cacti-db-server)
- [Konfiguration von DB Server](#konfiguration-von-db-server)
- [Installation 2. VM (Apache Webserver)](#installation-2-vm-apache-webserver)
---
## Einleitung
Mit unserer Arbeit wollen wir aufzeigen, wie die Automatisierung mittels Vagrant und Docker funktioniert. In dieser Arbeit, werden zwei VM's erstellt. Die erste VM dient für die Installation von der Cacti Datenbank. Die zweite VM installiert und konfiguriert den Apache Webserver.

---
## Installation 1. VM (Cacti DB Server)
> [&uarr; *Zum Inhaltsverzeichnis*](#inhaltsverzeichnis)

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
# Definition der Vagrant Box
Vagrant.configure("2") do |config|
# Definition des Box Namens
config.vm.define "db" do |db|
# Definition der zur Verfügung gestellten Box
  db.vm.box = "ubuntu/xenial64"
# Definiton der Hypervisors
  db.vm.provider "virtualbox" do |vb|
# Definition des RAM's auf 1GB
  vb.memory = "1024"
  end
# Definiert die statische IP Adresse
  db.vm.network "public_network", ip: "192.168.0.231", host: 4567, guest: 80, auto_correct: true
# Führt nach der VM Installation das unten definierte Shellscript aus
  db.vm.provision "shell", inline: <<-SHELL
  end
  db.vm.box = "ubuntu/xenial64"
end
```
---
## Konfiguration von DB Server
> [&uarr; *Zum Inhaltsverzeichnis*](#inhaltsverzeichnis)
```
# Aktualisiert alle neuen Repositories
apt-get update -y

# Installieren von Cacti-Datenbank #
dpkg-reconfigure tzdata

# NTP Date Zeitzone Config # 
apt-get update -y
apt-get install ntpdate -y
ntpdate pool.ntp.ch
timedatectl set-ntp 0
apt-get install ntp -y
date
			
		
# Installation des Datenbankdienstes #
apt-get update -y
export DEBIAN_FRONTEND="noninteractive"
debconf-set-selections <<< "mysql-server mysql-server/root_password password root"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password root"
apt-get install mysql-server -y
apt-get install mysql-client -y

# MySQLd Konfiguration #
cat << 'EOT' >> /etc/mysql/mysql.conf.d/mysqld.cnf
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
join_buffer_size = 31M
innodb_buffer_pool_size = 240M
innodb_doublewrite = OFF
innodb_flush_log_at_timeout = 3
innodb_read_io_threads = 32
innodb_write_io_threads = 16
innodb_buffer_pool_instances = 3
[client]
user=root
password="root"
EOT
	
# Datenbank und Benutzer Erstellung in MySQL und cacti Download #
service mysql restart
mysql -u root -Bse "CREATE DATABASE cacti;CREATE USER 'cactiuser'@'%' IDENTIFIED BY 'cacti';GRANT ALL PRIVILEGES ON cacti.* TO 'cactiuser'@'%';"
mkdir /downloads
cd /downloads
wget https://www.cacti.net/downloads/cacti-1.2.3.tar.gz

# Cacti-Datenbankvorlage in MySQL importieren #
sed -i 's/user=root/user=cactiuser/g' /etc/mysql/mysql.conf.d/mysqld.cnf
sed -i 's/password="root"/password="cacti"/g' /etc/mysql/mysql.conf.d/mysqld.cnf
tar -zxvf cacti-1.2.3.tar.gz
cd cacti-1.2.3
mysql -u cactiuser cacti < cacti.sql

# Importieren Sie die MySQL-Datenbankkonfiguration mithilfe des MySQL-Root-Kontos #
sed -i 's/user=cactiuser/user=root/g' /etc/mysql/mysql.conf.d/mysqld.cnf
sed -i 's/password="cacti"/password="root"/g' /etc/mysql/mysql.conf.d/mysqld.cnf
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql

# Gewähren Sie in MySQL Cacti Zugriff auf die TimeZone-Datenbank #
mysql -u root -Bse "GRANT SELECT ON mysql.time_zone_name TO cactiuser@'%';"


# Löscht das root Passwort aus der Datei #
sed -n -e :a -e '1,3!{P;N;D;};N;ba' /etc/mysql/mysql.conf.d/mysqld.cnf

# Verbindung um auf cacti DB #
cat << 'EOT' >> /etc/mysql/my.cnf
[mysqld]
bind-address=192.168.0.231
EOT

# MySQL neustarten #
/etc/init.d/mysql restart

# Port öffnen für DB Zugriff #
/sbin/iptables -A INPUT -i eth0 -p tcp --destination-port 3306 -j ACCEPT
		
SHELL
	
end

```
---
## Installation 2. VM (Apache Webserver)
Hier geben Wir die Konfiguration für den Apache Webserver mit.
> [&uarr; *Zum Inhaltsverzeichnis*](#inhaltsverzeichnis)
```

config.vm.define "web" do |web|

web.vm.box = "ubuntu/xenial64"
web.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1", auto_correct: true
web.vm.network "public_network", ip: "192.168.0.232"
  
web.vm.provider "virtualbox" do |vb|
vb.gui = true
vb.memory = "1024"
end

web.vm.provision "shell", inline: <<-SHELL
apt-get update -y

#Apache Webserver Installation #     
apt-get install apache2 php libapache2-mod-php php-cli php-snmp -y
apt-get install php-mysql php-mbstring php-gd php-xml -y
apt-get install php-ldap php-gmp php-intl php-recode php-gettext -y
apt-get install php-pear php-pspell php-memcache -y


# Lokalisierung der php.ini Datei und in Variable speichern # 
updatedb


# Bearbeitung der php.ini Datei (mit sed noch anpassen) # 
phpini=$(locate php.ini | grep -e "^/etc.*apache2")
sed -i 's/max_execution_time\ =\ 30/max_execution_time\ =\ 300/g' $phpini
sed -i 's/memory_limit\ =\ 128M/memory_limit\ =\ 500M/g' $phpini
sed -i 's/post_max_size\ =\ 8M/post_max_size\ =\ 32M/g' $phpini
sed -i 's/max_input_time\ =\ 60/max_input_time\ =\ 300/g' $phpini
sed -i 's/;date.timezone\ =/date.timezone\ = Europe\/Zurich/g' $phpini
sed -i 's/register_argc_argv\ =\ Off/register_argc_argv\ =\ On/g' $phpini


# Bearbeitung der php.ini Datei (mit sed noch anpassen) # 
phpini=$(locate php.ini | grep -e "^/etc.*cli")
sed -i 's/;date.timezone\ =/date.timezone\ = Europe\/Zurich/g' $phpini


# Apache Dienst neustarten # 
service apache2 restart


# Cacti Installation # 
apt-get update -y
apt-get install snmp snmpd rrdtool libmysql++-dev libsnmp-dev help2man -y
apt-get install dos2unix autoconf dh-autoreconf libssl-dev librrds-perl -y
apt-get install snmp-mibs-downloader -y


# Spine Installation # 
mkdir /downloads
cd /downloads
wget https://www.cacti.net/downloads/spine/cacti-spine-1.2.3.tar.gz
tar -zxvf cacti-spine-1.2.3.tar.gz
cd cacti-spine-1.2.3
mkdir m4
./bootstrap
./configure
make
make install
chown root:root /usr/local/spine/bin/spine
chmod +s /usr/local/spine/bin/spine


# Bearbeitung Konfigurationsdatei # 
cp /usr/local/spine/etc/spine.conf.dist /usr/local/spine/etc/spine.conf
sed -i 's/DB_Host       localhost/DB_Host       192.168.0.231/g' /usr/local/spine/etc/spine.conf
sed -i 's/DB_Pass       cactiuser/DB_Pass       cacti/g' /usr/local/spine/etc/spine.conf


# Verschiebung des Cacti Ordner in Apache # 
cd /
mv /downloads/cacti-spine-1.2.3 /var/www/html/cacti
mkdir /var/www/html/cacti/log
touch /var/www/html/cacti/log/cacti.log
touch /var/www/html/cacti/log/cacti_stderr.log
chown www-data.www-data /var/www/html/cacti -R


# Cacti Konfigurationsdatei bearbeitung # 
mkdir /var/www/html/cacti/include
confphp="/var/www/html/cacti/include/config.php"
touch $confphp
cat << 'EOT' > $confphp
$database_default  = 'cacti';
$database_hostname = '192.168.0.231';
$database_username = 'cactiuser';
$database_password = 'cacti';
$database_port     = '3306';
$database_retries  = 5;
$database_ssl      = false;
$database_ssl_key  = '';
$database_ssl_cert = '';
$database_ssl_ca   = '';
EOT

# PHP Datei für Verbindung zur Datenbank # 
dbphp='var/www/html/db.php'
cat << 'EOT' > $dbphp
<?php
$servername = "192.168.0.231";
$username = "cactiuser";
$password = "cacti";
$database = "cacti";
$conn = mysqli_connect($servername, $username, $password, $database);
if (mysqli_connect_errno()) {
    die("Connection failed");
}
echo "<h1>Tabellen</h1>
<table>";
$result = mysqli_query($conn, "SHOW TABLES"); // Fragt $
while ($row = mysqli_fetch_array($result,MYSQLI_NUM))
{
echo "</tr>
<td>$row[0]</td>
</tr>";
}
echo "</table>";

?>
EOT

SHELL
   
end

end
```
---

