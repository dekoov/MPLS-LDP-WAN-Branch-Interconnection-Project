# Github Conectividad entre Departamentos

## Información del Proyecto
---
Este fue mi proyecto de final de curso. En él nuestro equipo se dedicó a simular de forma realista un diseño e implementación de una infraestructura de red y su interconexión entre matriz y sucursal mediante un ISP usando como tecnología a MPLS-LDP. Para ello se usó planifico el proyecto usando la metodología HLD/LLD y fue simulado usando EVE-NG con routers y switches Cisco y máquinas virtuales Linux

## Objetivo del Proyecto
---
El objetivo principal es diseñar e implementar una infraestructura de red segura y escalable que interconecte la sede principal (Matriz) en Quito con una sucursal en Guayaquil. Para ello, se emplearán tecnologías modernas de networking que garanticen la alta disponibilidad de los servicios críticos.

## Sobre la Empresa
---
Usamos como punto de partida para la simulación de la empresa a Santillana.  Se trata de una empresa educativa especializada en la producción de contenidos físicos y digitales, plataformas de enseñanza multiidioma y servicios de capacitación.

![](https://i.imgur.com/KrQbPO1.png)

Gracias a un compañero que estaba trabajando en la empresa en su momento logramos analizar sus departamentos para la elaboración de este proyecto

Departamentos incluidos:
- Gerencia - Recursos Humanos
- Económico - Comercial
- Implementación/TI
- Editorial
- Marketing

## Arquitectura de Red
---

![](https://i.imgur.com/5hol3B1.png)

---
### Matriz (AS-500) - Diseño Jerárquico en 3 Capas

El diseño de la matriz fue jerárquico en 3 capas, lo que permite una segmentación adecuada y separación de funciones. Los routers de borde (RM1 y RM2) se encargan únicamente de la salida hacia el ISP, asegurando disponibilidad mediante **HSRP** y garantizando la propagación de rutas gracias a **OSPF interno** y **BGP/MPLS hacia el proveedor**.

El **Switch de Capa 3 (SW-L3)** centraliza el enrutamiento interno, gestionando las **inter-VLANs**, los servicios de **DHCP** para los distintos departamentos, así como el **acceso remoto por SSH**. Además, se configuró **EtherChannel** para aumentar el ancho de banda y mejorar la tolerancia a fallos en los enlaces principales.

En la **capa de acceso**, los switches (SW1, SW2, SW3) proporcionan segmentación por VLAN a cada departamento, con enlaces redundantes hacia el router interno y hacia el SW-L3, garantizando continuidad en caso de falla de un enlace.

```
Capa de Núcleo:       RM1 + RM2 (HSRP, OSPF, BGP/MPLS)
Capa de Distribución: SW-L3 (Enrutamiento Inter-VLAN, DHCP, SSH, EtherChannel)  
Capa de Acceso:       SW1, SW2, SW3 (VLANs por departamento, redundancia de enlaces)
```

---

### Sucursal (AS-700) - Núcleo Colapsado

El diseño de la sucursal se implementó con un **núcleo colapsado**, lo que reduce la complejidad y los costos de operación. En este esquema, el **router RS1** cumple múltiples funciones: actúa como **router de borde** para el intercambio de rutas con el ISP, gestiona el **enrutamiento inter-VLAN** mediante subinterfaces (_ROAS_), ofrece **DHCP para los hosts internos** y participa en **BGP/MPLS** para la conectividad hacia la matriz.

Los **switches de acceso (SW1 y SW2)** distribuyen el tráfico segmentado por VLAN hacia los departamentos, conectándose directamente con RS1. Esto simplifica el diseño, aunque limita la redundancia y la escalabilidad en comparación con la topología de la matriz.

```
Núcleo/Distribución: RS1 (ROAS, BGP/MPLS, DHCP, Enrutamiento Inter-VLAN)
Capa de Acceso:      SW1, SW2 (VLANs departamentales)
```

---

### ISP (AS-100) - Backbone MPLS

El proveedor de servicios está representado por un **backbone MPLS**, en donde los **routers PE1 y PE2** funcionan como _Provider Edge_ con VRF dedicada para la empresa (_VRF SANTILLANA_). Estos se interconectan mediante un **router P** (Provider Core), el cual únicamente transporta etiquetas MPLS y no almacena información de clientes.

La comunicación entre la matriz y la sucursal se asegura con **BGP** para el intercambio de prefijos y **OSPF** como protocolo de soporte, garantizando convergencia rápida y escalabilidad en el backbone.

```
PE1 ←→ P ←→ PE2
(VRF SANTILLANA con BGP/OSPF sobre MPLS)
```


## Plan de Direccionamiento
---
Matriz - 192.168.0.0/21

| Departamento   | VLAN | Subred           | Gateway       |
| -------------- | ---- | ---------------- | ------------- |
| TI             | 110  | 192.168.0.0/27   | 192.168.0.1   |
| Económico      | 120  | 192.168.0.32/27  | 192.168.0.33  |
| Editorial      | 130  | 192.168.0.64/27  | 192.168.0.65  |
| Marketing      | 140  | 192.168.0.128/28 | 192.168.0.129 |
| RR.HH          | 150  | 192.168.0.144/28 | 192.168.0.145 |
| Gerencia       | 160  | 192.168.0.160/28 | 192.168.0.161 |
| WiFi Invitados | 170  | 192.168.0.96/27  | 192.168.0.97  |

Sucursal - 172.16.0.0/20

| Departamento   | VLAN | Subred          | Gateway      |
| -------------- | ---- | --------------- | ------------ |
| TI             | 210  | 172.16.0.0/27   | 172.16.0.1   |
| Económico      | 220  | 172.16.0.32/27  | 172.16.0.33  |
| Marketing      | 230  | 172.16.0.96/28  | 172.16.0.97  |
| RR.HH          | 240  | 172.16.0.112/29 | 172.16.0.113 |
| Gerencia       | 250  | 172.16.0.120/29 | 172.16.0.121 |
| WiFi Invitados | 260  | 172.16.0.64/27  | 172.16.0.65  |

## Tecnologías Implementadas
---
### Protocolos de Enrutamiento
- **OSPF:** Enrutamiento interno en cada sede.
- **BGP:** Intercambio de rutas entre los sistemas autónomos (500 ↔ 100 ↔ 700).
- **MPLS-LDP:** Transporte etiquetado en el backbone del ISP.
- **VRF:** Aislamiento del tráfico del cliente en la red MPLS.

### Alta Disponibilidad
- **HSRP:** Implementación de un gateway virtual redundante (192.168.0.179).
- **EtherChannel:** Agregación de enlaces mediante PAgP.
- **Redundancia física:** Doble equipo CE en la Matriz.

### Servicios de Red
- **DHCP:** Asignación automática de direcciones IP por VLAN.
- **DNS:** Servidor interno (192.168.0.10) bajo el dominio santillana.local.
- **Web:** Servidor interno dedicado a los servicios corporativos.

### Seguridad
- **VLANs:** Segmentación lógica por departamento.
- **ACLs:** Control de tráfico a nivel granular.
- **SSH:** Acceso administrativo seguro.

## Servicios Validados
---
- Conectividad inter-VLAN operativa en ambas sedes.
- Comunicación entre la Matriz y la Sucursal a través de MPLS.
- Funcionamiento correcto del servicio DHCP en todas las VLANs.
- Resolución DNS para el dominio [www.santillana.com](http://www.santillana.com/).
- Redundancia HSRP activa.
- ACLs aplicadas para seguridad.
- Web: Acceso desde la sucursal al servidor de la matriz operativo.

## Pruebas y validaciones
---

DHCP

![](https://i.imgur.com/u3znfHU.png)


ACL

![](https://i.imgur.com/db3F7PT.png)


EtherChannel

![](https://i.imgur.com/NnAMpSu.png)


HSRP

![](https://i.imgur.com/dJzAeO2.png)

![](https://i.imgur.com/wYv9J7N.png)


DNS

![](https://i.imgur.com/4wzDy0h.png)


VLANs

![](https://i.imgur.com/Ay8jZEY.png)

![](https://i.imgur.com/ycOrv1u.png)


BGP

![](https://i.imgur.com/bfNpmpN.png)

![](https://i.imgur.com/PwBv4Fs.png)


OSPF

![](https://i.imgur.com/6urO6gt.png)


Marcaje LSP

![](https://i.imgur.com/MKJqoBp.png)


LDP

![](https://i.imgur.com/MoScATZ.png)


iBGP VPNv4

![](https://i.imgur.com/9gf0Zjb.png)


VRF

![](https://i.imgur.com/Bi3dmGN.png)


ACL y Acceso a Web Server desde Sucursal

![](https://i.imgur.com/357HGy4.png)


Ping
![](https://i.imgur.com/LeQUilN.png)
