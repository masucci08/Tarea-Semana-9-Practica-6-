# Configuraciones de VPN basadas en SITE TO SITE & CLIENT TO SITE

**Estudiante:** Masucci Franco Rincón  
**Matrícula:** 2024-1250  
**Asignatura:** Seguridad de Redes  
**Fecha:** 20/03/2026  
---
# Link VPN site to site with two fortigates: https://itlaedudo-my.sharepoint.com/:v:/g/personal/20241250_itla_edu_do/IQDfi07wg93jS7SnX9wN34bfAcfvc6bhrnPdDxF50z9h0Bk?e=2l6gX7


# 🚀 PRIMERA PARTE: VPN SITE TO SITE UTILIZANDO 2 FORTIGATES

### Descripción y Topología 
El propósito de este laboratorio es configurar un túnel cifrado (generalmente utilizando IPsec) entre los dos firewalls (FTG-1-WAN y FTG-2-WAN). 

<img width="573" height="388" alt="TOPOLOGIA" src="https://github.com/user-attachments/assets/ad3ecf05-a3d3-4601-8cc4-9d5e39693cbf" />



### Detalles de la Topología

| Dispositivo | Interfaz | Dirección IP | Máscara de Subred | Gateway Predeterminado |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | g1/0 | 10.12.50.129 | 255.255.255.252 (/30) |    |
| **R1** | g2/0 | 10.12.50.133 | 255.255.255.252 (/30) |    |
| **FTG-1-WAN** | port1 | 10.12.50.130 | 255.255.255.252 (/30) |    |
| **FTG-1-WAN** | port2 | 10.12.50.1 | 255.255.255.192 (/26) |    |
| **SWL1** | N/A | N/A |  N/A |  N/A |
| **WebTerm)** | e0 | 10.12.50.10 | 255.255.255.192 (/26) | 10.12.50.1 |
| **FTG-2-WAN** | port1 | 10.12.50.134 | 255.255.255.192 (/26) |    |
| **FTG-2-WAN** | port2 | 10.12.50.65 | 255.255.255.192 (/26) |    |
| **SWL2** | N/A | N/A |  N/A |  N/A |
| **WebTerm** | e0 | 10.12.50.66  | 255.255.255.192 (/26) | 10.20.12.65 |


### Objetivo de red
La red consta de dos sedes o "sitios" locales (representados a la izquierda y a la derecha) que buscan comunicarse de forma segura a través de un enrutador central, el cual simula el proveedor de servicios de internet (ISP) o la red WAN pública.


## Capturas de pantalla de puntos específicos de la configuración con su explicación.
## 1- SHOW IP INTERFACE BRIEF

<img width="492" height="285" alt="show ip int br r1" src="https://github.com/user-attachments/assets/23748a59-24d4-477b-b574-e635f4b3f021" />

En la consola del router R1, se verifica que las interfaces g1/0 y g2/0 están activas con sus respectivas IPs WAN. Al revisar la tabla de enrutamiento
<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">



## 2- SHOW IP ROUTE

<img width="490" height="288" alt="show ip route r1 routing static" src="https://github.com/user-attachments/assets/8ec81d23-0145-47f0-852c-fb25986bdffd" />

Aqui solo existen rutas conectadas directamente (C y L) correspondientes a las subredes /30. el ISP no tiene protocolos de enrutamiento dinámico ni rutas estáticas hacia las redes LAN. El ISP ignora por completo la existencia de las redes 10.12.50.0 y 10.12.50.64. Toda la comunicación entre ellas obligatoriamente dependerá del éxito del túnel VPN.
<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">



## 3- Interfaces de Red en el Firewall

<img width="577" height="432" alt="fortilan1 por1 y por2" src="https://github.com/user-attachments/assets/7503e445-6847-422b-a61d-630a3369c257" />

Se observa el port1 configurado con la IP WAN (10.12.50.130/30) y el port2 con la IP LAN (10.12.50.1/26). Ambos puertos tienen habilitados protocolos de acceso administrativo básico como PING y HTTP/HTTPS para su gestión.
<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">

## 4- Enrutamiento Estático en el FortiGate

<img width="578" height="428" alt="fortilan1 route static forti1y2" src="https://github.com/user-attachments/assets/a219146f-d8d3-4c68-b7c2-f4c289b04f58" />


 - Hay una ruta por defecto (0.0.0.0/0) apuntando al ISP (10.12.50.129) por el port1, necesaria para salir a internet.

 - Hay una ruta específica para la VPN: Todo tráfico destinado a la red remota (VPN-TO-FTG2_remote) se envía a través de la interfaz virtual del túnel VPN, apuntando a la IP pública del FortiGate remoto (10.12.50.134).
<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">


## 5- Políticas de Firewall para la VPN


<img width="577" height="431" alt="policy objects " src="https://github.com/user-attachments/assets/3d1010a3-9c0c-41db-901a-e6d57cdbc8a4" />





### Políticas bidireccionales:
```
 - port2 -> VPN-TO-FTG2: Permite la salida de la LAN local hacia la LAN remota a través del túnel.

 - VPN-TO-FTG2 -> port2: Permite la entrada desde el túnel hacia la LAN local.
```
Para que el tráfico fluya a través del túnel, el FortiGate requiere políticas de seguridad (Firewall Policies) que lo permitan explícitamente.

En ambas políticas, la opción de NAT está deshabilitada (Disabled). En una VPN Site-to-Site, es crucial no traducir las direcciones (NAT) para que los paquetes lleguen con su IP de origen privada intacta y el enrutamiento funcione correctamente.
<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">


## 6- Negociación y Levantamiento del Túnel 


<img width="576" height="430" alt="ipsec tunnels" src="https://github.com/user-attachments/assets/1d623833-228c-4578-88af-433bb02ed0f8" />

 + Este  es el registro de eventos del sistema (Logs) enfocado en la VPN, dentro de la interfaz gráfica del FortiGate. Se esta mostrando el proceso paso a paso de cómo los dos firewalls negocian la conexión segura.
<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">





## 7- Confirmación de Conectividad Extremo a Extremo 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5a28fa81-63fb-48b6-bbbf-ba929f7ca4da" />




> Una consola del equipo cliente de origen (webterm-1 en la LAN izquierda) ejecutando el comando de red básico ping hacia la IP destino 10.12.50.70 (el equipo webterm-2 en la LAN derecha). El parámetro -c 4 le indica que envíe exactamente 4 paquetes de prueba. Es la comprobación práctica. El resumen muestra 0% packet loss (cero paquetes perdidos), lo que significa que los cuatro paquetes enviados llegaron a la otra red y sus respectivas respuestas regresaron con éxito. Esto demuestra a nivel de capa de red que el tráfico está siendo correctamente enrutado hacia el FortiGate local, cifrado, enviado por el ISP, desencriptado por el FortiGate remoto y entregado a la IP destino.

<img src="https://placehold.co/1000x10/4CAF50/4CAF50.png" width="100%" height="10">


