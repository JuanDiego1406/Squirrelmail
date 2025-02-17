# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"
  config.vm.hostname = "mail"
  config.vm.network "private_network", ip: "192.168.57.10"
  config.vm.provision "shell", inline: <<-SHELL
  #instalar el postfix dentro de la maquina, he puesto como dominio ieszv.jd, subdominio mail.ieszv.jd
  
    #actualizamos la lista de paquetes
    
    apt update && apt upgrade -y

    #instalamos el dns

    apt install bind9 bind9-utils -y

    # instalamos los paquetes necesarios para el servidor de correo

    apt install -y apache2 php libapache2-mod-php dovecot-core dovecot-imapd dovecot-pop3d wget tar

        # Instalar SquirrelMail

    cp -v /vagrant/squirrel/squirrelmail-webmail-1.4.22.tar.gz /tmp/
    tar xvfz /tmp/squirrelmail-webmail-1.4.22.tar.gz -C /usr/local
    ln -s /usr/local/squirrelmail-webmail-* /var/www/mail

    # Crear carpetas de data y attach

    sudo mkdir -p /var/local/squirrelmail/data
    sudo chown www-data:www-data /var/local/squirrelmail/data
    sudo mkdir -p /var/local/squirrelmail/attach
    sudo chown www-data:www-data /var/local/squirrelmail/attach


    #copiando archivos DNS

    cp -v /vagrant/dns/db.ieszv.jd /var/lib/bind/
    cp -v /vagrant/dns/192.168.57.rev /var/lib/bind/
    cp -v /vagrant/dns/named.conf.local /etc/bind/
    cp -v /vagrant/dns/named.conf.options /etc/bind/
    cp -v /vagrant/dns/named /etc/default/

    # copiando archivos del servidor

    cp -v /vagrant/webmail/hostname /etc/
    cp -v /vagrant/webmail/hosts /etc/
    cp -v /vagrant/webmail/resolv.conf /etc/

    # copiando archivos del dovecot

    cp -v /vagrant/dovecot/10-auth.conf /etc/dovecot/conf.d/
    cp -v /vagrant/dovecot/10-mail.conf /etc/dovecot/conf.d/

    # copiar archivo apache mail conf

    cp -v /vagrant/apache/mail.conf /etc/apache2/sites-available/

    #reinicio del servicio bind9
    
    sudo systemctl enable named
    sudo systemctl restart named

    #reinicio del servicio dovecot

    sudo systemctl restart dovecot

    #reinicio del servicio apache2 y activación de sitios

    sudo a2ensite mail.conf
    sudo a2dissite 000-default
    sudo a2enmod rewrite
    sudo systemctl enable apache2
    sudo systemctl restart apache2

    # establecer idioma 

    sudo locale-gen es_ES

    SHELL

end
