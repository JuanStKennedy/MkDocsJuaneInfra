# :octicons-key-16: Samba AD DC (Identity Provider)

El Controlador de Dominio de Samba (Samba AD DC) nos proporciona la identidad en el laboratorio, brindando servicios de LDAP (Servicio de Directorio) y Kerberos (Autenticación). Permite la autenticación centralizada y el inicio de sesión único para todos los hosts y servicios.

## 1. Aprovisionamiento del Controlador de Dominio (JDCSR01)

El servicio Samba AD DC fue levantado sobre el servidor JDCSR01 con la siguiente configuración:

```bash 
#!/bin/bash

DOMAIN="JS-LAB-PY"
REALM="${DOMAIN}.DUCKDNS.ORG"
ADMINPASSWORD=""

sed -i 's/127.0.0.1/8.8.8.8/' /etc/resolv.conf

cp /etc/resolv.conf /etc/resolv.conf.bak

sudo apt update
apt-get install acl attr samba samba-client winbind libpam-winbind libnss-winbind dnsutils python3-setproctitle krb5-user -y 

systemctl stop smbd nmbd winbind
systemctl disable smbd nmbd winbind
systemctl mask smbd nmbd winbind

for VARIABLE in $(smbd -b | egrep "LOCKDIR|STATEDIR|CACHEDIR|PRIVATE_DIR" | cut -d ":" -f2)
do
    echo "Borrando *.tdb y *.ldb en $VARIABLE"
    rm -Rf $VARIABLE/*.tdb
    rm -Rf $VARIABLE/*.ldb
done

rm -f $(smbd -b | grep "CONFIGFILE" | cut -d ":" -f2)
echo "etc/samba/smb.conf borrado"
ls /etc/samba

samba-tool domain provision --server-role=dc --use-rfc2307 --dns-backend=SAMBA_INTERNAL --realm="${REALM}" --domain="${DOMAIN}" --adminpass="${ADMINPASSWORD}"

sed -i 's/8.8.8.8/127.0.0.1/' /etc/resolv.conf

systemctl unmask samba-ad-dc
systemctl enable samba-ad-dc
systemctl restart samba-ad-dc

sleep 10

host -t SRV _ldap._tcp.$REALM
host -t SRV _kerberos._udp.$REALM
host -t A $(hostname).$REALM

sleep 5
```

---

## 2. Estructura de Usuarios y Grupos

La estructura de AD fue creada mediante un script utilizando la herramienta `samba-tool` para estandarizar los nombres de usuario al formato nombre.apellido y organizar la red.
Este script crea OUs, grupos de seguridad, asigna a los usuarios a sus respectivos grupos y organiza todo de manera jerárquica. Además,
la contraseña por defecto para los usuarios es el nombre.apellido, pero cuando se inicia sesión por primera vez, se les solicita cambiarla:

```bash
#!/bin/bash

DOMAIN="JS-LAB-PY"
REALM="${DOMAIN}.DUCKDNS.ORG"
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

# Crear grupo proveedores
samba-tool group add proveedores --groupou="ou=Grupos de Seguridad,ou=Grupos,ou=${REALM}"

# Usuarios Sysadmins
samba-tool user create "carlos.martinez" carlos.martinez --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "laura.lopez" laura.lopez --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "juan.hernandez" juan.hernandez --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "jesus.guibert" jesus.guibert --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "juan.siecola" juan.siecola --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "miguel.torres" miguel.torres --userou="ou=Sysadmins,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login

# Usuarios Devs
samba-tool user create "david.gomez" david.gomez --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "sandra.ramirez" sandra.ramirez --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "alejandro.torres" alejandro.torres --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "patricia.morales" patricia.morales --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "javier.ortiz" javier.ortiz --userou="ou=Devs,ou=Sistemas,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
# Usuarios RRHH
samba-tool user create "beatriz.navarro" beatriz.navarro --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "raul.gil" raul.gil --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "maria.soto" maria.soto --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "antonio.castro" antonio.castro --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "isabel.ramos" isabel.ramos --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "jorge.vega" jorge.vega --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login
samba-tool user create "laura.mendez" laura.mendez --userou="ou=RRHH,ou=Empleados,ou=Usuarios,ou=${REALM}" --must-change-at-next-login

# Agregar usuarios a grupos
samba-tool group addmembers proxmox-admins "jesus.guibert,juan.siecola"
samba-tool group addmembers sysadmins "carlos.martinez,laura.lopez,juan.hernandez,jesus.guibert,juan.siecola" 
samba-tool group addmembers devs "david.gomez,sandra.ramirez,alejandro.torres,patricia.morales,javier.ortiz"
samba-tool group addmembers rrhh "beatriz.navarro,raul.gil,maria.soto,antonio.castro,isabel.ramos,jorge.vega"
samba-tool group addmembers proveedores "laura.mendez,beatriz.navarro"
```

---

## 3. Unión de Equipos al Dominio (Host Linux)

Todos los servidores Linux (JDEVSR01, JJUMSR01 y JDATSR01) se unieron al dominio utilizando realm para configurar SSSD, Kerberos y Winbind automáticamente.

Este script asegura que las máquinas puedan usar las credenciales de AD para iniciar sesión pot SSH por ejemplo y que se creen los directorios home automáticamente. **El requisito principal para que funcione es que el DNS primario apunte a la IP del host que corre el servicio samba (JDCSR01)**.

!!!note
    Como se mencionó antes, el DNS primario de las máquinas Linux debe apuntar a la IP del servidor Samba AD DC (JDCSR01) para que la unión al dominio funcione correctamente, en este caso `172.16.10.11` es la IP del servidor Samba AD DC.

```bash
#!/bin/bash
apt install sssd-ad sssd-tools realmd adcli packagekit sudo nfs-common -y
sudo realm join js-lab-py.duckdns.org -U administrator
sudo pam-auth-update --enable mkhomedir

#Configuración general de sssd

sudo cat << EOF | sudo tee /etc/sssd/sssd.conf
[sssd]
domains = JS-LAB-PY.DUCKDNS.ORG
config_file_version = 2
services = nss, pam

[domain/JS-LAB-PY.DUCKDNS.ORG]
default_shell = /bin/bash
krb5_store_password_if_offline = True
cache_credentials = True
krb5_realm = JS-LAB-PY.DUCKDNS.ORG
realmd_tags = manages-system joined-with-adcli
id_provider = ad
fallback_homedir = /home/%u@%d
ad_domain = JS-LAB-PY.DUCKDNS.ORG
ldap_id_mapping = True
access_provider = ad
ad_enable_gc = False
ldap_user_ssh_public_key = sshPublicKey
ldap_user_extra_attrs = sshPublicKey:sshPublicKey
EOF

sleep 5
sudo systemctl restart sssd
```

---

## 4. Demostración en una máquina con Windows 10.

Para unir una máquina con Windows 10 al dominio Samba AD DC:

