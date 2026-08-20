# Active Directory

**Active Directory (AD)** es el servicio de directorio de Windows Server: la base de datos central que guarda las identidades (usuarios, equipos, grupos) de una organización y decide **quién es cada uno** (autenticación) y **a qué puede acceder** (autorización). No es un servicio suelto, sino la columna vertebral sobre la que se apoyan el resto: File Server, Print Server, SharePoint y Exchange delegan en AD la identidad de sus usuarios.

Por debajo, AD es en realidad una pila de protocolos trabajando juntos:

- **Kerberos** — autentica a usuarios y equipos mediante tickets, sin enviar la contraseña por la red.
- **LDAP** — el lenguaje con el que se consulta y modifica el directorio.
- **DNS** — cómo los clientes *localizan* al controlador de dominio (registros SRV).
- **GPO + SYSVOL** — cómo se distribuyen configuraciones y scripts a miles de equipos.
- **Replicación** — cómo varios controladores de dominio mantienen una copia idéntica del directorio.

Un servidor que ejecuta AD DS y aloja esa base de datos se llama **controlador de dominio (DC)**.

---

## Entorno del laboratorio

| Elemento | Valor |
|---|---|
| Dominio | `btpcce.lab` (NetBIOS `BTPCCE`) |
| Bosque | Un único dominio, un único sitio |
| Controlador de dominio | `DC01` (10.10.10.11) — **único DC**, también servidor DNS |
| Red de clientes | 10.10.10.0/24 · GW 10.10.10.1 |

> **Nota:** este entorno tiene **un solo DC**, lo cual es un **punto único de fallo (SPOF)**: si cae, cae toda la autenticación del dominio. Es deliberado para practicar; varios ejercicios existen precisamente para que interiorices esa consecuencia.

---

## Marco de referencia: Modelo TCP/IP aplicado a AD

AD es un conjunto de protocolos apilados. Cuando algo falla, localiza en qué capa está el problema antes de tocar nada.

| Capa | Nombre | Qué usa AD en esa capa |
|:---:|---|---|
| **4** | Aplicación | Kerberos (autenticación), LDAP (directorio), DNS (localización del DC), SYSVOL/GPO, SMB |
| **3** | Transporte | Puertos: Kerberos 88, LDAP 389, LDAPS 636, Global Catalog 3268/3269, RPC 135, SMB 445, DNS 53 |
| **2** | Internet | Direccionamiento IP, alcanzar el DC, DNS del cliente correctamente configurado |
| **1** | Acceso a red | VLAN, port group del vSwitch, uplink |

> **Regla de diagnóstico:** siempre de capa 1 hacia capa 4. Antes de culpar a Kerberos o a las GPOs (capa 4), confirma que el DC responde (capa 2) y que los puertos están abiertos (capa 3). El 80% de los "problemas de AD" son en realidad DNS o red.

---

## Cómo se usan estos ejercicios

Cada caso te da un **escenario** y unos **síntomas**. Tu trabajo es **pensar el diagnóstico** respondiendo a las preguntas y llegando por tu cuenta a la causa raíz y a la solución. Aquí no encontrarás la respuesta escrita: esa es la idea. En los ejercicios de administración se te da el **objetivo** y las restricciones; el *cómo* lo averiguas tú.

---

## Índice de ejercicios

### [Nivel Básico →](active_directory_basico.md)
Fundamentos: identidad, localización del DC, inicio de sesión y gestión básica de objetos.

**Diagnóstico**
- AD-01 — El DC está caído: nadie puede iniciar sesión (no hay failover)
- AD-02 — Un equipo recién unido no encuentra el dominio
- AD-03 — "La relación de confianza entre esta estación y el dominio ha fallado"
- AD-04 — Un usuario no puede iniciar sesión: cuenta deshabilitada o caducada

**Administración**
- ADM-01 — Diseñar y crear la estructura de OUs del dominio
- ADM-02 — Alta de usuario nuevo con grupo y OU correctos
- ADM-03 — Crear un grupo de seguridad y añadir miembros
- ADM-04 — Restablecer una contraseña y desbloquear una cuenta

### [Nivel Intermedio →](active_directory_intermedio.md)
Profundidad: internals de GPO, SYSVOL/replicación, Kerberos, recuperación del directorio y cuentas de servicio.

**Diagnóstico**
- AD-01 — Una GPO no se aplica a un equipo concreto (filtrado de seguridad)
- AD-02 — Los usuarios entran, pero las GPOs y los scripts de inicio no se ejecutan
- AD-03 — El inicio de sesión falla en un equipo por desfase de reloj
- AD-04 — Una cuenta se bloquea sola varias veces al día
- AD-05 — Se borra por error una OU con todos los usuarios — recuperación
- AD-06 — Un servicio falla con error Kerberos por un SPN duplicado

**Administración**
- ADM-01 — Crear un grupo de seguridad y delegar control sobre una OU
- ADM-02 — Crear una GPO y vincularla a una OU
- ADM-03 — Política de contraseñas específica (PSO) para un grupo
- ADM-04 — Crear una cuenta de servicio administrada (gMSA)
- ADM-05 — Aplicar la estrategia de anidamiento AGDLP para permisos
- ADM-06 — Habilitar la Papelera de reciclaje de AD y probar una restauración
