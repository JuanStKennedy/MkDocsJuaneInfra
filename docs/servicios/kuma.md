# 📈 Uptime Kuma (Monitoreo)

> **URL de Acceso:** `https://kuma.js-lab-uy.duckdns.org`

Uptime Kuma es un dashboard de monitoreo simple y de código abierto. En nuestra topología
nos sirve para saber si nuestros dispositivos de red están activos así como también nuestras páginas.

## 1. Propósito (Observabilidad)

En una infraestructura, no es suficiente con "construir" los servicios, debemos asegurarnos de que se mantengan en línea. Uptime Kuma nos da una vista de controlador del estado de todos los componentes críticos del laboratorio.

Nos alerta visualmente (y podría enviar notificaciones) si un servicio o dispositivo se cae, permitiendo un diagnóstico rápido.

## 2. Dashboard de Estado

El dashboard principal centraliza el estado de todos los monitores configurados, mostrando un "UP" (verde) o "DOWN" (rojo) para cada servicio.

![KumaDashboard](../assets/kumaDash.png)

## 3. Monitores Clave

Kuma está configurado para vigilar los componentes más importantes de la topología usando diferentes métodos:

* **Monitores HTTP (Sitios Web):**
    * `NetBox`: Verifica que la interfaz web de NetBox responda correctamente.
    * `Homer Dashboard`: Asegura que el portal de inicio esté accesible.
    * `pfSense WebGUI`: Verifica el estado de la interfaz de gestión del firewall.

* **Monitores Ping (ICMP):**
    * `Gateway pfSense (10.10.0.1)`: El monitor más crítico. Si este se cae, la red está rota.
    * `Loopback JPROO2 (10.255.255.2)`: Verifica que el router de las VLANs esté en línea.
    * `Servidor Bastion (172.16.10.10)`: Asegura que el punto de acceso SSH esté vivo.

* **Monitores de Certificados SSL/TLS:**
    * `https://netbox.js-lab-uy.duckdns.org/`: Kuma también se encarga de monitorear la fecha de vencimiento de los certificados SSL/TLS, avisando 30 días antes de que expiren.

## 4. Despliegue (Docker)

Uptime Kuma se ejecuta como un contenedor Docker liviano en el servidor de servicios (el mismo que NetBox y Homer), gestionado a través de `docker-compose`.

```yaml title="docker-compose.yml"
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1.23.16
    container_name: uptime-kuma
    volumes:
      - ./uptime-kuma-data:/app/data
    ports:
      - 3000:3001  # <Host Port>:<Container Port>
    restart: unless-stopped
```