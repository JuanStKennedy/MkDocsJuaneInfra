# :octicons-graph-16: Servicios de Gestión y Monitoreo

Esta sección documenta las herramientas de soporte que hacen que el laboratorio sea funcional, observable, seguro y fácil de usar.

Casi todos estos servicios se ejecutan como contenedores **Docker** en los servidores Debian.

* [**NetBox (Source of Truth)**](netbox.md): La base de datos de qué existe en la red.
* [**Uptime Kuma (Monitoreo)**](kuma.md): El dashboard de monitoreo de los equipos de la topología y servicios.
* [**Homer (Portal)**](homer.md): La página de inicio centralizada para acceder a todo.
* [**Bastion (Jump Server)**](bastion.md): El punto de acceso seguro SSH a la red.
* [**Samba AD DC (Identity Provider)**](samba.md): El servicio de directorio y Controlador de Dominio para la autenticación centralizada (Kerberos/LDAP).
* [**Grafana & Prometheus (Observabilidad)**](grafana.md): El stack para la recolección de métricas históricas SNMP (Tráfico, CPU, RAM) y visualización de datos en tiempo real.