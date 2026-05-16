---
layout: post
title: "Cómo monitorear tu infraestructura de red sin gastar una fortuna"
subtitle: "Herramientas open source que usamos en producción con clientes reales"
date: 2025-05-08
categories: [Infraestructura]
tags: [redes, monitoreo, zabbix, grafana, open source]
author: "Conecta Solutions"
read_time: 7
excerpt: "El monitoreo de red no tiene que ser caro. Con Zabbix, Grafana y un poco de configuración bien hecha, puedes ver todo lo que pasa en tu infraestructura en tiempo real."
---

Uno de los errores más comunes que vemos en empresas medianas es enterarse de que algo falló cuando ya lo reportó un usuario. El monitoreo reactivo es caro: cuesta downtime, cuesta horas de diagnóstico, y cuesta confianza.

Este artículo describe el stack que usamos con clientes en Ecuador para monitoreo proactivo, con herramientas completamente open source.

## El stack que recomendamos

### Zabbix — el núcleo del monitoreo

Zabbix es el motor central. Recolecta métricas de servidores, switches, routers, firewalls y cualquier dispositivo con SNMP o agente instalado.

Lo que más valoramos de Zabbix:

- **Autodiscovery**: detecta automáticamente dispositivos nuevos en la red
- **Triggers configurables**: alertas cuando CPU supera el 85%, cuando un disco llega al 90%, cuando un servicio cae
- **Historial de métricas**: puedes ver qué pasó hace tres semanas a las 2am

Instalación básica del agente en un servidor Linux:

```bash
# Debian / Ubuntu
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
apt update && apt install zabbix-agent2

# Configurar el servidor Zabbix
echo "Server=192.168.1.10" >> /etc/zabbix/zabbix_agent2.conf
echo "Hostname=$(hostname)" >> /etc/zabbix/zabbix_agent2.conf

systemctl enable --now zabbix-agent2
```

### Grafana — la capa visual

Zabbix tiene sus propios gráficos, pero Grafana los supera en presentación y flexibilidad. Conectamos ambos vía plugin y construimos dashboards que nuestros clientes pueden entender sin ser técnicos.

Un dashboard útil para infraestructura debe mostrar:

- Estado de todos los hosts (UP/DOWN)
- Uso de CPU y RAM por servidor en tiempo real
- Tráfico de red por interfaz
- Espacio en disco con proyección de cuándo se llena

### Alertmanager + Email/Telegram

Las alertas de Zabbix las redirigimos a Telegram para el equipo técnico. Un mensaje a las 3am que dice "Servidor DB01 — CPU 98% por 10 minutos" vale más que cualquier dashboard.

## Lo que aprendimos implementando esto

**No monitorees todo desde el día uno.** Empieza con lo crítico: servidores de base de datos, firewalls y el servidor de correo. Agregar demasiados hosts sin madurar las alertas genera ruido y el equipo aprende a ignorarlas.

**Las alertas deben ser accionables.** Si recibes una alerta y no sabes qué hacer con ella, no debería existir. Cada alerta debe tener un runbook asociado.

**El historial de métricas resuelve discusiones.** "¿El servidor estuvo lento ayer?" pasa de ser una pregunta a tener una respuesta de 30 segundos.

¿Tienes una infraestructura que quieres monitorear? [Conversemos](/contacto/).
