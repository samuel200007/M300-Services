# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!

#Virtualbox für DHCP
#Im ersten Abschnitt wird das Guest-OS der VM bestimmt.
Vagrant.configure(2) do |config|
  #config.ssh.username = "admin"
  #config.ssh.password = "admin"
  config.vm.define "dhcp" do |dhcp|
    dhcp.vm.box = "ubuntu/xenial64"
    dhcp.vm.hostname = "dhcp"
    dhcp.vm.network "private_network", ip:"192.168.6.5" 
    dhcp.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"  
    end     
    dhcp.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update
        #Gruppe Myadmin erstellen
        sudo groupadd myadmin
        #User erstellen
        sudo useradd admin -g myadmin -m -s /bin/bash
        sudo useradd test -g myadmin -m -s /bin/bash
        #Password festlegen
        sudo chpasswd <<<admin:admin
        sudo chpasswd <<<test:test
        #DHCP Server installieren
        sudo apt-get -y install isc-dhcp-server
	#DHCP Config-File konfigurieren
        #Domainanme konfigurieren
        sudo sed -i 's/example.org/labor.local/g' /etc/dhcp/dhcpd.conf
        #DNS konfigurieren
        sudo sed -i 's/ns2.labor.local/8.8.8.8/g' /etc/dhcp/dhcpd.conf
        #DHCP Autorisierung aktiviert
        sudo sed -i 's/#authoritative/authoritative/g' /etc/dhcp/dhcpd.conf
        #DHCP Subnetz & Maske konfigurieren
        sudo sed -i '$asubnet 192.168.6.0 netmask 255.255.255.0 {' /etc/dhcp/dhcpd.conf
        #DHCP Range konfigurieren
        sudo sed -i '$arange 192.168.6.100 192.168.6.130;' /etc/dhcp/dhcpd.conf
        #DHCP Gateway konfigurieren
        sudo sed -i '$aoption routers 192.168.6.1;' /etc/dhcp/dhcpd.conf
        sudo sed -i '$a}' /etc/dhcp/dhcpd.conf
        #DHCP Server neustarten
	sudo service isc-dhcpd-server restart
        #Tastaturlayout anpassen
        sudo sed -i 's/XKBLAYOUT="us"/XKBLAYOUT="ch"/g' /etc/default/locale
        #Local Firewall installieren
        sudo apt-get -y install ufw gufw 
        sudo ufw allow from 10.0.2.2 to any port 22
        sudo ufw --force enable
SHELL
 end

#Virtualbox für FTP
#Im ersten Abschnitt wird das Guest-OS der VM bestimmt.
  config.vm.define "ftp" do |ftp|
    ftp.vm.box = "ubuntu/xenial64"
    ftp.vm.hostname = "ftp"
    ftp.vm.network "private_network", ip: "192.168.6.6"
    ftp.vm.network "forwarded_port", guest:3306, host:3306, auto_correct: true
    ftp.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"  
    end
    ftp.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update
        #FTP Server installieren
        sudo apt-get -y install pure-ftpd-common pure-ftpd
        #FTP Server konfigurieren
        sudo groupadd ftpgroup
        sudo useradd -g ftpgroup -d /dev/null -s /etc ftpuser
        sudo pure-pw useradd samgian -u ftpuser -g ftpgroup -d /home/pubftp/samgian -N 10
        #FTP Server neustarten
	#sudo service pure-ftpd-common pure-ftpd restart
        sudo /home/pubftp/samgian restart
        #Tastaturlayout anpassen
        sudo sed -i 's/XKBLAYOUT="us"/XKBLAYOUT="ch"/g' /etc/default/locale
        #Local Firewall installieren
        sudo apt-get -y install ufw gufw 
        sudo ufw allow from 10.0.2.2 to any port 22
        sudo ufw --force enable
SHELL
 end        

#Im ersten Abschnitt wird das Guest-OS der VM bestimmt.
  config.vm.define "apache" do |apache|
    config.vm.box = "ubuntu/xenial64"
    config.vm.hostname = "apache"
    config.vm.network "private_network",ip:"192.168.6.7",netmask:"255.255.255.0",default_gateway:"192.168.6.1"
    config.vm.network "forwarded_port", guest:80, host:8080, auto_correct: true
    config.vm.synced_folder ".", "/var/www/html"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end
# Ab hier werden Services, die auf dem Server laufen sollen, und deren Einrichtung beschrieben
  config.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      #Gruppe Myadmin erstellen
      sudo groupadd myadmin
      #User erstellen
      sudo useradd admin -g myadmin -m -s /bin/bash
      sudo useradd test -g myadmin -m -s /bin/bash
      #Password festlegen
      sudo chpasswd <<<admin:admin
      sudo chpasswd <<<test:test
      sudo apt-get -y install apache2
      sudo service apache2 restart
      #Tastaturlayout anpassen
      sudo sed -i 's/XKBLAYOUT="us"/XKBLAYOUT="ch"/g' /etc/default/locale
      #Local Firewall installieren
      sudo apt-get -y install ufw gufw 
      sudo ufw allow from 10.0.2.2 to any port 22
      sudo ufw allow from 10.0.2.2 to any port 80
      sudo ufw --force enable
SHELL
    end  
end
