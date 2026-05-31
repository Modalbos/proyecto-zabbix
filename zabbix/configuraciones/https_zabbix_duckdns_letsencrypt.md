# 🔐 HTTPS válido en Zabbix con DuckDNS + Let’s Encrypt

> En este apartado explico cómo configurar un dominio gratuito con **DuckDNS** y un certificado HTTPS válido con **Let’s Encrypt** para acceder a la interfaz web de **Zabbix** de forma segura.

---

## 🎯 Objetivo

El objetivo es poder acceder a Zabbix usando una dirección como:

```text
https://proyectoildezabbix.duckdns.org
```

en lugar de acceder solo por IP local:

```text
http://192.168.1.10
```
---

## 🧠 Esquema de funcionamiento

```text
Usuario / Navegador
        |
        | https://proyectoildezabbix.duckdns.org
        |
DuckDNS apunta a la IP pública
        |
Router Movistar
        |
Redirección de puertos 80 y 443
        |
Servidor Zabbix
192.168.1.10
        |
Nginx + Certbot + Let’s Encrypt
        |
Frontend de Zabbix
```

---

## 1. Crear dominio gratuito en DuckDNS

Primero se accede a la página de DuckDNS.

Se crea un subdominio gratuito, por ejemplo:

```text
proyectoildezabbix.duckdns.org
```

DuckDNS asociará ese subdominio con la IP pública de la red.

---

## 2. Comprobar la IP pública

Desde el PC principal se puede buscar en Google:

```text
cuál es mi IP
```

Esa IP pública es la que debe tener asociada DuckDNS.

También se puede comprobar desde terminal:

```bash
curl ifconfig.me
```

---

## 3. Comprobar que DuckDNS apunta correctamente

Después de crear el dominio en DuckDNS, se comprueba con:

```bash
nslookup proyectoildezabbix.duckdns.org
```

El resultado debe devolver la IP pública de la red.

También se puede probar:

```bash
ping proyectoildezabbix.duckdns.org
```

Aunque el ping puede estar bloqueado por algunos routers, lo importante es que el nombre resuelva a la IP pública.

---

## 4. Redirigir puertos en el router

Como el servidor Zabbix está en la máquina virtual:

```text
zabbix-server → 192.168.1.10
```

en el router se deben crear dos reglas NAT o port forwarding.

| Nombre | IP interna | Protocolo | Puerto externo | Puerto interno | Uso |
|---|---|---|---:|---:|---|
| `Zabbix_HTTP` | `192.168.1.10` | TCP | `80` | `80` | Validación HTTP y acceso inicial |
| `Zabbix_HTTPS` | `192.168.1.10` | TCP | `443` | `443` | Acceso HTTPS |

---

## ⚠️ Importante sobre los puertos

No se puede redirigir el mismo puerto externo a dos máquinas distintas.

Por ejemplo, esto no sería válido:

```text
80 → 192.168.1.10
80 → 192.168.1.11
```

Si se quiere configurar HTTPS para Zabbix, los puertos deben apuntar a:

```text
192.168.1.10
```

Si se quisiera configurar HTTPS para otra web, como TecnoStore, entonces habría que apuntarlos a la IP de esa máquina.

---

## 5. Abrir puertos en el firewall del servidor Zabbix

Aunque esto ya lo teniamos hecho pongo de nuevo como hacerlo por si acaso

En el servidor Zabbix:

```bash
su -
```

Abrimos HTTP y HTTPS:

```bash
ufw allow 80/tcp
ufw allow 443/tcp
ufw reload
ufw status
```

### Explicación

| Comando | Función |
|---|---|
| `ufw allow 80/tcp` | Permite tráfico HTTP |
| `ufw allow 443/tcp` | Permite tráfico HTTPS |
| `ufw reload` | Recarga las reglas del firewall |
| `ufw status` | Muestra las reglas activas |

---

## 6. Configurar Nginx de Zabbix con DuckDNS

En el servidor Zabbix editamos el archivo de configuración de Nginx usado por Zabbix:

```bash
nano /etc/zabbix/nginx.conf
```

Buscamos la línea:

```nginx
server_name _;
```

o similar, y la cambiamos por el dominio de DuckDNS:

```nginx
server_name proyectoildezabbix.duckdns.org;
```

También nos aseguramos de que escucha en el puerto 80:

```nginx
listen 80;
```

---

## 7. Comprobar Nginx

Después de modificar el archivo:

```bash
nginx -t
```
Aplicamos los cambios:

```bash
systemctl reload nginx
```

---

## 8. Comprobar acceso HTTP

Antes de pedir el certificado, debe funcionar el acceso por HTTP:

```text
http://proyectoildezabbix.duckdns.org
```

Si se está dentro de la misma red, puede que el router no permita acceder a la IP pública desde dentro. En ese caso se recomienda probar desde el móvil usando datos móviles.

También se puede comprobar con:

```bash
curl -I http://proyectoildezabbix.duckdns.org
```

El resultado esperado es algo parecido a:

```text
HTTP/1.1 200 OK
```

o una redirección normal.

---

## ⚠️ Error común: demasiadas redirecciones

Durante la configuración puede aparecer en el navegador:

```text
ERR_TOO_MANY_REDIRECTS
```

Esto significa que hay un bucle de redirecciones.

Puede ocurrir si Nginx redirige de HTTP a HTTPS antes de tener correctamente configurado HTTPS.

Para solucionarlo, antes de usar Certbot hay que revisar que no exista una línea como:

```nginx
return 301 https://$host$request_uri;
```

o:

```nginx
rewrite ^ https://$host$request_uri permanent;
```

Si existe, se comenta temporalmente hasta que HTTP funcione correctamente.

---

## 9. Instalar Certbot

En el servidor Zabbix instalamos Certbot y el plugin de Nginx:

```bash
apt update
apt install -y certbot python3-certbot-nginx
```

### Explicación

| Paquete | Función |
|---|---|
| `certbot` | Herramienta para solicitar certificados Let’s Encrypt |
| `python3-certbot-nginx` | Permite que Certbot configure Nginx automáticamente |

---

## 10. Solicitar certificado Let’s Encrypt

Cuando el dominio ya funciona por HTTP, ejecutamos:

```bash
certbot --nginx -d proyectoildezabbix.duckdns.org
```

Certbot pedirá algunos datos:

```text
Correo electrónico
Aceptar términos de uso
Elegir si redirigir HTTP a HTTPS
```

Cuando pregunte si se desea redirigir HTTP a HTTPS, se puede elegir que sí.

---

## 11. Comprobar acceso HTTPS

Después de completar Certbot, se accede desde el navegador:

```text
https://proyectoildezabbix.duckdns.org
```

Si todo está correcto, se mostrará la pantalla de inicio de sesión de Zabbix usando HTTPS.

---

## 12. Comprobar certificado instalado

Para ver los certificados instalados:

```bash
certbot certificates
```

Debe aparecer el dominio:

```text
proyectoildezabbix.duckdns.org
```

También se puede comprobar Nginx:

```bash
nginx -t
systemctl status nginx
```

Y que escucha en HTTPS:

```bash
ss -tulpn | grep ':443'
```

---

## 13. Probar renovación automática

Los certificados de Let’s Encrypt caducan, por lo que es importante comprobar la renovación automática.

Ejecutamos una prueba:

```bash
certbot renew --dry-run
```

Si no muestra errores, la renovación está correctamente preparada.

También se puede comprobar si existe temporizador de Certbot:

```bash
systemctl list-timers | grep certbot
```
---

## 14. Crear escenario web HTTPS

También se puede crear un escenario web para comprobar la página completa de Zabbix.

Ruta:

```text
Recopilación de datos → Equipos → zabbix-server → Web → Crear escenario web
```

Configuración del escenario:

```text
Nombre: Web HTTPS Zabbix
Intervalo de actualización: 1m
Intentos: 1
Agente: Zabbix
```

Paso del escenario:

```text
Nombre: Página login Zabbix
URL: https://proyectoildezabbix.duckdns.org
Códigos de estado requeridos: 200
```

Esto comprueba que la web no solo tiene el puerto abierto, sino que responde correctamente por HTTPS.

---

## 15. Pruebas recomendadas

| Prueba | Comando / Acción | Resultado esperado |
|---|---|---|
| Comprobar DNS | `nslookup proyectoildezabbix.duckdns.org` | Devuelve IP pública |
| Comprobar Nginx | `nginx -t` | Configuración correcta |
| Comprobar HTTP | `curl -I http://proyectoildezabbix.duckdns.org` | Respuesta HTTP |
| Comprobar HTTPS | `curl -I https://proyectoildezabbix.duckdns.org` | Respuesta HTTPS |
| Ver certificado | `certbot certificates` | Certificado listado |
| Probar renovación | `certbot renew --dry-run` | Sin errores |
| Comprobar puerto 443 | `ss -tulpn | grep ':443'` | Nginx escuchando |
| Probar caída HTTPS | Parar Nginx | Zabbix genera problema |

---

## 16. Prueba de fallo

Para comprobar que Zabbix detecta la caída del servicio HTTPS:

```bash
systemctl stop nginx
```

Después de unos segundos o minutos, debería aparecer un problema en Zabbix.

Para recuperar:

```bash
systemctl start nginx
```

---

## 17. Seguridad

Publicar la interfaz de Zabbix en Internet requiere aplicar medidas de seguridad.

Se recomienda:

```text
Cambiar la contraseña del usuario Admin
Deshabilitar o limitar el usuario guest
Usar HTTPS
No abrir SSH al exterior
Mantener Zabbix actualizado
Usar contraseñas fuertes
Mantener Tailscale para administración remota
Abrir solo los puertos necesarios
```

---

## 18. Por qué se usó DuckDNS

Se usó DuckDNS porque permite crear un subdominio gratuito sin comprar un dominio propio.

Esto es útil en un proyecto de laboratorio porque permite obtener un dominio público válido para Let’s Encrypt.

Ventajas:

```text
Es gratuito
Es fácil de configurar
Permite usar Let’s Encrypt
No requiere comprar dominio
Sirve para proyectos de pruebas y laboratorio
```

---
## 19. Scripts para duckdns y demas

1- actualizar la ip publica en duckdns

## 20. Por qué se usó Let’s Encrypt

Let’s Encrypt permite obtener certificados TLS gratuitos y reconocidos por los navegadores.

Con esto se evita el aviso de certificado no confiable que aparece con los certificados autofirmados.

Ventajas:

```text
Certificado válido
Reconocido por navegadores
Renovación automática
Gratuito
Integración sencilla con Certbot y Nginx
```

---

## 21. Comparación con otras opciones

| Opción | Ventajas | Desventajas |
|---|---|---|
| Certificado autofirmado | Fácil y no necesita dominio | El navegador muestra aviso |
| CA local | Más profesional en red local | Hay que instalar la CA en los clientes |
| DuckDNS + Let’s Encrypt | Certificado válido y gratuito | Hay que abrir puertos y exponer el servicio |
| Dominio propio + Let’s Encrypt | Opción más profesional | Puede requerir pagar dominio |

---

## 22. Conclusión

Con esta configuración se consiguió acceder al frontend de Zabbix mediante HTTPS usando un dominio gratuito de DuckDNS y un certificado válido de Let’s Encrypt.

Esto mejora la seguridad del acceso web, ya que la comunicación entre el navegador y el servidor se realiza cifrada.

Además, se puede monitorizar el propio servicio HTTPS desde Zabbix mediante una métrica sencilla, un iniciador y un escenario web.
