# ADMININFRA LABORATORIO FINAL

***____________________________________________________________________________________________________________________________________________***
***Http Server***(Contenedor 1)

`apt update`

`apt upgrade`

*Dependencias:*

`wget https://ufpr.dl.sourceforge.net/project/pcre/pcre/8.43/pcre-8.43.tar.gz && tar xzvf pcre-8.43.tar.gz`

`wget https://www.zlib.net/zlib-1.2.11.tar.gz && tar xzvf zlib-1.2.11.tar.gz`

`wget https://www.openssl.org/source/openssl-1.1.1c.tar.gz && tar xzvf openssl-1.1.1c.tar.gz`
        
*NginX:*

`wget https://nginx.org/download/nginx-1.20.1.tar.gz && tar zxvf nginx-1.20.1.tar.gz`

*Mas dependencias:*

`apt install -y perl libperl-dev libgd3 libgd-dev libgeoip1 libgeoip-dev geoip-bin libxml2 libxml2-dev libxslt1.1 libxslt1-dev`

`apt install build-essential`

*Copia del manual(Opcional):*

`cp ~/nginx-1.20.1/man/nginx.8 /usr/share/man/man8`
        
`gzip /usr/share/man/man8/nginx.8`

*Entrar a los 3 directorios nuevos e instalarlos:*

>`cd pcre-8.43/`
>
>`./configure`
>
>`make`
>
>`make install`

>`cd openssl-1.1.1c/`
>
>`./config`
>    
>`make`
>   
>`make install`
 
>`cd zlib-1.2.11/`
>
>`./configure`
>
>`make`
>
>`make install`

*Descargar el modulo de auth-ldap:*

`apt install git`

`git clone https://github.com/kvspb/nginx-auth-ldap.git`

`apt install libldap2-dev ldap-utils`

*Instalar NginX:*

`cd nginx-1.20.1/`

```
./configure --prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--modules-path=/usr/lib/nginx/modules \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--user=nginx \
--group=nginx \
--build=Debian \
--builddir=nginx-1.20.1 \
--with-select_module \
--with-poll_module \
--with-threads \
--with-file-aio \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_xslt_module=dynamic \
--with-http_image_filter_module=dynamic \
--with-http_geoip_module=dynamic \
--with-http_sub_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_auth_request_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_degradation_module \
--with-http_slice_module \
--with-http_stub_status_module \
--with-http_perl_module=dynamic \
--with-perl_modules_path=/usr/share/perl/5.26.1 \
--with-perl=/usr/bin/perl \
--http-log-path=/var/log/nginx/access.log \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--with-mail=dynamic \
--with-mail_ssl_module \
--with-stream=dynamic \
--with-stream_ssl_module \
--with-stream_realip_module \
--with-stream_geoip_module=dynamic \
--with-stream_ssl_preread_module \
--with-compat \
--with-pcre=../pcre-8.43 \
--with-pcre-jit \
--with-zlib=../zlib-1.2.11 \
--with-openssl=../openssl-1.1.1c \
--with-openssl-opt=no-nextprotoneg \
--add-module=$HOME/nginx-auth-ldap \
--with-debug
```
	
`make`

`make install`

`ln -s /usr/lib/nginx/modules /etc/nginx/modules`

*Podemos ver la version de NginX asi:*

`nginx -V`
        
*Crear el grupo y usuario para NginX:*

`adduser --system --home /nonexistent --shell /bin/false --no-create-home --disabled-login --disabled-password --gecos "nginx user" --group nginx`

`tail -n 1 /etc/passwd /etc/group /etc/shadow`

*Chequeo de errores:*

`nginx -t`

*Si te sale este error:*

```nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (2: No such file or directory)```

*Se soluciona asi:*

`mkdir -p /var/cache/nginx/client_temp /var/cache/nginx/fastcgi_temp /var/cache/nginx/proxy_temp /var/cache/nginx/scgi_temp /var/cache/nginx/uwsgi_temp`

`chmod 700 /var/cache/nginx/*`

`chown nginx:root /var/cache/nginx/*`

*Verificar solución*

`nginx -t`

*Crear el archivo de servicio:*

`nano /etc/systemd/system/nginx.service`

*pegar dentro:*

```
[Unit]
Description=nginx - high performance web server
Documentation=https://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```

*habilitar el servicio:*

`systemctl enable nginx.service`

`systemctl start nginx.service`

*Comprobar si se arranca al iniciar:*

`systemctl is-enabled nginx.service`


***____________________________________________________________________________________________________________________________________________***

***Monitorización***(Contenedor 2):




![alt text](https://media.discordapp.net/attachments/883488196036534297/907487706794295297/unknown.png?width=1026&height=670)

![alt text](https://media.discordapp.net/attachments/883488196036534297/905835068663148644/unknown.png?width=1440&height=608)

![alt text](https://cdn.discordapp.com/attachments/883488196036534297/906190460970733609/unknown.png)

![alt text](https://user-images.githubusercontent.com/57294943/141221321-ee2a293c-1d3a-409c-8579-1adf7fd089fb.png)


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

![alt text](https://cdn.discordapp.com/attachments/883488196036534297/907704717059174470/Desktop_Screenshot_2021.11.09_-_15.53.26.68.png)

![alt text](https://cdn.discordapp.com/attachments/883488196036534297/907704706925756476/Desktop_Screenshot_2021.11.09_-_15.53.20.98.png)

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

>COMPILAR SAMBA
`apt update`
`apt upgrade`

```
apt-get install acl attr autoconf bind9utils bison build-essential \
  debhelper dnsutils docbook-xml docbook-xsl flex gdb libjansson-dev krb5-user \
  libacl1-dev libaio-dev libarchive-dev libattr1-dev libblkid-dev libbsd-dev \
  libcap-dev libcups2-dev libgnutls28-dev libgpgme-dev libjson-perl \
  libldap2-dev libncurses5-dev libpam0g-dev libparse-yapp-perl \
  libpopt-dev libreadline-dev nettle-dev perl perl-modules pkg-config \
  python-all-dev python-crypto python-dbg python-dev python-dnspython \
  python3-dnspython python-gpg python3-gpg python-markdown python3-markdown \
  python3-dev xsltproc zlib1g-dev liblmdb-dev lmdb-utils \
```
  
`apt install libtasn1-bin libdbus-1-dev`

`./configure`
`make`
`make install`
`echo "export PATH=$PATH:/usr/local/samba/sbin:/usr/local/samba/bin" >> /root/.bashrc`
`source /root/.bashrc`

*Verificar que no esté en ejecución ningúno de los siguientes procesos, en caso de estarlos, pkill al proceso/s*
`ps ax | egrep "samba|smbd|nmbd|winbindd"`
`rm /etc/krb5.conf`

`samba-tool domain provision --server-role=dc --use-rfc2307 --dns-backend=SAMBA_INTERNAL --realm=EQUIPO10.ADMININFRA.EDU.UY --domain=EQUIPO10 --adminpass=Passw0rd`

*Verificar que las ip sean las correspondientes*
`cat /etc/resolv.conf`
`cat /etc/hosts`

`cp /usr/local/samba/private/krb5.conf /etc/krb5.conf`

*Ejecutar samba*
`samba`

*Comprobar que esté todo correcto*
`kinit administrator`
`klist`

*Configurar smb.conf*
`nano /usr/local/samba/etc/smb.conf`
*PEGAR*
```
# Global parameters
[global]
        dns forwarder = 10.0.2.113
        netbios name = CT-PDC
        realm = EQUIPO10.ADMININFRA.EDU.UY
        server role = active directory domain controller
        workgroup = EQUIPO10
        idmap_ldb:use rfc2307 = yes
        ldap server require strong auth = no
[sysvol]
        path = /usr/local/samba/var/locks/sysvol
        read only = No

[netlogon]
        path = /usr/local/samba/var/locks/sysvol/equipo10.admininfra.edu.uy/scripts
        read only = No
```

*Verificar que se resuelva bien el PDC*
`nslookup ct-pdc.equipo10.admininfra.edu.uy`

`nano /etc/systemd/system/samba-ad-dc.service`
 *PEGAR*
 ```
[Unit]
Description=Samba Active Directory Domain Controller
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/samba/sbin/samba -D
PIDFile=/usr/local/samba/var/run/samba.pid
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```
`systemctl daemon-reload`
`systemctl enable samba-ad-dc`
`reboot`

>PARA AGREGAR USUARIOS

`nano generarUsuarios.sh`

```
for (( i=0; i<=(valorDeIteracciones); i++ ))
do
    response=$(curl --silent https://randomuser.me/api/?nat=us)
    nombre=$(jq -r '.results[0].name.first' <<< "${response}")
    apellido=$(jq -r '.results[0].name.last' <<< "${response}")
    email=$(jq -r '.results[0].email' <<< "${response}")

    name_low=$(echo ${nombre} | tr '[:upper:]' '[:lower:]')
    sur_low=$(echo ${apellido} | tr '[:upper:]' '[:lower:]')
    nickname=$(echo $name_low"."$sur_low)
    uid=$(($i+10000))
    samba-tool user create "${nickname}" "Passw0rd" --given-name="${nombre}" --surname="${apellido}" --mail-address="${email}" --userou="OU=MONITOREO,OU=ADMININFRA" --gid-number=10000 --uid-number=$uid
done
```
`chmod u+x generarUsuarios.sh`

*Ejecutar*
`./generarUsuarios.sh`

*Crear OU*
`samba-tool ou create 'OU=WEB,OU=ADMININFRA,DC=equipo10,DC=admininfra,DC=edu,DC=uy' --description='Unidad Organizacional para acceso a la web de NGINX'`

***____________________________________________________________________________________________________________________________________________***



