# INFORME TÉCNICO: DISEÑO PRELIMINAR DE LA INFRAESTRUCTURA DE RED PARA LA ALCALDÍA DE MORONA

**Proyecto Práctico Experimental - Avance Inicial**

## 1. Descripción de la Organización y Necesidades de Red

### 1.1 Datos generales de la organización

La **Alcaldía de Morona** (Gobierno Autónomo Descentralizado Municipal del cantón Morona) es
una institución de carácter público, sin fines de lucro, cuya finalidad es la administración de
los recursos y servicios municipales, la planificación del desarrollo urbano y rural, y la
atención directa a la ciudadanía del cantón Morona. La institución depende de un presupuesto
público y presta servicios esenciales que deben permanecer disponibles durante toda la jornada
laboral, por lo que la continuidad de sus sistemas informáticos es un requisito crítico.

Su edificio central (objeto de este estudio) consta de **dos plantas** y alberga las áreas
administrativas y de atención al público. La organización se encuentra en proceso de
modernización tecnológica, y la red de datos actual se ha construido de forma empírica y
desordenada, sin segmentación ni documentación, lo que ha generado los problemas que este
informe busca resolver.

### 1.2 Estructura física y departamental

El edificio central alberga cuatro departamentos estratégicos, distribuidos de la siguiente
manera:

- **Planta Baja:**
  - **Atención Ciudadana y Registro:** Área de cara al público. Presenta alta concurrencia
    de usuarios y visitantes diarios, y requiere acceso rápido a sistemas de consulta,
    ventanillas de registro y bases de datos locales.
  - **Departamento Financiero:** Maneja recaudaciones, tesorería, nómina, presupuesto y
    transferencias bancarias. Exige los más altos niveles de seguridad, aislamiento y
    confidencialidad de la información, ya que administra datos sensibles de la
    institución y de terceros.
- **Planta Alta:**
  - **Planificación y Obras Públicas:** Requiere un elevado ancho de banda para la
    transferencia de archivos pesados (planos, modelos CAD, mapas GIS, imágenes
    satelitales) entre estaciones de trabajo y servidores de archivos.
  - **Tecnologías de la Información (TI) / Administración:** Encargado de la gestión de
    servidores, el monitoreo de la red, la administración de usuarios y el soporte técnico
    institucional. Aquí se ubica el rack principal (data center) del edificio.

Para mantener el proyecto manejable y verificable en Packet Tracer, se plantea un escenario
representativo de alrededor de **30 dispositivos simulados** en total. No se dibuja un equipo
por cada funcionario real, sino una muestra por departamento que permite validar la
segmentación, el direccionamiento y las políticas de seguridad.

#### 1.2.1 Distribución estimada de dispositivos por departamento

| Departamento | Planta | PCs / terminales | Impresoras | Servidores |
|---|---|---|---|---|
| Atención Ciudadana y Registro | Baja | 5 | 1 | — |
| Departamento Financiero | Baja | 4 | — | — |
| Planificación y Obras Públicas | Alta | 5 | 1 | — |
| TI / Administración | Alta | 4 | — | 2 |
| **Subtotal departamentos** | | **18** | **2** | **2** |

A estos se suman la **infraestructura de red** (1 router-firewall, 1 router ISP, 1 switch Core
de Capa 3, 2 switches de acceso —uno por planta— y 2 Access Points) y **2 laptops** para
demostrar las redes inalámbricas, alcanzando un total aproximado de **30 dispositivos**.

### 1.3 Situación actual de la red

La red existente presenta las siguientes deficiencias:

- **Red plana sin segmentación:** Todas las áreas comparten el mismo dominio de difusión
  (broadcast domain), lo que provoca degradación del rendimiento y **tormentas de broadcast**.
- **Ausencia de políticas de seguridad:** No existe separación lógica entre el tráfico
  financiero y el de atención pública, exponiendo información confidencial.
- **Cableado desordenado y sin categoría certificada:** Mezcla de cables y switches no
  administrables, sin documentación de puntos de red.
- **Sin monitoreo ni gestión centralizada:** Imposibilidad de identificar fallas, cuellos de
  botella o tráfico malicioso de forma proactiva.
- **Wi-Fi sin control:** Puntos de acceso aislados sin segmentación entre red institucional y
  red de invitados.

### 1.4 Análisis de Necesidades Tecnológicas

A partir de la situación descrita, se identifican las siguientes necesidades críticas que el
diseño de la nueva infraestructura debe satisfacer:

1. **Conectividad concurrente y estable:** Garantizar el acceso sin interrupciones a los
   sistemas institucionales, servidores locales y servicios de Internet (proporcionados por
   el ISP local), soportando la concurrencia simultánea del personal y del público.
2. **Seguridad y aislamiento de datos:** El Departamento Financiero no debe compartir
   visibilidad de red directa con las terminales de consulta pública ni con el resto de
   departamentos, aplicando segmentación lógica y políticas de control de acceso.
3. **Gestión centralizada:** Capacidad para monitorear el tráfico, aplicar políticas de
   Calidad de Servicio (QoS), priorizar aplicaciones críticas y resolver fallas rápidamente
   desde el área de TI.
4. **Alto rendimiento para datos pesados:** Ancho de banda suficiente y conmutación de alta
   velocidad para el traslado de archivos grandes (CAD/GIS) en Planificación y Obras Públicas.
5. **Conectividad inalámbrica segmentada:** Wi-Fi diferenciado para personal/institucional y
   para visitantes, con aislamiento entre ambas redes.
6. **Escalabilidad y disponibilidad:** Posibilidad de ampliar usuarios y servicios sin
   rediseñar la red, minimizando puntos únicos de falla y facilitando el mantenimiento.

## 2. Diseño de la Arquitectura de Red Propuesta

### 2.1 Criterios de diseño

La selección de la arquitectura parte de los siguientes criterios, derivados del análisis de
necesidades de la sección 1:

- **Tamaño y alcance:** LAN de un solo edificio con dos plantas y alrededor de 30-35
  dispositivos finales. No requiere enlaces WAN propios ni infraestructura de operador.
- **Segmentación:** cuatro departamentos con distintos niveles de confidencialidad y consumo
  de ancho de banda.
- **Gestión centralizada:** el área de TI debe poder administrar, monitorear y aplicar QoS
  desde un único punto.
- **Confiabilidad:** los fallos de un puesto o de un cable no deben afectar al resto de la red.
- **Escalabilidad:** debe permitir añadir áreas o servicios sin rediseñar la red existente.
- **Costo:** presupuesto público acotado, por lo que se busca el mejor equilibrio
  costo/beneficio.

### 2.2 Modelo jerárquico por capas (núcleo colapsado)

Se adopta el **modelo jerárquico de tres capas** (Acceso, Distribución y Núcleo), con la
salvedad de que en este proyecto la **Distribución y el Núcleo se combinan en un solo switch
de Capa 3** (núcleo colapsado). Esto es lo adecuado para una red de este tamaño: se obtiene el
orden, la segmentación y el control del modelo jerárquico sin el costo de un switch core
dedicado adicional.

- **Capa de Acceso:** switches de Capa 2 que conectan los dispositivos finales (PC,
  impresoras, Access Points). Aplican VLANs y PoE.
- **Capa de Distribución/Núcleo (colapsada):** switch de Capa 3 que realiza el
  **enrutamiento inter-VLAN**, concentra el backbone de fibra/cobre y aplica las ACL y
  políticas de QoS.
- **Capa de Borde/Perímetro:** router-firewall que conecta al ISP y separa la red interna de
  Internet (NAT, firewall, VPN).

### 2.3 Topología de red: Estrella Extendida (Jerárquica)

Se opta por una **topología en estrella extendida o jerárquica**. El switch de
Distribución/Núcleo (Capa 3) se centraliza en el rack principal ubicado en el departamento
de TI (Planta Alta), y desde él parten enlaces independientes hacia cada uno de los switches
de acceso, que a su vez actúan como el centro de las estrellas de cada departamento.

Ventajas que la justifican:

- El fallo de una estación o de un cable de acceso **no afecta** al resto de la red.
- Facilita el diagnóstico y el mantenimiento (problemas aislados por segmento).
- El tráfico entre departamentos se concentra y filtra en un único punto (el núcleo),
  simplificando seguridad y QoS.
- Es un diseño **probado, económico y escalable**, ideal para una red de campus pequeño.

### 2.4 Componentes e Infraestructura de Interconexión

| Dispositivo / Componente | Ubicación / Rol | Función Técnica |
|---|---|---|
| Router Firewall Perimetral | Departamento de TI (Planta Alta) | Conexión a Internet (ISP), NAT, seguridad perimetral mediante políticas de firewall, ACL y terminación de VPN. |
| Switch Core/Distribución (Capa 3) | Data Center / TI (Planta Alta) | Conmutación central de alta velocidad, administración de VLANs y enrutamiento inter-VLAN interno sin sobrecargar al router. Punto de agregación del backbone. |
| Switches de Acceso (Capa 2) | Uno por planta (2 en total) | Conexión directa a estaciones de trabajo, impresoras y Access Points. Aplican VLAN y soporte PoE (Power over Ethernet). |
| Puntos de Acceso (Wireless AP) | Pasillos y salas de espera (1 por planta) | Conectividad Wi-Fi segmentada: red institucional protegida (WPA2) y red de invitados pública aislada. |
| Cableado Estructurado Cat6 | Tendido vertical y horizontal | Transmisiones de hasta 1 Gbps a distancias menores a 100 m, reduciendo la diafonía. |
| Patch Panels, Rack y UPS | Rack principal (TI) y rack secundario (Planta Baja) | Organización del cableado, protección eléctrica y continuidad del servicio ante cortes de energía. |
| Servidor de Servicios de Red | Data Center / TI | Servidor para DHCP, DNS interno y compartición de archivos (archivos CAD/GIS de Planificación). |

#### 2.4.1 Equipos sugeridos (producción y laboratorio)

Dado que el proyecto se validará en **Cisco Packet Tracer**, se proponen equipos reales de
referencia y su equivalente en el simulador:

| Rol | Equipo recomendado (producción) | Equivalente Cisco / Packet Tracer | Cantidad |
|---|---|---|---|
| Router-Firewall | FortiGate 60F o Cisco RV340 | Cisco 4331 (ISR) / ASA 5506-X | 1 |
| Switch Core/Distribución L3 | Cisco Catalyst 3650 o Aruba 2930F | Cisco 3650-24PS (Capa 3) | 1 |
| Switch de Acceso L2 PoE | Cisco Catalyst 2960-X 24 puertos PoE | Cisco 2960-24TT / 3560 PoE | 2 |
| Access Point inalámbrico | Ubiquiti UniFi 6 / Cisco Aironet | AccessPoint-PT (autónomo) | 2 |
| Servidor | Servidor físico o virtual (Windows/Linux) | Server-PT | 2 |
| Cableado horizontal/vertical | Cat6 UTP/STP (certificado) | Cable directo/recto Cat6 | — |

> **Nota presupuestaria:** si no se dispone de presupuesto para equipos Cisco en producción,
> son válidas alternativas de menor costo (TP-Link Omada, Ubiquiti UniFi, MikroTik) que
> soportan VLAN 802.1Q, PoE y enrutamiento inter-VLAN.

### 2.5 Enlaces y backbone

- **Backbone vertical (uplinks):** el switch de acceso de la Planta Baja se conecta al switch
  Core de la Planta Alta mediante un enlace **troncal (trunk) 802.1Q** sobre cableado Cat6,
  garantizando la máxima tasa de transferencia (1 Gbps) entre plantas.
- **Enlaces de acceso:** cada switch de acceso se conecta al núcleo con un enlace troncal
  independiente, transportando las VLAN de su zona.
- **Borde:** el switch Core se conecta al router-firewall mediante un enlace enrutado
  (subred de tránsito) y este al ISP local.

### 2.6 Redundancia y disponibilidad (recomendación opcional)

Para un avance inicial es suficiente la estrella de núcleo único. Como mejora futura se puede:

- Duplicar el enlace Core ↔ Acceso mediante **EtherChannel (LACP)** para agregar ancho de
  banda y tolerancia a fallos.
- Añadir un segundo switch de núcleo en configuración de respaldo y usar **STP/RSTP**.
- Incorporar un segundo enlace a Internet (respaldo del ISP) con failover.

### 2.7 Anexos: diagramas

En los anexos se incluirán:

- **Anexo A – Diagrama de topología física** (ubicación de racks, switches y AP por planta).
- **Anexo B – Diagrama lógico** (VLANs, subredes y enrutamiento inter-VLAN).
- **Anexo C – Capturas de Packet Tracer** de la topología implementada.

## 3. Planificación del Direccionamiento IP (Subneteo y VLSM)

### 3.1 Método de trabajo

Para el direccionamiento se aplica **VLSM (Variable Length Subnet Mask)**, que consiste en
asignar a cada segmento una máscara de longitud distinta según la cantidad real de hosts que
necesita. El procedimiento seguido es:

1. **Determinar los segmentos lógicos (VLANs)** que requiere la institución.
2. **Contar los hosts** de cada segmento y agregar un margen de crecimiento.
3. **Ordenar los segmentos de mayor a menor** según su tamaño.
4. **Seleccionar un bloque base privado** y subdividirlo de forma jerárquica, asignando
   primero los bloques grandes para evitar la fragmentación del espacio.
5. **Reservar** las primeras direcciones de cada subred para el gateway (interfaz del switch
   de Capa 3) y direcciones fijas para la infraestructura.
6. **Dejar libre** el espacio restante para futuras ampliaciones (escalabilidad).

### 3.2 Datos de partida y VLANs

A partir de la estructura de la sección 1 y el diseño de la sección 2, se definen **siete
VLANs**: las cuatro de departamentos, una de gestión de la infraestructura y dos para el
servicio inalámbrico.

Es importante distinguir entre la **capacidad planificada** de cada segmento (que es lo que
dimensiona el subneteo) y el **número de dispositivos que se simulan** en Packet Tracer
(~30 en total). Se reserva capacidad adicional en cada VLAN para el crecimiento real de la
institución.

| VLAN | Nombre | Planta / Uso | Dispositivos simulados | Capacidad prevista |
|---|---|---|---|---|
| VLAN 10 | Atención Ciudadana | Planta Baja (público) | 6 | 25 |
| VLAN 20 | Financiero | Planta Baja (datos sensibles) | 4 | 10 |
| VLAN 30 | Planificación y Obras Públicas | Planta Alta (CAD/GIS) | 6 | 12 |
| VLAN 40 | TI / Administración | Planta Alta (servidores) | 6 | 10 |
| VLAN 50 | Gestión de Red | Switches, APs y router | 5 | 8 |
| VLAN 60 | Wi-Fi Invitados | Cobertura inalámbrica pública | 1 | 50 |
| VLAN 70 | Wi-Fi Institucional | Cobertura inalámbrica de personal | 1 | 20 |

> **Nota sobre el conteo de dispositivos:** los valores de capacidad de las VLAN 60 y 70
> representan la **capacidad del pool DHCP**, no equipos físicos. En el diseño se representan
> con **1 cliente inalámbrico por SSID** (laptop), ya que su direccionamiento es dinámico. Así
> el total simulado se mantiene en ~30 dispositivos, aunque la red queda dimensionada para
> atender hasta ~50 usuarios inalámbricos.

### 3.3 Cálculo del tamaño de cada subred (VLSM paso a paso)

Se clasifican los segmentos de mayor a menor y se elige la máscara mínima que cubra los hosts
requeridos (2^n − 2 hosts útiles):

| Orden | Segmento | Capacidad prevista | Máscara elegida | Hosts útiles |
|---|---|---|---|---|
| 1 | VLAN 60 – Wi-Fi Invitados | 50 | /26 | 62 |
| 2 | VLAN 10 – Atención Ciudadana | 25 | /27 | 30 |
| 3 | VLAN 70 – Wi-Fi Institucional | 20 | /27 | 30 |
| 4 | VLAN 50 – Gestión de Red | 8 | /28 | 14 |
| 5 | VLAN 30 – Planificación y Obras Públicas | 12 | /28 | 14 |
| 6 | VLAN 40 – TI / Administración | 10 | /28 | 14 |
| 7 | VLAN 20 – Financiero | 10 | /28 | 14 |
| 8 | Enlace Core ↔ Router | 2 | /30 | 2 |
| 9 | Enlace Router ↔ ISP | 2 | /30 | 2 |

Como la suma de los bloques (64 + 32 + 32 + 16 + 16 + 16 + 16 + 4 + 4 = **200 direcciones**)
cabe en una **/24**, pero se desea margen de crecimiento, se parte de un bloque base más
amplio: **`10.10.0.0/23`** (512 direcciones, rango `10.10.0.0 – 10.10.1.255`).

### 3.4 Asignación jerárquica de subredes

Partiendo de `10.10.0.0/23` y respetando el orden del punto anterior, se obtiene:

| VLAN / Enlace | Subred (CIDR) | Dirección de red | Gateway | Rango de hosts | Broadcast |
|---|---|---|---|---|---|
| VLAN 60 – Invitados | 10.10.0.0/26 | 10.10.0.0 | 10.10.0.1 | 10.10.0.1 – 10.10.0.62 | 10.10.0.63 |
| VLAN 10 – Atención Ciudadana | 10.10.0.64/27 | 10.10.0.64 | 10.10.0.65 | 10.10.0.65 – 10.10.0.94 | 10.10.0.95 |
| VLAN 70 – Wi-Fi Institucional | 10.10.0.96/27 | 10.10.0.96 | 10.10.0.97 | 10.10.0.97 – 10.10.0.126 | 10.10.0.127 |
| VLAN 50 – Gestión de Red | 10.10.0.128/28 | 10.10.0.128 | 10.10.0.129 | 10.10.0.129 – 10.10.0.142 | 10.10.0.143 |
| VLAN 30 – Planificación y OO.PP. | 10.10.0.144/28 | 10.10.0.144 | 10.10.0.145 | 10.10.0.145 – 10.10.0.158 | 10.10.0.159 |
| VLAN 40 – TI / Administración | 10.10.0.160/28 | 10.10.0.160 | 10.10.0.161 | 10.10.0.161 – 10.10.0.174 | 10.10.0.175 |
| VLAN 20 – Financiero | 10.10.0.176/28 | 10.10.0.176 | 10.10.0.177 | 10.10.0.177 – 10.10.0.190 | 10.10.0.191 |
| Enlace Core ↔ Router | 10.10.0.192/30 | 10.10.0.192 | — | 10.10.0.193 – 10.10.0.194 | 10.10.0.195 |
| Enlace Router ↔ ISP | 10.10.0.196/30 | 10.10.0.196 | — | 10.10.0.197 – 10.10.0.198 | 10.10.0.199 |
| **Reserva de crecimiento** | 10.10.0.200 – 10.10.1.255 | — | — | — | — |

> Todas las **puertas de enlace (gateway)** de las VLANs se configuran como **interfaz virtual
> (SVI)** en el switch de Capa 3, que realiza el enrutamiento inter-VLAN.

### 3.5 Direccionamiento estático de la infraestructura

Los equipos de red y servidores no usan DHCP: reciben **IP fija** para su administración.

| Dispositivo | VLAN | Dirección IP | Máscara | Gateway |
|---|---|---|---|---|
| Switch Core L3 (SVI gestión) | 50 | 10.10.0.129 | /28 | — |
| Switch de Acceso PB (Planta Baja) | 50 | 10.10.0.130 | /28 | 10.10.0.129 |
| Switch de Acceso PA (Planta Alta) | 50 | 10.10.0.131 | /28 | 10.10.0.129 |
| Router-Firewall (interfaz LAN) | tránsito | 10.10.0.194 | /30 | — |
| Router-Firewall (interfaz WAN) | — | 10.10.0.197 | /30 | — |
| Access Points (2) | 50 | 10.10.0.135 – 10.10.0.136 | /28 | 10.10.0.129 |
| Servidor DHCP/DNS | 40 | 10.10.0.162 | /28 | 10.10.0.161 |
| Servidor de Archivos (CAD/GIS) | 40 | 10.10.0.163 | /28 | 10.10.0.161 |

### 3.6 Asignación dinámica (DHCP) y DNS

- **DHCP:** el servidor `10.10.0.162` entrega direcciones a las VLANs 10, 30 y 40 para el
  personal, y a las VLANs 60 y 70 para las redes inalámbricas. Los switches de Capa 3 usan
  **`ip helper-address 10.10.0.162`** en cada SVI para reenviar las peticiones DHCP entre
  VLANs.
- **VLAN 20 (Financiero):** por política de seguridad se recomienda direccionamiento
  **estático o DHCP con reserva por MAC**, evitando que equipos no autorizados obtengan
  dirección en el segmento sensible.
- **DNS:** el servidor interno resuelve nombres institucionales y reenvía las consultas
  externas al DNS del ISP.
- **NAT:** el router-firewall aplica NAT (PAT) para todo el tráfico interno hacia Internet y
  una ruta por defecto `0.0.0.0/0` hacia el ISP.

### 3.7 Escalabilidad del direccionamiento

El bloque reservado **`10.10.0.200 – 10.10.1.255`** permite crear nuevos departamentos,
ampliar la VLAN de invitados o incorporar nuevos racks sin tocar las subredes existentes,
cumpliendo el requisito de escalabilidad del proyecto.

## 4. Selección y Configuración de Protocolos de Red

### 4.1 Criterios de selección

La elección de protocolos responde directamente a las necesidades de la sección 1:
segmentación, seguridad, alto rendimiento, gestión centralizada y conectividad inalámbrica.
Se priorizan **estándares abiertos (IEEE/IETF)**, ampliamente soportados y fáciles de mantener
con el personal de TI de la institución.

### 4.2 Protocolos seleccionados por capa

| Capa | Protocolo / Estándar | Función en el proyecto | Dónde se configura |
|---|---|---|---|
| Física/Enlace | Ethernet 802.3ab (1000BASE-T) + Cat6 | Enlaces de 1 Gbps a menos de 100 m | Cableado horizontal y vertical |
| Enlace | PoE 802.3af/at | Alimentar APs sin cable de energía | Switches de acceso |
| Enlace | VLAN 802.1Q | Segmentación lógica y troncales | Switches acceso y core |
| Enlace | STP / RSTP (802.1D / 802.1w) | Prevención de bucles de Capa 2 | Todos los switches |
| Enlace | LACP 802.3ad | Agregación de enlaces (mejora opcional) | Core ↔ Acceso |
| Enlace | DTP (deshabilitado) | Evitar negociación automática de troncales | Puertos troncales |
| Red | IPv4 + VLSM | Direccionamiento jerárquico y eficiente | Sección 3 |
| Red | Enrutamiento estático + ruta por defecto | Camino a Internet y entre segmentos | Core y router |
| Red | ICMP | Diagnóstico (ping, traceroute) | Todos |
| Servicios | DHCP (RFC 2131) + `ip helper-address` | Asignación automática de IP | Servidor + SVI del core |
| Servicios | DNS | Resolución de nombres interna/externa | Servidor + ISP |
| Servicios | NAT/PAT (RFC 3022) | Traducción de direcciones privadas a públicas | Router-firewall |
| Servicios | NTP | Sincronización de hora (logs) | Core/router |
| Servicios | Syslog + SNMP | Monitoreo y registro centralizado | Core/router → servidor |
| Servicios | SSH (deshabilitar Telnet) | Administración remota cifrada | Todos los switches/router |
| Seguridad | ACL y firewall | Filtrado del tráfico entre VLANs y hacia Internet | Core / router-firewall |
| Seguridad | Port Security, DHCP Snooping, Dynamic ARP Inspection | Mitigar ataques de Capa 2 | Switches de acceso |
| Inalámbrico | IEEE 802.11n/ac | Cobertura Wi-Fi institucional e invitados | Access Points |
| Inalámbrico | WPA2/WPA3 (802.11i) + AES | Cifrado del Wi-Fi institucional | Access Points |
| Calidad | 802.1p / DSCP + colas de prioridad | Priorizar voz, gestión y CAD/GIS | Core y switches |
| WAN | DHCP/PPPoE del ISP + IPsec VPN (opcional) | Enlace a Internet y acceso remoto seguro | Router-firewall |

### 4.3 Configuración base

A continuación se muestra la configuración de referencia sobre los equipos Cisco
(equivalente a lo que se implementará en Packet Tracer). Las IP corresponden al plan de la
sección 3.

#### 4.3.1 Switch Core de Capa 3 (VLANs, SVIs y enrutamiento)

```
hostname CORE-L3
ip routing

! Definición de VLANs
vlan 10
 name ATENCION_CIUDADANA
vlan 20
 name FINANCIERO
vlan 30
 name PLANIFICACION
vlan 40
 name TI_ADMIN
vlan 50
 name GESTION
vlan 60
 name WIFI_INVITADOS
vlan 70
 name WIFI_INSTITUCIONAL
vlan 999
 name NATIVA_NO_USADA

! Interfaces virtuales (gateway de cada VLAN)
interface Vlan10
 ip address 10.10.0.65 255.255.255.224
 ip helper-address 10.10.0.162
interface Vlan20
 ip address 10.10.0.177 255.255.255.240
interface Vlan30
 ip address 10.10.0.145 255.255.255.240
 ip helper-address 10.10.0.162
interface Vlan40
 ip address 10.10.0.161 255.255.255.240
interface Vlan50
 ip address 10.10.0.129 255.255.255.240
interface Vlan60
 ip address 10.10.0.1 255.255.255.192
 ip helper-address 10.10.0.162
interface Vlan70
 ip address 10.10.0.97 255.255.255.224
 ip helper-address 10.10.0.162

! Enlace hacia el router-firewall (subred de tránsito)
! El core es un Catalyst 3650-24PS (24 puertos Gigabit) para la estrella pura.
interface GigabitEthernet1/0/1
 description UPLINK_ROUTER
 no switchport
 ip address 10.10.0.193 255.255.255.252

! Ruta por defecto hacia el router
ip route 0.0.0.0 0.0.0.0 10.10.0.194
```

#### 4.3.2 Switches de acceso (VLAN, troncal, STP y seguridad de puerto)

```
hostname ACCESO-PB
vlan 10,20,30,40,50,60,70,999

! Puerto de usuario
interface FastEthernet0/1
 description PC_USUARIO
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 spanning-tree portfast

! Enlace troncal hacia el core (802.1Q)
interface GigabitEthernet0/1
 description TRONCAL_CORE
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60,70
 switchport trunk native vlan 999

! Prevención de bucles
spanning-tree mode rapid-pvst
spanning-tree bpduguard default
```

#### 4.3.3 Router-Firewall (perímetro, NAT y rutas)

```
hostname ROUTER-FW

interface GigabitEthernet0/0/0
 description WAN_ISP
 ip address 10.10.0.197 255.255.255.252
 ip nat outside
interface GigabitEthernet0/0/1
 description LAN_CORE
 ip address 10.10.0.194 255.255.255.252
 ip nat inside

! NAT de salida a Internet (PAT)
ip nat inside source list 10 interface GigabitEthernet0/0/0 overload
access-list 10 permit 10.10.0.0 0.0.1.255

! Rutas
ip route 10.10.0.0 255.255.254.0 10.10.0.193
ip route 0.0.0.0 0.0.0.0 10.10.0.198
```

#### 4.3.4 Servicios y administración segura (DHCP, SSH, NTP, Syslog)

- **DHCP:** el servidor `10.10.0.162` publica un *scope* por VLAN (10, 30, 40, 60 y 70); las
  SVIs del core reenvían las peticiones con `ip helper-address 10.10.0.162`. La VLAN 20
  (Financiero) usa direccionamiento estático/reserva.
- **SSH y deshabilitación de Telnet** en switches y router:

```
ip domain-name alcaldia-morona.gob.ec
crypto key generate rsa modulus 1024
username admin privilege 15 secret <clave>
line vty 0 4
 transport input ssh
 login local
```

- **NTP:** `ntp server 10.10.0.162` para una hora común en los registros.
- **Syslog/SNMP:** `logging host 10.10.0.162` y `snmp-server community <RO> ro` para que TI
  monitoree los equipos de forma centralizada.

#### 4.3.5 Seguridad de Capa 2 y control de tráfico entre VLANs

- **DHCP Snooping:** en los switches de acceso, marcando los uplinks hacia el core como
  `ip dhcp snooping trust`, para bloquear servidores DHCP no autorizados.
- **Dynamic ARP Inspection (DAI):** para prevenir ARP spoofing.
- **ACL inter-VLAN:** en el core se bloquea por defecto el acceso directo a la VLAN 20
  (Financiero); solo la VLAN 40 (TI) y las direcciones de administración pueden iniciar
  conexiones hacia ella.
- **ACL hacia Internet:** el router solo permite salida (NAT) y bloquea conexiones entrantes
  no solicitadas, salvo el servicio VPN si se habilita.

#### 4.3.6 Red inalámbrica

La red inalámbrica se implementó con **Access Points autónomos** (sin controladora WLC), ya
que la versión de Packet Tracer disponible no soporta la interfaz GUI del WLC-2504. Cada AP
emite un único SSID y se conecta a un puerto de acceso en la VLAN correspondiente:

- AP en **Planta Baja** → SSID **`MORONA-VISITANTES`** → VLAN 60 (invitados), cifrado
  **WPA2-PSK/AES**, con aislamiento de clientes y acceso solo a Internet.
- AP en **Planta Alta** → SSID **`ALCALDIA-MORONA`** → VLAN 70 (institucional), cifrado
  **WPA2-PSK/AES**, con acceso al servidor interno.

La lógica de ubicación responde al perfil de usuarios: los **visitantes** concurren a la
Planta Baja (Atención Ciudadana, salas de espera), mientras que el **personal** opera
principalmente en la Planta Alta (Planificación, TI, servidores).

> **Nota de implementación:** en una versión de Packet Tracer que soporte el WLC-2504 con
> pestaña GUI, se podría usar una controladora para emitir ambos SSIDs desde cada AP,
> asignando cada SSID a su VLAN mediante interfaces dinámicas.

#### 4.3.7 Calidad de Servicio (QoS)

Se aplican colas de prioridad en el core para el tráfico crítico:

```
class-map match-any CRITICO
 match dscp ef
class-map match-any ALTO
 match dscp af31 af21
policy-map QOS-SALIDA
 class CRITICO
  priority 512
 class ALTO
  bandwidth 2048
```

- **Prioridad alta:** VoIP y gestión de red.
- **Prioridad media:** tráfico CAD/GIS de Planificación y Obras Públicas.
- **Prioridad normal (best-effort):** navegación web y tráfico de invitados.

## 5. Justificación Técnica de la Solución

### 5.1 Principios de diseño

La arquitectura propuesta se fundamenta en principios modernos de ingeniería de redes:

- **Implementación de VLANs (Virtual LANs):** Se configuran **siete VLANs** (cuatro por
  departamento, una de gestión de red y dos para el servicio Wi-Fi). Esto garantiza que el
  tráfico sensible del área Financiera quede completamente aislado a nivel lógico del tráfico
  de Atención Ciudadana, mitigando riesgos de seguridad e interceptación de datos.
- **Segmentación con VLSM:** El uso de máscaras de longitud variable evita el desperdicio de
  direcciones y permite que cada segmento disponga del tamaño justo, dejando un bloque de
  reserva para el crecimiento.
- **Eficiencia de la Topología en Estrella:** Al centralizar las conexiones en switches
  dedicados, el fallo de un cable o una estación de trabajo individual no afectará al resto de
  la red corporativa, aislando los problemas y facilitando el mantenimiento.
- **Escalabilidad:** El uso de un switch Capa 3 en el Core permite que la Alcaldía añada
  nuevos departamentos o expanda su infraestructura física en el futuro sin necesidad de
  reestructurar la lógica de la red actual.

### 5.2 Análisis Costo-Beneficio y Justificación de la Inversión

Dado que se trata de una **institución pública con presupuesto limitado**, el diseño debe
demostrar que **cada dispositivo responde a una necesidad real** y que no existe
sobre-dimensionamiento. El criterio aplicado es *"el equipo necesario, ni más ni menos"*.

#### 5.2.1 Justificación de cada componente

| Componente | Necesidad que resuelve | ¿Es imprescindible? | Alternativa de menor costo |
|---|---|---|---|
| Router-Firewall | Salida a Internet, NAT, seguridad perimetral | Sí, sin él no hay conectividad externa ni protección | FortiGate 40F/60F, MikroTik RB4011, pfSense (mini-PC) |
| Switch Core L3 | Enrutamiento inter-VLAN y segmentación | Sí; centraliza y filtra el tráfico | Switch L3 económico (MikroTik CRS3xx, Ubiquiti) o router-on-a-stick si el presupuesto es mínimo |
| Switch de Acceso PoE | Conectar PCs/impresoras y alimentar APs | Sí; es el punto de conexión de cada planta | Switches administrables TP-Link Omada / Ubiquiti |
| Access Points | Cobertura Wi-Fi institucional e invitados | Sí, por la demanda de movilidad | Ubiquiti UniFi, TP-Link Omada |
| WLC (controladora) | Separar 2 SSID en VLAN distintas | **Opcional**; puede evitarse con APs autónomos o controladora en software | Controladora UniFi/Omada (software, sin costo de hardware) |
| Servidores (2) | DHCP, DNS y archivos CAD/GIS | Sí, pero **pueden unificarse** en un solo equipo virtualizado | Un servidor físico + máquinas virtuales, o NAS |
| Cableado Cat6, Rack, UPS | Base física, protección eléctrica | Sí | Cat6 certificado (no Cat6A, más caro e innecesario aquí) |

#### 5.2.2 Comparación de costos (valores referenciales en USD)

Los precios son **estimados y pueden variar** según proveedor y país; sirven para comparar
órdenes de magnitud entre una solución empresarial (Cisco) y una solución ajustada al tamaño
real de la Alcaldía.

| Componente | Opción Cisco (empresarial) | Opción costo-efectiva | Ahorro aprox. |
|---|---|---|---|
| Router-Firewall | ~$2,000 – $4,000 | FortiGate 60F ~$600 | ~70 % |
| Switch Core L3 | ~$2,500 – $5,000 | Ubiquiti/MikroTik L3 ~$400 – $800 | ~75 % |
| Switch de Acceso PoE (2) | ~$1,000 – $2,000 c/u | TP-Link Omada/UniFi ~$250 c/u | ~70 % |
| Access Points (2) | ~$300 – $500 c/u | UniFi U6 ~$150 – $200 c/u | ~50 % |
| WLC | ~$1,500 | Controladora por software $0 | 100 % |
| Servidores | 2 físicos ~$1,000 – $2,500 | 1 físico virtualizado (2 VM) | ~50 % |

#### 5.2.3 Conclusiones del análisis

- **No hay sobre-ingeniería en la arquitectura:** el número de switches, APs y enlaces
  corresponde exactamente a la escala del edificio (~30 dispositivos, 2 plantas).
- **Sí existe una diferencia de marca:** el equipamiento Cisco empleado en la simulación
  académica es sensiblemente más caro que el necesario para la operación real. Para la
  implementación productiva se recomienda la **opción costo-efectiva** (FortiGate, Ubiquiti,
  MikroTik, TP-Link Omada), que cumple los mismos requisitos de VLAN, PoE, enrutamiento y
  seguridad con un costo total estimado **60-70 % menor**.
- **Ahorros adicionales:** unificar los dos servidores en uno virtualizado y usar la
  controladora Wi-Fi por software elimina hardware innecesario.
- **Beneficio esperado:** la inversión se justifica por la reducción de fallas (segmentación),
  la protección de datos financieros y la capacidad de gestión centralizada, evitando a la vez
  gastos en equipos sobredimensionados.

> **Nota metodológica:** Packet Tracer permite simular con equipos Cisco porque son los
> modelados en la herramienta; esto **no implica** que la Alcaldía deba comprarlos. El informe
> presenta el diseño lógico y, en esta sección, la alternativa económicamente viable.

---

## REQUISITOS DEL INFORME (Checklist del proyecto)

El informe técnico debe incluir todos los siguientes detalles:

- [x] Descripción de la empresa y sus necesidades de red.
- [x] Diseño de la arquitectura de red, incluyendo topología y componentes
      (si corresponde, incluir diagramas en los anexos).
- [x] Planificación del direccionamiento IP, incluyendo subneteo y VLSM.
- [x] Selección y configuración de protocolos de red adecuados.
- [x] Justificación costo-beneficio de la solución y de los equipos seleccionados
      (sección 5.2).
- [x] Uso de herramientas de análisis para monitorear y evaluar el rendimiento de la red.
      Herramientas utilizadas: **Cisco Packet Tracer** (Simulation Mode) para la captura e
      inspección de paquetes ICMP, ARP, 802.1Q y NAT. Las capturas de pantalla y el proceso
      de verificación (Etapas 10 y 12) se documentan en el archivo Word adjunto. No se utilizó
      Wireshark directamente porque Packet Tracer corre en un entorno aislado; el Simulation
      Mode cumple la misma función de análisis de tramas dentro del simulador.
