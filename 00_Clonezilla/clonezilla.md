# Guía de Ejercicios: Despliegue Masivo (Hyper-V & Clonezilla)

## Introducción

### ¿Qué es Clonezilla y para qué sirve?
**Clonezilla** es una herramienta *open-source* para la clonación y despliegue de discos y particiones a bajo nivel (*bare-metal*). A diferencia de una copia tradicional de archivos, trabaja a nivel de bloques del sistema de archivos (usando herramientas como `Partclone`), guardando y restaurando únicamente los bloques utilizados. Esto reduce drásticamente el tiempo de copia y el tamaño de las imágenes.

En administración de sistemas, es un estándar para:
- **Despliegues masivos:** Configurar decenas de puestos de trabajo idénticos en minutos en lugar de instalar uno por uno.
- **Disaster Recovery:** Generar copias de respaldo completas de sistemas operativos y configuraciones críticas.

### Propósito del Laboratorio
En un entorno de producción real, este trabajo se realiza sobre hardware físico conectando equipos a un switch de red o usando medios externos. 

En este laboratorio vamos a **reproducir esa arquitectura real en un entorno virtualizado (Hyper-V)** para dominar dos métodos fundamentales de despliegue:
1. **Clonación local (Disco a Disco / Imagen):** Captura de una imagen maestra (*Golden Image*) a un almacenamiento secundario y su restauración individual en un equipo destino.
2. **Clonación masiva por red (PXE / Broadcast):** Configuración de la máquina maestra como servidor de despliegue (*Clonezilla Lite Server*) para enviar la imagen simultáneamente a múltiples clientes que arrancan por red, simulando el aprovisionamiento de una oficina o laboratorio completo.

---

## Hito 1: Preparación del Entorno Hyper-V (El Laboratorio)

- Tarea 1.1: Crear un conmutador virtual (Virtual Switch) de tipo "Privado" o "Interno".

- Tarea 1.2: Crear una Máquina Virtual (VM) que actuará como equipo "Maestro".

- Tarea 1.3: Crear tres VMs que actuarán como "Clientes".

- Tarea 1.4: Configurar todas las VMs (Maestro y Clientes) para que estén conectadas al conmutador virtual creado en la Tarea 1.1.

## Hito 2: Creación de la Imagen Base (Golden Image)

- Tarea 2.1: Instalar un sistema operativo Windows en la VM Maestra.

- Tarea 2.2: Instalar un paquete de software de prueba (ej. 7-Zip, un navegador web, un lector PDF) y crear un archivo de texto en el escritorio llamado "CLON_TEST".

- Tarea 2.3: Sellar el sistema utilizando la herramienta sysprep nativa de Windows y apagar la máquina para dejarla lista para su clonación.

## Hito 3: Captura de Equipo a Disco Externo (Backup a USB)

- Tarea 3.1: Conectar un disco duro USB físico (o crear un hdd virtual) al equipo host e integrarlo dentro de la VM Maestra.

- Tarea 3.2: Cargar la ISO de Clonezilla en la unidad de CD/DVD virtual de la VM Maestra y forzar el arranque desde ahí.

- Tarea 3.3: Utilizar Clonezilla para leer el disco de la VM Maestra y guardar una imagen completa de ese disco dentro del disco duro USB conectado (o del virtual).
> *Se recomienda hacerlo con HDD virtual, ya que es 100% más rápido. En el físico puede tardar hasta 90min, con el virtual en pocos minutos está listo*

## Hito 4: Despliegue Individual (De Disco USB a Equipo)

- Tarea 4.1: Retirar el disco duro USB (o virutal) de la VM Maestra y conectarlo a una de las VMs Clientes.

- Tarea 4.2: Arrancar la VM Cliente utilizando la ISO de Clonezilla.

- Tarea 4.3: Utilizar Clonezilla para volcar (restaurar) la imagen guardada en el disco USB (o virtual) hacia el disco duro local de la VM Cliente. Arrancar Windows para comprobar que el software de prueba está ahí.


## Hito 5: Clonación Masiva en Red (Modo Live/Broadcast)

- Tarea 5.1: Arrancar la VM Maestra con Clonezilla y configurar el modo "Clonezilla Lite Server" para servir el disco local a través de la red.

- Tarea 5.2: Configurar el servidor de Clonezilla seleccionando específicamente el modo de transmisión Broadcast.

- Tarea 5.3: Configurar las VMs Clientes restantes para que arranquen desde la red (PXE Boot).

- Tarea 5.4: Iniciar las VMs Clientes, esperar a que reciban IP del servidor Clonezilla, se unan a la sesión de Broadcast y comience la clonación masiva y simultánea.
  
> *Si el disco de destino es inferior en capacidad al de origen, se recomienda utilizar la opción -k1 para omitir la advertencia (buscar información acerca de esa opción para ver mas detalles)*

## Reglas e información adicional

Estas son las reglas del entorno y conceptos que debéis conocer antes de empezar a investigar cómo ejecutar los pasos:

- Analogía del hardware: En este laboratorio de Hyper-V, el Conmutador Virtual actúa exactamente igual que el Switch físico que utilizamos en ejercicios al que conectáis los equipos por cable. Asimismo, las Máquinas Virtuales (VMs) representan los portátiles o PCs físicos de los clientes. Todo lo que hagáis aquí se aplica al mundo real.

- Conflicto de DHCP: El conmutador virtual que creéis no debe tener ningún servicio DHCP habilitado ni dar salida a internet de forma automática. Cuando arranquéis Clonezilla en modo servidor para la clonación masiva, Clonezilla levantará su propio servidor DHCP para repartir IPs a los clientes que arranquen por red. Si hay dos DHCP en la misma red, la clonación fallará.

- Integración de USB en Hyper-V (Passthrough): A diferencia de otros programas, Hyper- V no tiene un botón de "Conectar USB" sencillo. Para pasar un disco USB físico a una VM, primero debéis ir al "Administrador de discos" del portátil host (vuestro Windows físico), buscar el USB y ponerlo en estado "Sin conexión" (Offline). Solo entonces Hyper-V os dejará añadirlo a la VM como un "Disco duro físico".

- La importancia del Sysprep: Antes de clonar un equipo Windows a otros, es obligatorio ejecutar la herramienta sysprep (System Preparation) con las opciones Generalizar (Generalize) y Apagar. Esto borra el SID (Identificador de Seguridad) único del sistema. Si clonáis un Windows sin hacer sysprep, todos los equipos de la red tendrán el mismo identificador, lo que causará caídas de red, problemas con el Directorio Activo y fallos masivos en las licencias y actualizaciones.

- Tamaño de los discos en Clonezilla: Por defecto, Clonezilla es estricto: el disco de destino debe ser igual o mayor que el disco de origen, sin importar cuánto espacio libre quede. Si el disco de la VM Maestra es de 50GB, los clientes deben tener 50GB o más. (Existen parámetros avanzados en Clonezilla como -icds para saltarse esta comprobación si la partición real es más pequeña que el disco, pero debéis respetar los tamaños en el diseño inicial del laboratorio).
