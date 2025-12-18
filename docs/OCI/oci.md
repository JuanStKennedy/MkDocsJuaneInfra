# Monitoreo con Grafana y Prometheus

## 1. Resumen de Arquitectura
El sistema de monitoreo opera en un modelo híbrido Cloud-OnPremise. El servidor principal se aloja en **Oracle Cloud Infraestructure (OCI)**, mientras que los dispositivos objetivos (Routers/Firewalls) se encuentran en una red local (Lab/On-Premise).

La comunicación entre la nube y el sitio local se asegura mediante **Tailscale (VPN Mesh)**.

## 2. Detalles de la Instancia (OCI)

| Recurso | Detalle |
| :--- | :--- |
| **Proveedor** | Oracle Cloud Infrastructure |
| **OS** | Ubuntu 24.04 LTS |
| **Shape** | VM.Standard.E3.Flex |
| **Rol** | Nodo de Monitoreo (Prometheus/Grafana) |
| **IP Pública** | `163.176.179.69` |
| **IP Privada (Tailscale)** | `100.124.242.122` (Tráfico de Gestión) |

### 2.1 Conectividad Segura (Tailscale)

Para alcanzar los objetivos SNMP que tienen IPs privadas, la VM de Oracle utiliza la red **Tailnet**, ya que
El firewall PfSense es quien comparte las redes, y la vm de Oracle acepta estas rutas para alcanzar estas redes. De esta manera tengo conectividad con los dispositivos de mi topología en GNS3.

![subnet-router-pfsense](../assets/subnet-advertise.png)

### 2.2 Flujo de Datos SNMP
1. **Origen:** Prometheus inicia el scrape en la VM de Oracle.
2. **Enrutamiento:** La tabla de rutas del OS dirige el tráfico hacia alguna IP destino de la topología local en la red interna, por ejemplo `192.168.1.0/24` hacia la interfaz `tailscale0`.
3. **Transporte:** Los paquetes se encriptan y viajan por internet hacia el **Subnet Router** en el sitio local.
4. **Destino:** El Subnet Router entrega la petición SNMP al PfSense en su red LAN.

"Ventaja de Seguridad"
Gracias a esta arquitectura, **no es necesario abrir el puerto UDP 161** en el firewall, minimizando posibles ataques a la red.

## 3. Reglas de Firewall (Security Lists OCI)

La superficie de ataque se ha reducido al mínimo. **No se permiten conexiones entrantes a los puertos 3000, 9090 o 9116 desde internet.**

Solo se mantienen las reglas por defecto de Oracle para administración básica.

![firewall-rules](../assets/rules-oci.png)

### 3.1 Acceso a los Servicios

Dado que los puertos no están abiertos en la IP Pública, el acceso se realiza utilizando la **IP de Tailscale** del servidor.

**Requisito:** El dispositivo desde donde se accede debe tener el cliente Tailscale conectado y estar dentro de la misma tailnet.

| Servicio | URL de Acceso |
| :--- | :--- |
| **Grafana** | `http://100.124.242.122:3000` |
| **Prometheus** | `http://100.124.242.122:9090` |
| **SNMP Exporter** | `http://100.124.242.122:9116` |