#  Networking Home Lab

Bienvenido a la documentación de mi laboratorio personal. Este espacio es el registro técnico de mi aprendizaje en Linux, Docker, Networking y Observabilidad. Mi enfoque está en la **conectividad híbrida**.

---

##  Arquitectura del Laboratorio

Mi laboratorio combina dispositivos locales en **GNS3** con máquinas virtuales en **Nubes Públicas** (Oracle/AWS), interconectados mediante una VPN (Tailscale).

### 1. Gestión de Identidad y Directorio 
He implementado un entorno de dominio centralizado para gestionar recursos y seguridad:
* **Controlador de Dominio:** Samba sobre **Debian**, configurado como Domain Controller (AD DC).
* **Integración Linux:** Autenticación centralizada en servidores Linux mediante SSSD y resolución de nombres integrada en el dominio.
* **Instalación de RSAT en Windows:** Para administrar de manaera gráfica los usuarios, ous, grupos y políticas.

### 2. Conectividad y Redes Híbridas 
El laboratorio está compuesto por dispositivos en una topología en **GNS3** local, y con instancias de OCI y AWS:
* **LAN->VPN:** Uso de **Tailscale** para crear un túnel persistente entre mi red local y máquinas virtuales públicas en la nube (Oracle Cloud / AWS).
* **Simulación de Topologías:** Diseño y testeo de enrutamiento estático y dinámico en **GNS3** con imágenes Cisco IOS, VyOS y PfSense.
* **Direccionamiento y Protocolos:** Implementación de direccionamiento IPv4, VLANs, NAT, VPN.



### 3. Stack de Monitoreo y Observabilidad 
Para garantizar la salud de los servicios, implementé servicios de métricas basado en contenedores:
* **Recolección:** **Prometheus** como motor de base de datos de series temporales.
* **Exporters:** Uso de `snmp_exporter` para obtener datos en tiempo real de equipos de red (Cisco/VyOS/PfSense).
* **Visualización:** Dashboards avanzados en **Grafana** para monitoreo de tráfico de interfaces y métricas de CPU y memoria.
* **Estado:** **Uptime Kuma** para alertas de disponibilidad, tiempos de respuesta y seguimiento de los certificados SSL.

---

##  Tecnologías Utilizadas

| Categoría | Herramientas y Protocolos |
| :--- | :--- |
| **Sistemas** | Debian (Server), Docker & Docker Compose, Windows 10 (Admin). |
| **Networking** | Cisco IOS, VyOS, PfSense, SNMP, DNS, Tailscale(VPN). |
| **Seguridad/Identidad** | Samba AD, SSH con autenticación de dominio, ACLs. |
| **Observabilidad** | Grafana, Prometheus, Uptime Kuma. |


---
> [!NOTE]
> Este laboratorio está en constante evolución. Cada configuración aquí documentada ha sido instalada, configurada y testeada manualmente.
