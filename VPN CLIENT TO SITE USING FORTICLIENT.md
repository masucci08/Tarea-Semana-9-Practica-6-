
# 🛠️ PARTE 2: VPN CLIENT TO SITE BASED POLICIES IKEv2

# Link VPN Client to site: https://itlaedudo-my.sharepoint.com/:v:/g/personal/20241250_itla_edu_do/IQDe7v6u4CjjRb8w_wPEke45ASCyMuGQD2R1Y5FlrMX8xYc?e=pokoag

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
