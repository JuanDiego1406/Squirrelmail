# Instalación y Configuración de SquirrelMail en Debian 11

Este repositorio contiene una guía paso a paso para instalar y configurar **SquirrelMail**, un cliente de correo web, en un servidor con **Debian 11 (Bullseye)**.

## 📌 Requisitos previos

Antes de instalar **SquirrelMail**, asegúrate de contar con los siguientes servicios configurados en tu servidor:

- **DNS**:
  - Servidor NS
  - Servidor MX
  - Zona inversa
- **Servidor Web** (Apache)
- **Postfix** (Servidor de correo)
- **Dovecot** (IMAP/POP3)
- **PHP** y su módulo para Apache

## 🛠️ Instalación de dependencias

Ejecuta los siguientes comandos para instalar los paquetes necesarios:

```bash
sudo apt update
sudo apt install apache2 php libapache2-mod-php postfix dovecot-imapd dovecot-pop3d bind9
```

## 🌐 Configuración de Apache

1. Copia la configuración por defecto para crear un nuevo virtual host:

    ```bash
    cd /etc/apache2/sites-available
    sudo cp 000-default.conf mail.conf
    ```

2. Edita `mail.conf` y añade la siguiente configuración:

    ```apache
    <VirtualHost *:80>
        ServerName mail.ieszv.jd
        ServerAdmin webmaster@ieszv.jd
        DocumentRoot /var/www/mail

        ErrorLog ${APACHE_LOG_DIR}/mail.ieszv.jd.error.log
        CustomLog ${APACHE_LOG_DIR}/mail.ieszv.jd.access.log combined

        <Directory /var/www/mail>
            DirectoryIndex index.php
        </Directory>
    </VirtualHost>
    ```

3. Habilita el sitio y reinicia Apache:

    ```bash
    sudo a2ensite mail
    sudo a2dissite 000-default
    sudo systemctl restart apache2
    ```

## 🌐 Configuración de DNS (BIND9)

1. Agrega la configuración de las zonas en `/etc/bind/named.conf.local`:

    ```conf
    zone "ieszv.jd" {
        type master;
        file "/var/lib/bind/db.ieszv.jd";
    };
    
    zone "57.168.192.in-addr.arpa" {
        type master;
        file "/var/lib/bind/192.168.57.rev";
    };
    ```

2. Configura el archivo de la zona directa `/var/lib/bind/db.ieszv.jd`:

    ```conf
    $TTL 604800
    @   IN  SOA mail.ieszv.jd. root.ieszv.jd. (
            2         ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            604800 )  ; Negative Cache TTL
    ;
    @   IN  NS  mail.ieszv.jd.
    @   IN  MX  10 mail.ieszv.jd.
    @   IN  A   192.168.57.10
    mail    IN  A   192.168.57.10
    ```

3. Configura la zona inversa en `/var/lib/bind/192.168.57.rev`:

    ```conf
    $TTL 604800
    @   IN  SOA mail.ieszv.jd. root.ieszv.jd. (
            2         ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            604800 )  ; Negative Cache TTL
    ;
    @   IN  NS  mail.ieszv.jd.
    10  IN  PTR mail.ieszv.jd.
    ```

4. Reinicia el servicio:

    ```bash
    sudo systemctl restart bind9
    ```

## 📩 Configuración de Postfix

1. Edita `/etc/dovecot/conf.d/10-auth.conf` y cambia la línea 10:

```dovecot
    #disable_plaintext_auth = no
```
a:
```dovecot
    disable_plaintext_auth = no
```

2. Edita `/etc/dovecot/conf.d/10-mail.conf` y cambia la línea 30:
```dovecot
    mail_location = maildir:~/Maildir
```
## 📩 Configuración de Postfix

1. Edita `/etc/postfix/main.cf` con los siguientes valores:

```conf
myhostname = mail.ieszv.jd
mydomain = ieszv.jd
myorigin = /etc/mailname
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
home_mailbox = Maildir/
relayhost = 
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all
``` 

2. Reinicia Postfix:

```bash
sudo systemctl restart postfix
```

## 🏠 Configuración del servidor

1. Edita `/etc/hosts` y añade:

```conf
192.168.57.10   mail.ieszv.jd mail
192.168.57.10   mail.ieszv.jd ieszv.jd
```

2. Edita `/etc/hostname` y añade (si no esta escrito):

```conf
    mail
```

3. Edita `/etc/resolv.conf` y añade:

```conf
    nameserver 192.168.57.10
    search ieszv.jd
```

    > Todos los archivos cofigurados en la carpeta webmail

## 🐿️ Instalación de SquirrelMail

1. Descarga la última versión estable desde el sitio oficial(está en la carpeta squirrel, solo se necesita copiar):

    ```bash
    cp -v /vagrant/squirrel/squirrelmail-webmail-1.4.22.tar.gz /tmp/
    ```

2. Extrae los archivos en `/usr/local/`:

    ```bash
    tar xvfz /tmp/squirrelmail-webmail-1.4.22.tar.gz -C /usr/local
    ```

3. Crea un enlace simbólico en el directorio de Apache:

    ```bash
    ln -s /usr/local/squirrelmail-webmail-* /var/www/mail
    ```

4. Configura los directorios para datos y adjuntos:

    ```bash
    sudo mkdir -p /var/local/squirrelmail/data
    sudo chown www-data:www-data /var/local/squirrelmail/data

    sudo mkdir -p /var/local/squirrelmail/attach
    sudo chown www-data:www-data /var/local/squirrelmail/attach
    ```

5. Configura SquirrelMail ejecutando el script interactivo:

    ```bash
    sudo /usr/local/squirrelmail-webmail-*/config/conf.pl
    ```

   - Configura el servidor IMAP: selecciona `D` y elige `dovecot`
   - Configura el dominio: Ve a `Server Settings (2)` y cambia el dominio a `mail.ieszv.jd`
   - Configura el idioma: Ve a `Languages (10)` y elige `es_ES`
   - Guarda los cambios (`S`)

   > Puedes encontrar este fichero en [config.php](/php/config.php)

## ✅ Verificación de la instalación

1. Crearemos los usuarios, en mi caso uno será profesor y otro mi nombre:

```bash
    sudo adduser juandiego
    sudo adduser profesor
```
2. Rellene los datos que usted desee y procedemos a crear los directorios para que reciban correo (si no está ya creada):

```bash
    sudo -u juandiego mkdir -p /home/juandiego/Maildir/{cur,new,tmp}

    sudo -u profesor mkdir -p /home/profesor/Maildir/{cur,new,tmp}
```

    > Crear los usuarios puedes hacerlo al principio pero yo lo hago al final.

3. Reinicia Apache y Postfix:

```bash
sudo systemctl restart apache2 postfix bind9
```

4. Accede a SquirrelMail desde tu navegador:

```
http://mail.ieszv.jd/mail
```
5. Inicias sesión y Mandar correo

    - Inicias sesión con uno de los usarios que has creado y mandas un correo al otro

    - Para poder mandar un correo solo tienes que pinchar en `Compose` y allí pondes `profesor@ieszv.jd` o `juandiego@ieszv.jd`

    - Al entrar en al otro podrás ver como te llego el correo que mandaste desde el otro usuario.

---

 2025 - Guía de instalación y configuración de Webmail en Debian 11.
