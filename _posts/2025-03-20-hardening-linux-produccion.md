---
layout: post
title: "Checklist de hardening para servidores Linux en producción"
subtitle: "Lo mínimo que debes configurar antes de exponer un servidor a internet"
date: 2025-03-20
categories: [Infraestructura]
tags: [linux, seguridad, hardening, servidores, SSH]
author: "Conecta Solutions"
read_time: 6
excerpt: "Cada semana vemos servidores comprometidos que tenían configuraciones por defecto. Esta lista de verificación cubre los pasos esenciales para endurecer un servidor Linux antes de ponerlo en producción."
---

Un servidor recién instalado con Ubuntu o CentOS está diseñado para funcionar, no para ser seguro. La diferencia entre ambos requiere configuración explícita.

Esta es la checklist que aplicamos en cada servidor que ponemos en producción.

## 1. SSH — el punto de entrada más atacado

```bash
# /etc/ssh/sshd_config — cambios críticos

# Deshabilitar login como root
PermitRootLogin no

# Deshabilitar autenticación por contraseña (solo llaves)
PasswordAuthentication no
PubkeyAuthentication yes

# Cambiar el puerto por defecto (reduce el ruido de bots)
Port 2222

# Limitar tiempo de conexión inactiva
ClientAliveInterval 300
ClientAliveCountMax 2

# Reiniciar el servicio después de cambios
systemctl restart sshd
```

**Por qué importa**: la mayoría de ataques automatizados prueban el puerto 22 con credenciales comunes. Deshabilitar la autenticación por contraseña y mover el puerto elimina el 95% de ese ruido.

## 2. Firewall con UFW o firewalld

```bash
# UFW (Ubuntu / Debian)
ufw default deny incoming
ufw default allow outgoing
ufw allow 2222/tcp    # SSH en el nuevo puerto
ufw allow 80/tcp      # HTTP
ufw allow 443/tcp     # HTTPS
ufw enable

# Verificar estado
ufw status verbose
```

Regla de oro: **deniega todo por defecto, permite solo lo necesario**.

## 3. Fail2ban — bloqueo automático de fuerza bruta

```bash
apt install fail2ban

# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = 2222
maxretry = 4
bantime = 3600
findtime = 600

systemctl enable --now fail2ban
```

Fail2ban revisa los logs y banea automáticamente IPs que intentan autenticarse repetidamente.

## 4. Actualizaciones automáticas de seguridad

```bash
apt install unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades
```

Los parches de seguridad deben aplicarse sin esperar a que alguien lo recuerde.

## 5. Auditoría con auditd

```bash
apt install auditd

# Registrar accesos a archivos sensibles
auditctl -w /etc/passwd -p wa -k passwd_changes
auditctl -w /etc/sudoers -p wa -k sudoers_changes

# Ver eventos
ausearch -k passwd_changes
```

## 6. Verificación rápida del estado

```bash
# Ver puertos abiertos
ss -tlnp

# Procesos corriendo como root
ps aux | awk '$1=="root"'

# Últimos logins
last -n 20

# Intentos de acceso fallidos
grep "Failed password" /var/log/auth.log | tail -20
```

## Lo que esta lista no reemplaza

Este hardening es el piso mínimo, no el techo. Para entornos críticos (bases de datos con PII, sistemas financieros) se necesita ir más lejos: SELinux/AppArmor, segmentación de red, IDS/IPS, y auditorías periódicas.

Si quieres una revisión de la postura de seguridad de tu infraestructura, [contáctanos](/contacto/).
