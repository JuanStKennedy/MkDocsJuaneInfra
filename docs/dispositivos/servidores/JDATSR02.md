# Configuración: JDATSR02 (Debian 12 Server)

```bash
# Loopback
auto lo
iface lo inet loopback

# Red principal
auto ens4
iface ens4 inet static
    address 172.16.5.101
    netmask 255.255.255.0
    gateway 172.16.5.1
    dns-nameservers 8.8.8.8 8.8.4.4
```
