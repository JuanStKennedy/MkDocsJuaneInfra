# :key: Samba AD DC (Identity Provider)

El Controlador de Dominio de Samba (Samba AD DC) es el corazón de la identidad en el laboratorio, proporcionando servicios de LDAP (Servicio de Directorio) y Kerberos (Autenticación). Permite la autenticación centralizada y el inicio de sesión único (SSO) para todos los hosts y servicios.

## 1. Aprovisionamiento del Controlador de Dominio (JDCSR01)

El servicio Samba AD DC fue levantado sobre el servidor JDCSR01 con la siguiente configuración:

```bash 
#!/bin/bash

DOMAIN="JS-LAB-UY"
REALM="${DOMAIN}.DUCKDNS.ORG"
ADMINPASSWORD="Passw0rd"
sed -i 's/127.0.0.1/8.8.8.8/' /etc/resolv.conf

apt update -y
apt upgrade -y

apt-get install acl attr samba samba-client winbind libpam-winbind libnss-winbind dnsutils python3-setproctitle krb5-user -y

for VARIABLE in $(smbd -b | egrep "LOCKDIR|STATEDIR|CACHEDIR|PRIVATE_DIR" | cut -d ":" -f2)
do
    echo "Borrando *.tdb y *.ldb en $VARIABLE"
    rm -Rf $VARIABLE/*.tdb
    rm -Rf $VARIABLE/*.ldb
done

systemctl stop winbind smbd nmbd
systemctl disable winbind smbd nmbd

rm -f $(smbd -b | grep "CONFIGFILE" | cut -d ":" -f2)
echo "etc/samba/smb.conf borrado"
ls /etc/samba

samba-tool domain provision --server-role=dc --use-rfc2307 --dns-backend=SAMBA_INTERNAL --realm="${REALM}" --domain="${DOMAIN}" --adminpass="${ADMINPASSWORD}"

sed -i 's/8.8.8.8/127.0.0.1/' /etc/resolv.conf

sleep 5
samba

sleep 5

host -t SRV _ldap._tcp.$REALM
host -t SRV _kerberos._udp.$REALM
host -t A $(hostname).$REALM

sleep 5
```

## 2. Estructura de Usuarios y Grupos

La estructura de AD fue creada mediante un script para estandarizar los nombres de usuario al formato nombre.apellido 
y organizar la red.
Este script crea OUs, grupos de seguridad y los usuarios con el formato nombre.apellido:

```bash
samba-tool ou create "ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"

# Usuarios
samba-tool ou create "ou=Usuarios,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Empleados,ou=Usuarios,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Proveedores,ou=Usuarios,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Ex-Empleados,ou=Usuarios,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"

# Service Accounts
samba-tool ou create "ou=ServiceAccounts,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"

# Equipos
samba-tool ou create "ou=Equipos,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Desktops,ou=Equipos,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Servers,ou=Equipos,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"

# Grupos
samba-tool ou create "ou=Grupos,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Grupos de Seguridad,ou=Grupos,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"
samba-tool ou create "ou=Grupos Distribuidos,ou=Grupos,ou=${REALM},dc=${DOMAIN},dc=duckdns,dc=org"

# Crear grupo sysadmins
samba-tool group add sysadmins --groupou="ou=Grupos de Seguridad,ou=Grupos,ou=${REALM}"

# Crear grupo devs
samba-tool group add devs --groupou="ou=Grupos de Seguridad,ou=Grupos,ou=${REALM}"

# Crear grupo rrhh
samba-tool group add rrhh --groupou="ou=Grupos de Seguridad,ou=Grupos,ou=${REALM}"

# Crear grupo proxmox-admins
samba-tool group add proxmox-admins --groupou="ou=Grupos de Seguridad,ou=Grupos,ou=${REALM}"

# Usuarios Sysadmins
samba-tool user create "carlos.martinez" Password123! --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "laura.lopez" Password123! --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "juan.hernandez" Password123! --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "jesus.guibert" Password123! --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "juan.siecola" Juaneduardo25/ --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"

# Usuarios Devs
samba-tool user create "david.gomez" Password123! --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "sandra.ramirez" Password123! --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "alejandro.torres" Password123! --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "patricia.morales" Password123! --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "javier.ortiz" Password123! --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}"

# Usuarios RRHH
samba-tool user create "beatriz.navarro" Password123! --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "raul.gil" Password123! --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "maria.soto" Password123! --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "antonio.castro" Password123! --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "isabel.ramos" Password123! --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}"
samba-tool user create "jorge.vega" Password123! --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}"

samba-tool group addmembers proxmox-admins "jesus.guibert,juan.siecola"
samba-tool group addmembers sysadmins "carlos.martinez,laura.lopez,juan.hernandez,jesus.guibert,juan.siecola" 
samba-tool group addmembers devs "david.gomez,sandra.ramirez,alejandro.torres,patricia.morales,javier.ortiz"
samba-tool group addmembers rrhh "beatriz.navarro,raul.gil,maria.soto,antonio.castro,isabel.ramos,jorge.vega"
```

## 3. Unión de Equipos al Dominio (Host Linux)

Todos los servidores Linux (JDEVSR01, JJUMSR01, JDATSR01, JDATSR02) se unieron al dominio utilizando realm para configurar SSSD, Kerberos y Winbind automáticamente.

Este script asegura que las máquinas puedan usar las credenciales de AD para iniciar sesión pot SSH por ejemplo y que se creen los directorios home automáticamente. **El requisito principal para que funcione es que el DNS primario apunte a la IP del host que corre el servicio samba (JDCSR01)**.

```bash
#!/bin/bash
apt install sssd-ad sssd-tools realmd adcli packagekit sudo nfs-common -y
sudo realm join js-lab-uy.duckdns.org -U administrator
sudo pam-auth-update --enable mkhomedir

#Configuración general de sssd

sudo cat << EOF | sudo tee /etc/sssd/sssd.conf
[sssd]
domains = JS-LAB-UY.DUCKDNS.ORG
config_file_version = 2
services = nss, pam

[domain/JS-LAB-UY.DUCKDNS.ORG]
default_shell = /bin/bash
krb5_store_password_if_offline = True
cache_credentials = True
krb5_realm = JS-LAB-UY.DUCKDNS.ORG
realmd_tags = manages-system joined-with-adcli
id_provider = ad
fallback_homedir = /home/%u@%d
ad_domain = JS-LAB-UY.DUCKDNS.ORG
ldap_id_mapping = True
access_provider = ad
ad_enable_gc = False
# LÍNEA ELIMINADA: ldap_use_tokengroups
ldap_user_ssh_public_key = sshPublicKey
ldap_user_extra_attrs = sshPublicKey:sshPublicKey
EOF

sleep 5
sudo systemctl restart sssd
```