**<u>M300 - LB1</u>**
======

Diese Repository behandelt die Umsetzung von der LB1.

[TOC]



## Einleitung
Diese Dokumentation wurde von Samuel Briegel im Rahmen des Moduls M300 (Plattformübergreifende Dienste in ein Netzwerk integrieren) erarbeitet und zeigt alle Schritte auf, die es braucht um die LB1-Kriterien zu erfüllen.

## Voraussetzungen
* VirtualBox 
* Vagrant
* Text-Editor (z.B. Visual Studio Code)
* Github Account

***



```# -*- mode: ruby -*-

```

## Vagrant-Befehle

Die wichtigsten Befehle:

![befehle](https://user-images.githubusercontent.com/47855918/54756122-d84d8b80-4be7-11e9-8d4b-ee62e54122f7.jpg)

# Teil1 - Toolumgebung 

Hiermit verweise ich auf das File FirstStepsReadme.

Ich habe gedacht, dass ich auch die Konfiguration Toolumgebung dokumentieren muss. Heute Morgen habe ich aber von Herrn Berger erfahren, dass ich diese nur haben muss und nicht dokumentiert sein muss. Da ich das bereits im Voraus gemacht habe, möchte ich diese Doku nicht einfach wegschmeissen und deswegen wird sie in einem separatem File abgegeben.



# Teil 2 - Iaac

## Eingerichtete Umgebung

```
+---------------------------------------------------------------+
! Notebook - Schulnetz 10.x.x.x und Privates Netz 192.168.6.1   !             
!                               								!	
!                                                               !	
!    +--------------------+          +---------------------+    !
!    ! Apache Server      !          ! DHCP Server         !    !       
!    ! Host: apache       !          ! Host: dhcp          !    !
!    ! IP: 192.168.6.7    ! 	     ! IP: 192.168.6.5     !    !
!    ! Port: 80           !          ! Port: -             !    !
!    ! Nat: 8080          !          !                     !    !
!    +--------------------+          +---------------------+    !
!                                                               !
!                                                               !
!                                                               !
!                  +---------------------+                      !
!                  ! FTP Server          !                      !
!                  ! Host: ftp           !                      !
!                  ! IP: 192.168.6.6     !                      !
!                  ! Port 3306           !                      !
!                  !             	 !                      !
!                  +---------------------+                      !                                                           
!                                                               !
!	                                                        !
+---------------------------------------------------------------
```

## Funktionsweise Testen

Um zum überprüfen ob der Webserver läuft wird: http://localhost:8080/ im Browser eingegeben und dann sollte der Apache Server auftauchen. 

Um die Database zu testen, gibt man im Browser die folgende URL ein: http://localhost:8080/adminer.php.

Um die Funktionsweise vom DHCP-Server zu testen muss man eine vm erstellen mit der gleichen Netzwerkkarte wie der DHCP-Server und dann die Einstellungen ändern unter DHCP. Danach erhählt diese VM eine IP vom erstellten DHCP-Server. 

## Mein Vagrantfile
`So habe ich mein Vagrant konfiguriert:`

-*- mode: ruby -*-

#vi: set ft=ruby :



`#Virtualbox für DHCP`
`#Im ersten Abschnitt wird das Guest-OS der VM bestimmt.`
`Vagrant.configure(2) do |config|`
  `#config.ssh.username = "admin"`
  `#config.ssh.password = "admin"`
  `config.vm.define "dhcp" do |dhcp|`
    `dhcp.vm.box = "ubuntu/xenial64"`
    `dhcp.vm.hostname = "dhcp"`
    `dhcp.vm.network "private_network", ip:"192.168.6.5"` 
    `dhcp.vm.provider "virtualbox" do |vb|`
      `vb.memory = "1024"  
    end     
    dhcp.vm.provision "shell", inline: <<-SHELL`
        `sudo apt-get update`
        `#Gruppe Myadmin erstellen`
        `sudo groupadd myadmin`
        `#User erstellen`
        `sudo useradd admin -g myadmin -m -s /bin/bash`
        `sudo useradd test -g myadmin -m -s /bin/bash`
        `#Password festlegen`
        `sudo chpasswd <<<admin:admin`
        `sudo chpasswd <<<test:test`
        `#DHCP Server installieren`
        `sudo apt-get -y install isc-dhcp-server`
	`#DHCP Config-File konfigurieren`
        `#Domainanme konfigurieren`
        `sudo sed -i 's/example.org/labor.local/g' /etc/dhcp/dhcpd.conf`
        `#DNS konfigurieren`
        `sudo sed -i 's/ns2.labor.local/8.8.8.8/g' /etc/dhcp/dhcpd.conf`
        `#DHCP Autorisierung aktiviert`
        `sudo sed -i 's/#authoritative/authoritative/g' /etc/dhcp/dhcpd.conf`
        `#DHCP Subnetz & Maske konfigurieren`
        `sudo sed -i '$asubnet 192.168.6.0 netmask 255.255.255.0 {' /etc/dhcp/dhcpd.conf`
        `#DHCP Range konfigurieren`
        `sudo sed -i '$arange 192.168.6.100 192.168.6.130;' /etc/dhcp/dhcpd.conf`
        `#DHCP Gateway konfigurieren`
        `sudo sed -i '$aoption routers 192.168.6.1;' /etc/dhcp/dhcpd.conf`
        `sudo sed -i '$a}' /etc/dhcp/dhcpd.conf`
        `#DHCP Server neustarten`
	`sudo service isc-dhcpd-server restart`
        `#Tastaturlayout anpassen`
        `sudo sed -i 's/XKBLAYOUT="us"/XKBLAYOUT="ch"/g' /etc/default/locale`
        `#Local Firewall installieren`
        `sudo apt-get -y install ufw gufw` 
        `sudo ufw allow from 10.0.2.2 to any port 22`
        `sudo ufw --force enable`
`SHELL`
 `end`

`#Virtualbox für FTP`
`#Im ersten Abschnitt wird das Guest-OS der VM bestimmt.`
  `config.vm.define "ftp" do |ftp|`
    `ftp.vm.box = "ubuntu/xenial64"`
    `ftp.vm.hostname = "ftp"`
    `ftp.vm.network "private_network", ip: "192.168.6.6"`
    `ftp.vm.network "forwarded_port", guest:3306, host:3306, auto_correct: true`
    `ftp.vm.provider "virtualbox" do |vb|`
      `vb.memory = "1024"  
    end`
    `ftp.vm.provision "shell", inline: <<-SHELL`
        `sudo apt-get update`
        `#FTP Server installieren`
        `sudo apt-get -y install pure-ftpd-common pure-ftpd`
        `#FTP Server konfigurieren`
        `sudo groupadd ftpgroup`
        `sudo useradd -g ftpgroup -d /dev/null -s /etc ftpuser`
        `sudo pure-pw useradd samgian -u ftpuser -g ftpgroup -d /home/pubftp/samgian -N 10`
        `#FTP Server neustarten`
	`#sudo service pure-ftpd-common pure-ftpd restart`
        `sudo /home/pubftp/samgian restart`
        `#Tastaturlayout anpassen`
        `sudo sed -i 's/XKBLAYOUT="us"/XKBLAYOUT="ch"/g' /etc/default/locale`
        `#Local Firewall installieren`
        `sudo apt-get -y install ufw gufw` 
        `sudo ufw allow from 10.0.2.2 to any port 22`
        `sudo ufw --force enable`
`SHELL`
 `end`        

`#Im ersten Abschnitt wird das Guest-OS der VM bestimmt.`
  `config.vm.define "apache" do |apache|`
    `config.vm.box = "ubuntu/xenial64"`
    `config.vm.hostname = "apache"`
    `config.vm.network "private_network",ip:"192.168.6.7",netmask:"255.255.255.0",default_gateway:"192.168.6.1"`
    `config.vm.network "forwarded_port", guest:80, host:8080, auto_correct: true`
    `config.vm.synced_folder ".", "/var/www/html"`
    `config.vm.provider "virtualbox" do |vb|`
      `vb.memory = "1024"`
    `end`

`#Ab hier werden Services, die auf dem Server laufen sollen, und deren Einrichtung beschrieben`

  `config.vm.provision "shell", inline: <<-SHELL`
      `sudo apt-get update`
      `#Gruppe Myadmin erstellen`
      `sudo groupadd myadmin`
      `#User erstellen`
      `sudo useradd admin -g myadmin -m -s /bin/bash`
      `sudo useradd test -g myadmin -m -s /bin/bash`
      `#Password festlegen`
      `sudo chpasswd <<<admin:admin`
      `sudo chpasswd <<<test:test`
      `sudo apt-get -y install apache2`
      `sudo service apache2 restart`
      `#Tastaturlayout anpassen`
      `sudo sed -i 's/XKBLAYOUT="us"/XKBLAYOUT="ch"/g' /etc/default/locale`
      `#Local Firewall installieren`
      `sudo apt-get -y install ufw gufw` 
      `sudo ufw allow from 10.0.2.2 to any port 22`
      `sudo ufw allow from 10.0.2.2 to any port 80`
      `sudo ufw --force enable`
`SHELL`
    `end  
end`



Sorry für die Formatierung





## Benutzer- und Rechtevergabe ist eingerichtet
Im nächsten Schritt wird eine Gruppe, inklusive Benutzer mit Passwort erstellt. Dies geht wie folgt:

```
#Gruppe Myadmin erstellen
sudo groupadd myadmin
#User erstellen
sudo useradd admin -g myadmin -m -s /bin/bash
sudo useradd test -g myadmin -m -s /bin/bash
#Password festlegen
sudo chpasswd <<<admin:admin
sudo chpasswd <<<test:test
```

```bash

```

## DHCP-Server
***
Die DHCP VM hat folgende Spezifikationen (Die Änderungen kann man im Vagrant-File vornehmen):
* IP: 192.168.6.5
* Hostname: dhcp
* RAM: 1024 MB
* VM Box: Ubuntu

```
config.vm.define "dhcp" do |dhcp|
  dhcp.vm.box = "ubuntu/xenial64"
  dhcp.vm.hostname = "dhcp"
  dhcp.vm.network "private_network", ip:"192.168.6.5" 
	dhcp.vm.provider "virtualbox" do |vb|
	  vb.memory = "1024"  
end  
```
Der erstes Schritt ist die Paketverzeichnis aktualisiert wird. 

```
sudo apt-get update
```

Im nächsten Schritt wird eine Gruppe mit inklusvie Benutzer mit Passwort erstellt. Es sieht so aus: 

```
#Gruppe Myadmin erstellen
sudo groupadd myadmin
#User erstellen
sudo useradd admin -g myadmin -m -s /bin/bash
sudo useradd test -g myadmin -m -s /bin/bash
#Password festlegen
sudo chpasswd <<<admin:admin
sudo chpasswd <<<test:test
```

Bei nächsten Schritt wird der DHCP Server installiert. Das Paket lautet: ISC-DHCP-SERVER.

```
sudo apt-get -y install isc-dhcp-server
```

Nach der Installation von DHCp-Server muss man die Konfiguration anpassen. Das Konfigurationfile vom DHCP Server befindet sich im Pfad /etc/dhcp/dhcpd.conf. Bei dem Konfigurationfile wird folgendes geändert:

* Domainname
* DNS
* DHCP Scope

Der Domainname lautet Standardmässig example.org und will neu labor.local umändern. 
```
#Domainanme konfigurieren
sudo sed -i 's/example.org/labor.local/g' /etc/dhcp/dhcpd.conf
```
Der DNS wird nun auf 8.8.8.8 (Google DNS) konfiguriert.
```
#DNS konfigurieren
sudo sed -i 's/ns2.labor.local/8.8.8.8/g' /etc/dhcp/dhcpd.conf
```
Im Bereich beim Scope hat folgende Parameter:
* Subnet --> 192.168.6.0/24
* Range --> 192.168.6.100 - 130
* Gateway --> 192.168.6.1
```
#DHCP Autorisierung aktiviert
sudo sed -i 's/#authoritative/authoritative/g' /etc/dhcp/dhcpd.conf
#DHCP Subnetz & Maske konfigurieren
sudo sed -i '$asubnet 192.168.6.0 netmask 255.255.255.0 {' /etc/dhcp/dhcpd.conf
#DHCP Range konfigurieren
sudo sed -i '$arange 192.168.6.100 192.168.6.130;' /etc/dhcp/dhcpd.conf
#DHCP Gateway konfigurieren
sudo sed -i '$aoption routers 192.168.6.1;' /etc/dhcp/dhcpd.conf
sudo sed -i '$a}' /etc/dhcp/dhcpd.conf
```
Nach der Änderung des Konfigurationsfile wird der DHCP Service neu gestartet.
```
#DHCP Server neustarten
sudo service isc-dhcp-server restart
```
Die Tastaturlayout muss man noch auf Deutsch Schweiz anpassen.
```
#Tastaturlayout anpassen
sudo sed -i 's/XKBLAYOUT="us"/XKBLAYOUT="ch"/g' /etc/default/locale

Am Ende wird noch eine lokale Firewall installiert und anschliessend auch konfiguriert. Dabei öffnen man den Port 22 um via SSH darauf zuzugreifen. INFO-INPUT SSH Port 22 | TELNET Port 23

​```
#Local Firewall installieren
sudo apt-get -y install ufw gufw 
sudo ufw allow from 10.0.2.2 to any port 22
sudo ufw --force enable    
​```

Nun ist die DHCP-Part abgeschlossen. Man kann jetzt Clients VM erstellen und mit dem DHCP-Server innerhalb IP-Range IP-Adresse bekommen.
```


FTP-Server
----------
Die FTP VM hat folgende Spezifikationen (Die Änderungen kann man im Vagrant-File vornehmen):
* IP: 192.168.6.6
* Hostname: ftp
* RAM: 1024 MB
* VM Box: Ubuntu

```
config.vm.define "ftp" do |ftp|
  ftp.vm.box = "ubuntu/xenial64"
  ftp.vm.hostname = "ftp"
  ftp.vm.network "private_network", ip: "192.168.6.6"
  ftp.vm.network "forwarded_port", guest:3306, host:3306, auto_correct: true
  ftp.vm.provider "virtualbox" do |vb|
	 vb.memory = "1024"  
end
```
Der erstes Schritt ist erneut die Paketverzeichnis zu akualisieren. 

```
sudo apt-get update
```
Bei nächsten Schritt wird der FTP Server installiert. Das Paket lautet: pure-ftpd
```
#FTP Server installieren
sudo apt-get -y install pure-ftpd-common pure-ftpd
```
Nach der Installation kann ich die FTP-Server konfigurieren. Bei meinem Fall sieht es so aus: 
```
sudo groupadd ftpgroup
sudo useradd -g ftpgroup -d /dev/null -s /etc ftpuser
sudo pure-pw useradd samgian -u ftpuser -g ftpgroup -d /home/pubftp/samgian -N 10
```
Nach der Änderung wird der FTP Service neu gestartet.
```
#FTP Server neustarten
sudo service pure-ftpd-common pure-ftpd restart
#sudo /home/pubftp/samgian restart
```
Die Tastaturlayout muss man noch auf Deutsch Schweiz anpassen.
```
#Tastaturlayout anpassen
sudo sed -i 's/XKBLAYOUT="us"/XKBLAYOUT="ch"/g' /etc/default/locale
```
Am Ende wird noch eine lokale Firewall installiert und anschliessend auch konfiguriert. Dabei öffnen man den Port 22 um via SSH darauf zuzugreifen. INFO-INPUT SSH Port 22 | TELNET Port 23
```
#Local Firewall installieren
sudo apt-get -y install ufw gufw 
sudo ufw allow from 10.0.2.2 to any port 22
sudo ufw --force enable    
```
Nun ist die FTP-Part abgeschlossen.

Apache-Server
----------
Die Apache VM hat folgende Spezifikationen (Die Änderungen kann man im Vagrant-File vornehmen):
* IP: 192.168.6.7
* Hostname: apache
* RAM: 1024 MB
* VM Box: Ubuntu

```
config.vm.define "apache" do |apache|
  config.vm.box = "ubuntu/xenial64"
  config.vm.hostname = "apache"
  config.vm.network "private_network",ip:"192.168.6.7",netmask:"255.255.255.0",default_gateway:"192.168.6.1"
  config.vm.network "forwarded_port", guest:80, host:8080, auto_correct: true
  config.vm.synced_folder ".", "/var/www/html"
  config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
end
```
Der erstes Schritt ist ebenfalls erneut die Paketverzeichnis zu aktualisieren. 

```
sudo apt-get update
```

Bei nächsten Schritt wird der Apache Server installiert und die Service wird neugestartet. Das Paket lautet: apache2

```
#Installation Apache2
sudo apt-get -y install apache2
#Apache2 Service neustarten
sudo service apache2 restart
```
Das Tastaturlayout wird noch auf Deutsch Schweiz angepasst:
```
#Tastaturlayout anpassen
sudo sed -i 's/XKBLAYOUT="us"/XKBLAYOUT="ch"/g' /etc/default/locale
```

Hier definieren wir noch die Firewall

```
#Local Firewall installieren
sudo apt-get -y install ufw gufw 
sudo ufw allow from 10.0.2.2 to any port 22
sudo ufw allow from 10.0.2.2 to any port 80
sudo ufw --force enable    
```

Nun ist auch die Apache-Part abgeschlossen.

## Zusatz Reverse-Proxy einrichten

Leider hat mir die Zeit nicht gereicht den Proxy zu testen. aber so könne man so etwas realisieren. Ich wollte nicht riskieren, dass das VagrantFile nicht funktioniert und habe das nicht ins Vagrantfile übertragen

sudo ufw allow from 10.0.2.2 to any port 22
sudo ufw allow 80/tcp
	sudo ufw --force enable
	sudo apt-get install libapache2-mod-proxy-html -y
	sudo apt-get install libxml2-dev -y
	a2enmod proxy
	a2enmod proxy_html
	a2enmod proxy_http
	sed -i '$aServerName localhost' /etc/apache2/apache2.conf
	service apache2 restart
	cd 	/etc/apache2/sites-enabled
	wget https://pastebin.com/raw/GbjFC2ii
	cp GbjFC2ii 001-reverseproxy.conf



## User und Passwort

User1: test

Password: test

User2: admin

Password: admin

## Funktionsweise Testen

Um zum überprüfen ob der Webserver läuft wird: <http://localhost:8080/> im Browser eingegeben und dann sollte der Apache Server auftauchen.

Um die Database zu testen, gibt man im Browser die folgende URL ein: <http://localhost:8080/adminer.php>.

Um die Funktionsweise vom DHCP-Server zu testen muss man eine vm  erstellen mit der gleichen Netzwerkkarte wie der DHCP-Server und dann  die Einstellungen ändern unter DHCP. Danach erhählt diese VM eine IP vom  erstellten DHCP-Server.


## Was habe ich gelernt
Ich habe gelernt wie man mit einem Vagrant File eine VM erstellt nach seinen Bedürfnissen. Ausserdem habe ich mir sehr viel Fachwissen bezüglich Git, Marktdown , Linux und Virtualisierung angeeignet. Am Anfang hatte ich nur Basis Wissen in den meisten Bereichen. Durch dieses Modul konnte ich mein Wissen sehr vertiefen. 

## Mein Samba Anlauf

Zuerst wollte ich Samba als Service realisieren. Leider hatte ich sehr viele Probleme und habe so viel Zeit verloren, dass ich einen neuen Service nahm. Der Server war Pingbar, Der Ortner auf dem Server wurde mit chmod 777 bearbeitet(alle volle rechte) und der Netzwerkadapter wurde gebridged. Zur Sicherheit habe ich dann auch noch global-user gemacht welcher ohne rechte zugreifen soll. Funktionierte immer noch nicht. Ich habe viel nach meinem Problem im Internet gesucht und nach verschiedenen Anleitungen Samba installiert und konfiguriert. Hat alles nichts gebracht.

## Reflexion
Ich hatte noch nie etwas von Vagrant gehört. Deswegen hatte ich am Anfang noch etwas Mühe. Spannend finde ich, dass man eine VM erstellen kann mit einem Text-File nach seinen Bedürfnissen? Das ist sehr praktisch und zeitsparend. Neu war auch für mich den Funktionsumfang von GitHub in Kombination mit Markdown den ich nicht kannte. Da für mich alles sehr neu war, musste ich mich in einer ersten Phase erst einmal in die einzelnen Bereiche einarbeiten und Schritt für Schritt die Anweisungen befolgen. Am Schluss hatte ich ein bisschen Zeitdruck konnte jedoch alles noch gut erledigen. 

In Zukunft werde ich Github auf jeden Fall für nachfolgende Projekte brauchen. Hiermit möchte ich mich auch bei unserem Lehrer Herr Berger bedanken, ich habe viel gelernt. Der Unterricht war stets spannend und machte spass. 
