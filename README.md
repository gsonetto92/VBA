Installation von Cacti Network Monitor unter Ubuntu Linux

 
Möchten Sie erfahren, wie Sie eine Cacti-Installation unter Ubuntu Linux ausführen? In diesem Tutorial zeigen wir Ihnen, wie Sie Cacti Dashboards auf einem Computer mit Ubuntu Linux installieren, konfigurieren und darauf zugreifen.

• Ubuntu-Version: 18.04

Was ist Cacti?
Cacti ist eine Open-Source-Plattform für die Datenüberwachung, die vollständig PHP-gesteuert ist.

Auf der Webschnittstelle können Benutzer Cacti als Frontend für RRDtool verwenden, Diagramme erstellen und sie mit in MySQL gespeicherten Daten füllen.

Cacti bietet auch SNMP-Unterstützung für Benutzer, um Diagramme für die Netzwerküberwachung zu erstellen.


 
Cacti Playlist:
Auf dieser Seite bieten wir schnellen Zugriff auf eine Liste von Videos, die sich auf die Installation von Cacti beziehen.

Playlist

Vergessen Sie nicht, unseren Youtube-Kanal mit dem Namen FKIT.


 
Cacti-Tutorial:
Auf dieser Seite bieten wir schnellen Zugriff auf eine Liste von Cacti-Tutorials

Cacti-Tutorial

Zabbix-Tutorial

Grafana-Tutorial

Prometheus-Tutorial


 
Tutorial - Installieren Sie die Cacti-Datenbank
Zuerst werden wir das System so konfigurieren, dass das korrekte Datum und die korrekte Uhrzeit unter Verwendung von NTP verwendet werden.

Verwenden Sie in der Linux-Konsole die folgenden Befehle, um die korrekte Zeitzone festzulegen.

# dpkg-reconfigure tzdata

Installieren Sie das Paket Ntpdate und stellen Sie sofort das richtige Datum und die richtige Uhrzeit ein.

# apt-get update
# apt-get install ntpdate
# ntpdate pool.ntp.br

Der Befehl Ntpdate wurde verwendet, um das korrekte Datum und die korrekte Uhrzeit unter Verwendung des Servers einzustellen: pool.ntp.br

Lassen Sie uns den NTP-Dienst installieren.

# timedatectl set-ntp 0
# apt-get install ntp

NTP ist der Dienst, der unseren Server auf dem neuesten Stand hält.

Verwenden Sie den Befehl date, um das Datum und die Uhrzeit zu überprüfen, die in Ubuntu Linux konfiguriert sind.

# date

Wenn das System Datum und Uhrzeit korrekt anzeigt, haben Sie alle Schritte korrekt ausgeführt.

Nun können wir mit der Installation des Datenbankdienstes fortfahren.

Verwenden Sie in der Linux-Konsole die folgenden Befehle, um die erforderlichen Pakete zu installieren.

# apt-get update
# apt-get install mysql-server mysql-client

Bearbeiten Sie die MySQL-Server-Konfigurationsdatei mysqld.cnf.

# vi /etc/mysql/mysql.conf.d/mysqld.cnf

Fügen Sie im Abschnitt MYSQLD die folgenden Optionen hinzu.

Copy to Clipboard
1
[mysqld]
2
​
3
character-set-client-handshake = FALSE
4
character-set-server = utf8mb4
5
collation-server = utf8mb4_unicode_ci
6
join_buffer_size = 31M
7
innodb_buffer_pool_size = 240M
8
innodb_doublewrite = OFF
9
innodb_flush_log_at_timeout = 3
10
innodb_read_io_threads = 32
11
innodb_write_io_threads = 16
12
innodb_buffer_pool_instances = 3
Starten Sie den MySQL-Dienst neu.

# service mysql restart

Verwenden Sie nach Abschluss der Installation den folgenden Befehl, um auf den MySQL-Datenbankserver zuzugreifen.

Um auf den Datenbankserver zuzugreifen, geben Sie das im MySQL Server-Installationsassistenten festgelegte Kennwort ein.

# mysql -u root -p

Verwenden Sie den folgenden SQL-Befehl, um eine Datenbank mit dem Namen Cacti zu erstellen.

CREATE DATABASE cacti;

Verwenden Sie den folgenden SQL-Befehl, um einen Datenbankbenutzer mit dem Namen Cacti zu erstellen.

CREATE USER 'cactiuser'@'%' IDENTIFIED BY 'kamisama123';

Erteilen Sie dem SQL-Benutzer mit dem Namen "cactiuser" die Berechtigung für die Datenbank mit dem Namen "cacti".

GRANT ALL PRIVILEGES ON cacti.* TO 'cactiuser'@'%';
quit;

Verwenden Sie in der Linux-Konsole die folgenden Befehle, um das Cacti-Installationspaket herunterzuladen.

# mkdir /downloads
# cd /downloads
# wget https://www.cacti.net/downloads/cacti-1.2.3.tar.gz

Nun müssen wir die Cacti-Datenbankvorlage in MySQL importieren.

Extrahieren Sie das Cacti-Installationspaket und importieren Sie die Datenbankvorlage in MySQL.

Das System fordert jedes Mal, wenn Sie versuchen, eine Datei zu importieren, das Kennwort des MySQL-Cactiuser an.

# tar -zxvf cacti-1.2.3.tar.gz
# cd cacti-1.2.3
# mysql -u cactiuser -p cacti < cacti.sql

Cacti erfordern die Konfiguration der MySQL-Zeitzonen-Datenbank.

Importieren Sie die MySQL-Datenbankkonfiguration mithilfe des MySQL-Root-Kontos.

# mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql

Greifen Sie auf den MySQL Server zu.

Gewähren Sie in MySQL Cacti Zugriff auf die TimeZone-Datenbank.

# mysql -u root -p

GRANT SELECT ON mysql.time_zone_name TO cactiuser@'%';
quit;

Sie haben die Datenbankinstallation abgeschlossen.

Sie haben die Cacti-Datenbankvorlagen auf dem MySQL-Server importiert.


 
Tutorial - Installieren Sie das Cacti Apache Frontend
Als Nächstes müssen wir den Apache-Webserver und die erforderliche Software installieren.

Verwenden Sie in der Linux-Konsole die folgenden Befehle, um die erforderlichen Pakete zu installieren.

# apt-get install apache2 php libapache2-mod-php php-cli php-snmp
# apt-get install php-mysql php-mbstring php-gd php-xml
# apt-get install php-ldap php-gmp php-intl php-recode php-gettext
# apt-get install php-pear php-pspell php-memcache

Nun sollten Sie den Speicherort der Datei php.ini auf Ihrem System finden.

Nach dem Finden müssen Sie die php.ini-Datei bearbeiten.

# updatedb
# locate php.ini

/etc/php/7.2/apache2/php.ini
/etc/php/7.2/cli/php.ini

Beachten Sie, dass Ihre PHP-Version und der Speicherort der Datei möglicherweise nicht mit mir identisch sind.

Sie benötigen beide php.ini-Dateien.

Zuerst bearbeiten wir die Datei: /etc/php/7.2/apache2/php.ini

# vi /etc/php/7.2/apache2/php.ini

Hier ist die neue Datei mit unserer Konfiguration.

max_execution_time = 300
memory_limit = 500M
post_max_size = 32M
max_input_time = 300
date.timezone = America/Sao_Paulo
register_argc_argv = On

Jetzt bearbeiten wir die Datei: /etc/php/7.2/cli/php.ini

# vi /etc/php/7.2/cli/php.ini

Hier ist die neue Datei mit unserer Konfiguration.

date.timezone = America/Sao_Paulo

Denken Sie daran, dass Sie Ihre PHP-Zeitzone einstellen müssen.

In unserem Beispiel haben wir die Zeitzone America / Sao_Paulo verwendet

Sie sollten Apache auch manuell neu starten und den Dienststatus überprüfen.

# service apache2 restart

Hier ist ein Beispiel für die Ausgabe des Apache-Dienststatus.

● apache2.service - LSB: Apache2 web server
Loaded: loaded (/etc/init.d/apache2; bad; vendor preset: enabled)
Drop-In: /lib/systemd/system/apache2.service.d
└─apache2-systemd.conf
Active: active (running) since Mon 2018-04-23 00:02:09 -03; 1min 4s ago

Tutorial - Cacti-Installation auf Ubuntu
Nun müssen wir den Cacti-Server unter Ubuntu Linux installieren.

Verwenden Sie in der Linux-Konsole die folgenden Befehle, um die erforderlichen Pakete zu installieren.

# apt-get update
# apt-get install snmp snmpd rrdtool libmysql++-dev libsnmp-dev help2man
# apt-get install dos2unix autoconf dh-autoreconf libssl-dev librrds-perl
# apt-get install snmp-mibs-downloader

Starte deinen Computer neu.

# reboot

Verwenden Sie die folgenden Befehle, um Spine herunterzuladen und zu installieren.

# cd /downloads
# wget https://www.cacti.net/downloads/spine/cacti-spine-1.2.3.tar.gz
# tar -zxvf cacti-spine-1.2.3.tar.gz
# cd cacti-spine-1.2.3
# mkdir m4
# ./bootstrap
# ./configure
# make
# make install
# chown root:root /usr/local/spine/bin/spine
# chmod +s /usr/local/spine/bin/spine

Erstellen und bearbeiten Sie die Spine-Konfigurationsdatei.

# cp /usr/local/spine/etc/spine.conf.dist /usr/local/spine/etc/spine.conf
# vi /usr/local/spine/etc/spine.conf

Hier ist die Datei spine.conf mit unserer Konfiguration.

Copy to Clipboard
1
DB_Host       localhost
2
DB_Database   cacti
3
DB_User       cactiuser
4
DB_Pass       kamisama123
5
DB_Port       3306
Verschieben Sie den Cacti-Ordner in der Linux-Konsole im Apache-Root-Ordner.

# mv /downloads/cacti-1.2.3 /var/www/html/cacti
# touch /var/www/html/cacti/log/cacti.log
# touch /var/www/html/cacti/log/cacti_stderr.log
# chown www-data.www-data /var/www/html/cacti -R

Nun müssen Sie die Cacti-Konfigurationsdatei bearbeiten.

# vi /var/www/html/cacti/include/config.php

Hier ist die neue Datei mit unserer Konfiguration.

Copy to Clipboard
1
$database_type     = 'mysql';
2
$database_default  = 'cacti';
3
$database_hostname = 'localhost';
4
$database_username = 'cactiuser';
5
$database_password = 'kamisama123';
6
$database_port     = '3306';
7
$database_retries  = 5;
8
$database_ssl      = false;
9
$database_ssl_key  = '';
10
$database_ssl_cert = '';
11
$database_ssl_ca   = '';
Cacti Web Installer
Öffnen Sie Ihren Browser und geben Sie die IP-Adresse Ihres Webservers plus / Kakteen ein.

In unserem Beispiel wurde die folgende URL in den Browser eingegeben:

• http://35.162.85.57/cacti

Die Cacti-Weboberfläche sollte vorgestellt werden.

Cacti login
Geben Sie in der Eingabeaufforderung die Anmeldeinformationen für das Cacti Default Password ein.

• Benutzername: admin
• Passwort: admin

Das System fordert Sie auf, das Standardkennwort von Cacti zu ändern.

Cacti default password
Akzeptieren Sie die Open Source-Lizenzvereinbarung für Cacti Network Monitor.

Cacti network monitor open source
Auf dem nächsten Bildschirm müssen Sie überprüfen, ob alle Anforderungen erfüllt wurden.

Cacti php requirements
Prüfen Sie, ob alle PHP-Modulanforderungen erfüllt wurden.

cacti php module requirements
Wählen Sie im nächsten Bildschirm die Option Neuer Primärserver aus.

cacti primary server
Auf dem nächsten Bildschirm überprüft Cacti, ob Probleme mit der Dateiberechtigung vorliegen.

Cacti permission issues
Führen Sie im nächsten Bildschirm die folgende Konfiguration aus:

• Spine-Konfigurationsdateipfad: /usr/local/spine/etc/spine.conf
• Cacti-Protokollpfad: /var/www/html/cacti/log/cacti.log

Cacti Critical binary locations
Deaktivieren Sie im nächsten Bildschirm den Scan-Modus und fahren Sie fort.

Cacti Default profile
Importieren Sie im nächsten Bildschirm Cacti-Vorlagen.

Cacti install templates
Mach weiter.

Cacti database format
Aktivieren Sie das Kontrollkästchen Installation bestätigen und fahren Sie fort.

Cacti Confirm installation
Die Cacti-Installation wird gestartet.

Schauen Sie sich das Cacti-Installationsprotokoll an.

Cacti installation log
Nach Abschluss der Installation wird das Cacti Dashboard angezeigt.

Cacti dashboard
Rufen Sie im Cacti-Dashboard das Konfigurationsmenü auf und wählen Sie die Option Einstellungen.

Rufen Sie die Registerkarte Poller auf und konfigurieren Sie die Option Poller-Typ von cmd.php bis Spine.

Klicken Sie auf die Schaltfläche Speichern.

Cacti spine configuration
Erstellen Sie eine geplante Task mit Cron, um die Datei poler.php alle 5 Minuten als Benutzer www-data auszuführen.

# crontab -u www-data -e

Fügen Sie Crontab die folgende Konfiguration hinzu:

*/5 * * * * /usr/bin/php /var/www/html/cacti/poller.php

Warten Sie 15 Minuten, bis der Auswahlvorgang abgeschlossen ist, um Informationen zu erhalten.

Rufen Sie das Grafikmenü auf und wählen Sie Ihren Linux-Computer aus, um die Grafiken anzuzeigen

Cacti graph
Herzliche Glückwünsche! Die Installation des Cacti-Servers wurde erfolgreich abgeschlossen.
