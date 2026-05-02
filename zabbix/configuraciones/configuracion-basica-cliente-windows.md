# Configuración del cliente Windows en Zabbix

Aquí configuramos el cliente Windows.

Primero configuramos la **IP estática** y además **desactivamos el firewall**.

---

## 1. Instalación del agente en Windows

La instalación en Windows es prácticamente lo mismo.

Vamos a la página de **Zabbix Agents**:

[Descargar Zabbix Agents](https://www.zabbix.com/download_agents)

Bajando en la página, descargamos esta versión, que es la que utilizamos en los apartados anteriores:

![Versión del agente 1](../imagenes/con-win/version1.png)

Nos aseguramos de que ponga esto:

![Versión del agente 2](../imagenes/con-win/version2.png)

---

## 2. Instalación

Iniciamos el ejecutable:

![Ejecutable 1](../imagenes/con-win/eje1.png)

Y lo configuramos tal que así:

![Ejecutable 2](../imagenes/con-win/eje2.png)

![Ejecutable 3](../imagenes/con-win/eje3.png)

Debemos darle que **sí** a lo siguiente:

![Ventana emergente](../imagenes/con-win/popup.png)

---

## 3. Comprobación de servicios

Comprobamos que los servicios estén activos:

![Servicios 1](../imagenes/con-win/servicios.png)

![Servicios 2](../imagenes/con-win/sertvicios2.png)

---

## 4. Comprobar la configuración del agente

Para asegurarnos de que está configurado, podemos meternos en este archivo:

```text
ruta
```

Y mirar estos datos:

![Datos 1](../imagenes/con-win/dat1.png)

![Datos 2](../imagenes/con-win/dat2.png)

---

## 5. Comprobación desde el servidor

Y comprobamos desde el servidor:

![Comprobación desde servidor](../imagenes/con-win/comp1.png)

---

## 6. Configuración en la web de Zabbix

Y lo configuramos en la web de Zabbix:

![Configuración web 1](../imagenes/con-win/cap1.png)

![Configuración web 2](../imagenes/con-win/cap2.png)
