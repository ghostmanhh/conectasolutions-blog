---
layout: post
title: "Certificado SSL en Zimbra: guía paso a paso y errores comunes"
date: 2026-10-20
categories: [Servidores]
excerpt: "Generar el CSR, validar la cadena e instalar un certificado comercial en Zimbra es sencillo si sigues el orden correcto. Estos son los pasos y los errores que más se repiten."
---

Instalar o renovar el certificado SSL de un servidor de correo **Zimbra** intimida la primera vez, pero el proceso es corto si respetas el orden: generar el CSR, validar los archivos, desplegar y reiniciar. Aquí está el flujo completo.

> Los comandos de `zmcertmgr` se ejecutan como **root**. Los de `zmcontrol`, como usuario **zimbra**.

## Antes de empezar

- El nombre del servidor (por ejemplo `mail.tudominio.com`) debe resolver por DNS a la IP pública.
- Debes tener acceso al correo o al DNS del dominio para que la entidad certificadora valide la solicitud.
- Haz un respaldo del certificado actual:

```bash
cp -a /opt/zimbra/ssl/zimbra /opt/zimbra/ssl/zimbra.bak-$(date +%F)
```

## 1. Generar el CSR

```bash
/opt/zimbra/bin/zmcertmgr createcsr comm -new \
  -subject "/C=EC/ST=Pichincha/L=Quito/O=Tu Empresa/CN=mail.tudominio.com" \
  -subjectAltNames mail.tudominio.com
```

Esto crea dos archivos en `/opt/zimbra/ssl/zimbra/commercial/`:

- `commercial.csr`: lo envías a la entidad certificadora.
- `commercial.key`: la llave privada. **No la compartas ni la borres.**

## 2. Preparar los archivos que entrega la entidad

Necesitas:

- `certificado.crt`: el certificado de tu dominio.
- `cadena.crt`: los certificados intermedios y la raíz, en ese orden. Algunas entidades los entregan separados y hay que unirlos en un solo archivo.

## 3. Verificar antes de desplegar

```bash
/opt/zimbra/bin/zmcertmgr verifycrt comm \
  /opt/zimbra/ssl/zimbra/commercial/commercial.key \
  certificado.crt cadena.crt
```

Debe terminar con `Valid Certificate`. Si falla aquí, no avances: es el paso que evita dejar el servidor con un certificado roto.

## 4. Desplegar y reiniciar

```bash
/opt/zimbra/bin/zmcertmgr deploycrt comm certificado.crt cadena.crt
su - zimbra -c "zmcontrol restart"
```

## 5. Comprobar el resultado

```bash
/opt/zimbra/bin/zmcertmgr viewdeployedcrt
echo | openssl s_client -connect mail.tudominio.com:443 -servername mail.tudominio.com 2>/dev/null | openssl x509 -noout -dates -issuer
```

También conviene abrir el webmail desde un navegador y revisar la cadena, y probar los puertos de correo (465/587 y 993) con un cliente.

## Errores comunes

**`verifycrt` dice que la llave no coincide.** Generaste un CSR nuevo después de pedir el certificado, así que la llave actual ya no corresponde. Hay que usar el CSR original o solicitar el certificado de nuevo.

**Cadena en el orden equivocado.** Debe ir primero el intermedio y al final la raíz. Un orden incorrecto produce advertencias en algunos clientes de correo aunque el navegador funcione.

**El navegador muestra el certificado nuevo, pero el correo sigue con el viejo.** No se reinició algún servicio. Ejecuta `zmcontrol restart` completo, no solo el proxy.

**Nombre que no coincide.** El `CN` o el SAN deben ser exactamente el nombre que usan los clientes para conectarse.

**Certificado vencido por no llevar control.** Anota la fecha de vencimiento en un calendario o programa una revisión mensual de `viewdeployedcrt`.

## Conclusión

La clave es verificar antes de desplegar y tener siempre un respaldo del directorio `ssl`. Con eso, renovar el certificado toma menos de media hora.
