# ☁️ Cloud Ingress (Azure Reverse Proxy)
> **Punto de Entrada Público:** `https://netbox.js-lab-uy.ddnsfree.com`, `https://kuma.js-lab-uy.ddnsfree.com`, `https://homer.js-lab-uy.ddnsfree.com`

Este componente es una Máquina Virtual (VM) en **Microsoft Azure** que actúa como el **único punto de entrada público** para los servicios internos del laboratorio.

## 1. Diseño

Se utiliza una VM pequeña y económica en Azure como un proxy reverso.

1.  **Nginx:** Actúa como el proxy reverso. Recibe todo el tráfico público.
2.  **SSL (Let's Encrypt):** Nginx (junto con Certbot) se encarga de gestionar los certificados SSL, proporcionando https a los usuarios finales.
3.  **Tailscale:** La VM de Azure y el firewall pfSense del laboratorio están **ambos en la misma Tailnet (VPN)**.

Esta arquitectura nos permite exponer servicios al público **sin tener que hacer muchos reenvio de puertos en PfSense (Port-Forwarding)**.

## 2. Flujo de Tráfico (Público a Interno)

Este es el flujo exacto que sigue una petición de un usuario:

1.  Un usuario accede a `https://netbox.js-lab-uy.ddnsfree.com`.
2.  El DNS público (un DNS dinámico) apunta ese dominio a la **IP pública de la VM en Azure**.
3.  La VM de Azure recibe la petición en el puerto 443.
4.  **Nginx** la intercepta, la descifra y gestiona la conexión SSL.
5.  Nginx ve la regla de `proxy_pass` que apunta a la IP *interna del laboratorio* (ej. `http://172.16.200.50:8000`).
6.  El sistema operativo de la VM en Azure (gracias a Tailscale) enruta ese paquete a través del túnel VPN hacia el pfSense.
7.  **pfSense** recibe el tráfico de la Tailnet y como comparte las rutas de la LAN, lo reenvía al servidor NetBox.
8.  La respuesta vuelve por el mismo camino.

## 3. El Enlace: Tailscale (Enrutamiento de Subred)

El componente clave es Tailscale.

* El **pfSense** en el laboratorio está configurado para "anunciar" (advertise) las subredes internas (ej. `172.16.5.0/24` y `172.16.200.0/24`) a la Tailnet.
```bash
    tailscale up --advertise-routes=172.16.5.0/24,172.16.200.0/24
```

* La **VM de Azure** está configurada para "aceptar" esas rutas.
```bash
    sudo tailscale up --accept-routes
```
### 3.1 Máquinas de la tailnet

![TailscaleMachines](../assets/tailscale1.png)

### 3.2 Subnet Routes
![TailscaleSubnets](../assets/tailscale2.png)

Esto significa que la VM de Azure sabe cómo llegar a `172.16.5.100` como si estuviera en su misma red local.

## 4. Configuración del Proxy (Nginx)

La configuración de Nginx en la VM de Azure es la que define qué dominio público se mapea a qué servicio interno.

```bash title="/etc/nginx/sites-available/netbox.conf"
server {
    server_name netbox.js-lab-uy.ddnsfree.com;

    location / {
        proxy_pass http://172.16.200.50:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/netbox.js-lab-uy.ddnsfree.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/netbox.js-lab-uy.ddnsfree.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = netbox.js-lab-uy.ddnsfree.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    server_name netbox.js-lab-uy.ddnsfree.com;
    listen 80;
    return 404; # managed by Certbot
}
```