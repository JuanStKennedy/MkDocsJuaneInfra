# 🛠️ Networking Home Lab

Bienvenido a la documentación de mi laboratorio personal. Este espacio es el registro técnico de mi aprendizaje en Linux, networking y observabilidad. Mi enfoque está en la **interconectividad híbrida**.

---

## 🏗️ Arquitectura del Laboratorio

Mi laboratorio combina dispositivos locales **On-Premise** con **Cloud Publica** (Oracle/AWS), interconectados mediante una VPN (Tailscale).

### 1. Gestión de Identidad y Directorio 
He implementado un entorno de dominio centralizado para gestionar recursos y seguridad:
* **Controlador de Dominio:** Samba sobre **Debian**, configurado como Domain Controller (AD DC).
* **Integración Linux:** Autenticación centralizada en servidores Linux mediante SSSD y resolución de nombres integrada en el dominio.

### 2. Conectividad y Redes Híbridas 
El laboratorio no es una isla; está diseñado para simular un entorno empresarial real:
* **SD-WAN / Mesh VPN:** Uso de **Tailscale** para crear un túnel persistente entre mi red local y nodos en la nube (Oracle Cloud / AWS).
* **Simulación de Topologías:** Diseño y testeo de enrutamiento estático y dinámico en **GNS3** con imágenes Cisco IOS.
* **Core Técnico:** Implementación de direccionamiento IPv4, VLANs y servicios de red esenciales (DNS/DHCP).



### 3. Stack de Monitoreo y Observabilidad 
Para garantizar la salud de los servicios, implementé un pipeline de métricas basado en contenedores:
* **Recolección:** **Prometheus** como motor de base de datos de series temporales.
* **Exporters:** Uso de `snmp_exporter` para obtener datos en tiempo real de equipos de red (Cisco/VyOS).
* **Visualización:** Dashboards avanzados en **Grafana** para monitoreo de tráfico de interfaces y disponibilidad de servicios.
* **Estado:** **Uptime Kuma** para alertas de disponibilidad y tiempos de respuesta.

---

## 🛠️ Tecnologías Utilizadas

| Categoría | Herramientas y Protocolos |
| :--- | :--- |
| **Sistemas** | Debian (Server), Docker & Docker Compose, Windows 10 (Admin). |
| **Networking** | Cisco IOS, SNMP, DNS, Tailscale. |
| **Seguridad/Identidad** | Samba AD, SSH con autenticación de dominio, ACLs. |
| **Observabilidad** | Grafana, Prometheus, Uptime Kuma. |

---

---
> [!NOTE]
> Este laboratorio está en constante evolución. Cada configuración aquí documentada ha sido instalada, configurada y testeada manualmente.
