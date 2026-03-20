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
<img width="476" height="489" alt="ping -c 4" src="https://github.com/user-attachments/assets/df23f911-8f7a-4a96-9273-2eefebaf85da" />


> Una consola del equipo cliente de origen (webterm-1 en la LAN izquierda) ejecutando el comando de red básico ping hacia la IP destino 10.12.50.70 (el equipo webterm-2 en la LAN derecha). El parámetro -c 4 le indica que envíe exactamente 4 paquetes de prueba. Es la comprobación práctica. El resumen muestra 0% packet loss (cero paquetes perdidos), lo que significa que los cuatro paquetes enviados llegaron a la otra red y sus respectivas respuestas regresaron con éxito. Esto demuestra a nivel de capa de red que el tráfico está siendo correctamente enrutado hacia el FortiGate local, cifrado, enviado por el ISP, desencriptado por el FortiGate remoto y entregado a la IP destino.

<img src="https://placehold.co/1000x10/4CAF50/4CAF50.png" width="100%" height="10">



# 🛠️ PARTE 2: VPN CLIENT TO SITE BASED POLICIES IKEv2

# Link VPN Client to site:

# Objetivo de la Red
El propósito de esta infraestructura simulada es implementar y verificar un entorno de VPN Client-to-Site (cliente a sitio) utilizando la solución de software FortiClient.

A diferencia de la VPN Site-to-Site (que conecta dos redes fijas), este escenario permite que un usuario remoto individual (representado por el nodo Forticlient) se conecte de forma segura a través de una red pública no confiable (simulada por R1) hacia el gateway central de la empresa (Fortigate). Una vez autenticado y establecido el túnel cifrado, el dispositivo Forticlient recibe una dirección IP interna y puede acceder a los recursos de la red LAN protegida (webterm-1), como si estuviera físicamente conectado dentro de las oficinas.

El objetivo técnico final es configurar los parámetros del túnel dial-up en el FortiGate, autenticar al cliente remoto y confirmar que el tráfico de red privado fluye exclusivamente a través del túnel VPN utilizando herramientas de diagnóstico como trace-route.



# Topología y Detalle de Direccionamiento IP
<img width="554" height="336" alt="image" src="https://github.com/user-attachments/assets/647309a4-a9ad-4c24-a2f5-7319c08ac7c3" />


| Dispositivo | Interfaz | Dirección IP | Máscara de Subred | Gateway Predeterminado |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | e0/0 | 10.12.50.129 | 255.255.255.252 (/30) |    |
| **R1** | e0/1 | 10.12.50.133 | 255.255.255.252 (/30) |    |
| **R1** | e0/2 | 10.12.50.133 | 255.255.255.252 (/30) |    |
| **Fortigate** | port1 | 10.12.50.130 | 255.255.255.252 (/30) |    |
| **Fortigate** | port2 | 10.12.50.1 | 255.255.255.192 (/26) |    |
| **NAT2** | nat0 | N/A |  N/A |  N/A |
| **WebTerm)** | e0 | 10.12.50.10 | 255.255.255.192 (/26) | 10.12.50.1 |
| **Windows 10 con Forticlient** | e0 | 10.12.50.200 | 255.255.255.255 (/32) |VPN   |
| **Windows 10 sin Forticlient** | e0 | 10.12.50.66 | 255.255.255.192 (/26) | 10.12.50.65    |







## Capturas de pantalla de puntos específicos de la configuración con su explicación.
## 1- Configuración de Enrutamiento para el Túnel (FortiGate)
<img width="1292" height="737" alt="image" src="https://github.com/user-attachments/assets/74cba20a-e3d5-4bdd-808c-585479ef45db" />


> [!NOTE]  
> Esta red 10.12.50.192/26 es el pool de direcciones IP que el FortiGate ha reservado para asignar a los usuarios remotos (como el FortiClient) cuando se conectan. Esta ruta es vital para que la red LAN interna sepa cómo enviar tráfico de respuesta hacia los clientes remotos a través del túnel VPN. 

<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">


## 2- Configuración del Perfil VPN en el Cliente Remoto 
<img width="698" height="650" alt="image" src="https://github.com/user-attachments/assets/6da5c7b3-01de-4601-a44e-6fba04d4b4c8" />


 - Remote Gateway: Está apuntando a 10.12.50.130. Esta es la IP pública de la interfaz port1 del FortiGate (el "destino" público al que el cliente debe llegar para tocar la puerta).

 - Authentication Method: Se utiliza Pre-shared key (una contraseña compartida configurada previamente en el FortiGate) para la autenticación inicial mutua.

 - IKE Proposal (Fase 1): Se negocian los algoritmos de seguridad iniciales. Aquí se ve que se permiten tanto DES con SHA256 como AES256 con SHA1 para encriptar el canal de control.

<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">

## 3- Configuración Avanzada del Perfil VPN 
<img width="694" height="607" alt="image" src="https://github.com/user-attachments/assets/d1b7881d-bda9-40c6-ac49-bd43c65b8de7" />


 1. Options: Está seleccionado Mode Config. Esto es crucial, ya que le dice al FortiClient que, tras autenticarse, debe solicitarle al FortiGate que le asigne una dirección IP virtual, una máscara de subred y (opcionalmente) servidores DNS de forma dinámica desde el pool configurado (el 10.12.50.192/26 que vimos antes).

 2. DH Group: Se selecciona el grupo 14 para el intercambio de claves Diffie-Hellman, que es un estándar seguro moderno.

 3. Local ID: Se configuró como Itla-mas. Este identificador local se envía al FortiGate para que este último sepa qué perfil de VPN aplicar a este usuario en particular, permitiendo tener múltiples configuraciones para distintos grupos de usuarios.
<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">

## 4- Conexión Exitosa y Asignación de IP
<img width="702" height="658" alt="image" src="https://github.com/user-attachments/assets/30a086ec-1b28-481d-93ae-7d728d233096" />

> [!NOTE]
> Esta captura demuestra el éxito de la negociación anterior (Mode Config). El FortiGate aceptó la conexión y le asignó dinámicamente al FortiClient una IP (.200) que pertenece precisamente al pool 10.12.50.192/26 definido en la regla de enrutamiento que vimos en la configuracion del perfil VPN
<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">

## 5- Pruebas de Enrutamiento y Conectividad (Trace-route)
<img width="1668" height="752" alt="image" src="https://github.com/user-attachments/assets/02f88c87-f88b-405d-823b-57af44639e5c" />



>A la izquierda, la terminal de webterm-1 (en la LAN interna).

>A la derecha, el CMD de la máquina Windows (FortiClient).


```
En el CMD de Windows con Forticlient: El comando ipconfig confirma que la máquina física tiene la IP pública 10.12.50.66 (asignada al adaptador Ethernet0), pero el adaptador VPN virtual recibió la IP 10.12.50.201 (una variación del pool, demostrando asignación dinámica).

En webterm-1: El usuario intenta hacer un traceroute hacia la IP física del cliente de Windows (10.12.50.66). Vemos que el tráfico salta al gateway (10.12.50.1), luego al router del ISP (10.12.50.129) y después se pierde.
```
<img src="https://placehold.co/1000x10/f44336/f44336.png" width="100%" height="10">
