# Ejercicios de Interiorización

**Sgto. Narros — PAIB2.0**
**Plataforma de virtualización:** VMware ESXi 8.0 + vCenter

---

## Marco de referencia: Modelo TCP/IP

| Capa | Nombre | Qué controla |
|:---:|---|---|
| **4** | Aplicación | Protocolos de servicio: DNS, HTTP/S, SMTP, LDAP, SMB, Kerberos, RDP... |
| **3** | Transporte | Puertos TCP/UDP, sesiones, fiabilidad de entrega |
| **2** | Internet | Direccionamiento IP, enrutamiento, ICMP |
| **1** | Acceso a red | Tramas Ethernet, VLANs, MACs, switches físicos, vSwitches ESXi |

> **Regla de diagnóstico:** siempre se sube de capa 1 a capa 4. Si no hay red, no hay servicio.

---

## Tabla resumen

|   #   | Título                                                               | Módulo                 | Capa principal | Dificultad |
| :---: | -------------------------------------------------------------------- | ---------------------- | :------------: | :--------: |
| VJ-01 | VM del nodo no arranca tras revertir snapshot                        | Virtualización         |     **1**      |    ★☆☆     |
| VJ-02 | vMotion falla — la VM queda en estado desconocido                    | Virtualización         |     **1**      |    ★★☆     |
| VJ-03 | Todas las VMs de un hipervisor pierden red simultáneamente           | Virtualización         |     **1**      |    ★★☆     |
| SW-01 | El hipervisor desaparece de vCenter pero sigue activo                | Switch de núcleo       |     **1**      |    ★★☆     |
| AD-01 | Ningún usuario del nodo puede iniciar sesión                         | Active Directory       |   **2 / 4**    |    ★★☆     |
| AD-02 | La GPO de escritorio no se aplica en los equipos del N131            | Active Directory       |     **4**      |    ★★☆     |
| AD-03 | Los DCs del N121 no sincronizan con el PDC del N111                  | Active Directory       |   **3 / 4**    |    ★★★     |
| FS-01 | Un usuario puede acceder a la carpeta pero no puede guardar archivos | File Server            |     **4**      |    ★☆☆     |
| FS-02 | El disco del File Server está al 100% sin previo aviso               | File Server            |     **4**      |    ★★☆     |
| FS-03 | El espacio de nombres DFS deja de resolver tras caída de un nodo     | File Server            |   **2 / 4**    |    ★★☆     |
| PS-01 | La cola de impresión está bloqueada — nadie puede imprimir           | Print Server           |     **4**      |    ★☆☆     |
| PS-02 | La impresora no aparece en los equipos de un nodo concreto           | Print Server           |     **4**      |    ★★☆     |
| EX-01 | Un usuario no puede enviar correo — su buzón está lleno              | Exchange               |     **4**      |    ★☆☆     |
| EX-02 | La base de datos de buzones no está montada — nadie recibe correo    | Exchange               |     **4**      |    ★★☆     |
| EX-03 | Outlook no configura la cuenta automáticamente (Autodiscover falla)  | Exchange               |   **2 / 4**    |    ★★★     |
| SP-01 | SharePoint devuelve error 503 — servicio no disponible               | SharePoint             |   **3 / 4**    |    ★★☆     |
| SP-02 | Un usuario recibe "Acceso denegado" en un sitio de SharePoint        | SharePoint             |     **4**      |    ★★☆     |
| SP-03 | El navegador avisa de certificado no confiable en SharePoint         | SharePoint             |     **4**      |    ★★★     |
| SD-01 | El equipo no arranca por PXE — no recibe imagen del servidor         | Servidor de despliegue |   **2 / 3**    |    ★★☆     |
| SD-02 | La task sequence falla al intentar unir el equipo al dominio         | Servidor de despliegue |     **4**      |    ★★☆     |
| SD-03 | La imagen desplegada tiene el mismo SID en todos los equipos         | Servidor de despliegue |     **4**      |    ★★★     |

### Ejercicios de administración

| # | Título | Módulo | Capa principal | Dificultad |
|:---:|---|---|:---:|:---:|
| ADM-VJ-01 | Configurar vSwitches y port groups en un ESXi recién desplegado | Virtualización | **1** | ★★☆ |
| ADM-VJ-02 | Configurar los VMkernel adapters (gestión, vMotion, almacenamiento) | Virtualización | **1 / 2** | ★★☆ |
| ADM-VJ-03 | Añadir y configurar un datastore al hipervisor | Virtualización | **1** | ★★★ |
| ADM-AD-01 | Alta de usuario nuevo en el dominio con grupos y OU correctos | Active Directory | **4** | ★☆☆ |
| ADM-AD-02 | Crear grupo de seguridad y delegar control sobre una OU | Active Directory | **4** | ★★☆ |
| ADM-DNS-01 | Registrar un nuevo servidor en DNS (registro A y PTR) | DNS | **2 / 4** | ★☆☆ |
| ADM-DNS-02 | Crear un alias CNAME para un servicio web interno | DNS | **4** | ★☆☆ |
| ADM-FS-01 | Crear carpeta compartida con permisos SMB y NTFS correctos | File Server | **4** | ★★☆ |
| ADM-FS-02 | Añadir un nuevo destino al espacio de nombres DFS | File Server | **4** | ★★☆ |
| ADM-EX-01 | Crear buzón de correo para un usuario nuevo | Exchange | **4** | ★☆☆ |
| ADM-EX-02 | Crear buzón compartido y asignar permisos de acceso | Exchange | **4** | ★★☆ |
| ADM-EX-03 | Crear lista de distribución y cargar miembros masivamente | Exchange | **4** | ★★☆ |
| ADM-SP-01 | Crear colección de sitios y asignar administrador | SharePoint | **4** | ★★☆ |
| ADM-SP-02 | Configurar permisos de un sitio con grupos de AD | SharePoint | **4** | ★★☆ |
| ADM-PS-01 | Agregar impresora al servidor y desplegarla por GPO | Print Server | **4** | ★★☆ |

---

## MÓDULO: VIRTUALIZACIÓN (ESXi + vCenter)

---

### VJ-01 — VM del nodo no arranca tras revertir snapshot
**Capa principal: 1 — Acceso a red**

**Escenario:**
Un operador del N121 revierte un snapshot de `N121DC1` para recuperar un estado anterior después de una configuración incorrecta. Tras la reversión, la VM arranca sin errores visibles en vCenter pero no tiene conectividad de red. El resto de VMs del mismo hipervisor funcionan con normalidad.

**Síntomas:**
- `N121DC1` arranca y el guest OS responde en consola de vCenter
- Sin IP asignada — `ipconfig` muestra la NIC sin configuración válida
- VMware Tools reporta: `Guest NIC disconnected from virtual switch`
- El port group del resto de VMs en el mismo host funciona correctamente

**Preguntas de diagnóstico:**
1. ¿Qué captura un snapshot en ESXi además del estado de la memoria y el disco? ¿Puede capturar la configuración del vSwitch?
2. En vCenter, ¿cómo verificarías que la NIC virtual de la VM está conectada al port group correcto?
3. Si la NIC aparece conectada al port group pero sigue sin red, ¿cuál sería el siguiente nivel de diagnóstico?

**Diagnóstico y resolución:**
- vCenter → VM → Editar configuración → Adaptador de red
- Verificar que el port group asignado existe y tiene el VLAN ID correcto para el N121 (VLAN 1209 para clientes, VLAN 1210 para servidores)
- Comprobar que "Conectado" y "Conectar al encender" están marcados
- Si el port group fue renombrado o eliminado entre el snapshot y la reversión: reasignar al port group correcto
- Tras reasignar: comprobar conectividad con `ping` al gateway del nodo (`102.197.133.129`)

**Concepto clave:** Los snapshots en ESXi capturan el estado del disco (VMDK delta) y la memoria RAM de la VM, pero NO la configuración del vSwitch ni los port groups del host. Si el port group cambió de nombre o fue eliminado después de crear el snapshot, al revertir la VM queda referenciando un port group que ya no existe o tiene otro nombre. La NIC aparece "conectada" pero sin port group válido = sin red.

---

### VJ-02 — vMotion falla — la VM queda en estado desconocido
**Capa principal: 1 — Acceso a red**

**Escenario:**
Se intenta migrar en caliente `N111EX01` desde el ESXSRV01 al ESXSRV02 del N111 para realizar mantenimiento planificado. La migración llega al 37% y falla. La VM queda en estado "Invalid" en vCenter.

**Síntomas:**
- Error en vCenter: `vMotion migration [VM] failed. The migration has exceeded the maximum switchover time of 100 seconds`
- La VM no responde en origen ni en destino

**Preguntas de diagnóstico:**
1. ¿Qué transfiere vMotion por la red durante una migración en caliente? ¿Por qué necesita una red dedicada?
2. Una VM en estado "Invalid" en vCenter, ¿está corriendo en algún host? ¿Cuál es el riesgo de dejarla en ese estado?
3. ¿Qué pasos seguirías para recuperar la VM antes de volver a intentar la migración?

**Diagnóstico y resolución:**
```bash
# Verificar conectividad VMkernel de vMotion entre hosts (desde ESXi por SSH):
vmkping -I vmk1 "IP_RED_VMOTION"    # ping al VMkernel de vMotion del host destino

# Verificar que vmk1 tiene habilitado el servicio de vMotion:
esxcli network ip interface list    # ver interfaces VMkernel
# En vCenter: Host → Configure → VMkernel Adapters → vmk1 → verificar "vMotion" marcado

# Para recuperar la VM en estado "Invalid":
# vCenter → VM → clic derecho → "Remove from Inventory" (NO delete)
# Ir al host origen → Datastores → Examinar → localizar el .vmx
# Clic derecho sobre el .vmx → "Register VM"
# La VM vuelve al inventario con sus datos intactos
```

**Concepto clave:** vMotion copia la RAM activa de la VM por la red mientras la VM sigue corriendo. Cuando la copia alcanza un umbral de consistencia, congela la VM por milisegundos, transfiere el estado final y reanuda en el destino. Si la red de vMotion (vmk1) es lenta o inestable, el proceso supera el tiempo máximo de switchover y falla. El estado "Invalid" significa que vCenter ha perdido el rastro de la VM — los datos en el VMDK están intactos, pero hay que re-registrar la VM manualmente.

---

### VJ-03 — Todas las VMs de un hipervisor pierden red simultáneamente
**Capa principal: 1 — Acceso a red**

**Escenario:**
A las 14:32, todas las VMs alojadas en `ESXSRV03CESH2` (172.21.0.4) pierden conectividad de red. El hipervisor sigue respondiendo en su IP de gestión desde vCenter. Las VMs del ESXSRV01 y ESXSRV02 no están afectadas.

**Síntomas:**
- Todas las VMs del ESXSRV03 sin red simultáneamente
- vCenter mantiene conexión con el hipervisor (su VMkernel de gestión `vmk0` sigue activo)
- En vCenter, el vSwitch de producción del ESXSRV03 muestra el uplink en estado "down"

**Preguntas de diagnóstico:**
1. ¿Por qué el hipervisor sigue respondiendo en vCenter si todas sus VMs están sin red? ¿Qué redes distintas tiene un host ESXi?
2. El uplink del vSwitch de producción está "down". ¿Qué componentes físicos o lógicos pueden causar ese estado? ¿Cuál de ellos está en tu ámbito y cuál en el del administrador de redes?

**Diagnóstico y resolución (tu ámbito — ESXi/vCenter):**
```bash
# Desde vCenter: Host → Configure → Virtual Switches
# Verificar estado del uplink del vSwitch de producción
# Si el uplink (vmnic1 o vmnic2) aparece "down":

# Desde SSH al ESXi — confirmar estado de las NICs físicas:
esxcli network nic list
# Si la NIC aparece "Down" con link speed 0 → problema físico o en el switch

# Verificar si hay uplink redundante (NIC Teaming):
esxcli network vswitch standard policy failover get --vswitch-name vSwitch1
```
> Si el uplink físico está down en la NIC del ESXi: escalar al administrador de redes con `Hostname: ESXSRV03CESH2, NIC: vmnic1, Switch de producción: vSwitch1, VLAN afectada: VLANs de misión`.

**Concepto clave:** ESXi separa el tráfico de gestión (vmk0 — con la IP del hipervisor) del tráfico de VMs (port groups en vSwitch de producción). Pueden usar NICs físicas distintas. Por eso el hipervisor sigue siendo gestionable aunque sus VMs pierdan red. Alertas masivas y simultáneas de un único host siempre apuntan al vSwitch o al uplink físico — nunca a un problema individual de cada VM.

---

## MÓDULO: SWITCH DE NÚCLEO

---

### SW-01 — El hipervisor desaparece de vCenter pero sigue activo
**Capa principal: 1 — Acceso a red**

> Este ejercicio requiere identificar el problema con precisión y comunicarlo al administrador de redes. No se espera que configures el switch.

**Escenario:**
`ESXSRV02CESH2` (172.21.0.3, red de gestión VLAN 3900) desaparece de vCenter con estado "Not Responding". Las VMs alojadas en ese hipervisor dejan de ser visibles desde vCenter, pero los usuarios del N111 siguen accediendo a los servicios con normalidad.

**Síntomas:**
- vCenter: ESXSRV02 en estado "Not Responding"
- Ping a `172.21.0.3` desde la red de gestión → timeout
- Ping a `172.21.0.3` desde ESXSRV01 (misma VLAN 3900) → timeout también
- Las VMs del ESXSRV02 siguen dando servicio (Exchange, SharePoint responden)

**Preguntas de diagnóstico:**
1. ¿Por qué las VMs siguen funcionando si vCenter no puede contactar con el hipervisor?
2. El hecho de que ESXSRV01 (misma VLAN, mismo switch) tampoco llegue al ESXSRV02, ¿qué descarta? ¿En qué componente empieza a estar el problema?
3. ¿Qué información concreta le darías al administrador de redes para localizar el problema en el switch?

**Lo que confirmás desde tu ámbito:**
- Acceder al ESXSRV02 por consola física o iLO/IPMI: si el hipervisor responde → el problema es de red de gestión, no del host
- Si el host está vivo: escalar con exactamente esto: _"ESXSRV02CESH2, IP 172.21.0.3, VLAN 3900 — el host responde por consola directa pero es inaccesible desde la red. Otro host en la misma VLAN tampoco le llega. Necesito que revises el puerto del switch core al que está conectado."_
- Mientras se resuelve: las VMs siguen corriendo solas — ESXi no necesita vCenter para funcionar

**Concepto clave:** vCenter es el plano de gestión, no el plano de datos. Un hipervisor ESXi funciona perfectamente sin vCenter — las VMs siguen corriendo con total normalidad. vCenter solo es necesario para gestionar, migrar o monitorizar. La pérdida de visibilidad en vCenter no implica caída de servicio, pero sí pérdida de control: no podés apagar, mover ni crear VMs hasta recuperar la conexión.

---

## MÓDULO: ACTIVE DIRECTORY

---

### AD-01 — Ningún usuario del nodo puede iniciar sesión
**Capa principal: 2 — Internet · 4 — Aplicación (Kerberos)**

**Escenario:**
A las 08:15, ningún usuario del N141 puede autenticarse en sus equipos. El error en pantalla es "No se puede iniciar sesión. El servicio de inicio de sesión no está disponible". Los equipos son miembros de `btpcce.esp`. Los DCs del nodo son `N141DC1` (102.197.141.131) y `N141DC2` (102.197.141.132).

**Síntomas:**
- Error de autenticación en todos los equipos del N141
- Ping a `N141DC1` → timeout
- Ping a `N141DC2` → timeout también
- Los usuarios con sesión ya abierta pueden seguir trabajando sin interrupciones

**Preguntas de diagnóstico:**
1. ¿Qué protocolo usa Windows para autenticar contra el dominio? ¿Por qué necesita comunicación de red con el DC para iniciar sesión?
2. ¿Por qué los usuarios con sesión ya abierta pueden seguir trabajando? ¿Qué mecanismo de caché interviene?
3. Si ambos DCs del nodo están caídos, ¿hacia dónde intentaría ir el cliente para autenticarse?

**Diagnóstico y resolución:**
```powershell
# Desde un equipo del N141:
ping 102.197.141.131                      # DC1
ping 102.197.141.132                      # DC2
nltest /dsgetdc:btpcce.esp               # qué DC está usando el cliente
nltest /sc_query:btpcce.esp             # estado del canal seguro con el dominio

# Si los DCs no responden: verificar estado de las VMs N141DC1/N141DC2 en vCenter
# Si los DCs responden pero la autenticación sigue fallando:
dcdiag /test:netlogons                   # desde el DC — verificar servicio Netlogon
Get-Service Netlogon
```

**Concepto clave:** Kerberos necesita contactar con el DC para obtener un ticket antes de iniciar sesión. Sin DC alcanzable, la autenticación falla completamente. Sin embargo, Windows cachea las últimas credenciales localmente — por eso quien ya tiene sesión abierta puede continuar trabajando. Al cerrar sesión, si el DC sigue caído, no podrán volver a entrar. El primer paso siempre es confirmar si las VMs de los DCs están encendidas en vCenter antes de escalar a un problema de red.

---

### AD-02 — La GPO de escritorio no se aplica en los equipos del N131
**Capa principal: 4 — Aplicación (LDAP / GPO)**

**Escenario:**
Se publica desde el N111 una nueva GPO que configura el fondo de escritorio corporativo para todos los equipos de `btpcce.esp`. Al día siguiente, los usuarios del N131 siguen con el fondo anterior. Los del N111 y N121 tienen el fondo nuevo.

**Síntomas:**
- GPO aplicada correctamente en N111 y N121
- N131: `gpresult /r` en un equipo del nodo — la GPO **no aparece en la lista de GPOs aplicadas**
- `gpupdate /force` en ese equipo tarda 2 minutos sin resultado visible
- `repadmin /showrepl` desde `N131DC1` muestra errores de replicación hacia `N111PDC`

**Preguntas de diagnóstico:**
1. ¿Cómo llegan las GPOs a los equipos? ¿Qué rol juega el DC local del nodo en ese proceso?
2. Si el DC del N131 no ha recibido la GPO desde el PDC, ¿qué recibe el equipo cuando hace `gpupdate`?
3. ¿Cuál es el orden correcto de diagnóstico: primero los permisos de la GPO o primero el estado de la replicación?

**Diagnóstico y resolución:**
```powershell
# 1. Verificar replicación — este es el problema raíz:
repadmin /showrepl N131DC1
repadmin /replsummary

# 2. Forzar replicación si hay conectividad con el N111:
repadmin /replicate N131DC1 N111PDC dc=btpcce,dc=esp

# 3. Una vez replicado, confirmar que el DC tiene la GPO:
Get-GPO -All | Where-Object {$_.DisplayName -like "*escritorio*"}

# 4. En el equipo cliente — informe detallado:
gpresult /h C:\gp_report.html
gpupdate /force
```

**Concepto clave:** Las GPOs viven en SYSVOL y en la base de datos de AD del DC. Para que un equipo las reciba, necesita que su DC local las tenga replicadas. Si la replicación entre el PDC (N111) y el DC del N131 está rota, los equipos del N131 reciben la versión antigua — sin ningún error visible en el cliente. `gpresult` es la herramienta diagnóstica definitiva: te dice qué GPOs se aplicaron, cuáles no, y por qué.

---

### AD-03 — Los DCs del N121 no sincronizan con el PDC del N111
**Capa principal: 3 — Transporte · 4 — Aplicación (replicación AD)**

**Escenario:**
Tres días después del despliegue del N121, se crea en `N111PDC` (102.194.249.11) una nueva cuenta de usuario. Al intentar usarla desde el N121, el error es "El nombre de usuario o la contraseña no son correctos".

**Síntomas:**
- La cuenta existe y funciona perfectamente desde el N111 y el N141
- `repadmin /showrepl` desde `N121DC1` muestra errores hacia `N111PDC`
- Error concreto: `8453 — Replication access was denied`
- La cuenta de servicio de replicación en N121DC1 no tiene permisos correctos

**Preguntas de diagnóstico:**
1. ¿Qué información sincroniza la replicación de AD? ¿Solo cuentas de usuario o algo más?
2. El error `8453 — Replication access was denied` es un problema de permisos en AD (capa 4). ¿Qué capas inferiores hay que descartar primero antes de llegar a esa conclusión?
3. Si la replicación lleva 3 días rota en silencio, ¿qué otros problemas derivados pueden haberse acumulado además de cuentas de usuario no sincronizadas?

**Diagnóstico y resolución:**
```powershell
# Paso 1 — descartar problemas de red:
Test-NetConnection -ComputerName 102.194.249.11 -Port 389    # LDAP
Test-NetConnection -ComputerName 102.194.249.11 -Port 135    # RPC Endpoint Mapper
Test-NetConnection -ComputerName 102.194.249.11 -Port 3268   # Global Catalog

# Paso 2 — si la red está bien, investigar el error de permisos:
repadmin /showrepl N121DC1 /errorsonly
dcdiag /test:replications /v

# Paso 3 — corregir permisos en AD:
# ADUC → dominio → Propiedades → Seguridad → "Enterprise Domain Controllers"
# → debe tener "Replicating Directory Changes" y "Replicating Directory Changes All"

# Paso 4 — forzar sincronización tras corregir:
repadmin /syncall /Ade
```

**Concepto clave:** La replicación de AD sincroniza TODO: cuentas, contraseñas, GPOs, DNS integrado en AD, relaciones de confianza, cambios de esquema. Una replicación rota no genera ningún error visible en el cliente — se acumula en silencio como objetos desincronizados. Lo más peligroso: cambios de contraseña y bloqueos de cuentas no llegan al nodo afectado, lo que puede dejar usuarios sin acceso o con contraseñas antiguas activas. Hay que monitorizar `repadmin /replsummary` regularmente.

---

## MÓDULO: FILE SERVER

---

### FS-01 — Un usuario puede acceder a la carpeta pero no puede guardar archivos
**Capa principal: 4 — Aplicación (permisos SMB/NTFS)**

**Escenario:**
El Sgto. García, del N121, accede sin problema a `\\N121FS\operaciones` y ve todos los archivos. Cuando intenta guardar un documento modificado recibe "No tiene permiso para guardar en esta ubicación". El administrador confirma que García pertenece al grupo `GG_Operaciones` en AD, y ese grupo sí tiene permiso de escritura.

**Síntomas:**
- Lectura funciona correctamente
- Escritura falla con error de permisos
- Otros miembros de `GG_Operaciones` escriben sin problema
- García fue añadido al grupo hace 20 minutos — justo antes de que empezara a tener el problema

**Preguntas de diagnóstico:**
1. Un usuario pertenece a un grupo en AD pero el permiso no se refleja en el acceso. ¿Qué mecanismo de Windows es responsable de esto y cómo se resuelve?
2. Explica la "regla de intersección" de permisos SMB. Si el permiso de share es "Control Total" y el permiso NTFS es "Lectura", ¿qué permiso efectivo tiene el usuario?
3. ¿Cómo verificarías los permisos efectivos de García sobre esa carpeta sin necesidad de pedirle que cierre sesión?

**Diagnóstico y resolución:**
```powershell
# Verificar permisos efectivos desde el servidor:
# Propiedades de la carpeta → Seguridad → Opciones avanzadas → Acceso efectivo
# → Seleccionar usuario "btpcce\garcia" → Ver acceso efectivo

# Verificar permisos del share:
Get-SmbShareAccess -Name "operaciones"

# Verificar permisos NTFS:
Get-Acl "D:\shares\operaciones" | Format-List

# La solución inmediata:
# García debe cerrar sesión y volver a iniciarla
# → Windows reconstruye el token de acceso incluyendo el nuevo grupo
```

**Concepto clave:** Cuando un usuario inicia sesión, Windows construye un token de acceso que contiene todos sus grupos en ese momento. Si se añade al usuario a un grupo DESPUÉS de que inició sesión, el token no incluye ese grupo hasta que cierra sesión y vuelve a entrar. Es la causa más frecuente de "está en el grupo pero no tiene acceso". La regla de intersección es inamovible: el permiso efectivo es siempre el más restrictivo entre Share y NTFS.

---

### FS-02 — El disco del File Server está al 100% sin previo aviso
**Capa principal: 4 — Aplicación (FSRM)**

**Escenario:**
Los usuarios no pueden guardar archivos desde las 23:30. No hubo ninguna alerta previa de tendencia de crecimiento.

**Síntomas:**
- Volumen `D:\` al 100%
- Nuevas escrituras en shares fallan
- FSRM no tiene cuotas configuradas en ese volumen
- La carpeta `D:\shares\videoconferencia` ocupa 180 GB — subió 60 GB en las últimas 24 horas

**Preguntas de diagnóstico:**
1. ¿Qué es FSRM y qué dos mecanismos preventivos debería haber tenido configurados para evitar esta situación?
2. ¿Cuál es la diferencia entre una cuota **estricta** y una cuota **blanda** en FSRM? ¿Cuál habría sido más adecuada aquí?
3. ¿Qué tipo de filtro de archivos aplicarías para evitar que se guarden archivos de vídeo en carpetas de documentos?

**Diagnóstico y resolución (acción inmediata):**
```powershell
# Identificar qué está ocupando el espacio:
Get-ChildItem "D:\shares" -Recurse | Sort-Object Length -Descending | Select-Object -First 20 FullName, Length

# Liberar espacio de emergencia (verificar con responsable antes de borrar)

# Configurar cuota para el futuro en FSRM:
# File Server Resource Manager → Quota Management → Quotas → Create Quota
# Ruta: D:\shares\videoconferencia | Límite: 50 GB | Tipo: Estricta
# Alerta al 80% → notificación por correo al administrador

# Configurar filtro de archivos:
# File Screening Management → File Screens → Create File Screen
# Ruta: D:\shares\documentos | Plantilla: Block Video Files | Tipo: Activo
```

**Concepto clave:** FSRM es la herramienta de gobierno del File Server — sin cuotas y sin filtros, el disco es tierra de nadie. Cuota blanda: alerta pero no bloquea (flexible, útil cuando hay picos puntuales). Cuota estricta: bloquea sin excepción (adecuada cuando el desbordamiento afecta a todos los usuarios). La gestión reactiva siempre llega tarde

---

### FS-03 — El espacio de nombres DFS deja de resolver tras caída de un nodo
**Capa principal: 2 — Internet · 4 — Aplicación (DFS)**

**Escenario:**
El N131 sufre una caída de red de 2 horas. Al recuperarse, los usuarios de todos los nodos reportan que `\\btpcce.esp\datos` no funciona, aunque el File Server del N111 está operativo. El acceso directo `\\N111FS1\datos` sí funciona.

**Síntomas:**
- `\\btpcce.esp\datos` → "No se puede obtener acceso a la ruta de acceso de red"
- `\\N111FS1\datos` → funciona sin problema
- `N131FS` era uno de los servidores de namespace DFS y quedó en estado inconsistente tras la caída
- La caída de N131FS afectó al namespace entero, no solo al nodo 131

**Preguntas de diagnóstico:**
1. ¿Qué ventaja aporta un namespace DFS respecto al UNC directo? ¿Y qué riesgo introduce si uno de los servidores de namespace falla mal?
2. Si el namespace tiene dos servidores (N111FS1 y N131FS), ¿por qué la caída del N131 afecta también a usuarios del N111?
3. ¿Cómo marcarías como offline el servidor de namespace problemático sin eliminar la configuración?

**Diagnóstico y resolución:**
```powershell
# Verificar estado del namespace y sus servidores:
Get-DfsnRoot -Path "\\btpcce.esp\datos"
Get-DfsnRootTarget -Path "\\btpcce.esp\datos"

# Marcar N131FS como offline para que el namespace funcione solo con N111FS1:
Set-DfsnRootTarget -Path "\\btpcce.esp\datos" -TargetPath "\\N131FS\datos" -State Offline

# Verificar estado de replicación DFS en N131FS una vez recuperado:
Get-DfsrState -ComputerName N131FS
dfsrdiag replicationstate /member:N131FS

# Forzar resincronización con el servidor principal:
dfsrdiag syncnow /partner:N111FS1 /rgname:"DatosReplicacion" /member:N131FS

# Volver a poner N131FS online una vez verificada la consistencia:
Set-DfsnRootTarget -Path "\\btpcce.esp\datos" -TargetPath "\\N131FS\datos" -State Online
```

**Concepto clave:** DFS Namespaces da una ruta única independiente del servidor físico. Si un servidor de namespace queda en estado inconsistente (no simplemente caído), puede contaminar las respuestas de referencia para todos los clientes, haciendo inaccesible el namespace aunque los otros servidores estén perfectos. La diferencia entre "servidor caído" (namespace falla sobre ese servidor, redirige al otro) y "servidor en estado inválido" (envenena las referencias = todos afectados) es la que explica por qué los usuarios del N111 también notaron el problema.

---

## MÓDULO: PRINT SERVER

---

### PS-01 — La cola de impresión está bloqueada — nadie puede imprimir
**Capa principal: 4 — Aplicación (Spooler)**

**Escenario:**
A las 10:15, el personal del N111 reporta que no puede imprimir. Los trabajos se envían pero quedan en estado "Eliminando" o "Error" sin moverse. Reiniciar la impresora física no tiene ningún efecto.

**Síntomas:**
- Trabajos atascados con estado "Eliminando" en la consola de impresión
- No se pueden cancelar manualmente desde el cliente ni desde el servidor
- La impresora física está encendida, con papel y sin error en el panel

**Preguntas de diagnóstico:**
1. ¿Qué es el servicio Spooler y qué función cumple en el flujo de impresión? ¿Por qué reiniciar la impresora física no resuelve un bloqueo de Spooler?
2. ¿Dónde están almacenados físicamente los trabajos de impresión en el servidor? ¿Qué ocurre si hay un archivo corrupto en esa carpeta?
3. ¿Cuál es el procedimiento correcto para limpiar una cola atascada sin perder la configuración de las impresoras?

**Diagnóstico y resolución:**
```powershell
# Procedimiento estándar — en el servidor de impresión:

# 1. Detener el servicio Spooler:
net stop spooler

# 2. Eliminar los trabajos bloqueados (.SHD y .SPL):
Remove-Item "C:\Windows\System32\spool\PRINTERS\*" -Force

# 3. Reiniciar el servicio:
net start spooler

# 4. Verificar que la cola está limpia:
# printmanagement.msc → servidor → Impresoras → colas

# Si el problema es recurrente — revisar logs del sistema:
Get-EventLog -LogName System -Source "Print Spooler" -Newest 20
```

**Concepto clave:** El Spooler recibe los trabajos de impresión, los almacena en `C:\Windows\System32\spool\PRINTERS` como archivos `.SPL` (datos) y `.SHD` (cabecera) y los envía a la impresora en orden. Un archivo corrupto bloquea toda la cola porque el Spooler no puede saltar al siguiente trabajo. Parar el servicio, borrar los archivos físicos y reiniciar es el único procedimiento fiable — desde la consola gráfica no es posible eliminar un trabajo verdaderamente bloqueado.

---

### PS-02 — La impresora no aparece en los equipos de un nodo concreto
**Capa principal: 4 — Aplicación (GPO / SMB)**

**Escenario:**
Se configura en el servidor de impresión del N111 la impresora `\\N111-PS\Impresora-PlantaMando` y se despliega por GPO a todos los usuarios del dominio. Los usuarios del N111 la ven y la usan sin problema. Los del N121 y N131 no la ven.

**Síntomas:**
- `gpresult /r` en un equipo del N121: la GPO de impresoras aparece como aplicada
- La impresora sigue sin estar en "Dispositivos e impresoras"
- `Test-NetConnection -ComputerName N111-PS -Port 445` desde N121 → timeout
- El servidor de impresión está en VLAN 1110 del N111 (`102.194.249.0/25`)

**Preguntas de diagnóstico:**
1. La GPO está aplicada pero la impresora no aparece. ¿Qué dos condiciones necesita el cliente para instalar una impresora desplegada por GPO?
2. ¿Por qué TCP/445 (SMB) es crítico para el despliegue de impresoras? ¿Qué descarga el cliente por ese canal?
3. Si el puerto 445 está bloqueado entre el N121 y el servidor de impresión del N111, ¿qué componente lo está bloqueando y a quién corresponde resolverlo?

**Diagnóstico y resolución:**
```powershell
# Verificar conectividad con el servidor de impresión:
Test-NetConnection -ComputerName 102.194.249.X -Port 445    # SMB — necesario para drivers

# Verificar resolución de nombres:
Resolve-DnsName N111PS.btpcce.esp

# Si TCP/445 no llega:
# → El ASA está bloqueando SMB inter-VLAN entre N121 (VLAN 1209) y N111 (VLAN 1110)
# → Escalar: origen VLAN 1209, destino IP del Print Server, puerto TCP/445

# Una vez resuelta la conectividad, forzar re-aplicación de GPO:
gpupdate /force

# Verificar resultado:
Get-Printer | Where-Object {$_.Name -like "*Mando*"}
```

**Concepto clave:** La GPO de impresoras tiene dos fases: (1) la GPO le dice al cliente qué impresora instalar, (2) el cliente descarga el driver del servidor de impresión por SMB/TCP-445. Si la fase 2 falla por firewall, la impresora nunca se instala aunque la GPO esté aplicada. En entornos multi-nodo con VLANs separadas por el ASA, este bloqueo es la causa más frecuente de "GPO aplicada, impresora no aparece".

---

## MÓDULO: EXCHANGE SERVER 2019

---

### EX-01 — Un usuario no puede enviar correo — su buzón está lleno
**Capa principal: 4 — Aplicación (cuotas Exchange)**

**Escenario:**
El Cpt  (N111) llama reportando que no puede enviar correos desde hace una hora. Puede recibir correos sin problema, pero al intentar enviar Outlook muestra "Su buzón está lleno".

**Síntomas:**
- Error al enviar: `Your mailbox is full`
- Puede recibir pero no enviar
- Su buzón ocupa 4.8 GB de los 5 GB asignados
- Lleva semanas sin vaciar la papelera

**Preguntas de diagnóstico:**
1. Exchange tiene tres umbrales de cuota distintos. ¿Cuáles son y qué comportamiento activa cada uno?
2. Vaciar la papelera en Outlook, ¿libera espacio inmediatamente en Exchange? ¿Hay algo más que verificar?
3. ¿Cómo verificarías desde EMS el tamaño actual y los límites del buzón del cpt?

**Diagnóstico y resolución:**
```powershell
# Verificar tamaño y cuotas:
Get-Mailbox -Identity "cpt" | Select ProhibitSendQuota, ProhibitSendReceiveQuota, IssueWarningQuota
Get-MailboxStatistics -Identity "cpt" | Select TotalItemSize, ItemCount, DeletedItemCount

# Limpiar elementos eliminados recuperables (papelera de 2º nivel de Exchange):
Search-Mailbox -Identity "cpt" -SearchDumpsterOnly -DeleteContent -Force

# Si se necesita ampliar la cuota temporalmente:
Set-Mailbox -Identity "cpt" -ProhibitSendQuota 7GB -IssueWarningQuota 6GB

# Ver todos los buzones que superan el 90% de su cuota:
Get-MailboxStatistics | Where-Object {$_.TotalItemSize -gt 4.5GB} | Select DisplayName, TotalItemSize
```

**Concepto clave:** Exchange tiene tres umbrales: `IssueWarningQuota` (aviso al usuario), `ProhibitSendQuota` (puede recibir, no puede enviar — aquí está el cpt) y `ProhibitSendReceiveQuota` (ni envía ni recibe). Vaciar la papelera en Outlook mueve los elementos a la "papelera recuperable" de Exchange — no libera espacio real hasta que esa segunda papelera también se vacía con `Search-Mailbox -SearchDumpsterOnly`. Sin ese paso, el buzón puede parecer vacío visualmente y seguir sin espacio.

---

### EX-02 — La base de datos de buzones no está montada
**Capa principal: 4 — Aplicación (Exchange BD)**

**Escenario:**
Tras el reinicio planificado del servidor `N121EX` por Windows Update (03:00), a las 08:00 los usuarios del N121 reportan que no reciben correo. Outlook muestra "No se puede conectar con el servidor de correo". Los correos enviados desde el N111 hacia el N121 rebotan con error 451.

**Síntomas:**
- OWA `https://N121EX/owa` carga la página pero da error al abrir el buzón
- EAC: la base de datos `BD-N121` aparece como `Dismounted`
- Log de aplicación: `MSExchangeIS — Database BD-N121 failed to mount`
- Los archivos `.edb` existen en `D:\BD-N121\` con tamaño normal

**Preguntas de diagnóstico:**
1. ¿Qué significa que una base de datos Exchange esté desmontada? ¿Los datos están en riesgo?
2. ¿Por qué Exchange a veces no monta automáticamente una base de datos tras un reinicio?
3. ¿Qué verificarías antes de montar la BD para asegurarte de que no hay riesgo de pérdida de datos?

**Diagnóstico y resolución:**
```powershell
# Verificar estado:
Get-MailboxDatabase -Status | Select Name, Mounted, DatabaseSize

# Comprobar integridad antes de montar:
eseutil /mh "D:\BD-N121\BD-N121.edb"
# Resultado esperado: "State: Clean Shutdown"
# Si dice "Dirty Shutdown": hay logs sin aplicar — NO montar todavía

# Si está en "Clean Shutdown" — montar directamente:
Mount-Database -Identity "BD-N121"

# Si está en "Dirty Shutdown" — reproducir logs primero:
eseutil /r E00 /l "D:\BD-N121\Logs" /d "D:\BD-N121"
Mount-Database -Identity "BD-N121"

# Verificar que los buzones responden:
Get-MailboxDatabase "BD-N121" -Status | Select Name, Mounted
Test-MapiConnectivity -Database "BD-N121"
```

**Concepto clave:** BD desmontada ≠ datos perdidos. El riesgo está en montar una BD en estado "Dirty Shutdown" sin reproducir los logs de transacciones primero. Si se omite ese paso, Exchange puede truncar transacciones no confirmadas y perder correos de los últimos minutos antes del apagado. `eseutil /mh` es el primer comando siempre — te dice el estado exacto de la BD antes de actuar.

---

### EX-03 — Outlook no configura la cuenta automáticamente (Autodiscover falla)
**Capa principal: 2 — Internet · 4 — Aplicación (DNS / Autodiscover)**

**Escenario:**
Se despliegan 15 equipos nuevos en el N111 con Outlook 2019 desde MDT. Al configurar la cuenta de correo, Outlook pide al usuario que introduzca manualmente el servidor. Los equipos ya existentes configuran la cuenta automáticamente en segundos.

**Síntomas:**
- Outlook equipo nuevo: "No se puede iniciar sesión. Compruebe que está conectado a la red"
- Outlook equipo antiguo: configura la cuenta en 10 segundos sin intervención
- `Resolve-DnsName -Name "_autodiscover._tcp.btpcce.esp" -Type SRV` → sin resultado
- El registro SRV de Autodiscover no existe en el DNS del dominio

**Preguntas de diagnóstico:**
1. ¿Qué es Autodiscover y cómo sabe Outlook a qué servidor conectarse sin que el usuario escriba nada?
2. ¿Qué tipo de registro DNS necesita Autodiscover? ¿Cuál sería el valor correcto apuntando a `N111EX01`?
3. ¿Por qué los equipos antiguos funcionan si el registro DNS no existe?

**Diagnóstico y resolución:**
```powershell
# Verificar estado del DNS:
Resolve-DnsName -Name "_autodiscover._tcp.btpcce.esp" -Type SRV

# Crear el registro SRV que falta (desde el DC con rol DNS):
Add-DnsServerResourceRecord -ZoneName "btpcce.esp" -Srv `
    -Name "_autodiscover._tcp" `
    -DomainName "N111EX01.btpcce.esp" `
    -Priority 0 -Weight 0 -Port 443

# Verificar tras crear:
Resolve-DnsName -Name "_autodiscover._tcp.btpcce.esp" -Type SRV

# Probar desde Outlook (equipo cliente):
# Ctrl + clic derecho sobre icono Outlook en bandeja del sistema
# → "Probar configuración automática del correo electrónico"
```

**Concepto clave:** Autodiscover es el mecanismo por el que Outlook descubre automáticamente servidor de Exchange, URL de OWA, configuración de ActiveSync y política de buzón. Lo hace buscando un registro SRV `_autodiscover._tcp.dominio` en DNS. Sin ese registro, cada equipo nuevo necesita configuración manual. Los equipos antiguos tienen la configuración cacheada en el perfil de Outlook y no necesitan Autodiscover en cada arranque.

---

## MÓDULO: SHAREPOINT SERVER 2019

---

### SP-01 — SharePoint devuelve error 503 — servicio no disponible
**Capa principal: 3 — Transporte · 4 — Aplicación**

**Escenario:**
A las 09:00, todos los usuarios que acceden a `http://N111SHP01/` reciben HTTP 503 "Servicio no disponible". El servidor está encendido, responde a ping e IIS está en estado Running.

**Síntomas:**
- HTTP 503 en todos los accesos a SharePoint
- IIS: estado "Running"
- El Pool de aplicaciones `SharePoint - 80` aparece como "Stopped" en IIS Manager
- Log de aplicación: `Application pool 'SharePoint - 80' was disabled due to a series of failures`
- Los fallos del pool empezaron 13 minutos antes de las primeras quejas de usuarios

**Preguntas de diagnóstico:**
1. ¿Qué relación hay entre el Pool de aplicaciones de IIS y SharePoint? ¿Por qué la caída del pool tumba todo SharePoint?
2. HTTP 503 indica que el servidor web recibe la petición pero no puede procesarla. ¿Cómo lo diferencias de un HTTP 500 (error interno)?
3. ¿Qué verificarías antes de reiniciar el Pool para entender por qué se cayó y evitar que vuelva a caerse?

**Diagnóstico y resolución:**
```powershell
# Ver por qué se cayó el pool:
Get-EventLog -LogName Application -Source "SharePoint*" -Newest 20
Get-EventLog -LogName Application -Source "ASP.NET*" -Newest 20

# Verificar que SQL Server es accesible (causa más frecuente):
Test-NetConnection -ComputerName N111-SQL -Port 1433

# Reiniciar el Pool:
Restart-WebAppPool "SharePoint - 80"
# O: IIS Manager → Grupos de aplicaciones → SharePoint - 80 → Iniciar

# Si el pool vuelve a caerse:
# Verificar que la contraseña de la cuenta de servicio del pool no ha expirado
# IIS Manager → Grupos de aplicaciones → SharePoint - 80 → Configuración avanzada → Identidad
```

**Concepto clave:** SharePoint corre dentro de IIS bajo un pool de aplicaciones. Si el pool cae (por error de la aplicación, por pérdida de conectividad con SQL Server o por credenciales de servicio expiradas), IIS sigue corriendo pero no tiene nada que servir → 503. La causa más habitual en producción: SQL Server no está disponible o la contraseña de la cuenta de servicio del pool expiró.

---

### SP-02 — Un usuario recibe "Acceso denegado" en un sitio de SharePoint
**Capa principal: 4 — Aplicación (permisos SharePoint / AD)**

**Escenario:**
La Sgto. Ruiz necesita acceder a `http://N111SHP01/sites/operaciones`. Recibe "Acceso denegado". Su jefe confirma que "la añadió al sitio" hace dos semanas. Ruiz cambió de cuenta de dominio recientemente (renombrado de usuario en AD, la cuenta nueva es `btpcce\ruiz` y la antigua era `btpcce\ruiz_old`).

**Síntomas:**
- Error 403 al acceder al sitio
- En permisos del sitio: aparece `btpcce\ruiz_old` como miembro, no `btpcce\ruiz`
- Otros miembros del mismo grupo de SharePoint acceden sin problema

**Preguntas de diagnóstico:**
1. ¿Por qué SharePoint sigue mostrando la cuenta antigua si en AD el usuario ya tiene cuenta nueva?
2. ¿Por qué se recomienda añadir grupos de AD a SharePoint en lugar de usuarios individuales? ¿Qué problema concreto habría evitado esto?
3. ¿Cómo migrarías los permisos de la cuenta antigua a la nueva sin perder el historial de documentos creados por Ruiz?

**Diagnóstico y resolución:**
```powershell
# Desde SharePoint Management Shell:
$web = Get-SPWeb "http://N111SHP01/sites/operaciones"

# Migrar cuenta antigua a nueva:
Move-SPUser -Identity "btpcce\ruiz_old" -NewAlias "btpcce\ruiz" -IgnoreSID -Web $web

# Verificar resultado:
Get-SPUser -Web $web | Where-Object {$_.DisplayName -like "*Ruiz*"}

# Lección para el futuro — gestionar grupos AD, no usuarios individuales:
# Administración del sitio → Grupos de SharePoint → Miembros
# → Eliminar usuarios individuales → Añadir grupo "GG_Operaciones" de AD
# Si alguien cambia de cuenta, se actualiza en AD — SharePoint no necesita tocarse
```

**Concepto clave:** SharePoint vincula permisos a la identidad de AD. Un renombrado de cuenta en AD crea técnicamente una cuenta distinta desde la perspectiva del SID. SharePoint no detecta este cambio automáticamente — sigue apuntando al nombre anterior. La práctica correcta es siempre añadir grupos de AD, no usuarios individuales: cuando un usuario cambia de cuenta, basta con actualizar el grupo en AD sin tocar SharePoint.

---

### SP-03 — El navegador avisa de certificado no confiable en SharePoint
**Capa principal: 4 — Aplicación (PKI / TLS)**

**Escenario:**
SharePoint en `N111SHP01` se ha configurado con HTTPS usando un certificado emitido por `N111CASUB01` (CA intermediaria del dominio). Los usuarios reciben `NET::ERR_CERT_AUTHORITY_INVALID` en el navegador. El certificado es válido, no ha expirado y el CN es correcto.

**Síntomas:**
- Advertencia de certificado en todos los navegadores
- El certificado muestra la cadena: `N111CAROOT → N111CASUB01 → N111SHP01`
- `N111CAROOT` no está en el almacén de certificados de confianza de los equipos cliente
- `N111CASUB01` sí está instalada en algunos equipos de forma manual

**Preguntas de diagnóstico:**
1. ¿Por qué no basta con tener la CA intermediaria instalada? ¿Qué necesita el cliente para considerar la cadena como confiable?
2. ¿Qué mecanismo usarías para distribuir `N111CAROOT` a todos los equipos del dominio de forma automática?
3. ¿Cómo verificarías desde un equipo cliente que la cadena de confianza está completa tras aplicar la GPO?

**Diagnóstico y resolución:**
```powershell
# Distribuir N111CAROOT por GPO:
# Computer Configuration → Policies → Windows Settings → Security Settings
# → Public Key Policies → Trusted Root Certification Authorities → Import → N111CAROOT.cer

# Forzar aplicación en los clientes:
gpupdate /force

# Verificar en cliente que la CA raíz está instalada:
certutil -store root | findstr /i "N111"

# Verificar que la URL de SharePoint usa HTTPS correctamente:
# Central Admin → Manage Web Applications → N111SHP01
# → Alternate Access Mappings → HTTPS debe estar como URL pública
```

**Concepto clave:** La cadena de confianza PKI: el cliente verifica el certificado del servidor → busca el emisor (N111CASUB01) → busca el emisor de la intermediaria (N111CAROOT) → comprueba si la raíz está en su almacén de confianza. Si la raíz no está, la cadena se rompe independientemente de que la intermediaria sí esté instalada. La distribución por GPO es el único método escalable en producción.

---

## MÓDULO: SERVIDOR DE DESPLIEGUE (WDS + MDT)

---

### SD-01 — El equipo no arranca por PXE — no recibe imagen del servidor
**Capa principal: 2 — Internet · 3 — Transporte (DHCP / TFTP)**

**Escenario:**
Se intenta desplegar un equipo nuevo en el N121 mediante PXE. Al arrancar, aparece: `PXE-E51: No DHCP or proxyDHCP offers were received`. El equipo está en VLAN de clientes del N121 (VLAN 1209 / `102.197.133.0/25`). El servidor WDS está en el N111 (VLAN 1110 / `102.194.249.0/25`).

**Síntomas:**
- El equipo no recibe respuesta DHCP al arrancar por PXE
- Otros equipos de la misma VLAN reciben IP normalmente
- El servidor WDS y el servidor DHCP están en el N111, en VLAN distinta al cliente

**Preguntas de diagnóstico:**
1. ¿Por qué el proceso PXE usa DHCP? ¿Qué información adicional necesita el cliente PXE que un cliente DHCP normal no necesita?
2. El cliente PXE está en una VLAN diferente al servidor DHCP. Los broadcasts DHCP no cruzan VLANs por defecto. ¿Qué mecanismo de red lo resuelve y quién lo configura?
3. ¿Qué opciones DHCP son necesarias para PXE y qué valor debe tener cada una?

**Diagnóstico y resolución:**
```powershell
# Verificar opciones DHCP en el servidor (N111PDC):
Get-DhcpServerv4OptionValue -ScopeId 102.197.133.0 | Where-Object {$_.OptionId -in @(66,67)}

# Si no existen:
Set-DhcpServerv4OptionValue -ScopeId 102.197.133.0 -OptionId 66 -Value "102.194.249.X"   # IP del WDS
Set-DhcpServerv4OptionValue -ScopeId 102.197.133.0 -OptionId 67 -Value "boot\x64\wdsnbp.com"

# Si WDS y DHCP están en el mismo servidor:
# WDS Manager → Propiedades → DHCP → marcar "Do not listen on DHCP ports"
# y marcar "Configure DHCP option 60 to indicate that this server is a PXE server"

# Verificar que el firewall/ASA permite el tráfico WDS:
Test-NetConnection -ComputerName 102.194.249.X -Port 4011   # UDP 4011 WDS
```
> **Nota:** Si la VLAN 1209 no tiene un IP Helper / DHCP Relay apuntando al servidor DHCP del N111, ninguna de las opciones anteriores funcionará. En ese caso escalar al administrador de redes para configurar el relay en la VLAN 1209.

**Concepto clave:** PXE usa DHCP para obtener IP, pero necesita dos datos extra que DHCP normal no da: opción 66 (IP del servidor WDS) y opción 67 (nombre del archivo de arranque). Sin ellos, el cliente recibe IP pero no sabe de dónde descargar el entorno de preinstalación. En entornos multi-VLAN, los broadcasts DHCP no cruzan VLANs sin IP Helper — es la causa más frecuente de PXE fallido en los despliegues.

---

### SD-02 — La task sequence falla al intentar unir el equipo al dominio
**Capa principal: 4 — Aplicación (MDT / AD)**

**Escenario:**
El despliegue por MDT arranca correctamente, instala Windows y las aplicaciones, pero falla en el paso "Recover from Domain". El equipo termina instalado pero en grupo de trabajo, no en `btpcce.esp`.

**Síntomas:**
- `BDD.log` en `C:\Windows\Temp\DeploymentLogs\`: `Failed to join domain btpcce.esp`
- Error: `0x00000005 — Access Denied`
- Las credenciales en `CustomSettings.ini` son correctas según el administrador
- La cuenta de unión al dominio ha realizado más de 10 uniones previas

**Preguntas de diagnóstico:**
1. ¿Por qué existe un límite de uniones al dominio por cuenta de usuario? ¿Dónde se configura?
2. ¿Qué permisos mínimos debe tener la cuenta que MDT usa para unir equipos al dominio?
3. Las credenciales de unión están almacenadas en `Bootstrap.ini` en texto claro. ¿Qué implica eso en términos de seguridad?

**Diagnóstico y resolución:**
```powershell
# Verificar cuántas uniones ha realizado la cuenta de servicio MDT:
# ADUC → buscar la cuenta → Propiedades → Attribute Editor → ms-DS-CreatorSID en objetos de equipo

# Solución recomendada — delegar permisos específicos en la OU de destino:
# ADUC → OU "Equipos_Desplegados" → clic derecho → Delegar control
# → Seleccionar la cuenta MDT → Solo permiso "Unir equipos al dominio"
# Esto elimina el límite de 10 y evita usar una cuenta de administrador de dominio

# Si se necesita solución inmediata — ampliar el límite vía atributo del dominio:
# ADSI Edit → Default Naming Context → dominio → Propiedades
# → ms-DS-MachineAccountQuota → cambiar a valor mayor o 0 (ilimitado)

# Verificar Bootstrap.ini — riesgo de seguridad:
# Las credenciales están en texto claro → usar cuenta con mínimos privilegios, nunca admins
```

**Concepto clave:** Por defecto, cualquier usuario de AD puede unir hasta 10 equipos al dominio (atributo `ms-DS-MachineAccountQuota`). La cuenta MDT no es excepción. Al llegar al límite, el error es "Acceso denegado" — idéntico al de contraseña incorrecta. La solución correcta no es usar una cuenta de administrador de dominio: con las credenciales en texto claro en `Bootstrap.ini`, cualquiera con acceso al Deployment Share puede verlas. La cuenta MDT debe tener solo el permiso de unión delegado en la OU de destino.

---

### SD-03 — La imagen desplegada tiene el mismo SID en todos los equipos
**Capa principal: 4 — Aplicación (Sysprep / identidad de equipo)**

**Escenario:**
Tras desplegar 30 equipos desde una imagen WIM personalizada, el administrador de AD detecta comportamientos extraños: equipos que se solapan en el dominio, GPOs que se aplican a equipos que no deberían, y eventos de inicio de sesión duplicados en los logs. La imagen fue capturada sin ejecutar Sysprep antes.

**Síntomas:**
- Varios equipos comparten el mismo SID de equipo en AD
- `whoami /user` desde equipos distintos muestra el mismo SID de equipo
- Los equipos se unieron al dominio sin error, pero en AD aparecen como "duplicados"
- Las GPOs de equipo se comportan de forma impredecible

**Preguntas de diagnóstico:**
1. ¿Qué es el SID de equipo y por qué debe ser único en el dominio?
2. Si una imagen Windows se captura sin Sysprep, ¿qué ocurre cuando se despliega en 30 equipos?
3. ¿Cuál es el procedimiento correcto para preparar el equipo master antes de capturar la imagen?

**Diagnóstico y resolución:**
```powershell
# Verificar SIDs duplicados en AD:
Get-ADComputer -Filter * -Properties SID | Group-Object SID | Where-Object {$_.Count -gt 1}

# Solución en los equipos afectados:
# 1. Retirar del dominio (volver a workgroup)
# 2. Ejecutar Sysprep:
sysprep /generalize /oobe /shutdown
# 3. Arrancar por PXE → capturar imagen limpia
# 4. Redesplegar desde la imagen correcta → el sistema generará un SID único en cada equipo

# Procedimiento correcto para construir la imagen master:
# 1. Instalar Windows en equipo limpio (sin unir al dominio)
# 2. Instalar drivers, aplicaciones base y configuración inicial
# 3. sysprep /generalize /oobe /shutdown   ← OBLIGATORIO antes de capturar
# 4. Arrancar por PXE → "Capture Image" en WDS → guardar nueva WIM
# 5. Cada despliegue desde esa WIM generará automáticamente un SID único
```

**Concepto clave:** El SID de equipo es el identificador único que AD usa para reconocer cada máquina. Sin Sysprep, todos los clones tienen el mismo SID — AD los trata como el mismo equipo, las GPOs se comportan de forma impredecible y los logs de seguridad mezclan eventos de máquinas distintas. Sysprep con `/generalize` elimina el SID, los identificadores únicos de hardware y la activación de Windows, permitiendo que cada despliegue arranque con una identidad propia. Es el paso que más frecuentemente se omite y el que más problemas causa.

---

---

## EJERCICIOS DE ADMINISTRACIÓN

---

## MÓDULO: VIRTUALIZACIÓN — Administración (ESXi + vCenter)

---

### ADM-VJ-01 — Configurar vSwitches y port groups en un ESXi recién desplegado
**Capa principal: 1 — Acceso a red**

**Escenario:**
Se despliega un tercer hipervisor (`ESXSRV03`) en el N121. El host tiene 4 NICs físicas. Hay que configurar la red virtual para que las VMs del nodo tengan acceso a las VLANs correctas, separando el tráfico de gestión del de producción y del de almacenamiento.

**Concepto previo — cómo se interconecta el vSwitch con la red física:**

```
  NIC FÍSICA (vmnic)          vSwitch (lógico)         Port Groups (VLANs)
  ─────────────────           ────────────────         ──────────────────
  vmnic0 ──────────────────→  vSwitch0 (gestión)  ──→  PG-Gestion    (VLAN 1200)
  vmnic1 ──────────────────↗                       ──→  VMkernel vmk0 (IP del host)
  
  vmnic2 ──────────────────→  vSwitch1 (producción)──→  PG-Servers    (VLAN 1210)
  vmnic3 ──────────────────↗                       ──→  PG-Clients    (VLAN 1209)
                                                    ──→  PG-vMotion    (VLAN 3902)
```

> **La clave:** el vSwitch es el puente entre las NICs físicas (que conectan con el switch físico del administrador de redes) y las NICs virtuales de las VMs. El switch físico debe tener esas VLANs en modo trunk en el puerto donde conecta el servidor. Eso es responsabilidad del administrador de redes — tu responsabilidad empieza en el vSwitch.

**Tarea:**
Crear dos vSwitches: uno para gestión del host y otro para tráfico de producción de las VMs, con los port groups y VLAN IDs correctos para el N121.

**Procedimiento (desde vCenter → Host → Configure → Virtual Switches):**
```bash
# Desde SSH al ESXi o desde vCenter — verificar NICs disponibles:
esxcli network nic list

# Crear vSwitch0 para gestión (uplinks: vmnic0 + vmnic1 para redundancia):
esxcli network vswitch standard add --vswitch-name vSwitch0
esxcli network vswitch standard uplink add --vswitch-name vSwitch0 --uplink-name vmnic0
esxcli network vswitch standard uplink add --vswitch-name vSwitch0 --uplink-name vmnic1

# Crear port group de gestión con VLAN 1200 (gestión del nodo N121):
esxcli network vswitch standard portgroup add --vswitch-name vSwitch0 --portgroup-name "PG-Gestion-N121"
esxcli network vswitch standard portgroup set --portgroup-name "PG-Gestion-N121" --vlan-id 1200

# Crear vSwitch1 para producción (uplinks: vmnic2 + vmnic3):
esxcli network vswitch standard add --vswitch-name vSwitch1
esxcli network vswitch standard uplink add --vswitch-name vSwitch1 --uplink-name vmnic2
esxcli network vswitch standard uplink add --vswitch-name vSwitch1 --uplink-name vmnic3

# Crear port groups de producción con sus VLANs:
esxcli network vswitch standard portgroup add --vswitch-name vSwitch1 --portgroup-name "PG-Servers-N121"
esxcli network vswitch standard portgroup set --portgroup-name "PG-Servers-N121" --vlan-id 1210

esxcli network vswitch standard portgroup add --vswitch-name vSwitch1 --portgroup-name "PG-Clients-N121"
esxcli network vswitch standard portgroup set --portgroup-name "PG-Clients-N121" --vlan-id 1209

esxcli network vswitch standard portgroup add --vswitch-name vSwitch1 --portgroup-name "PG-vMotion-N121"
esxcli network vswitch standard portgroup set --portgroup-name "PG-vMotion-N121" --vlan-id 3902

# Verificar configuración:
esxcli network vswitch standard list
esxcli network vswitch standard portgroup list
```

**Preguntas:**
1. ¿Por qué se usan dos NICs físicas por vSwitch (vmnic0+vmnic1 / vmnic2+vmnic3)? ¿Qué ocurre si una falla?
2. Si el port group tiene VLAN ID `1210` pero el trunk del switch físico no incluye esa VLAN, ¿qué pasa con el tráfico de las VMs conectadas a ese port group?
3. ¿Qué diferencia hay entre un vSwitch estándar y un vSwitch distribuido (dvSwitch)? ¿Cuál usamos en el PAIB y por qué?

**Concepto clave:** El VLAN ID del port group en el vSwitch es la mitad del trabajo. La otra mitad es que el puerto del switch físico al que conecta el servidor esté configurado como trunk con esas VLANs permitidas — eso lo hace el administrador de redes. Si hay desacuerdo entre el VLAN ID del port group y lo que permite el switch físico, el tráfico se descarta silenciosamente en capa 1. El síntoma: VMs sin red aunque el port group parezca correcto.

---

### ADM-VJ-02 — Configurar los VMkernel adapters (gestión, vMotion, almacenamiento)
**Capa principal: 1 — Acceso a red · 2 — Internet**

**Escenario:**
El ESXSRV03 del N121 tiene los vSwitches configurados (ADM-VJ-01) pero no tiene asignadas las IPs de los interfaces VMkernel. Sin ellas el hipervisor no tiene IP de gestión, vMotion no funciona y el almacenamiento compartido no es accesible.

**Concepto previo — qué es un VMkernel adapter:**

```
  vSwitch0 (gestión)
  ├── PG-Gestion-N121  (VLAN 1200)
  │   └── vmk0  ← IP de gestión del hipervisor (la que ve vCenter)
  │               IP: 102.197.132.X  GW: 102.197.132.1
  │
  vSwitch1 (producción)
  ├── PG-vMotion-N121  (VLAN 3902)
  │   └── vmk1  ← IP de vMotion (solo tráfico de migración entre hosts)
  │               IP: 172.21.0.X
  ├── PG-Storage-N121  (VLAN dedicada almacenamiento)
  │   └── vmk2  ← IP de iSCSI/NFS (acceso a almacenamiento compartido)
  └── PG-Servers/Clients → NICs virtuales de las VMs (sin VMkernel)
```

> Los VMkernel adapters son interfaces IP del propio hipervisor (no de las VMs). Cada servicio del host (gestión, vMotion, almacenamiento) usa su propio VMkernel en una VLAN dedicada.

**Tarea:**
Crear y configurar los tres VMkernel adapters con sus IPs y habilitar el servicio correcto en cada uno.

**Procedimiento:**
```bash
# VMkernel 0 — gestión del host (vmk0, VLAN 1200):
esxcli network ip interface add --interface-name vmk0 --portgroup-name "PG-Gestion-N121"
esxcli network ip interface ipv4 set --interface-name vmk0 --type static \
    --ipv4 102.197.132.4 --netmask 255.255.255.224
# Habilitar servicio de gestión:
esxcli network ip interface tag add --interface-name vmk0 --tagname Management

# VMkernel 1 — vMotion (vmk1, VLAN 3902):
esxcli network ip interface add --interface-name vmk1 --portgroup-name "PG-vMotion-N121"
esxcli network ip interface ipv4 set --interface-name vmk1 --type static \
    --ipv4 172.21.0.52 --netmask 255.255.255.240
# Habilitar servicio de vMotion:
esxcli network ip interface tag add --interface-name vmk1 --tagname vMotion

# VMkernel 2 — almacenamiento iSCSI/NFS (vmk2):
esxcli network ip interface add --interface-name vmk2 --portgroup-name "PG-Storage-N121"
esxcli network ip interface ipv4 set --interface-name vmk2 --type static \
    --ipv4 172.21.0.35 --netmask 255.255.255.240

# Verificar todos los VMkernel configurados:
esxcli network ip interface list
esxcli network ip interface ipv4 get

# Añadir gateway de gestión:
esxcli network ip route ipv4 add --network default --gateway 102.197.132.1
```

**Preguntas:**
1. Si `vmk0` (gestión) y `vmk1` (vMotion) están en la misma red, ¿qué problema puede surgir?
2. ¿Por qué el tráfico de vMotion y el de almacenamiento necesitan redes dedicadas y no pueden compartir la NIC de producción?
3. Después de configurar `vmk0` con la IP de gestión, ¿desde dónde podrías verificar que vCenter ya ve el nuevo host?

**Concepto clave:** Los VMkernel adapters son los "ojos y oídos" del hipervisor en la red. Sin `vmk0` con el servicio Management, vCenter no puede ver ni gestionar el host. Sin `vmk1` con vMotion habilitado, las migraciones en caliente fallan aunque haya red entre los hosts. Sin `vmk2` dedicado para almacenamiento, el rendimiento de I/O de todas las VMs compite con el tráfico de producción. Cada servicio en su red: es el principio de separación de planos de tráfico aplicado a la virtualización.

---

### ADM-VJ-03 — Añadir y configurar un datastore al hipervisor
**Capa principal: 1 — Acceso a red**

**Escenario:**
El ESXSRV03 del N121 solo tiene el datastore local (disco interno del servidor). Para poder hacer vMotion y tener alta disponibilidad, las VMs deben estar en almacenamiento compartido accesible por todos los hipervisores del nodo. El N121 usa un NAS con NFS en `172.21.0.36`.

**Concepto previo — tipos de almacenamiento en ESXi:**

```
  TIPOS DE DATASTORE
  ──────────────────
  VMFS local    → disco físico del servidor
                  Solo accesible por ese host. Sin vMotion. Sin HA.
                  Útil para: VMs de test, ISOs.

  VMFS iSCSI    → disco en cabina SAN por red IP (TCP)
                  Compartido entre hosts. vMotion ✓. HA ✓.
                  Requiere VMkernel con tag iSCSI.

  NFS           → carpeta exportada por un NAS (protocolo NFS)
                  Compartido entre hosts. vMotion ✓. HA ✓.
                  Más simple de configurar que iSCSI.
                  Requiere VMkernel accesible al NAS.
```

> **Lo que tienes que tener en cuenta del almacenamiento:**
> - Almacenamiento local = sin movilidad de VMs. Nunca usar para producción en multi-host.
> - Almacenamiento compartido (NFS/iSCSI) = las VMs pueden vivir en cualquier host. Requisito para vMotion y HA.
> - El rendimiento del almacenamiento determina el rendimiento de TODAS las VMs. Una red de almacenamiento lenta o saturada afecta a todos los servicios.

**Tarea:**
Añadir el datastore NFS compartido al ESXSRV03 y verificar que es accesible junto con los otros hipervisores del nodo.

**Procedimiento:**
```bash
# Verificar conectividad con el NAS antes de añadir el datastore:
vmkping -I vmk2 172.21.0.36    # ping desde el VMkernel de almacenamiento

# Añadir datastore NFS desde vCenter:
# Host → Configure → Storage → Datastores → Add Datastore → NFS
# → NFS server: 172.21.0.36
# → NFS share: /vol/n121_vms
# → Datastore name: DS-N121-NFS

# O desde ESXi CLI:
esxcli storage nfs add \
    --host 172.21.0.36 \
    --share /vol/n121_vms \
    --volume-name DS-N121-NFS

# Verificar que el datastore es visible:
esxcli storage filesystem list | grep DS-N121

# Verificar que el mismo datastore es visible desde ESXSRV01 y ESXSRV02:
# vCenter → Datastores → DS-N121-NFS → Hosts → debe aparecer en los 3 hosts

# Crear una VM de prueba en el datastore compartido y verificar que se puede hacer vMotion:
# vCenter → VM de prueba → Migrate → Change compute resource only → destino: ESXSRV01
# Si tiene acceso al mismo datastore → vMotion funciona
```

**Preguntas:**
1. ¿Por qué el `vmkping` al NAS se lanza desde `vmk2` y no desde `vmk0`? ¿Qué ocurre si la conectividad de almacenamiento pasa por el VMkernel de gestión?
2. Si dos hipervisores ven el mismo datastore NFS pero no el mismo datastore local, ¿entre cuáles se puede hacer vMotion?
3. El NAS tiene un único enlace de red de 1 Gbps. Hay 3 hipervisores con 10 VMs cada uno. ¿Qué problema puede surgir y cómo se mitigaría?

**Concepto clave:** vMotion requiere que el host origen y el destino tengan acceso al mismo datastore — si la VM está en almacenamiento local del host A, no se puede migrar al host B porque B no ve ese disco. El almacenamiento compartido (NFS o iSCSI) es el requisito fundamental para tener flexibilidad en la infraestructura virtual. En el PAIB, cada nodo tiene su propio NAS/cabina para que el almacenamiento no dependa de la conectividad entre nodos.

---

## MÓDULO: ACTIVE DIRECTORY — Administración

---

### ADM-AD-01 — Alta de usuario nuevo en el dominio con grupos y OU correctos
**Capa principal: 4 — Aplicación (LDAP / AD)**

**Escenario:**
Se incorpora al N121 el nuevo Sgto. Martínez. Necesita acceso al dominio `btpcce.esp`, carpetas compartidas del nodo y correo corporativo. Hay que crear su cuenta en la OU correcta y asignarle los grupos adecuados antes de que llegue.

**Tarea:**
Crear la cuenta de usuario con los parámetros correctos, asignarla a la OU del nodo N121, añadirla a los grupos necesarios y verificar que puede autenticarse.

**Procedimiento:**
```powershell
# Crear el usuario en la OU correcta del nodo N121:
New-ADUser `
    -Name "Martínez García, Pedro" `
    -GivenName "Pedro" `
    -Surname "Martínez García" `
    -SamAccountName "p.martinez" `
    -UserPrincipalName "p.martinez@btpcce.esp" `
    -Path "OU=Usuarios,OU=N121,DC=btpcce,DC=esp" `
    -AccountPassword (ConvertTo-SecureString "Btpcce2024!" -AsPlainText -Force) `
    -ChangePasswordAtLogon $true `
    -Enabled $true

# Verificar que se creó correctamente:
Get-ADUser -Identity "p.martinez" -Properties *

# Añadir a los grupos del nodo:
Add-ADGroupMember -Identity "GG_Usuarios_N121" -Members "p.martinez"
Add-ADGroupMember -Identity "GG_FS_N121_Lectura" -Members "p.martinez"

# Verificar grupos asignados:
Get-ADUser "p.martinez" -Properties MemberOf | Select -ExpandProperty MemberOf
```

**Preguntas:**
1. ¿Por qué es importante crear el usuario en la OU correcta del nodo y no en el contenedor `CN=Users` por defecto?
2. Si la cuenta se crea habilitada pero sin `ChangePasswordAtLogon`, ¿qué riesgo existe?
3. ¿Cómo crearías 15 usuarios de golpe a partir de una lista en CSV?

```powershell
# Alta masiva desde CSV (columnas: Nombre, Apellidos, SamAccountName):
Import-Csv "C:\nuevos_usuarios.csv" | ForEach-Object {
    New-ADUser `
        -Name "$($_.Apellidos), $($_.Nombre)" `
        -GivenName $_.Nombre `
        -Surname $_.Apellidos `
        -SamAccountName $_.SamAccountName `
        -UserPrincipalName "$($_.SamAccountName)@btpcce.esp" `
        -Path "OU=Usuarios,OU=N121,DC=btpcce,DC=esp" `
        -AccountPassword (ConvertTo-SecureString "Btpcce2024!" -AsPlainText -Force) `
        -ChangePasswordAtLogon $true `
        -Enabled $true
}
```

**Concepto clave:** La OU determina qué GPOs se aplican al usuario. Un usuario en la OU equivocada puede tener políticas de otro nodo — o ninguna. En un entorno multi-nodo como el PAIB, cada nodo tiene su propio subárbol de OUs (`OU=N121`, `OU=N131`...) exactamente para que las GPOs y los permisos sean independientes por nodo.

---

### ADM-AD-02 — Crear grupo de seguridad y delegar control sobre una OU
**Capa principal: 4 — Aplicación (LDAP / AD)**

**Escenario:**
El N131 tiene su propio administrador de sistemas que necesita gestionar usuarios y equipos de su nodo sin tener privilegios de administrador de dominio. Hay que crear un grupo de seguridad y delegarle el control sobre la OU del N131.

**Tarea:**
Crear el grupo `GG_Admins_N131`, añadir al administrador del nodo, y delegar sobre `OU=N131` los permisos de creación/modificación de usuarios y equipos únicamente.

**Procedimiento:**
```powershell
# Crear el grupo de administradores del nodo:
New-ADGroup `
    -Name "GG_Admins_N131" `
    -GroupScope Global `
    -GroupCategory Security `
    -Path "OU=Grupos,OU=N131,DC=btpcce,DC=esp" `
    -Description "Administradores locales del nodo N131"

# Añadir el administrador del nodo al grupo:
Add-ADGroupMember -Identity "GG_Admins_N131" -Members "admin.n131"

# La delegación de control se hace desde ADUC (GUI):
# ADUC → clic derecho sobre OU=N131 → "Delegar control..."
# → Añadir grupo "GG_Admins_N131"
# → Tareas a delegar:
#     ✓ Crear, eliminar y administrar cuentas de usuario
#     ✓ Restablecer contraseñas de usuario
#     ✓ Leer toda la información del usuario
#     ✓ Crear, eliminar y administrar grupos
#     ✓ Unir equipo a un dominio

# Verificar permisos delegados:
# ADUC → OU=N131 → Propiedades → Seguridad → Opciones avanzadas
# → Verificar que GG_Admins_N131 aparece con los permisos delegados
```

**Preguntas:**
1. ¿Qué diferencia hay entre un grupo de seguridad de ámbito **Global**, **Universal** y **Local de dominio**? ¿Por qué usamos Global aquí?
2. Si el administrador del N131 tiene permisos delegados sobre `OU=N131`, ¿puede ver o modificar usuarios de `OU=N121`?
3. ¿Qué riesgo supone darle permisos de administrador de dominio en lugar de delegación específica?

**Concepto clave:** La delegación de control en AD permite aplicar el principio de mínimos privilegios: el administrador del N131 gestiona su nodo sin poder tocar nada del resto del dominio. Esto es crítico en una infraestructura multi-nodo donde cada compañía tiene su propio responsable. Un administrador de dominio puede hacerlo todo en cualquier nodo — es demasiado poder para la operativa diaria.

---

## MÓDULO: DNS — Administración

---

### ADM-DNS-01 — Registrar un nuevo servidor en DNS (registro A y PTR)
**Capa principal: 2 — Internet · 4 — Aplicación (DNS)**

**Escenario:**
Se despliega un nuevo servidor en el N121: `N121ZBX` (Zabbix) con IP `102.197.133.150`. Hay que registrarlo en DNS para que sea alcanzable por nombre desde cualquier nodo del dominio, y crear el registro inverso para que los logs y herramientas de monitorización muestren el nombre en lugar de la IP.

**Tarea:**
Crear el registro A directo y el registro PTR inverso en la zona DNS integrada en AD.

**Procedimiento:**
```powershell
# Crear registro A (directo: nombre → IP):
Add-DnsServerResourceRecordA `
    -ZoneName "btpcce.esp" `
    -Name "N121ZBX" `
    -IPv4Address "102.197.133.150" `
    -CreatePtr    # crea el PTR automáticamente si la zona inversa existe

# Verificar el registro A:
Resolve-DnsName -Name "N121ZBX.btpcce.esp" -Type A

# Si la zona inversa no existe todavía, crearla primero:
Add-DnsServerPrimaryZone `
    -NetworkID "102.197.133.0/25" `
    -ReplicationScope "Domain"

# Crear el registro PTR manualmente si -CreatePtr no funcionó:
Add-DnsServerResourceRecordPtr `
    -ZoneName "133.197.102.in-addr.arpa" `
    -Name "150" `
    -PtrDomainName "N121ZBX.btpcce.esp"

# Verificar resolución inversa:
Resolve-DnsName -Name "102.197.133.150" -Type PTR
```

**Preguntas:**
1. ¿Qué diferencia hay entre un registro A y un registro PTR? ¿Cuándo es imprescindible el PTR?
2. El DNS de `btpcce.esp` está integrado en AD. ¿Qué ventaja tiene eso frente a un DNS estándar?
3. Si el registro A se crea en el DC del N111 pero el servidor Zabbix está en el N121, ¿cuánto tarda ese registro en llegar al DC del N121?

**Concepto clave:** El registro A resuelve nombre → IP (lo que usan los usuarios y aplicaciones). El registro PTR resuelve IP → nombre (lo que usan los sistemas de log, seguridad y monitorización). Sin PTR, Zabbix muestra IPs en lugar de nombres, los logs de Exchange muestran IPs en las cabeceras y las herramientas de auditoría no pueden identificar hosts por nombre. En producción, siempre se crean los dos.

---

### ADM-DNS-02 — Crear un alias CNAME para un servicio web interno
**Capa principal: 4 — Aplicación (DNS)**

**Escenario:**
El servidor SharePoint del N111 se llama `N111SHP01` pero los usuarios acceden a través de la URL `intranet.btpcce.esp`. Si en el futuro se migra SharePoint a otro servidor, la URL debe seguir funcionando sin avisar a nadie. Hay que crear el alias CNAME correspondiente.

**Tarea:**
Crear el registro CNAME `intranet` apuntando a `N111SHP01.btpcce.esp` y verificar que la resolución funciona desde un cliente del dominio.

**Procedimiento:**
```powershell
# Crear el registro CNAME:
Add-DnsServerResourceRecordCName `
    -ZoneName "btpcce.esp" `
    -Name "intranet" `
    -HostNameAlias "N111SHP01.btpcce.esp"

# Verificar:
Resolve-DnsName -Name "intranet.btpcce.esp" -Type CNAME
Resolve-DnsName -Name "intranet.btpcce.esp"   # resolución completa hasta la IP

# En el futuro, si SharePoint se mueve a N111SHP02:
# Solo hay que cambiar el CNAME — los usuarios siguen usando intranet.btpcce.esp
Set-DnsServerResourceRecord `
    -ZoneName "btpcce.esp" `
    -OldInputObject (Get-DnsServerResourceRecord -ZoneName "btpcce.esp" -Name "intranet" -RRType CName) `
    -NewInputObject (... nuevo destino ...)

# IMPORTANTE: si SharePoint usa autenticación Kerberos, el SPN debe incluir el alias:
setspn -A HTTP/intranet.btpcce.esp N111SHP01$
setspn -A HTTP/intranet N111SHP01$
```

**Preguntas:**
1. ¿Cuál es la diferencia entre un registro A y un CNAME? ¿Cuándo usarías uno y cuándo el otro?
2. Si el CNAME apunta a `N111SHP01.btpcce.esp` y ese registro A no existe, ¿qué ocurre?
3. ¿Por qué hay que añadir el SPN con el nombre del alias si SharePoint usa Kerberos?

**Concepto clave:** Un CNAME es un alias — apunta a otro nombre, no a una IP directamente. Esto permite cambiar la infraestructura detrás sin cambiar la URL que usan los usuarios. La trampa con Kerberos: el cliente construye el SPN usando el nombre con el que accede (`intranet`), no el nombre real del servidor. Si el SPN no incluye el alias, Kerberos falla aunque el registro DNS esté correcto.

---

## MÓDULO: FILE SERVER — Administración

---

### ADM-FS-01 — Crear carpeta compartida con permisos SMB y NTFS correctos
**Capa principal: 4 — Aplicación (SMB / NTFS)**

**Escenario:**
El N121 necesita una carpeta compartida `\\N121FS\operaciones_n121` para uso exclusivo del personal del nodo. El grupo `GG_Usuarios_N121` debe tener lectura y escritura. Los administradores del nodo (`GG_Admins_N121`) deben tener control total. Nadie más debe tener acceso.

**Tarea:**
Crear la carpeta, configurar el share SMB y los permisos NTFS correctamente aplicando la regla de intersección.

**Procedimiento:**
```powershell
# 1. Crear la carpeta en disco:
New-Item -Path "D:\shares\operaciones_n121" -ItemType Directory

# 2. Crear el share SMB:
New-SmbShare `
    -Name "operaciones_n121" `
    -Path "D:\shares\operaciones_n121" `
    -FullAccess "btpcce\GG_Admins_N121" `
    -ChangeAccess "btpcce\GG_Usuarios_N121" `
    -Description "Carpeta operacional nodo N121"

# 3. Configurar permisos NTFS (más granulares que el share):
$acl = Get-Acl "D:\shares\operaciones_n121"

# Eliminar herencia y limpiar permisos heredados:
$acl.SetAccessRuleProtection($true, $false)

# Añadir permisos explícitos:
$acl.AddAccessRule((New-Object System.Security.AccessControl.FileSystemAccessRule(
    "btpcce\GG_Admins_N121", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")))
$acl.AddAccessRule((New-Object System.Security.AccessControl.FileSystemAccessRule(
    "btpcce\GG_Usuarios_N121", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")))
# SYSTEM y Administrators locales siempre deben mantenerse:
$acl.AddAccessRule((New-Object System.Security.AccessControl.FileSystemAccessRule(
    "SYSTEM", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")))

Set-Acl "D:\shares\operaciones_n121" $acl

# 4. Verificar permisos efectivos:
Get-SmbShareAccess -Name "operaciones_n121"
Get-Acl "D:\shares\operaciones_n121" | Format-List
```

**Preguntas:**
1. El share tiene `ChangeAccess` para `GG_Usuarios_N121` y el NTFS tiene `Modify`. ¿Cuál es el permiso efectivo resultante?
2. ¿Por qué se elimina la herencia de permisos NTFS en una carpeta compartida de producción?
3. ¿Qué ocurre si se deja a `Everyone` con lectura en el share, aunque el NTFS esté bien configurado?

**Concepto clave:** La regla de doble capa: los permisos de share controlan el acceso por red, los permisos NTFS controlan el acceso al sistema de archivos. El permiso efectivo es siempre el MÁS RESTRICTIVO de los dos. La práctica habitual es poner `Control Total` en el share a los grupos relevantes y gestionar toda la granularidad en NTFS — así el share no interfiere con la lógica de permisos y se puede auditar en un solo lugar.

---

### ADM-FS-02 — Añadir un nuevo destino al espacio de nombres DFS
**Capa principal: 4 — Aplicación (DFS)**

**Escenario:**
Se despliega el N131 y su File Server (`N131FS`) ya está operativo. Hay que incorporarlo al espacio de nombres DFS `\\btpcce.esp\datos` para que los usuarios del N131 accedan a los datos localmente en lugar de cruzar la red hacia el N111.

**Tarea:**
Añadir `\\N131FS\datos` como nuevo destino en el namespace DFS y configurar la replicación DFSR con el servidor del N111.

**Procedimiento:**
```powershell
# 1. Crear la carpeta compartida en N131FS (si no existe):
# (en el servidor N131FS)
New-SmbShare -Name "datos" -Path "D:\shares\datos" -FullAccess "btpcce\GG_Admins_N131"

# 2. Añadir el nuevo destino al namespace DFS:
New-DfsnFolderTarget `
    -Path "\\btpcce.esp\datos" `
    -TargetPath "\\N131FS\datos" `
    -State Online

# 3. Verificar que el nuevo destino está activo:
Get-DfsnFolderTarget -Path "\\btpcce.esp\datos"

# 4. Crear grupo de replicación DFSR entre N111FS1 y N131FS:
New-DfsReplicationGroup -GroupName "Replicacion-Datos-N131"
Add-DfsrMember -GroupName "Replicacion-Datos-N131" -ComputerName "N111FS1"
Add-DfsrMember -GroupName "Replicacion-Datos-N131" -ComputerName "N131FS"

New-DfsReplicatedFolder `
    -GroupName "Replicacion-Datos-N131" `
    -FolderName "datos" `
    -DfsnPath "\\btpcce.esp\datos"

# 5. Verificar estado de replicación:
Get-DfsrState -ComputerName N131FS
```

**Preguntas:**
1. Una vez añadido el nuevo destino, ¿cómo decide el cliente DFS a qué servidor conectarse — al del N111 o al del N131?
2. Hasta que la replicación inicial no termina, el N131FS tiene la carpeta vacía. ¿Qué ocurre si un usuario del N131 accede a `\\btpcce.esp\datos` antes de que termine?
3. ¿Cómo configurarías la prioridad de destinos para que los usuarios del N131 siempre vayan a su servidor local primero?

**Concepto clave:** DFS usa el concepto de "sitio" AD para dirigir a cada cliente al destino más cercano. Si los sitios y subredes están bien configurados en AD Sites and Services, un cliente del N131 irá automáticamente a `\\N131FS\datos`. Si no están configurados, el cliente elige aleatoriamente y puede acabar cruzando la red WAN para acceder a datos que tiene local. La configuración de sitios AD es imprescindible para que DFS funcione de forma inteligente en multi-nodo.

---

## MÓDULO: EXCHANGE — Administración

---

### ADM-EX-01 — Crear buzón de correo para un usuario nuevo
**Capa principal: 4 — Aplicación (Exchange / AD)**

**Escenario:**
El nuevo Sgto. Martínez (creado en ADM-AD-01 como `p.martinez`) necesita buzón de correo en el servidor Exchange del N121 (`N121EX`). La base de datos de buzones del N121 es `BD-N121`.

**Tarea:**
Habilitar el buzón de correo para el usuario existente en AD y verificar que puede recibir y enviar correo.

**Procedimiento:**
```powershell
# El usuario ya existe en AD — solo hay que habilitar el buzón:
Enable-Mailbox `
    -Identity "p.martinez" `
    -Database "BD-N121"

# Verificar que el buzón se creó:
Get-Mailbox -Identity "p.martinez" | Select Name, Database, PrimarySmtpAddress

# Configurar cuota del buzón (aplicar política del nodo):
Set-Mailbox "p.martinez" `
    -IssueWarningQuota 4GB `
    -ProhibitSendQuota 5GB `
    -ProhibitSendReceiveQuota 6GB `
    -UseDatabaseQuotaDefaults $false

# Verificar que el buzón recibe correo (enviar correo de prueba desde EMS):
Send-MailMessage `
    -To "p.martinez@btpcce.esp" `
    -From "admin@btpcce.esp" `
    -Subject "Prueba buzón" `
    -Body "Verificación de buzón creado" `
    -SmtpServer "N121EX"

# Verificar estadísticas del buzón:
Get-MailboxStatistics -Identity "p.martinez" | Select DisplayName, TotalItemSize, ItemCount
```

**Preguntas:**
1. ¿Cuál es la diferencia entre `New-Mailbox` y `Enable-Mailbox`? ¿Cuándo usarías cada uno?
2. Si el usuario ya tiene cuenta en AD y usas `New-Mailbox`, ¿qué ocurre?
3. ¿Cómo crearías buzones para 15 usuarios de golpe a partir de un CSV?

```powershell
# Alta masiva de buzones desde CSV:
Import-Csv "C:\nuevos_usuarios.csv" | ForEach-Object {
    Enable-Mailbox -Identity $_.SamAccountName -Database "BD-N121"
    Set-Mailbox $_.SamAccountName `
        -IssueWarningQuota 4GB `
        -ProhibitSendQuota 5GB `
        -ProhibitSendReceiveQuota 6GB `
        -UseDatabaseQuotaDefaults $false
}
```

**Concepto clave:** `Enable-Mailbox` habilita un buzón sobre una cuenta AD que ya existe. `New-Mailbox` crea la cuenta AD y el buzón al mismo tiempo. En el PAIB el flujo correcto es siempre: primero crear el usuario en AD con la OU y grupos correctos (`New-ADUser`), después habilitar el buzón (`Enable-Mailbox`). Esto mantiene el control sobre dónde se crea la cuenta y a qué grupos pertenece desde el primer momento.

---

### ADM-EX-02 — Crear buzón compartido y asignar permisos de acceso
**Capa principal: 4 — Aplicación (Exchange)**

**Escenario:**
El N121 necesita un buzón compartido `secretaria.n121@btpcce.esp` al que puedan acceder varios usuarios del nodo para leer y responder correos en nombre de la secretaría. Los usuarios `p.martinez` y `a.garcia` deben poder abrir el buzón y enviar correos desde él.

**Tarea:**
Crear el buzón compartido, asignar permisos de acceso completo y permisos de envío como.

**Procedimiento:**
```powershell
# Crear el buzón compartido (no necesita licencia de usuario):
New-Mailbox `
    -Shared `
    -Name "Secretaria N121" `
    -Alias "secretaria.n121" `
    -UserPrincipalName "secretaria.n121@btpcce.esp" `
    -Database "BD-N121"

# Asignar permiso FullAccess (ver y gestionar el buzón):
Add-MailboxPermission `
    -Identity "secretaria.n121" `
    -User "p.martinez" `
    -AccessRights FullAccess `
    -InheritanceType All `
    -AutoMapping $true   # Outlook lo añade automáticamente

Add-MailboxPermission `
    -Identity "secretaria.n121" `
    -User "a.garcia" `
    -AccessRights FullAccess `
    -AutoMapping $true

# Asignar permiso SendAs (enviar como secretaria.n121):
Add-RecipientPermission `
    -Identity "secretaria.n121" `
    -Trustee "p.martinez" `
    -AccessRights SendAs `
    -Confirm:$false

Add-RecipientPermission `
    -Identity "secretaria.n121" `
    -Trustee "a.garcia" `
    -AccessRights SendAs `
    -Confirm:$false

# Verificar permisos asignados:
Get-MailboxPermission -Identity "secretaria.n121" | Where-Object {$_.User -notlike "NT AUTHORITY*"}
Get-RecipientPermission -Identity "secretaria.n121"
```

**Preguntas:**
1. ¿Cuál es la diferencia entre el permiso `FullAccess` y el permiso `SendAs`?
2. Si `p.martinez` tiene `FullAccess` pero no `SendAs`, ¿qué puede y qué no puede hacer?
3. ¿Qué es `AutoMapping` y por qué es útil en Outlook?

**Concepto clave:** `FullAccess` permite abrir el buzón, leer y mover correos. `SendAs` permite enviar correos que aparecen como si los enviara `secretaria.n121` (el destinatario ve la dirección del buzón compartido, no la del usuario real). Son permisos independientes — en la mayoría de casos se asignan juntos. `AutoMapping` hace que Outlook detecte automáticamente el buzón compartido y lo añada a la vista sin que el usuario tenga que configurar nada.

---

### ADM-EX-03 — Crear lista de distribución y cargar miembros masivamente
**Capa principal: 4 — Aplicación (Exchange)**

**Escenario:**
Hay que crear una lista de distribución `todos.n121@btpcce.esp` que incluya a todos los usuarios del N121. El comandante del nodo quiere poder enviar comunicados a todos de una sola vez. Se tienen 45 usuarios en un CSV.

**Tarea:**
Crear la lista de distribución, cargar los miembros desde CSV y restringir quién puede enviar a la lista.

**Procedimiento:**
```powershell
# Crear la lista de distribución:
New-DistributionGroup `
    -Name "Todos N121" `
    -Alias "todos.n121" `
    -Type Distribution `
    -MemberJoinRestriction Closed `       # solo admins pueden añadir miembros
    -MemberDepartRestriction Closed `     # solo admins pueden quitar miembros
    -PrimarySmtpAddress "todos.n121@btpcce.esp"

# Cargar miembros desde CSV (columna: SamAccountName):
Import-Csv "C:\usuarios_n121.csv" | ForEach-Object {
    Add-DistributionGroupMember `
        -Identity "todos.n121" `
        -Member $_.SamAccountName
}

# Verificar cuántos miembros tiene:
(Get-DistributionGroupMember -Identity "todos.n121").Count

# Restringir quién puede enviar a esta lista (solo el comandante y los admins):
Set-DistributionGroup `
    -Identity "todos.n121" `
    -AcceptMessagesOnlyFromSendersOrMembers @("comandante.n121", "GG_Admins_N121")

# Verificar configuración:
Get-DistributionGroup -Identity "todos.n121" | Select Name, PrimarySmtpAddress, AcceptMessagesOnlyFrom*
```

**Preguntas:**
1. ¿Cuál es la diferencia entre una lista de distribución y un grupo de seguridad habilitado para correo? ¿Cuándo usarías cada uno?
2. Si no se restringe `AcceptMessagesOnlyFrom`, ¿qué riesgo existe?
3. ¿Cómo actualizarías automáticamente la lista cuando se incorpore un nuevo usuario, sin tener que añadirlo manualmente?

**Concepto clave:** Una lista de distribución es solo para correo — no sirve para permisos de carpetas ni GPOs. Un grupo de seguridad habilitado para correo sirve para ambas cosas. Sin restricción en `AcceptMessagesOnlyFrom`, cualquier remitente dentro o fuera del dominio puede enviar a `todos.n121@btpcce.esp` — en un entorno militar, una lista de distribución sin control de remitentes es un vector de desinformación.

---

## MÓDULO: SHAREPOINT — Administración

---

### ADM-SP-01 — Crear colección de sitios y asignar administrador
**Capa principal: 4 — Aplicación (SharePoint)**

**Escenario:**
El N121 necesita su propio sitio en SharePoint para documentación operacional, partes y comunicados internos del nodo. Hay que crear la colección de sitios bajo `http://N111SHP01/sites/n121` y asignar al administrador del nodo como propietario.

**Tarea:**
Crear la colección de sitios usando la plantilla de equipo, asignar administrador y verificar el acceso.

**Procedimiento:**
```powershell
# Desde SharePoint Management Shell en N111SHP01:

# Crear la colección de sitios del N121:
New-SPSite `
    -Url "http://N111SHP01/sites/n121" `
    -OwnerAlias "btpcce\admin.n121" `
    -SecondaryOwnerAlias "btpcce\p.martinez" `
    -Name "Portal Operacional N121" `
    -Description "Documentación y comunicados del Nodo CIA 12" `
    -Template "STS#0"    # STS#0 = Sitio de equipo

# Verificar que se creó:
Get-SPSite -Identity "http://N111SHP01/sites/n121" | Select Url, Owner

# Verificar acceso como administrador del nodo:
# Navegar a http://N111SHP01/sites/n121 con la cuenta admin.n121
# → Debería ver el sitio con permisos de propietario

# Listar todas las colecciones de sitios:
Get-SPSite | Select Url, Owner, ContentDatabase
```

**Preguntas:**
1. ¿Qué diferencia hay entre una colección de sitios y un subsitio? ¿Cuándo crear una colección nueva en lugar de un subsitio?
2. La plantilla `STS#0` crea un "Sitio de equipo". ¿Qué otras plantillas existen y cuándo las usarías?
3. ¿Cómo limitarías el espacio en disco que puede usar esta colección de sitios?

```powershell
# Establecer cuota de almacenamiento:
Set-SPSite -Identity "http://N111SHP01/sites/n121" `
    -MaxSize 10GB `
    -WarningSize 8GB
```

**Concepto clave:** Una colección de sitios es una unidad administrativa independiente con su propia base de contenido en SQL, su propio administrador y sus propias cuotas. Subsitios comparten la base de contenido y el administrador de la colección padre. En el PAIB, cada nodo debe ser una colección de sitios independiente — así si un nodo tiene problemas en SharePoint no afecta a los demás, y cada administrador de nodo controla solo su colección.

---

### ADM-SP-02 — Configurar permisos de un sitio con grupos de AD
**Capa principal: 4 — Aplicación (SharePoint / AD)**

**Escenario:**
El sitio `http://N111SHP01/sites/n121` tiene que tener tres niveles de acceso: los oficiales del nodo (`GG_Oficiales_N121`) con permiso de propietario, el personal de tropa (`GG_Usuarios_N121`) con permiso de miembro (lectura y contribución), y observadores de otros nodos (`GG_Admins_N111`) con solo lectura.

**Tarea:**
Configurar los grupos de SharePoint usando grupos de AD y eliminar los permisos individuales que pudieran existir.

**Procedimiento:**
```powershell
# Desde SharePoint Management Shell:
$web = Get-SPWeb "http://N111SHP01/sites/n121"

# Añadir grupos AD a los grupos de SharePoint correspondientes:
# Grupo Propietarios del sitio → GG_Oficiales_N121:
$ownersGroup = $web.SiteGroups["Portal Operacional N121 - Propietarios"]
$ownersGroup.AddUser("btpcce\GG_Oficiales_N121", $null, "GG Oficiales N121", "")

# Grupo Miembros del sitio → GG_Usuarios_N121:
$membersGroup = $web.SiteGroups["Portal Operacional N121 - Miembros"]
$membersGroup.AddUser("btpcce\GG_Usuarios_N121", $null, "GG Usuarios N121", "")

# Grupo Visitantes → GG_Admins_N111 (solo lectura):
$visitorsGroup = $web.SiteGroups["Portal Operacional N121 - Visitantes"]
$visitorsGroup.AddUser("btpcce\GG_Admins_N111", $null, "GG Admins N111", "")

$web.Update()

# Verificar permisos del sitio:
Get-SPWeb "http://N111SHP01/sites/n121" | 
    Select -ExpandProperty RoleAssignments | 
    ForEach-Object { "$($_.Member.Name) → $($_.RoleDefinitionBindings.Name)" }
```

**Desde la GUI (alternativa):**
```
Configuración del sitio → Permisos del sitio → Conceder permisos
→ Añadir "btpcce\GG_Oficiales_N121" → Nivel de permiso: Control total
→ Añadir "btpcce\GG_Usuarios_N121" → Nivel de permiso: Colaborar
→ Añadir "btpcce\GG_Admins_N111" → Nivel de permiso: Leer
```

**Preguntas:**
1. ¿Por qué se añaden grupos de AD en lugar de usuarios individuales? ¿Qué trabajo se ahorra a futuro?
2. Si la colección de sitios tiene herencia de permisos desde el sitio raíz, ¿qué hay que hacer antes de configurar permisos específicos para este sitio?
3. ¿Cómo comprobarías qué permiso efectivo tiene un usuario concreto sobre el sitio?

**Concepto clave:** SharePoint + grupos AD = administración cero en SharePoint. Cuando alguien se incorpora al N121, se le añade al grupo AD correspondiente y automáticamente tiene acceso a SharePoint, carpetas compartidas, impresoras y cualquier otro recurso basado en ese grupo. Sin grupos AD, cada recurso hay que gestionarlo individualmente — en un despliegue con decenas de usuarios es inviable.

---

## MÓDULO: PRINT SERVER — Administración

---

### ADM-PS-01 — Agregar impresora al servidor y desplegarla por GPO
**Capa principal: 4 — Aplicación (Print Server / GPO)**

**Escenario:**
Llega al N121 una nueva impresora de red (`Impresora-N121-PLANA`, IP `102.197.133.200`). Hay que añadirla al servidor de impresión del N111, compartirla y desplegarla automáticamente a todos los usuarios del grupo `GG_Usuarios_N121` mediante GPO.

**Tarea:**
Añadir la impresora al servidor, instalar el driver, compartirla y crear la GPO de despliegue.

**Procedimiento:**
```
# PASO 1 — Instalar la impresora en el servidor de impresión (printmanagement.msc):
# Servidores de impresión → N111-PS → Impresoras → Agregar impresora
# → Impresora de red TCP/IP → IP: 102.197.133.200 → Protocolo: RAW, Puerto: 9100
# → Seleccionar driver (o instalar desde CD/descarga del fabricante)
# → Nombre del recurso compartido: "N121-Plana"
# → Publicar en Active Directory: ✓

# PASO 2 — Verificar que la impresora responde:
# printmanagement.msc → N111-PS → Impresoras → N121-Plana → Imprimir página de prueba

# PASO 3 — Crear la GPO de despliegue:
# GPMC → btpcce.esp → OU=Usuarios → OU=N121 → Crear GPO: "Deploy-Impresoras-N121"
# Editar GPO:
# Configuración de usuario → Directivas → Configuración de Windows → Impresoras implementadas
# → Agregar impresora → \\N111-PS\N121-Plana → Acción: Implementar (conexión)

# PASO 4 — Vincular la GPO a la OU correcta y forzar aplicación:
```
```powershell
# Verificar desde PowerShell que la impresora está compartida y publicada en AD:
Get-Printer -ComputerName N111-PS | Where-Object {$_.Name -like "*N121*"} | 
    Select Name, ShareName, Published

# Desde un equipo del N121 — verificar que se desplegó:
gpupdate /force
Get-Printer | Where-Object {$_.Name -like "*N121*"}
```

**Preguntas:**
1. ¿Por qué se instala la impresora en el servidor de impresión en lugar de directamente en cada equipo cliente?
2. La GPO se vincula a `OU=Usuarios,OU=N121`. ¿Por qué a la OU de usuarios y no a la de equipos?
3. Si la impresora está publicada en AD, ¿cómo puede un usuario encontrarla manualmente sin que se la despliegue por GPO?

**Concepto clave:** El servidor de impresión centraliza la gestión de drivers — si el fabricante saca un driver nuevo, se actualiza en el servidor y todos los clientes lo reciben automáticamente. Sin servidor centralizado, habría que actualizar el driver en cada equipo individualmente. La GPO de usuario (no de equipo) es la correcta para impresoras: la impresora sigue al usuario, no a la máquina — si el Sgto. Martínez inicia sesión en cualquier equipo del nodo, su impresora aparece.

---

## Progresión pedagógica sugerida

```
SESIÓN 1 — Virtualización y Switch de Núcleo
  VJ-01 → VJ-03 → SW-01 → VJ-02

SESIÓN 2 — Active Directory
  AD-01 → AD-02 → AD-03

SESIÓN 3 — File Server y Print Server
  FS-01 → FS-02 → PS-01 → PS-03 → FS-03 → PS-02

SESIÓN 4 — Exchange
  EX-01 → EX-02 → EX-03

SESIÓN 5 — SharePoint y Servidor de Despliegue
  SP-01 → SP-02 → SP-03 → SD-01 → SD-02 → SD-03
```

---

*Sgto. Narros — PAIB 2.0*
