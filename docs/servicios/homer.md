# :doughnut: Homer (Portal de Servicios)

> **URL de Acceso:** `https://homer.js-lab-uy.duckdns.org`

Homer es un dashboard estático y simple, auto-alojado. Es la **"página de inicio"** de todo nuestro laboratorio.

## 1. Propósito

El objetivo de Homer es simple: proporcionar un **punto de acceso único y centralizado** a todas las interfaces web de nuestros servicios.

## 2. Dashboard Principal

El dashboard de Homer está organizado en grupos lógicos (Gestión, Red, etc.) y muestra una tarjeta para cada servicio.

![HomerDashboard](../assets/homer-dashboard.png)



## 3. Configuración

Toda la configuración del dashboard se maneja en un único archivo de texto: `config.yml`.

Este archivo se encuentra en la carpeta de assets (`/opt/homer/assets/`) que se mapea al contenedor. Para añadir un nuevo servicio, simplemente se edita este archivo.

```yaml title="/opt/homer/assets/config.yml (extracto incompleto)"
---

title: "Juane's Infrastructure Dashboard"
subtitle: "Dashboard de Servicios"
icon: "fas fa-house-laptop"

# Columnas de la página de inicio
columns: "4"
defaults:
  layout: list
  colorTheme: dark

# Lista de servicios
services:
  # --- Grupo 1: Gestión y Monitoreo ---
  - name: "Gestión y Monitoreo"
    icon: "fas fa-heartbeat"
    items:
      # --- NetBox ---
      - name: "NetBox (SoT)"
        logo: "https://raw.githubusercontent.com/walkxcode/dashboard-icons/master/png/netbox.png"
        subtitle: "Base de Datos de Red (SoT)"
        tag: "infra"
        url: "https://netbox.js-lab-uy.duckdns.org"
        target: "_blank"
      # --- Uptime Kuma ---
      - name: "Uptime Kuma"
        logo: "https://raw.githubusercontent.com/walkxcode/dashboard-icons/master/png/uptime-kuma.png"
        subtitle: "Monitoreo de Servicios"
        tag: "monitoring"
        url: "https://kuma.js-lab-uy.duckdns.org"
        target: "_blank"

  - name: "Red y Laboratorio"
    icon: "fas fa-network-wired"
    items:
      # --- PfSense ---
      - name: "pfSense (Firewall)"
        logo: "https://raw.githubusercontent.com/walkxcode/dashboard-icons/master/png/pfsense.png"
        subtitle: "Gateway y Firewall"
        tag: "firewall"
        url: "http://100.127.26.75/"
        target: "_blank"
      # --- Nginx proxy Manager ---
      - name: "Nginx Proxy Manager (Proxy Reverso)"
        logo: "https://raw.githubusercontent.com/walkxcode/dashboard-icons/master/png/nginx-proxy-manager.png"
        subtitle: "Proxy reverso en AWS"
        tag: "reverse proxy"
        url: "http://100.74.135.89:81"
        target: "_blank"
      # --- Tailscale ---
      - name: "Tailscale"
        logo: "https://raw.githubusercontent.com/walkxcode/dashboard-icons/master/png/tailscale.png"
        subtitle: "Maquinas de la Tailnet"
        tag: "vpn"
        url: "https://login.tailscale.com/admin/machines"
        target: "_blank"

  - name: "Observabilidad"
    icon: "fas fa-chart-line"
    items:
      - name: "Grafana"
        logo: "https://raw.githubusercontent.com/walkxcode/dashboard-icons/master/png/grafana.png"
        subtitle: "Monitoreo de equipos"
        tag: "observability"
        url: "http://100.124.242.122:3000"
        target: "_blank"
      - name: "Prometheus"
        logo: "https://raw.githubusercontent.com/walkxcode/dashboard-icons/master/png/prometheus.png"
        subtitle: "Recolector de métricas"
        tag: "metrics"
        target: "_blank"
        url: "http://100.124.242.122:9090/"

  # --- Grupo 3: Documentación ---
  - name: "Documentación"
    icon: "fas fa-book"
    items:
      - name: "Portfolio (MkDocs)"
        logo: "assets/icons/mkdocs-light.svg"
        subtitle: "Documentación del Lab"
        tag: "docs"
        url: "http://192.168.1.50:8000"
        target: "_blank"
```

## 4. Despliegue con Docker

```yaml title="docker-compose.yml"
services:
  homer:
    image: b4bz/homer
    container_name: homerDash
    volumes:
      - /opt/homer/assets:/www/assets
    ports:
      - 3000:8080
    user: 1000:1000 # default
    environment:
      - INIT_ASSETS=1
    restart: unless-stopped
```