# Monitoreo con Grafana y Prometheus

## 1. Resumen de Arquitectura
El sistema de monitoreo opera en un modelo híbrido, en el cuál se ejecuta una instancia en Oracle Cloud, mientras que los dispositivos objetivos (Switches/Routers/Firewalls) se encuentran en una red local **(GNS3)**.

La comunicación entre la nube y el sitio local se asegura mediante **Tailscale (VPN)**.

---

## 2. Habilitar SNMP en dispositivos de red
Para habilitar el protocolo SNMP es los dispositivos ubicados en GNS3 se necesitan ejecutar ciertos comandos que se detallan debajo:

=== "Cisco"

    ```bash
    configure terminal
    snmp-server community public RO
    snmp-server location "Topologia red interna"
    exit
    write memory
    ```

=== "VyOS"

    ```bash
    configure
    set service snmp community public authorization ro
    set service snmp listen-address 10.255.255.1
    commit
    save
    exit
    ```

---

## 3. Conectividad Segura (Tailscale)

Para alcanzar los objetivos SNMP que tienen IPs privadas, la VM de Oracle utiliza la red **Tailnet**, ya que
El firewall PfSense es quien comparte las redes, y la vm de Oracle acepta estas rutas para alcanzar estas redes. De esta manera tengo conectividad con los dispositivos de mi topología en GNS3.


![subnet-router-pfsense](../assets/tailscale-routing-pfsense.png)

### 3.1 Flujo de Datos SNMP
1. **Origen:** Prometheus inicia el scrape en la VM de Oracle.
2. **Enrutamiento:** La tabla de rutas del OS dirige el tráfico hacia alguna IP destino de la topología local en la red interna, por ejemplo `192.168.1.0/24` hacia la interfaz `tailscale0`.
3. **Transporte:** Los paquetes se encriptan y viajan por internet hacia el **Subnet Router** en el sitio local.
4. **Destino:** El Subnet Router entrega la petición SNMP al PfSense en su red LAN.

!!! info 
    Gracias a esta arquitectura, **no es necesario abrir el puerto UDP 161** en el firewall.

---

## 4. Reglas de Firewall (Security Lists OCI)

La superficie de ataque se ha reducido al mínimo. **No se permiten conexiones entrantes a los puertos 3000, 9090 o 9116 desde internet.**

Solo se mantienen las reglas por defecto de Oracle para administración básica.

![firewall-rules](../assets/rules-oci.png)

### 4.1 Acceso a los Servicios

Dado que los puertos no están abiertos en la IP Pública, el acceso se realiza utilizando la **IP de Tailscale** del servidor.

**Requisito:** El dispositivo desde donde se accede debe tener el cliente Tailscale conectado y estar dentro de la misma Tailnet.

| Servicio | URL de Acceso |
| :--- | :--- |
| **Grafana** | `http://100.124.242.122:3000` |
| **Prometheus** | `http://100.124.242.122:9090` |
| **SNMP Exporter** | `http://100.124.242.122:9116` |

> *En el apartado de servicios -> grafana está todo más detallada su configuración, o haciendo click en el enlace* [Grafana](../servicios/grafana.md)