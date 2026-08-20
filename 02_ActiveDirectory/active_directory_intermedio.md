# Active Directory — Nivel Intermedio

> Entorno: dominio `btpcce.lab`, único DC `DC01` (10.10.10.11), clientes en 10.10.10.0/24.
> Marco de diagnóstico y modo de uso: ver el [README](README.md).

Aquí se asume que dominas los fundamentos del [nivel básico](active_directory_basico.md). Estos ejercicios entran en los *internals*: por qué una GPO se procesa o se deniega, cómo se distribuyen las directivas por SYSVOL, cómo Kerberos depende del tiempo y de los SPN, y cómo se recupera el directorio cuando algo se borra.

## Índice

**Diagnóstico**
| # | Título | Capa | Dificultad |
|:---:|---|:---:|:---:|
| AD-01 | Una GPO no se aplica a un equipo concreto | **4** | ★★☆ |
| AD-02 | Los usuarios entran, pero las GPOs y los scripts no se ejecutan | **4** | ★★☆ |
| AD-03 | El inicio de sesión falla en un equipo por desfase de reloj | **4** | ★★☆ |
| AD-04 | Una cuenta se bloquea sola varias veces al día | **4** | ★★☆ |
| AD-05 | Se borra por error una OU con todos los usuarios — recuperación | **4** | ★★★ |
| AD-06 | Un servicio falla con error Kerberos por un SPN duplicado | **3 / 4** | ★★★ |

**Administración**
| # | Título | Capa | Dificultad |
|:---:|---|:---:|:---:|
| ADM-01 | Crear un grupo de seguridad y delegar control sobre una OU | **4** | ★★☆ |
| ADM-02 | Crear una GPO y vincularla a una OU | **4** | ★★☆ |
| ADM-03 | Política de contraseñas específica (PSO) para un grupo | **4** | ★★☆ |
| ADM-04 | Crear una cuenta de servicio administrada (gMSA) | **4** | ★★★ |
| ADM-05 | Aplicar la estrategia de anidamiento AGDLP para permisos | **4** | ★★☆ |
| ADM-06 | Habilitar la Papelera de reciclaje de AD y probar una restauración | **4** | ★★☆ |

---

# DIAGNÓSTICO

---

### AD-01 — Una GPO no se aplica a un equipo concreto
**Capa principal: 4 — Aplicación (GPO / filtrado de seguridad)**

**Escenario:**
Se crea una GPO que configura el fondo de escritorio corporativo. Se filtra para que solo la reciba el grupo `GG_Ventas`. Los usuarios de Ventas que llevan tiempo la tienen bien, pero un equipo nuevo recién unido al dominio, cuyo usuario también está en `GG_Ventas`, no la aplica.

**Síntomas:**
- `gpresult /r` en el equipo nuevo muestra la GPO en la sección **"GPO denegadas → Motivo: Seguridad (Denied - Security)"**
- El usuario sí pertenece a `GG_Ventas`
- En el filtrado de seguridad de la GPO solo aparece `GG_Ventas`; se quitó `Usuarios autenticados`
- El resto de equipos, más antiguos, siguen aplicándola sin problema

**Preguntas de diagnóstico:**
1. Con un único DC no hay replicación de por medio. Entonces, ¿por qué la GPO se aplica a unos objetos y a otros no?
2. Además del usuario, ¿qué otra identidad necesita permiso de **Lectura** sobre la GPO para poder procesarla? (pista: la GPO también la lee el equipo, no solo el usuario)
3. ¿Qué representa exactamente la sección "Denegado (Seguridad)" en la salida de `gpresult`?
4. Si el filtrado de seguridad solo lista un grupo de usuarios, ¿qué le falta al objeto equipo para poder leer la directiva?

---

### AD-02 — Los usuarios entran, pero las GPOs y los scripts de inicio no se ejecutan
**Capa principal: 4 — Aplicación (SYSVOL / NETLOGON)**

**Escenario:**
Los usuarios inician sesión con normalidad, pero desde ayer ninguna GPO se aplica y los scripts de inicio de sesión (mapeo de unidades de red) no se ejecutan en ningún equipo. No se ha tocado ninguna directiva.

**Síntomas:**
- La autenticación funciona (los usuarios entran)
- `gpupdate /force` en cualquier cliente devuelve error al acceder a las plantillas de directiva
- En `DC01`, `net share` **no** muestra los recursos compartidos `SYSVOL` ni `NETLOGON`
- El log de DFS Replication en DC01 registra un `journal wrap` en el volumen de SYSVOL

**Preguntas de diagnóstico:**
1. Un usuario se autentica bien pero no recibe ninguna GPO. ¿Desde qué recurso compartido del DC descargan los clientes las plantillas de GPO y los scripts?
2. ¿Por qué la autenticación (Kerberos/LDAP) puede seguir funcionando aunque SYSVOL no esté compartido? ¿Por qué caminos distintos viajan la autenticación y la distribución de directivas?
3. ¿Qué significa un `journal wrap` de DFSR y por qué deja de publicarse SYSVOL cuando ocurre?
4. ¿Qué comando ejecutado en el DC te confirma de un vistazo si SYSVOL y NETLOGON se están publicando?

---

### AD-03 — El inicio de sesión falla en un equipo por desfase de reloj
**Capa principal: 4 — Aplicación (Kerberos / sincronización horaria)**

**Escenario:**
Un equipo que estuvo apagado y con la pila de la BIOS agotada arranca con la fecha y hora desajustadas. Su usuario no puede iniciar sesión en el dominio; el resto de equipos entran sin problema. El error en el visor de eventos del equipo es `KRB_AP_ERR_SKEW`.

**Síntomas:**
- Login de dominio imposible solo en ese equipo
- `w32tm /query /status` en el equipo muestra una desviación de +9 minutos respecto a `DC01`
- El resto de equipos, con la hora correcta, se autentican sin problema
- La conectividad con `DC01` es correcta (ping y puertos OK)

**Preguntas de diagnóstico:**
1. Kerberos rechaza tickets con un desfase horario mayor a cierto umbral. ¿Cuál es ese umbral por defecto y por qué existe? (pista: ataques de repetición)
2. En un dominio AD, ¿quién es la fuente de tiempo autoritativa y de quién debe sincronizar cada cliente?
3. ¿Por qué el problema afecta solo a ese equipo y no a todo el dominio?
4. En este laboratorio de un solo DC, ¿qué rol es el que fija la hora de referencia y de dónde debería tomarla ese DC?

---

### AD-04 — Una cuenta se bloquea sola varias veces al día
**Capa principal: 4 — Aplicación (directiva de bloqueo)**

**Escenario:**
La cuenta `btpcce\lopez` se bloquea sola 3-4 veces al día. El usuario jura que escribe bien la contraseña y, de hecho, cuando el administrador la desbloquea, entra sin problema... hasta que se vuelve a bloquear una hora después. López cambió su contraseña ayer.

**Síntomas:**
- Bloqueos recurrentes sin que el usuario falle la contraseña conscientemente
- La directiva de dominio bloquea la cuenta tras 5 intentos fallidos
- Los intentos fallidos vienen de un dispositivo que el usuario no está usando activamente
- El teléfono móvil de López tenía guardada la contraseña **anterior** para el correo

**Preguntas de diagnóstico:**
1. ¿Qué evento del log de seguridad registra un bloqueo de cuenta y en qué servidor hay que buscarlo?
2. ¿Qué campo de ese evento te dice desde dónde vino el intento que provocó el bloqueo?
3. ¿Qué fuentes habituales generan bloqueos "fantasma" con credenciales antiguas cacheadas?
4. Si solo desbloqueas la cuenta sin buscar el origen, ¿por qué el problema vuelve a aparecer?

---

### AD-05 — Se borra por error una OU con todos los usuarios — recuperación
**Capa principal: 4 — Aplicación (recuperación del directorio)**

**Escenario:**
Durante una limpieza, un administrador elimina por error la OU `Usuarios`, que contenía las 60 cuentas del personal. Nadie de esa OU puede iniciar sesión. Al haber un único DC, no existe otro DC del que "volver a copiar" los objetos: la copia autoritativa del directorio es la de `DC01`.

**Síntomas:**
- Las 60 cuentas han desaparecido de ADUC
- Los usuarios afectados no pueden autenticarse
- No hay un segundo DC del que recuperar por replicación
- Existe una copia de seguridad de estado del sistema de anoche

**Preguntas de diagnóstico:**
1. Con un solo DC, la replicación no puede ayudarte a recuperar objetos borrados. ¿Cuáles son entonces tus dos únicas redes de seguridad?
2. ¿Qué es la Papelera de reciclaje de AD y qué ventaja tiene frente a una restauración autoritativa desde copia de seguridad?
3. Si la Papelera de AD NO estuviera habilitada, ¿qué procedimiento y en qué modo de arranque tendrías que usar?
4. ¿Por qué habilitar la Papelera de AD el primer día habría convertido esta emergencia en un trámite de minutos?

---

### AD-06 — Un servicio falla con error Kerberos por un SPN duplicado
**Capa principal: 3 — Transporte · 4 — Aplicación (Kerberos / SPN)**

**Escenario:**
Se publica una aplicación web interna en `http://intranet.btpcce.lab`, servida por un servidor `WEB01` bajo una cuenta de servicio de dominio. Al acceder por el nombre, los usuarios reciben un error de autenticación; si acceden por la IP, entran (cae a NTLM). En el log del servidor aparece `KRB_AP_ERR_MODIFIED`.

**Síntomas:**
- Acceso por `http://intranet.btpcce.lab` → falla la autenticación integrada
- Acceso por IP → funciona (usa NTLM en lugar de Kerberos)
- `setspn -X` en el dominio reporta un **SPN duplicado** para `HTTP/intranet.btpcce.lab`
- El SPN está registrado a la vez en la cuenta de equipo `WEB01` y en la cuenta de servicio de la aplicación

**Preguntas de diagnóstico:**
1. ¿Qué es un SPN (Service Principal Name) y qué papel juega en que Kerberos localice la cuenta correcta de un servicio?
2. ¿Por qué un SPN debe ser **único** en todo el dominio? ¿Qué provoca exactamente el error `KRB_AP_ERR_MODIFIED`?
3. ¿Por qué el acceso funciona por IP (NTLM) pero falla por nombre (Kerberos)? ¿Qué te dice eso sobre dónde está el problema?
4. ¿Cómo localizarías el SPN duplicado y en qué cuenta debe quedarse registrado y en cuál eliminarse?

---

# ADMINISTRACIÓN

---

### ADM-01 — Crear un grupo de seguridad y delegar control sobre una OU
**Capa principal: 4 — Aplicación · Dificultad: ★★☆**

**Objetivo:** El personal de soporte debe poder resetear las contraseñas de los usuarios **sin** ser administradores del dominio. Crea el grupo de seguridad `GG_Soporte`, añade a los técnicos y delega sobre la OU de Usuarios únicamente la capacidad de restablecer contraseñas, aplicando el principio de mínimo privilegio. Debes ser capaz de explicar por qué se delega **sobre la OU** y no sobre cada usuario.

---

### ADM-02 — Crear una GPO y vincularla a una OU
**Capa principal: 4 — Aplicación · Dificultad: ★★☆**

**Objetivo:** Aplica un fondo de escritorio corporativo únicamente a los equipos de la OU `Equipos`. Crea la GPO, configura el ajuste correspondiente y vincúlala al nivel adecuado para que no afecte a objetos fuera de esa OU. Ten claro el orden de procesamiento **LSDOU** y por qué el nivel del vínculo determina el alcance.

---

### ADM-03 — Política de contraseñas específica (PSO) para un grupo
**Capa principal: 4 — Aplicación · Dificultad: ★★☆**

**Objetivo:** Las cuentas privilegiadas (`GG_Administradores`) deben tener una contraseña más larga y de caducidad más corta que el resto del dominio, **sin** modificar la política general. Crea una Política de Contraseñas Detallada (PSO) más estricta y aplícala solo a ese grupo. Debes saber justificar por qué una PSO se asigna a un grupo y no se vincula a una OU como una GPO.

---

### ADM-04 — Crear una cuenta de servicio administrada (gMSA)
**Capa principal: 4 — Aplicación · Dificultad: ★★★**

**Objetivo:** Un servicio programado que corre en un servidor miembro necesita una cuenta de dominio cuya contraseña se gestione y rote sola, sin intervención manual. Prepara el requisito previo en el bosque (la clave raíz KDS), crea la gMSA autorizando qué equipos pueden usarla e instálala en el servidor de destino. Debes entender qué problema de las cuentas de servicio tradicionales resuelve una gMSA.

---

### ADM-05 — Aplicar la estrategia de anidamiento AGDLP para permisos
**Capa principal: 4 — Aplicación · Dificultad: ★★☆**

**Objetivo:** Da al equipo de Operaciones acceso de **modificación** a la carpeta `\\FS01\operaciones` siguiendo la estrategia AGDLP (Accounts → Global → Domain Local → Permissions). El permiso NTFS debe recaer en un grupo Domain Local, nunca en el usuario ni en el grupo Global directamente, de modo que el modelo sea escalable. Debes ser capaz de explicar qué se gana desacoplando "quién es la persona" de "a qué recurso accede".

---

### ADM-06 — Habilitar la Papelera de reciclaje de AD y probar una restauración
**Capa principal: 4 — Aplicación · Dificultad: ★★☆**

**Objetivo:** Habilita la Papelera de reciclaje de AD en el bosque `btpcce.lab` y valida su funcionamiento: crea un usuario de prueba, bórralo, y restáuralo comprobando que recupera todos sus atributos y pertenencias a grupos. Ten presente antes de empezar que habilitar la Papelera es una operación **irreversible**, y sé capaz de explicar qué diferencia hay entre este método y una restauración autoritativa desde copia de seguridad.
