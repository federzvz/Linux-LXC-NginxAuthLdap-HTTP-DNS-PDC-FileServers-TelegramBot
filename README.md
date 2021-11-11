# ADMININFRA LABORATORIO FINAL

***____________________________________________________________________________________________________________________________________________***
***Http Server***(Contenedor 1)

`apt update`

`apt upgrade`

*Dependencias:*

`wget https://sourceforge.net/projects/pcre/files/pcre/8.45/pcre-8.45.tar.gz/download`

`mv download pcre-8.45.tar.gz` *Esto porque lo decarga con el nombre de "download"*

`apt install build-essential`

`wget http://zlib.net/zlib-1.2.11.tar.gz`

`apt install libldap2-dev ldap-utils libssl-dev`
        
*NginX:*

`wget https://nginx.org/download/nginx-1.18.0.tar.gz`

*Entrar a los 2 directorios nuevos e instalarlos:*

>`cd pcre-8.45/`
>
>`./configure`
>
>`make`
>
>`make install`

>`cd cd zlib-1.2.11/`
>
>`./configure`
>    
>`make`
>   
>`make install`

*Descargar el modulo de auth-ldap:*

`apt install git`

`git clone https://github.com/kvspb/nginx-auth-ldap.git`

*Instalar NginX:*

`cd nginx-1.20.1/`

`./configure --add-module=/root/nginx-auth-ldap`
	
`make`

`make install`

`echo "export PATH=$PATH:/usr/local/nginx/sbin" >> /root/.bashrc`

`source /root/.bashrc`

*Modificamos el archivo nginx.conf*

`nano /usr/local/nginx/conf/nginx.conf`

*pegar dentro:*

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    ldap_server dashboard {
        url ldap://10.0.2.111:3268/OU=WEB,OU=ADMININFRA,DC=EQUIPO10,DC=ADMININFRA,DC=EDU,DC=UY?sAMAccountName?sub?(objectClass=person);
        binddn "EQUIPO10\\administrator";
        binddn_passwd "Passw0rd";
        #group_attribute uniquemember;
        #group_attribute_is_dn on;
        require valid_user;
    }
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;
        root /usr/local/nginx/www/dashboard.equipo10.admininfra.edu.uy;
        index access_log.html;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        auth_ldap "Forbidden";
        auth_ldap_servers dashboard;

        location / {
            #root   html;
            #index  index.html index.htm;
            try_files $uri $uri/ /access_log.html?$args;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

***____________________________________________________________________________________________________________________________________________***

***Monitorización***(Contenedor 2):

![alt text](https://media.discordapp.net/attachments/883488196036534297/907487706794295297/unknown.png?width=1026&height=670)

![alt text](https://media.discordapp.net/attachments/883488196036534297/905835068663148644/unknown.png?width=1440&height=608)

![alt text](https://cdn.discordapp.com/attachments/883488196036534297/906190460970733609/unknown.png)

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

![alt text](https://cdn.discordapp.com/attachments/883488196036534297/907704717059174470/Desktop_Screenshot_2021.11.09_-_15.53.26.68.png)

![alt text](https://cdn.discordapp.com/attachments/883488196036534297/907704706925756476/Desktop_Screenshot_2021.11.09_-_15.53.20.98.png)

***____________________________________________________________________________________________________________________________________________***



