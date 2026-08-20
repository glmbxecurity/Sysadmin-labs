# Laboratorio de Introducción a la Virtualización

# Objetivo final del laboratorio

Al completar todas las prácticas, deberías ser capaz de:

- comprender los componentes básicos de un ordenador;
- entender qué es la virtualización;
- crear una máquina virtual;
- configurar su CPU y RAM;
- configurar sus discos virtuales;
- configurar sus tarjetas de red virtuales;
- comprender el almacenamiento de una VM;
- crear redes virtuales;
- conectar máquinas virtuales a diferentes redes;
- comprender los conceptos de vSwitch, Port Group, bridge y Virtual Switch;
- investigar y resolver dudas utilizando documentación;
- aplicar los mismos conceptos generales en ESXi, Proxmox y Hyper-V;
- diseñar una pequeña infraestructura virtual desde cero.

La meta no es memorizar una interfaz concreta.

La meta es que, cuando te encuentres delante de un hipervisor que nunca has utilizado, puedas pensar:

> **"No sé exactamente dónde está esta opción, pero sé qué quiero configurar y qué concepto estoy buscando."**

## Introducción

En este laboratorio aprenderás los conceptos fundamentales de la virtualización y su aplicación en distintos hipervisores.

El objetivo no es aprender a utilizar únicamente una herramienta concreta. La finalidad es que entiendas **qué estás haciendo, por qué lo haces y qué conceptos hay detrás de cada configuración**.

A lo largo del laboratorio podrás trabajar con plataformas como:

- VMware ESXi
- Proxmox VE
- Microsoft Hyper-V

Los nombres y las interfaces cambian entre plataformas, pero los conceptos fundamentales son los mismos.

No se proporcionará un procedimiento paso a paso para cada tarea. Tendrás que **investigar, consultar documentación y utilizar Internet para resolver los ejercicios**, aprendiendo a localizar información técnica por tu cuenta.

---

# Normas del laboratorio

Durante las prácticas:

- NO ESTÁ PERMITIDO EL USO DE INTELIGENCIA ARTIFICIAL
- Está permitido y recomendado consultar documentación oficial.
- Debes intentar comprender cada concepto antes de configurarlo.
- Cuando tomes una decisión técnica, debes ser capaz de justificarla.
- Si una práctica utiliza un concepto que no conoces, tu primera tarea es investigar qué significa.
- Los nombres de las opciones pueden variar según el hipervisor utilizado.
- No memorices únicamente dónde está una opción: intenta comprender qué problema resuelve.


---

# Módulo 0 — Conceptos básicos de informática

## Objetivo

Antes de comenzar con la virtualización, debes comprender los componentes básicos de un ordenador.

## Conceptos que debes investigar

Debes ser capaz de explicar con tus propias palabras:

- CPU.
- Core.
- Thread.
- RAM.
- Disco.
- HDD.
- SSD.
- Tarjeta de red.
- Dirección MAC.
- Dirección IP.
- Sistema operativo.
- Servidor.
- Cliente.
- BIOS.
- UEFI.
- ISO.

## Práctica 0.1 — Identificar componentes

Supón que tienes un ordenador físico con estas características:

```text
CPU: 8 cores
RAM: 32 GB
Almacenamiento: SSD de 1 TB
Red: 1 tarjeta Ethernet
```

Responde con tus palabras:

1. ¿Qué función realiza la CPU?
2. ¿Qué diferencia existe entre un core y un thread?
3. ¿Qué función tiene la RAM?
4. ¿Qué diferencia existe entre RAM y almacenamiento?
5. ¿Qué función realiza una tarjeta de red?
6. ¿Qué es una dirección MAC?
7. ¿Qué es una dirección IP?
8. ¿Qué diferencia existe entre BIOS y UEFI?
9. ¿Qué es una ISO?


---

# Módulo 1 — Introducción a la virtualización

## Objetivo

Comprender qué es la virtualización y cómo un ordenador físico puede ejecutar varias máquinas virtuales.

## Conceptos que debes investigar

- Máquina física.
- Máquina virtual.
- Host.
- Guest.
- Hipervisor.
- Virtualización.
- Hipervisor de tipo 1.
- Hipervisor de tipo 2.
- VMware ESXi.
- Proxmox VE.
- Microsoft Hyper-V.
- VirtualBox.

## Concepto fundamental

Una infraestructura virtualizada puede representarse de forma simplificada así:

```text
SERVIDOR FÍSICO
│
├── VM 1
├── VM 2
├── VM 3
└── VM 4
```

Las máquinas virtuales utilizan recursos físicos del host.

## Práctica 1.1 — Diseñar una infraestructura virtual

Dispones de un servidor físico con:

```text
CPU: 8 cores
RAM: 32 GB
SSD: 1 TB
NIC: 1
```

Debes diseñar tres máquinas virtuales.

Para cada una debes especificar:

- Sistema operativo.
- CPU.
- RAM.
- Disco.
- Tarjeta de red.

### Preguntas

1. ¿Por qué no puedes asignar 32 GB de RAM a cada una de las tres máquinas?
2. ¿Qué ocurre si asignas más recursos virtuales de los que realmente existen?
3. ¿Qué recursos son físicos?
4. ¿Qué recursos son virtuales?
5. ¿Qué componente se encarga de gestionar esos recursos?

---

# Módulo 2 — Creación de una máquina virtual

## Objetivo

Crear una máquina virtual funcional y comprender qué representa cada uno de sus componentes.

## Práctica 2.1 — Primera máquina virtual

Crea una máquina virtual con las siguientes características:

```text
CPU: 2 vCPU
RAM: 2 GB
Disco: 20 GB
Red: 1 NIC
Sistema operativo: Linux
```

Debes investigar por tu cuenta cómo realizar la creación en el hipervisor utilizado.

## Antes de crearla

Investiga:

- Qué es una ISO.
- Qué significa vCPU.
- Qué significa RAM asignada.
- Qué es un disco virtual.
- Qué es una NIC virtual.
- Qué es el firmware de una máquina virtual.
- Qué diferencia existe entre BIOS y UEFI.
- Qué es el orden de arranque.

## Preguntas

1. ¿Qué diferencia existe entre una CPU física y una vCPU?
2. ¿La máquina virtual tiene realmente una CPU física propia?
3. ¿La RAM asignada a una VM es físicamente independiente?
4. ¿Dónde se almacena el disco virtual?
5. ¿Qué función tiene la NIC virtual?

---

# Módulo 3 — Hardware virtual

## Objetivo

Comprender la relación entre el hardware físico y el hardware que observa una máquina virtual.

---

## 3.1 CPU virtual

Investiga:

- CPU física.
- Core.
- Thread.
- vCPU.
- Asignación de CPU.
- Sobreasignación de CPU.

### Práctica

Crea o modifica una máquina virtual para que tenga:

```text
2 vCPU
```

Después modifica su configuración para disponer de más o menos CPU.

Debes observar qué cambia en la configuración de la VM y explicar qué significa.

### Preguntas

1. ¿Qué representa una vCPU?
2. ¿Qué relación existe entre vCPU y los cores físicos?
3. ¿Qué significa sobreasignar CPU?

---

## 3.2 Memoria

Investiga:

- RAM física.
- RAM virtual.
- Asignación de memoria.
- Sobreasignación de memoria.

### Práctica

Crea una VM con:

```text
2 GB de RAM
```

Modifica posteriormente su cantidad de memoria.

Explica qué está ocurriendo desde el punto de vista del host y del guest.

---

## 3.3 Disco virtual

Investiga:

- Disco físico.
- Disco virtual.
- VMDK.
- VHD.
- VHDX.
- qcow2.
- thin provisioning.
- thick provisioning.

### Preguntas

1. ¿Es un disco virtual un disco físico?
2. ¿Dónde se almacena?
3. ¿Qué ventajas tiene utilizar un disco virtual?
4. ¿Qué diferencias existen entre los formatos anteriores?

---

## 3.4 Tarjeta de red virtual

Investiga:

- NIC física.
- NIC virtual.
- Dirección MAC.
- Driver de red.
- Modelo de adaptador virtual.

### Práctica

Crea una VM con una única tarjeta de red.

Localiza:

- el modelo de la tarjeta;
- la dirección MAC;
- la configuración de red desde el sistema operativo.

Explica qué relación existe entre la NIC virtual de la VM y la NIC física del host.

---

# Módulo 4 — Almacenamiento virtual

## Objetivo

Entender cómo pasa la información desde un almacenamiento físico hasta los archivos de una máquina virtual.

## Concepto fundamental

La estructura simplificada es:

```text
DISCO FÍSICO
     ↓
STORAGE DEL HIPERVISOR
     ↓
DISCO VIRTUAL
     ↓
PARTICIÓN
     ↓
SISTEMA DE ARCHIVOS
     ↓
ARCHIVOS
```

Los nombres concretos pueden variar según el hipervisor.

Por ejemplo:

- ESXi utiliza conceptos como datastore.
- Proxmox utiliza diferentes tipos de storage.
- Hyper-V utiliza almacenamiento para los archivos de las máquinas virtuales.

---

## Práctica 4.1 — Crear un disco virtual

Crea una VM con:

```text
Disco: 20 GB
```

Localiza desde el hipervisor dónde está almacenado ese disco virtual.

Después localiza el disco desde el sistema operativo invitado.

### Debes responder

1. ¿Cómo se llama el disco virtual en tu hipervisor?
2. ¿Qué formato utiliza?
3. ¿Dónde está almacenado?
4. ¿Cómo lo identifica el sistema operativo?

---

## Práctica 4.2 — Ampliar un disco

Parte de un disco virtual de:

```text
20 GB
```

Amplíalo hasta:

```text
40 GB
```

Después comprueba desde el sistema operativo qué tamaño observa.

### Objetivo

Descubrir por qué:

> ampliar el disco virtual no implica necesariamente que el sistema operativo pueda utilizar automáticamente todo el espacio añadido.

Investiga qué elementos intervienen entre el disco virtual y el espacio disponible para los archivos.

---

## Práctica 4.3 — Añadir un segundo disco

Añade un segundo disco virtual de:

```text
10 GB
```

Identifícalo desde el sistema operativo.

Debes ser capaz de explicar:

- qué disco es;
- cómo lo identifica el sistema operativo;
- qué diferencia existe entre el primer y el segundo disco.

---

# Módulo 5 — Redes virtuales

## Objetivo

Comprender cómo una máquina virtual puede comunicarse con otras máquinas virtuales mediante una red virtual.

## Conceptos que debes investigar

- Red.
- NIC.
- NIC física.
- NIC virtual.
- MAC.
- IP.
- Máscara de red.
- Gateway.
- DNS.
- DHCP.
- Switch.
- Switch virtual.
- Bridge.
- VLAN.
- Port Group.

---

## Concepto fundamental

Una simplificación de la comunicación puede representarse así:

```text
VM
 ↓
NIC virtual
 ↓
Switch virtual
 ↓
NIC física
 ↓
Switch físico
 ↓
Red
```

No todos los entornos tienen exactamente los mismos componentes visibles, pero el concepto general es similar.

---

## 5.1 Comparación entre hipervisores

Investiga cómo se representa una red virtual en:

### VMware ESXi

```text
VM
 ↓
vNIC
 ↓
Port Group
 ↓
vSwitch
 ↓
vmnic
```

### Hyper-V

```text
VM
 ↓
vNIC
 ↓
Virtual Switch
 ↓
NIC física
```

### Preguntas

1. ¿Qué función tiene un switch virtual?
2. ¿Qué diferencia existe entre una NIC física y una NIC virtual?
3. ¿Qué es una MAC?
4. ¿Qué es un Port Group?
5. ¿Qué es un bridge?
6. ¿Qué es una VLAN?
7. ¿Por qué los nombres cambian entre hipervisores?

---

# Módulo 6 — Construcción de una red virtual

## Objetivo

Crear una pequeña infraestructura de red utilizando máquinas virtuales.

En este módulo no se utilizará conexión a Internet.

El objetivo es trabajar exclusivamente con **comunicación interna entre máquinas virtuales**.

---

## Práctica 6.1 — Dos máquinas en la misma red

Crea dos máquinas virtuales:

```text
VM-01
VM-02
```

Ambas deben:

- disponer de una NIC virtual;
- estar conectadas a la misma red virtual;
- tener una dirección IP válida;
- poder comunicarse entre ellas.

Representación:

```text
          RED VIRTUAL
           /       \
        VM-01     VM-02
```

### Debes comprobar

- La configuración IP de cada máquina.
- La dirección MAC de cada NIC.
- Que existe conectividad entre las dos máquinas.

### Preguntas

1. ¿Qué permite que ambas máquinas se comuniquen?
2. ¿Qué componente actúa como punto de conexión entre ambas?
3. ¿Qué ocurriría si una de las máquinas estuviera conectada a una red virtual diferente?

---

## Práctica 6.2 — Crear una segunda red

Ahora debes crear una segunda red virtual independiente.

Debes tener:

```text
RED 1
 ├── VM-01
 └── VM-02

RED 2
 ├── VM-03
 └── VM-04
```

### Objetivo

- VM-01 y VM-02 deben poder comunicarse.
- VM-03 y VM-04 deben poder comunicarse.
- Las máquinas de RED 1 no deben comunicarse directamente con las máquinas de RED 2.

### Debes investigar

- Cómo crear una segunda red virtual.
- Cómo conectar una VM a una red diferente.
- Qué diferencia existe entre dos redes virtuales independientes.
- Qué necesitarías para permitir comunicación entre ambas redes.

---

# Proyecto final — Pequeña infraestructura virtual

## Objetivo

Construir desde cero una infraestructura virtual sencilla utilizando los conocimientos adquiridos durante el laboratorio.

No existe un procedimiento paso a paso.

Debes investigar, diseñar, configurar y comprobar la solución.

---

## Escenario

Una pequeña empresa necesita disponer de una infraestructura informática básica.

La empresa tendrá:

- 2 ordenadores de trabajadores.
- 1 servidor.
- 1 firewall/router virtual.
- 1 red interna empresarial dividida en 3 subredes.

No es necesario desplegar servidores adicionales.


El objetivo es aprender los fundamentos de la virtualización, el hardware virtual, el almacenamiento y las redes.

---

# Requisitos del proyecto



## 1. Firewall / Router

Debes crear una máquina virtual que actúe como dispositivo de red para la infraestructura.

Debe disponer de las interfaces de red necesarias para cumplir la función que hayas diseñado.

Investiga:

- qué es pfsense, ¿serviría como router?
- qué sistema operativo o solución utilizar;
- cuántas tarjetas de red necesita;
- qué función tiene cada tarjeta.

---

## 2. Servidor

Debes crear un servidor virtual.

Puedes utilizar Linux o Windows Server.

El servidor debe ofrecer un servicio sencillo a los equipos cliente.

Puedes utilizar, por ejemplo:

- servidor web;
- servidor de archivos.

Debes elegir una opción y justificarla.

---

## 3. Equipos cliente

Debes crear dos máquinas virtuales que representen los puestos de trabajo:

```text
PC-01
PC-02
```

Ambas deberán poder comunicarse con el servidor.

---

# Arquitectura mínima

Tu infraestructura debe ser equivalente, a nivel conceptual, a:

                         ┌─────────────────────┐
                         │  Firewall / Router   │
                         └──────────┬──────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
        ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
        │ Switch virtual │ │ Switch virtual │ │ Switch virtual │
        └───────┬────────┘ └───────┬────────┘ └───────┬────────┘
                │                  │                  │
                ▼                  ▼                  ▼
        ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
        │       PC       │ │       PC       │ │     SERVER     │
        └────────────────┘ └────────────────┘ └────────────────┘

La implementación concreta dependerá del hipervisor utilizado.

---

# Recursos de las máquinas

Para cada máquina debes decidir:

- Sistema operativo.
- CPU.
- RAM.
- Disco.
- Número de NIC.
- Red virtual a la que estará conectada.

No se proporcionan los valores exactos.

Debes elegirlos de forma razonable y justificar tus decisiones.

---

# Direccionamiento IP

Debes diseñar un esquema de direccionamiento IP para la red.

Debes especificar:

- Dirección IP.
- Máscara de red.
- Gateway.
- DNS, cuando sea necesario.

No se proporciona un rango concreto.

Debes investigar cómo diseñar una red IP sencilla y elegir un esquema adecuado.

---

# Entregables

## 1. Diagrama de la infraestructura

Debes representar gráficamente la solución.

Ejemplo conceptual:

```text
              FIREWALL
                  |
             RED INTERNA
          /       |       \
      SERVER    PC-01    PC-02
```

Puedes utilizar cualquier herramienta de diagramación.

---

## 2. Tabla de máquinas virtuales

Debes crear una tabla como la siguiente:

| Máquina | Sistema operativo | CPU | RAM | Disco | NIC | IP |
|---|---|---:|---:|---:|---:|---|
| Firewall | | | | | | |
| Server | | | | | | |
| PC-01 | | | | | | |
| PC-02 | | | | | | |

---

## 3. Justificación

Debes explicar brevemente:

- por qué has elegido cada sistema operativo;
- por qué has asignado esos recursos;
- por qué has diseñado la red de esa forma;
- qué función tiene cada NIC;
- qué almacenamiento utiliza cada VM.

---

## 4. Comprobaciones

Debes demostrar que:

- todas las máquinas virtuales arrancan correctamente;
- las NIC están correctamente configuradas;
- los equipos tienen las direcciones IP esperadas;
- PC-01 puede comunicarse con el servidor;
- PC-02 puede comunicarse con el servidor;
- las máquinas de la infraestructura están conectadas a la red correcta;
- el servicio del servidor funciona correctamente.

---

# Reflexión final

Al terminar el proyecto debes ser capaz de responder, con tus propias palabras, a las siguientes preguntas:

1. ¿Qué es una máquina virtual?
2. ¿Qué es un hipervisor?
3. ¿Qué diferencia existe entre host y guest?
4. ¿Qué es una vCPU?
5. ¿Qué es una NIC virtual?
6. ¿Qué es un disco virtual?
7. ¿Qué diferencia existe entre el almacenamiento físico y el disco virtual?
8. ¿Qué es un switch virtual?
9. ¿Qué es un Port Group?
10. ¿Qué es un bridge?
11. ¿Qué es una VLAN?
12. ¿Qué diferencia existe entre una red física y una red virtual?
13. ¿Qué conceptos de este laboratorio son iguales en ESXi, Proxmox y Hyper-V?
14. ¿Qué elementos cambian entre diferentes hipervisores?

---


