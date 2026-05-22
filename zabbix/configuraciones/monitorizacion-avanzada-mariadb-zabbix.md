# 🛢️ Monitorización avanzada de MariaDB con Zabbix Agent 2

> En esta sección se explica cómo monitorizar MariaDB de una forma más profesional usando **Zabbix Agent 2** y la plantilla **MySQL by Zabbix agent 2**.

---

## 🎯 Objetivo

Hasta ahora se podía comprobar MariaDB de forma básica con una métrica como:

```text
net.tcp.service[tcp,,3306]
```

Esa comprobación sirve para saber si el puerto `3306` responde, pero **no permite ver métricas internas** de la base de datos.

Con la monitorización avanzada podremos ver información como:

```text
Conexiones activas
Consultas por segundo
Errores de conexión
Uptime de MariaDB
Bytes enviados y recibidos
Uso de InnoDB
Estado interno del servicio
```

---

## 🧠 Diferencia entre monitorización básica y avanzada

| Tipo | Qué comprueba | Nivel |
|---|---|---|
| Básica | Si el puerto `3306` está abierto | Sencillo |
| Avanzada | Métricas internas reales de MariaDB | Profesional |

La forma avanzada es mejor para la memoria porque demuestra que no solo comprobamos si el servicio está abierto, sino también si **funciona correctamente por dentro**.

---

## 1. Crear usuario de monitorización en MariaDB

Primero entramos en MariaDB como root:

```bash
mariadb
```

Creamos un usuario específico para Zabbix:

```sql
CREATE USER IF NOT EXISTS 'zbx_monitor'@'localhost' IDENTIFIED BY 'zabbix_monitor_pass';
```

Este usuario se usará solo para consultar métricas, no para administrar la base de datos.

Ahora le damos permisos mínimos necesarios:

```sql
GRANT USAGE, PROCESS, REPLICATION CLIENT, SHOW DATABASES, SHOW VIEW
ON *.* TO 'zbx_monitor'@'localhost';
```

Aplicamos cambios:

```sql
FLUSH PRIVILEGES;
```

Comprobamos permisos:

```sql
SHOW GRANTS FOR 'zbx_monitor'@'localhost';
```

Salimos:

```sql
EXIT;
```

---

## 🔐 ¿Por qué crear un usuario específico?

No es recomendable usar el usuario `root` de MariaDB para monitorización.

Crear un usuario separado permite aplicar el principio de mínimo privilegio:

```text
Zabbix puede leer métricas
Zabbix no puede modificar datos
Zabbix no usa credenciales administrativas
```

Esto mejora la seguridad del sistema.

---

## 2. Probar el usuario manualmente

Antes de configurar Zabbix, probamos que el usuario funciona:

```bash
mariadb -u zbx_monitor -p -e "SHOW GLOBAL STATUS LIKE 'Uptime';"
```

Contraseña usada:

```text
zabbix_monitor_pass
```

Si funciona, veremos algo parecido a:

```text
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Uptime        | 1234  |
+---------------+-------+
```

Esto confirma que el usuario puede consultar información interna de MariaDB.

---

## 3. Configurar el plugin MySQL de Zabbix Agent 2

Editamos el archivo del plugin:

```bash
nano /etc/zabbix/zabbix_agent2.d/plugins.d/mysql.conf
```

Al final del archivo añadimos:

```text
Plugins.Mysql.Sessions.Local.Uri=tcp://localhost:3306
Plugins.Mysql.Sessions.Local.User=zbx_monitor
Plugins.Mysql.Sessions.Local.Password=zabbix_monitor_pass
```

### 📌 Explicación

| Línea | Función |
|---|---|
| `Uri` | Indica dónde está MariaDB |
| `User` | Usuario usado para monitorizar |
| `Password` | Contraseña del usuario de monitorización |

En este caso usamos:

```text
tcp://localhost:3306
```

porque MariaDB está en la misma máquina que el agente Zabbix.

---

## 4. Reiniciar Zabbix Agent 2

Después de modificar la configuración:

```bash
systemctl restart zabbix-agent2
```

Comprobamos el estado:

```bash
systemctl status zabbix-agent2
```

Debe aparecer:

```text
active (running)
```

Si hay errores, podemos revisar el log:

```bash
tail -n 50 /var/log/zabbix/zabbix_agent2.log
```

---

## 5. Probar el plugin localmente

Desde el servidor donde está MariaDB ejecutamos:

```bash
zabbix_agent2 -t 'mysql.ping["tcp://localhost:3306","zbx_monitor","zabbix_monitor_pass"]'
```

Resultado correcto:

```text
[s|1]
```

También podemos comprobar la versión:

```bash
zabbix_agent2 -t 'mysql.version["tcp://localhost:3306","zbx_monitor","zabbix_monitor_pass"]'
```

Si esto funciona, significa que:

```text
El plugin MySQL funciona
El usuario es correcto
La contraseña es correcta
MariaDB responde
```

---

## 6. Probar desde el servidor Zabbix

Desde el servidor Zabbix principal:

```bash
zabbix_get -s 192.168.1.11 -k 'mysql.ping["tcp://localhost:3306","zbx_monitor","zabbix_monitor_pass"]'
```

Resultado esperado:

```text
1
```

También podemos probar:

```bash
zabbix_get -s 192.168.1.11 -k 'mysql.version["tcp://localhost:3306","zbx_monitor","zabbix_monitor_pass"]'
```

---

## 7. Añadir la plantilla correcta en Zabbix

En la interfaz web de Zabbix vamos a:

```text
Recopilación de datos → Equipos → servidor-extra-debian → Plantillas
```

La plantilla que debemos usar es:

```text
MySQL by Zabbix agent 2
```

⚠️ Importante:

No usar:

```text
MySQL by Zabbix agent
```

si estamos trabajando con **Zabbix Agent 2**, ya que usa diferentes macros y plugings, usaremos agent 2 debido sobretodo por que lo que monitorizaremos sera un cliente.

---

## 8. Configurar macros del host

En el host vamos a:

```text
Recopilación de datos → Equipos → servidor-extra-debian → Macros
```

Añadimos estas macros:

| Macro | Valor |
|---|---|
| `{$MYSQL.DSN}` | `tcp://localhost:3306` |
| `{$MYSQL.USER}` | `zbx_monitor` |
| `{$MYSQL.PASSWORD}` | `zabbix_monitor_pass` |

Después guardamos los cambios.

---

## ⚠️ Error común: Access denied for user '3306'@'localhost'

Durante la configuración puede aparecer un error como:

```text
Cannot fetch data: Access denied for user '3306'@'localhost'
```

Este error significa que Zabbix está usando `3306` como si fuera el usuario de MariaDB.

Esto suele ocurrir por:

```text
Macro mal configurada
Plantilla incorrecta
Macros heredadas cruzadas
```

---

## ✅ Configuración correcta de macros

| Elemento | Valor correcto |
|---|---|
| Usuario | `zbx_monitor` |
| Contraseña | `zabbix_monitor_pass` |
| Puerto | `3306` |
| DSN | `tcp://localhost:3306` |

La macro:

```text
{$MYSQL.USER}
```

debe tener este valor:

```text
zbx_monitor
```

Nunca debe tener:

```text
3306
```

---

## 9. Revisar macros heredadas

Si el error continúa, revisamos:

```text
Macros → Macros heredadas y de equipo
```

Ahí comprobamos que no exista una macro heredada incorrecta como:

```text
{$MYSQL.USER} = 3306
```

Si aparece algo mal, se debe corregir o sobrescribir en las macros del equipo.

---

## 10. Comprobar métricas en Zabbix

Después de aplicar la plantilla y las macros, esperamos unos minutos y vamos a:

```text
Monitorización → Datos más recientes
```

Filtramos por:

```text
servidor-extra-debian
```

Buscamos métricas relacionadas con:

```text
MySQL
MariaDB
Connections
Uptime
Queries
Status variables
```

Si todo está bien, las métricas deberían aparecer como **activadas** y con datos.

---

## 11. Qué métricas se obtienen

Con la plantilla avanzada se pueden obtener datos como:

| Métrica | Utilidad |
|---|---|
| Uptime | Tiempo que MariaDB lleva activo |
| Threads connected | Conexiones activas |
| Questions | Consultas ejecutadas |
| Slow queries | Consultas lentas |
| Bytes sent/received | Tráfico de datos |
| Connection errors | Errores de conexión |
| InnoDB status | Estado interno de InnoDB |

---

## 12. Comandos útiles de diagnóstico

```bash
systemctl status mariadb
```

Comprueba si MariaDB está activo.

```bash
systemctl status zabbix-agent2
```

Comprueba si el agente Zabbix funciona.

```bash
ss -tulpn | grep 3306
```

Comprueba si MariaDB escucha en el puerto 3306.

```bash
tail -f /var/log/zabbix/zabbix_agent2.log
```

Muestra errores del agente Zabbix.

```bash
mariadb -u zbx_monitor -p -e "SHOW GLOBAL STATUS LIKE 'Uptime';"
```

Comprueba manualmente el usuario de monitorización.

---

## 14. Resumen final

Con esta configuración conseguiremos una monitorización más profesional de MariaDB.

En vez de limitarse a comprobar si el puerto `3306` está abierto, Zabbix puede consultar métricas internas del servicio mediante Zabbix Agent 2.

Esto permite detectar problemas de rendimiento, errores de conexión, caída del servicio y otros datos importantes para la administración de la base de datos y el sistema.

---
