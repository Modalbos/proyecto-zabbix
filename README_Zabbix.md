# Proyecto Zabbix

**Fecha:** 04/09/20XX  
**Autor:** Idefonso Sánchez Romero

![Portada Zabbix](assets/page1_image1_500x500.png)

---

## Índice

1. [Introducción](#introducción)
   - [Visión general](#visión-general)
   - [Objetivos](#objetivos)
   - [Máquinas](#máquinas)
   - [Software](#software)
   - [Esquema](#esquema)
2. [Organización](#organización)
3. [Elecciones](#elecciones)
4. [Instalación](#instalación)
   - [Preparación inicial](#preparación-inicial)
   - [Instalación de repositorios de Zabbix](#instalación-de-repositorios-de-zabbix)
   - [Instalación de Zabbix, Nginx y MariaDB](#instalación-de-zabbix-nginx-y-mariadb)
   - [Resumen de paquetes instalados](#resumen-de-paquetes-instalados)

---

## Introducción

### Visión general

En este proyecto utilizaré **Zabbix**, una herramienta para monitorizar sistemas centralizados y redes. Además, permite crear alertas para detectar problemas o cambios importantes en los equipos monitorizados.

En este documento se explicará paso a paso todo el proceso realizado, incluyendo posibles fallos y algunas soluciones. Todo el entorno se realizará usando máquinas virtuales.

### Objetivos

1. Crear un entorno donde **Zabbix** sea la herramienta principal de monitorización y aprender a utilizarla.
2. Implementar IA de manera creativa. Esta parte es opcional.

### Máquinas

> Pendiente: cambiar algunas cosas y crear de nuevo algunas máquinas.

| Máquina | Sistema | Función principal | Características |
|---|---|---|---|
| VM-1 | Debian 13 | Servidor de Zabbix | 4 GB RAM / Disco 30 GB |
| VM-2 | Ubuntu Server | Cliente | 2 GB RAM / Disco 30 GB |
| VM-3 | Windows 10 | Cliente | 4 GB RAM / Disco 50 GB |
| VM-4 | Debian | Servidor web y base de datos de pruebas | 2 GB RAM / Disco 50 GB |

### Software

- Zabbix
- MariaDB
- Nginx
- Telegram

### Esquema

![Esquema de red del proyecto](assets/page3_image2_1314x373.png)

El esquema plantea un servidor Zabbix principal con IP `192.168.1.10`, desde el cual se monitorizan diferentes clientes y servicios de la red.

---

## Organización

Para facilitar la configuración, se asignará una **IP fija** a todas las máquinas.

> **Importante para GitHub:** no es recomendable publicar contraseñas reales en un repositorio público. Por seguridad, las contraseñas se han marcado como `***`.

| Máquina | Datos |
|---|---|
| VM1 - Servidor Zabbix | Usuario: `admin`<br>Contraseña: `***`<br>Usuario: `servidor`<br>Contraseña: `***`<br>IP: `192.168.1.10` |
| VM2 - Cliente Ubuntu Server | Nombre servidor: `servidorcliente`<br>Usuario: `cliente1`<br>Contraseña: `***`<br>IP: `192.168.1.120` |
| VM3 - Cliente Windows | Usuario: `usuario`<br>Contraseña: sin contraseña<br>IP: `192.168.1.30` |
| VM4 - Debian servidor web | Usuario: `admin`<br>Contraseña: `***`<br>Usuario 2: `ilde`<br>Contraseña: `***`<br>IP: `192.168.1.40` |

---

## Elecciones

Elegí **Nginx** para la parte web porque consume menos recursos y se integra bien con Zabbix y PHP.

Además, quería utilizar una alternativa distinta a lo usado en clase, por lo que Nginx era una buena opción.

El servidor principal se hará en **Debian**, debido a su estabilidad, su bajo consumo de recursos y su buena compatibilidad con Zabbix.

---

## Instalación

Se irán añadiendo capturas de los comandos ejecutados y de los archivos modificados.

### Preparación inicial

Lo primero será configurar el servidor con una **IP estática** y configurar correctamente el **hostname**.

Después, se actualizará el sistema con:

```bash
apt update
apt upgrade
```

### Instalación de repositorios de Zabbix

Para comenzar con la instalación de Zabbix, se descarga el paquete que añade los repositorios oficiales de Zabbix 7.4 para Debian 13:

```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb
```

Con `wget` se descarga el archivo `.deb` necesario para instalar los repositorios oficiales de la versión 7.4 para Debian 13.

Después, se instala el paquete descargado:

```bash
dpkg -i zabbix-release_latest_7.4+debian13_all.deb
```

Este comando instala los repositorios de Zabbix en el sistema.

A continuación, se actualiza la lista de paquetes:

```bash
apt update
```

![Capturas de instalación de repositorios](assets/page6_image3_1278x701.png)

![Captura de actualización de repositorios](assets/page6_image4_1277x544.png)

### Instalación de Zabbix, Nginx y MariaDB

Una vez instalados los repositorios, se comienza con la instalación de Zabbix, Nginx y MariaDB:

```bash
apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent2 mariadb-server nginx
```

![Captura de instalación de paquetes](assets/page7_image5_1284x771.png)

### Resumen de paquetes instalados

| Paquete | Función |
|---|---|
| `zabbix-server-mysql` | Servidor Zabbix con soporte MySQL/MariaDB |
| `zabbix-frontend-php` | Interfaz web de Zabbix |
| `zabbix-nginx-conf` | Configuración de Nginx para Zabbix |
| `zabbix-sql-scripts` | Scripts iniciales de la base de datos |
| `zabbix-agent2` | Agente moderno de Zabbix |
| `mariadb-server` | Base de datos |
| `nginx` | Servidor web |

---

## Notas pendientes

- Añadir las capturas restantes del proceso.
- Completar la configuración de MariaDB.
- Configurar el frontend web de Zabbix con Nginx.
- Configurar los agentes Zabbix en los clientes.
- Crear triggers, alertas y notificaciones.
- Configurar Telegram para recibir alertas.
- Documentar errores encontrados y soluciones aplicadas.
