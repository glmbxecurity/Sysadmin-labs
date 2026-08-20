# Active Directory — Nivel Básico

> Entorno: dominio `btpcce.lab`, único DC `DC01` (10.10.10.11), clientes en 10.10.10.0/24.
> Marco de diagnóstico y modo de uso: ver el [README](README.md).

Fundamentos de AD: cómo el cliente localiza al DC, cómo se inicia sesión y cómo se gestionan los objetos básicos del directorio (usuarios, grupos, OUs). Si estos cimientos no están firmes, nada de lo intermedio se sostiene.

## Índice

**Diagnóstico**
| # | Título | Capa | Dificultad |
|:---:|---|:---:|:---:|
| AD-01 | El DC está caído: nadie puede iniciar sesión | **2 / 4** | ★★☆ |
| AD-02 | Un equipo recién unido no encuentra el dominio | **2 / 4** | ★★☆ |
| AD-03 | "La relación de confianza... ha fallado" | **4** | ★☆☆ |
| AD-04 | Un usuario no puede iniciar sesión: cuenta deshabilitada o caducada | **4** | ★☆☆ |

**Administración**
| # | Título | Capa | Dificultad |
|:---:|---|:---:|:---:|
| ADM-01 | Diseñar y crear la estructura de OUs | **4** | ★☆☆ |
| ADM-02 | Alta de usuario nuevo con grupo y OU correctos | **4** | ★☆☆ |
| ADM-03 | Crear un grupo de seguridad y añadir miembros | **4** | ★☆☆ |
| ADM-04 | Restablecer una contraseña y desbloquear una cuenta | **4** | ★☆☆ |

---

# DIAGNÓSTICO

---

### AD-01 — El DC está caído: nadie puede iniciar sesión (no hay failover)
**Capa principal: 2 — Internet · 4 — Aplicación (Kerberos)**

**Escenario:**
A las 08:15, ningún usuario puede autenticarse en sus equipos. El error en pantalla es "No se puede iniciar sesión. El servicio de inicio de sesión no está disponible". Los equipos son miembros de `btpcce.lab`. El único DC del dominio es `DC01` (10.10.10.11).

**Síntomas:**
- Error de autenticación en todos los equipos del dominio
- Ping a `DC01` (10.10.10.11) → timeout
- Los usuarios con sesión ya abierta pueden seguir trabajando sin interrupciones
- No hay ningún otro DC al que los clientes puedan recurrir

**Preguntas de diagnóstico:**
1. ¿Qué protocolo usa Windows para autenticar contra el dominio? ¿Por qué necesita comunicación de red con el DC para iniciar sesión?
2. ¿Por qué los usuarios con sesión ya abierta pueden seguir trabajando? ¿Qué mecanismo de caché interviene?
3. Con un único DC, ¿qué opciones tiene el cliente si ese DC no responde? ¿Qué cambiaría si hubiera un segundo DC?
4. ¿Cuál sería tu primer paso de comprobación antes de dar por hecho que es un problema de red?

---

### AD-02 — Un equipo recién unido no encuentra el dominio
**Capa principal: 2 — Internet · 4 — Aplicación (DNS)**

**Escenario:**
Se instala un equipo nuevo y al intentar unirlo al dominio aparece "No se pudo encontrar un controlador de dominio para el dominio btpcce.lab". El equipo tiene IP, navega por Internet y hace ping a `DC01` por su IP sin problema.

**Síntomas:**
- Ping por IP a `DC01` (10.10.10.11) → responde
- `nslookup btpcce.lab` → falla o devuelve una dirección de Internet
- El equipo tiene configurado como DNS un servidor público (8.8.8.8), no `DC01`
- La consulta de registros SRV del dominio no devuelve resultados

**Preguntas de diagnóstico:**
1. Un equipo que ya tiene IP y hasta navega por Internet, ¿por qué no encuentra el dominio? ¿Qué usa exactamente el cliente para localizar el DC?
2. ¿Qué tipo de registro DNS publica el DC para anunciarse y bajo qué nombre? (pista: `_ldap._tcp...`)
3. ¿Por qué usar un DNS público en un cliente de dominio rompe AD aunque haya conectividad perfecta?
4. ¿Qué DNS debería tener configurado un miembro del dominio y por qué?

---

### AD-03 — "La relación de confianza entre esta estación y el dominio ha fallado"
**Capa principal: 4 — Aplicación (canal seguro del equipo)**

**Escenario:**
Un equipo que estuvo apagado tres meses vuelve a encenderse. Al intentar iniciar sesión con una cuenta de dominio, aparece: "La relación de confianza entre esta estación de trabajo y el dominio principal ha fallado". Un administrador local sí puede entrar.

**Síntomas:**
- Login de dominio imposible; login con cuenta local sí funciona
- El equipo tiene conectividad con `DC01`
- El evento 3210 / 5719 (Netlogon) aparece en el log del sistema del equipo
- La contraseña de la cuenta de equipo en AD ya no coincide con la que guarda la máquina

**Preguntas de diagnóstico:**
1. Los equipos también tienen contraseña en AD y la cambian solos periódicamente. ¿Cada cuánto y por qué?
2. ¿Por qué un equipo apagado mucho tiempo pierde la confianza aunque nadie haya tocado nada?
3. ¿Cuál es la forma correcta de reparar la confianza SIN sacar y volver a meter el equipo en el dominio? ¿Qué efecto secundario tendría reunirlo? (pista: el SID)
4. ¿Por qué puede entrar un administrador local pero no una cuenta de dominio?

---

### AD-04 — Un usuario no puede iniciar sesión: cuenta deshabilitada o caducada
**Capa principal: 4 — Aplicación (estado de la cuenta)**

**Escenario:**
El Sgto. `Ortega`, que estuvo de baja dos meses, vuelve al trabajo y no puede iniciar sesión. El mensaje varía: en un equipo dice "Su cuenta ha sido deshabilitada, consulte al administrador"; en otro, "La cuenta ha caducado". Sus compañeros entran con normalidad.

**Síntomas:**
- El resto del personal inicia sesión sin problema
- La contraseña que teclea Ortega es la correcta
- En ADUC, su cuenta aparece con el icono de deshabilitada
- La pestaña "Cuenta" muestra una fecha de caducidad de la cuenta ya pasada

**Preguntas de diagnóstico:**
1. Diferencia entre estos cuatro estados de una cuenta: **deshabilitada**, **caducada**, **bloqueada** y **con la contraseña caducada**. ¿Qué mensaje de error produce cada uno?
2. ¿Por qué una baja temporal suele traducirse en una cuenta deshabilitada y no eliminada?
3. ¿Dónde se configura la fecha de caducidad de una cuenta y para qué casos tiene sentido usarla?
4. ¿Cómo distinguirías, solo por el mensaje y sin tocar nada aún, si el problema es de estado de cuenta o de contraseña?

---

# ADMINISTRACIÓN

---

### ADM-01 — Diseñar y crear la estructura de OUs del dominio
**Capa principal: 4 — Aplicación · Dificultad: ★☆☆**

**Objetivo:** Crea la estructura de Unidades Organizativas (OUs) sobre la que se apoyará todo el dominio: como mínimo `Usuarios`, `Equipos`, `Grupos` y `Servidores`, colgando de la raíz `btpcce.lab`. Diséñala pensando en que cada OU podrá recibir GPOs distintas y delegarse por separado, y asegúrate de que las OUs queden **protegidas contra el borrado accidental**.

---

### ADM-02 — Alta de usuario nuevo con grupo y OU correctos
**Capa principal: 4 — Aplicación · Dificultad: ★☆☆**

**Objetivo:** Da de alta al Sgto. `Mateo Sanz` en el dominio `btpcce.lab`. Debe quedar ubicado en la OU de Usuarios, pertenecer al grupo de su unidad (`GG_Operaciones`) y estar obligado a cambiar la contraseña en el primer inicio de sesión.

---

### ADM-03 — Crear un grupo de seguridad y añadir miembros
**Capa principal: 4 — Aplicación · Dificultad: ★☆☆**

**Objetivo:** Crea el grupo de seguridad `GG_Operaciones` en la OU de Grupos y añade a los usuarios del departamento. Ten claro, y sé capaz de justificar, dos decisiones de diseño: por qué eliges un grupo de **seguridad** y no de distribución, y por qué el ámbito **Global** frente a Domain Local o Universal en este caso.

---

### ADM-04 — Restablecer una contraseña y desbloquear una cuenta
**Capa principal: 4 — Aplicación · Dificultad: ★☆☆**

**Objetivo:** Un usuario ha olvidado su contraseña y, tras varios intentos, ha bloqueado su cuenta. Restablece la contraseña forzando el cambio en el próximo inicio de sesión y desbloquea la cuenta. Distingue claramente las dos acciones: **restablecer** (cambiar la contraseña) no es lo mismo que **desbloquear** (levantar el bloqueo por intentos fallidos).
