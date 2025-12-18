# Configuración de VLANs

## 1. Introducción

Las VLANs (Redes de Área Local Virtuales) se utilizan para segmentar la red en dominios de broadcast más pequeños. En esta topología, la arquitectura utilizada es **Router-on-a-Stick**.

* El switch de acceso (JPSWA01): Utiliza puertos de acceso para asignar los dispositivos a sus respectivas VLANs y conecta con el Core mediante un enlace troncal.

* El switch core (JPSWC01): Agrega las conexiones utilizando puertos troncales (trunks) para permitir el paso de todas las VLANs.

* El router (JPROO2): Recibe el tráfico etiquetado a través de un enlace troncal y realiza el enrutamiento Inter-VLAN.

## 2. Tabla de VLANs

| VLAN ID | Nombre (Propósito) | Red (Subred) | Gateway (en JPROO2) |
| :---: | :--- | :--- | :--- |
| 1 | Default | 172.16.1.0/24 | 172.16.1.1 |
| 5 | Datacenter | 172.16.5.0/24 | 172.16.5.1 |
| 10 | Sysadmin | 172.16.10.0/24 | 172.16.10.1 |
| 200 | Dev | 172.16.200.0/24 | 172.16.200.1 |

---

## 3. Configuración del Switch de Acceso (JPSWA01)

`JPSWA01` es un switch Cisco de Capa 2 que conecta a los dispositivos finales pero también tiene un enlace troncal hacia `JPSWC01`.

### Puertos de Acceso (Access Ports)

Estos puertos están asignados a una única VLAN.

* **Interfaz `G0/0` y `G0/1` (Datacenter):**
    * VLAN: 5
    * Dispositivos: `JDATSR01`, `JDATSR02`
* **Interfaz `G1/0` (Dev):**
    * VLAN: 200
    * Dispositivo: `JDEVSR01`
* **Interfaz `G2/0` (Sysadmin):**
    * VLAN: 10
    * Dispositivo: `JJUMSR01`

**Configuración del JPSWA01**
```bash title="Salida de JPSWA01#show vlan brief"
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3, Gi1/1, Gi1/2
                                                Gi1/3, Gi2/1, Gi2/2, Gi2/3
                                                Gi3/0, Gi3/2, Gi3/3
5    datacenter                       active    Gi0/0, Gi0/1
10   sysadmin                         active    Gi2/0
200  dev                              active    Gi1/0

```

---

## 4. Configuración del Switch Core (JPSWC01)

`JPSWC01` actúa como un switch de Capa 2, agregando los enlaces troncales de la red. **No realiza enrutamiento**, simplemente pasa el tráfico de todas las VLANs hacia el router.

### Puertos Troncales (Trunk Ports)

* **Interfaz `G3/1`:**
    * Conexión: Hacia `JPSWA01` (Acceso).
* **Interfaz `G0/0`:**
    * Conexión: Hacia `JPROO2` (Router).

**Configuración de JPSWC01**
```bash title="Configuración de JPSWC01"
! --- Troncal hacia el Switch de Acceso ---
interface GigabitEthernet3/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 media-type rj45
 negotiation auto

! --- Troncal hacia el Router ---
 interface GigabitEthernet3/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 media-type rj45
 negotiation auto
```

