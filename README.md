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


![alt text](https://cdn.discordapp.com/attachments/883488196036534297/886380346931826718/unknown.png)
