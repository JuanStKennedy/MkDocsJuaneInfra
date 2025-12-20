# Configuración: JDEVSR01 (Debian 12 Server)

```bash
# Loopback
auto lo
iface lo inet loopback
#
# Red principal (DHCP)
###auto ens4
#iface ens4 inet dhcp
# Configuración estática anterior (comentada)
 auto ens4
 iface ens4 inet static
     address 172.16.200.50
     netmask 255.255.255.0
     gateway 172.16.200.1
     dns-nameservers 172.16.10.11 8.8.4.4
```
