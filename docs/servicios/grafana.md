# :octicons-meter-16: Grafana & Prometheus

Este stack proporciona la capa de **Observabilidad y Telemetría** de la red. A diferencia de Uptime Kuma (que solo verifica si un equipo responde), este sistema recolecta métricas históricas detalladas: consumo de ancho de banda, carga de CPU por núcleo, uso de memoria RAM y estabilidad de las peticiones SNMP.

Este servicio se ejecuta en una instancia de **Oracle Cloud Infrastructure (OCI)**, fuera de la red física del laboratorio.
    
La conexión con los dispositivos locales se realiza de forma segura y transparente a través de un túnel **Tailscale**.

## 🏗️ Arquitectura de Red

* **No hay puertos expuestos a Internet:** Los puertos 3000 (Grafana) y 9090 (Prometheus) están bloqueados en el firewall de Oracle.
* **Acceso VPN:** Para ver los dashboards o consultar datos, es obligatorio estar conectado a la red Tailscale.
* **Recolección de Datos:** Prometheus (en la nube) alcanza las IPs privadas de la LAN (`192.168.1.x`) enrutando el tráfico a través del nodo Tailscale local (Subnet Router).



## 🚀 Despliegue (Docker Compose)

El stack se levanta mediante contenedores Docker en la VM de Oracle.

**Ruta:** `~/monitoreo-red/docker-compose.yml`

```yaml
services:
  grafana:
    image: grafana/grafana:11.1.0
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - monitoring
    restart: unless-stopped

  prometheus:
    image: prom/prometheus:v2.53.0
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    networks:
      - monitoring
    restart: unless-stopped

  snmp_exporter:
    image: prom/snmp-exporter:v0.26.0 
    container_name: snmp_exporter
    ports:
      - "9116:9116"
    volumes:
      - ./snmp.yml:/etc/snmp_exporter/snmp.yml
    command:
      - '--config.file=/etc/snmp_exporter/snmp.yml'
    networks:
      - monitoring
    restart: unless-stopped

volumes:
  grafana_data:
  prometheus_data:

networks:
  monitoring:
```

##  Prometheus (prometheus.yml)

Define los objetivos. Nótese el uso de IPs privadas gracias a Tailscale y la separación de módulos por tipo de dispositivo.

```yaml 
scrape_configs:
  - job_name: 'snmp-routers'
    scrape_interval: 60s
    scrape_timeout: 30s
    static_configs:
      - targets: ['172.16.99.20']
        labels:
          module: vyos
          device_type: router
          hostname: vyos-gw

      - targets: ['172.16.99.1']
        labels:
          module: pfsense
          device_type: firewall
          hostname: pfsense-gw
```