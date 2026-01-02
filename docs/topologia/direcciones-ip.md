# Direccionamiento IP por Interfaz

Esta página sirve como un inventario detallado de todas las direcciones IP asignadas a las interfaces de los dispositivos de red y servidores.

## 1. Routers y Firewall

### JPROO2 (Router Cisco)

Este router maneja el enrutamiento Inter-VLAN (Router-on-a-Stick).

| Interfaz | Descripción | VLAN | Dirección IP | Máscara (CIDR) |
| :--- | :--- | :---: | :--- | :---: |
| G0/3.5 | Gateway VLAN 5 (Datacenter) | 5 | 172.16.5.1 | /24 |
| G0/3.10 | Gateway VLAN 10 (Sysadmin) | 10 | 172.16.10.1 | /24 |
| G0/3.99 | Gateway VLAN 99 (Admin Switches) | 99 | 172.16.99.1 | /24 |
| G0/3.200 | Gateway VLAN 200 (Dev) | 200 | 172.16.200.1 | /24 |
| G0/0 | Enlace a JPROO1 (VyOS) | N/A | 10.10.1.2 | /30 |
| LoopBack | Administración | N/A | 10.255.255.2 | /32 |

### JPROO1 (Router VyOS)

Este router actúa como el núcleo de la red, conectando el router Cisco con el firewall.

| Interfaz | Descripción | Dirección IP | Máscara (CIDR) |
| :--- | :--- | :--- | :---: |
| eth1 | Enlace a JPROO2 (Cisco) | 10.10.1.1 | /30 |
| eth0 | Enlace a JPFW01 (pfSense) | 10.10.0.2 | /30 |
| LoopBack | Administración | 10.255.255.1 | /32 |


### JPFW01 (Firewall pfSense)

Este es el dispositivo de borde, encargado del NAT y la seguridad.

| Interfaz | Propósito | Red | Dirección IP | Máscara (CIDR) |
| :--- | :--- | :--- | :--- | :---: |
| em2 | Conexión a JPROO1 | Enlace | 10.10.0.1 | /30 |
| em1 | Red de Management | Management | 192.168.1.1 | /24 |
| em0 | Conexión a Internet (WAN) | WAN | DHCP | N/A |
| Tailscale0 | Conexión VPN para compartir rutas | Tailscale | 100.127.26.75 | N/A|

---

## 2. Gestión de Red (Switches)

Para poder administrar los switches por SSH necesitamos una IP, en los Routers por ejemplo usamos la IP asignada en la interfaz de loopback, pero estos necesitan una IP en una VLAN de gestión. 

| Dispositivo | Propósito | VLAN | Dirección IP | Gateway |
| :--- | :--- | :---: | :--- | :--- |
| JPSWC01 | IP de Gestión | 99 | 172.16.99.10 | 172.16.99.1 (Router Cisco) |
| JPSWA01 | IP de Gestión | 99 | 172.16.99.20 | 172.16.99.1 (Router Cisco) |

---

## 3. Servidores y Hosts

Estos son los dispositivos finales en cada red.

| Host | Red (VLAN) | Interfaz | Dirección IP | Gateway |
| :--- | :--- | :--- | :--- | :--- |
| JDATSR01| Datacenter (5) | ens4 | 172.16.5.100 | 172.16.5.1 |
| JDATSR02| Datacenter (5) | ens4 | 172.16.5.101 | 172.16.5.1 |
| JDEVSR01| Dev (200) | ens4 | 172.16.200.50 | 172.16.200.1 |
| JJUMSR01 | Sysadmin (10) | ens4 | 172.16.10.10 | 172.16.10.1 |
| JDCSR01  | Sysadmin (10) | ens4 | 172.16.10.11 | 172.16.10.1 |
| JPPC01 | Management (N/A) | eth0 | 192.168.1.50 | 192.168.1.1 |