# ADMININFRA LABORATORIO FINAL


***Http Server***(Contenedor 1)

***____________________________________________________________________________________________________________________________________________***

***Monitorización***(Contenedor 2):


***____________________________________________________________________________________________________________________________________________***

***File Server(smb)***(Contenedor 3):

`apt update`

`apt upgrade`

>-NO en DHCP-
>
>`apt install samba`


`ps ax | egrep "samba|smbd|nmbd|winbindd"`

`pkill smbd`
`pkill nmbd`

>-VERIFICAR SI NO HAY NINGUN PROCESO DE SAMBA-
>
>`ps ax | egrep "samba|smbd|nmbd|winbindd"`



>`smbd -b | egrep "LOCKDIR|STATEDIR|CACHEDIR|PRIVATE_DIR"`
>
>```
>rm /var/run/samba/*.tdb
>rm /var/run/samba/*.ldb
>rm /var/lib/samba/*.tdb
>rm /var/lib/samba/*.ldb
>rm /var/cache/samba/*.ldb
>rm /var/cache/samba/*.tdb
>rm /var/lib/samba/private/*.tdb
>rm /var/lib/samba/private/*.ldb
>```

>ASEGURARSE DE TENER EL PDC EN EL /etc/resolv.conf
>
>`cat /etc/resolv.conf`
>```
># --- BEGIN PVE ---
>search equipo10.admininfra.edu.uy
>nameserver 10.0.2.111
># --- END PVE ---
>```

`apt install dnsutils`

>testear nslookup
>
>nslookup ct-pdc.equipo10.admininfra.edu.uy
>```
>Server:         10.0.2.111
>Address:        10.0.2.111#53
>
>Name:   ct-pdc.equipo10.admininfra.edu.uy
>Address: 192.168.200.136
>Name:   ct-pdc.equipo10.admininfra.edu.uy
>Address: 10.0.2.111
>```

`nano /etc/krb5.conf`

```
[libdefaults]
        default_realm = EQUIPO10.ADMININFRA.EDU.UY
        dns_lookup_realm = false
        dns_lookup_kdc = true
```

>VERIFICAR QUE ESTÉ LA IP BIEN
>
> `getent hosts`
> 
> ```10.0.2.112      ct-file-server.equipo10.admininfra.edu.uy ct-file-server```

`/etc/samba/smb.conf`

```
[global]
   workgroup = EQUIPO10
   security = ADS
   realm = EQUIPO10.ADMININFRA.EDU.UY

   winbind refresh tickets = Yes
   vfs objects = acl_xattr
   map acl inherit = Yes
   store dos attributes = Yes

   dedicated keytab file = /etc/krb5.keytab
   kerberos method = secrets and keytab

   winbind use default domain = yes

   winbind enum users = yes
   winbind enum groups = yes

   load printers = no
   printing = bsd
   printcap name = /dev/null
   disable spoolss = yes

   idmap config * : backend = tdb
   idmap config * : range = 3000-7999
   # - You must set a DOMAIN backend configuration
   # idmap config for the SAMDOM domain
   idmap config EQUIPO10:backend = ad
   idmap config EQUIPO10:schema_mode = rfc2307
   idmap config EQUIPO10:range = 10000-999999
   idmap config EQUIPO10:unix_nss_info = yes

   vfs objects = acl_xattr
   map acl inherit = yes
   store dos attributes = yes
```

`net ads join -U administrator`

```
Enter administrator's password:
Using short domain name -- EQUIPO10
Joined 'CT-FILE-SERVER' to dns domain 'equipo10.admininfra.edu.uy'
```

`apt install winbind`

`apt install libnss-winbind libpam-winbind`

`nano /etc/nsswitch.conf`

```
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd winbind
group:          files systemd winbind
shadow:         files
gshadow:        files

hosts:          files dns
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

>CAMBIAR CON ADMINTOOLS EL GID NUMBER DE DOMAIN USERS A 10000

>PARA MONITOREAR
>
> `apt install zabbix-agent`
>
>`nano /etc/zabbix/zabbix_agentd.conf`
>
>```
>PidFile=/tmp/zabbix_agentd.pid
>LogFile=/tmp/zabbix_agentd.log
>LogFileSize=3
>Server=10.0.2.114,127.0.0.1
>ServerActive=10.0.2.114
>HostMetadata=e13f6fcf1e80acf28bcd308cfa9776c1
>Include=/etc/zabbix/zabbix_agentd.conf.d
>```


***____________________________________________________________________________________________________________________________________________***

***DNS Server***(Contenedor 4):

**EN UN CONTENEDOR NUEVO QUE FUNCIONARÁ COMO UN SERVIDOR DNS**

`apt update`

*Instalar dnsmasq*

`apt install dnsmasq -y`

*Instalas las utilidades DNS*

`apt install dnsutils`

>`dig debian.org @localhost`
>
>Verificar que esté todo OK

*Remplazar el nameserver default por los siguientes servidores ISP Antel(Realizarlo desde Proxmox)*

`nano /etc/resolv.conf`

```
nameserver 200.40.30.245
nameserver 200.40.220.245
````

*Verificar que se resuelvan las DNS correctamente (Si nos devuelve la IP está funcionadno.)*

`nslookup antel.com.uy`

***____________________________________________________________________________________________________________________________________________***

***PDC Server***(Contenedor 5):


***____________________________________________________________________________________________________________________________________________***


***CONFIGURACIÓN DEL PRIMER CONTENEDOR: (ct-node1)***

>**equipo-nro-admininfra.edu.uy**: contendrá un sitio web básico de wordpress.

`apt install nginx php-cli php-fpm php-mysql php-json php-opcache php-mbstring php-xml php-gd php-curl mariadb-server`

`systemctl start nginx.service`

`systemctl enable nginx.service`

`mkdir /var/www/equipo-10-admininfra.edu.uy`

`cd /var/www/equipo-10-admininfra.edu.uy`

`wget https://wordpress.org/latest.tar.gz`

`tar -xzvf latest.tar.gz`

`cd wordpress`

`mv wp-config-sample.php wp-config.php`

`chown -R www-data:www-data /var/www/equipo-10-admininfra.edu.uy/wordpress`

`chmod -R 755 /var/www/equipo-10-admininfra.edu.uy/wordpress`

`nano /etc/nginx/sites-available/wordpress.conf`

***PEGAR***

```
	server {
		listen 80;
		listen [::]:80;
		root /var/www/equipo-10-admininfra.edu.uy/wordpress;
		index  index.php index.html index.htm;
		server_name equipo-10-admininfra.edu.uy;

		error_log /var/log/nginx/mysite.com_error.log;
		access_log /var/log/nginx/mysite.com_access.log;

		client_max_body_size 100M;
		location / {
				try_files $uri $uri/ /index.php?$args;
		}
		location ~ \.php$ {
				include snippets/fastcgi-php.conf;
				fastcgi_pass unix:/run/php/php7.3-fpm.sock;
				fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
		}
	}
```
***GUARDAR***

`ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/`

`nginx -t`

`systemctl restart nginx`

***MODIFICAR ARCHIVO HOSTS DE WINDOWS***

>Ejecutar CMD como administrador

`notepad drivers/etc/hosts`

***PEGAR ABAJO DEL TODO:***

`192.168.200.105 equipo-10-admininfra.edu.uy`

>192.168.200.105 Corresponde a la IP de mi contenedor 1, tu debes poner la IP correspondiente a tu contenedor.

Ir a tu navegador predeterminado y acceder a la URL:

`equipo-10-admininfra.edu.uy`
        
![alt text](https://markontech.com/wp-content/uploads/2021/05/install-wordpress-nginx-debian5.png)

***CREAR PROXY REVERSO PARA REDIRECCIONAR TRAFICO DEL CONTENEDOR 1 AL CONTENEDOR 2***

>**equipo-nro-emby.admininfra.edu.uy**: redireccionara el trafico al nodo ct-node2.

`nano /etc/nginx/sites-available/equipo-10-emby.admininfra.edu.uy`

***PEGAR:***
```
	server {
		listen 80;
		server_name equipo-10-emby.admininfra.edu.uy;
		location / {
				proxy_pass http://192.168.200.106:8096/;
		}
	}
```
***GUARDAR***

`ln -s /etc/nginx/sites-available/equipo-10-emby.admininfra.edu.uy /etc/nginx/sites-enabled/`

`nginx -s reload`

***MODIFICAR ARCHIVO HOSTS DE WINDOWS***

>Ejecutar CMD como administrador

`notepad drivers/etc/hosts`

***PEGAR ABAJO DEL TODO:***

`192.168.200.105 equipo-10-emby.admininfra.edu.uy`

>192.168.200.105 Corresponde a la IP de mi contenedor 1, tu debes poner la IP correspondiente a tu contenedor.

***CREAR PAGINA WEB QUE MONITOREA EL TRÁFICO DE NUESTROS SERVIDORES NGINX***

>**dashboard-equipo-nro.admininfra.edu.uy:** mostrara una instancia del dashboard goaccess

***INSTALAR GOACCES***

`apt install libncursesw5-dev libgeoip-dev apt-transport-https `

`cd /usr/src`

`wget https://tar.goaccess.io/goaccess-1.4.tar.gz`

`tar -xzvf goaccess-1.4.tar.gz`

`cd goaccess-1.4/`

`apt install build-essential`

`./configure --enable-utf8 --enable-geoip=legacy`

`make`

`make install`

`apt install goaccess`

`nano /usr/local/etc/goaccess/goaccess.conf`

>Descomentar `time-format`, `date-format` y `log-format`. Dejarlos como se muestra a continuación:

```
//time-format:
time-format %T

//date-format Opción 1:
date-format %Y-%m-%d
//date-format Opción 2:
date-format %d/%b/%Y

//log-format:
log-format %h %^[%d:%t %^] "%r" %s %b

```
`mkdir /var/www/dashboard-equipo-10.admininfra.edu.uy`

`cd /var/www/dashboard-equipo-10.admininfra.edu.uy`

`goaccess /var/log/nginx/access.log -o access_log.html --real-time-html --addr=192.168.200.105  --> IP DEL CT-NODO1`
> Es un monitoreo en tiempo real por lo tanto el comando queda corriendo en primer plano.

`nano /etc/nginx/sites-available/dashboard`
***PEGAR:***

```
	server {
		listen 80;
		listen [::]:80;
		root /var/www/dashboard-equipo-10.admininfra.edu.uy;
		index access_log.html;
		server_name dashboard-equipo-10.admininfra.edu.uy;
		location / {
				try_files $uri $uri/ /access_log.html?$args;
		}
	}
```

`ln -s /etc/nginx/sites-available/dashboard /etc/nginx/sites-enabled/`

`nginx -s reload`


***MODIFICAR ARCHIVO HOSTS DE WINDOWS***

>Ejecutar CMD como administrador

`notepad drivers/etc/hosts`

***PEGAR ABAJO DEL TODO:***

`192.168.200.105 dashboard-equipo-10.admininfra.edu.uy`

>192.168.200.105 Corresponde a la IP de mi contenedor 1, tu debes poner la IP correspondiente a tu contenedor.


***____________________________________________________________________________________________________________________________________________***

***CONFIGURACIÓN DEL SEGUNDO CONTENEDOR: (ct-node2)***

> ct-node2: tendrá configurado el servidor de multimedia emby, donde se deberá de alojar un video, la instancia de nginx deberá retornar un sitio dinámico en php que retornara >el contenido del directorio de videos de emby.

`wget https://github.com/MediaBrowser/Emby.Releases/releases/download/4.6.4.0/emby-server-deb_4.6.4.0_amd64.deb`

`dpkg -i emby-server-deb_4.6.4.0_amd64.deb`

*** AGREGAR UN PUERTO EN LA VM ***
          
![alt text](https://cdn.discordapp.com/attachments/883488196036534297/886380346931826718/unknown.png)

> `192.168.31.186` Es dirección IPv4 de tu equipo host. 
> 
> `65003` Número de puerto a elección.
> 
> `10.0.2.220` Es la IP correspondiente a la red net0 del contenedor 2 (ct-node2)
> 
> `8096` Puerto utilizado para el servicio Emby.

*** CREAR CARPETA DONDE GUARDAR LOS VIDEOS: ***

`mkdir -p /emby-library/Movies`
				
***MODIFICAR ARCHIVO HOSTS DE WINDOWS SI NO LO HAS HECHO***

>Ejecutar CMD como administrador

`notepad drivers/etc/hosts`

***PEGAR ABAJO DEL TODO:***

`192.168.200.105 equipo-10-emby.admininfra.edu.uy`

>192.168.200.105 Corresponde a la IP de mi contenedor 1, tu debes poner la IP correspondiente a tu contenedor.


***____________________________________________________________________________________________________________________________________________***

***CONFIGURACIÓN DEL TERCER CONTENEDOR: (ct-node3)***

> Tendrá configurado el gestor de base de datos(mysql) para la instancia de wordpress que se configuró en el Contenedor1(ct-node1).

Instalar MySql server

`Apt install default-mysql-server`

`systemctl edit mariadb`

***PEGAR***
```
	# /lib/systemd/system/mariadb.service
	[Service]
	ProtectHome=false
	ProtectSystem=false
	PrivateDevices=false
```
***GUARDAR***

`systemctl daemon-reload`

`systemctl start mariadb`

`systemctl enable mariadb.service`

`apt install build-essential`

`mysq_secure_installation`
        
![alt text](https://markontech.com/wp-content/uploads/2021/05/install-wordpress-nginx-debian1-768x740.png)

`mysql -u root -p`

`CREATE DATABASE sampledbwp;`

`GRANT ALL PRIVILEGES ON sampledbwp.* TO sample-admin@’192.168.200.105’ IDENTIFIED BY ‘SamplePassword1’;`

> La IP `192.168.200.105` corresponde a la IP del Contenedor1(ct-node1)

`quit`

***MODIFICAR EL SIGUIENTE ARCHIVO PARA BRINDAR ACCESO REMOTO AL NUESTRO SERVIDOR MSQL***

`nano /etc/mysql/mariadb.conf.d/50-server.cnf`

```

Blind-Adress 0.0.0.0

```

![alt text](https://phoenixnap.com/kb/wp-content/uploads/2021/04/bind-address-loaction-mysqld-remote-mysql-connection.png)
> Modificar `Blind-Adress 127.0.0.1` -> `Blind-Adress 0.0.0.0`

`systemctl restart mysql`

`iptables -A INPUT -p tcp --dport 3306 -j ACCEPT`

`iptables -A INPUT -p tcp -s 192.168.200.105 --dport 3306 -j ACCEPT`

> La IP `192.168.200.105` corresponde al Contenedor3.

***EL CONTENEDOR3(ct-node3) HA SIDO CONFIGURADO, AHORA VOLVER AL PRIMER CONTENEDOR(ct-node1) Y REALIZAR LO SIGUIENTE***

`cd /var/www/equipo-10-admininfra.edu.uy/wordpress`

`nano wp-config.php`

MODIFICAR COMO SE MUESTRA EN LA IMAGEN EN AMARILLO
          
![alt text](https://markontech.com/wp-content/uploads/2021/05/install-wordpress-nginx-debian4.png)

> En `DB_HOST`, cambiar `localhost` por `192.168.200.107`(IP del Contenedor3(ct-node3)

`systemctl restart mysql`

Para probar la conexión a la base de datos remota puedes realizar lo siguiente:

`mysql -u sample-admin -h 192.168.200.107 -p`

> `sample-admin` es el Usuario creado en el Contenedor3(ct-node3)
>
> `192.168.200.107` es la IP dónde está alojada la base de datos remota, la cual corresponde al contenedor3(ct-node3).



