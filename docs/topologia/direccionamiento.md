## Tabla de direccionamiento de las subredes

| Propósito / Red | VLAN ID | Red (Subred) | Máscara (CIDR) | Gateway |
| :--- | :---: | :--- | :---: | :--- |
| Default Switches | 1 | 172.16.1.0 | /24 | 172.16.1.1 |
| Datacenter | 5 | 172.16.5.0 | /24 | 172.16.5.1 |
| Dev | 200 | 172.16.200.0 | /24 | 172.16.200.1 |
| Sysadmin | 10 | 172.16.10.0 | /24 | 172.16.10.1 |
| Management | N/A | 192.168.1.0 | /24 | 192.168.1.1 |
| Enlace (JPROO1-JPROO2) | N/A | 10.10.1.0 | /30 | N/A |
| Enlace (JPROO1-JPFW01) | N/A | 10.10.0.0 | /30 | N/A |