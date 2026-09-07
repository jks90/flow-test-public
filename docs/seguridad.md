# Seguridad de FlowTest

> Documento para equipos técnicos y de seguridad que evalúan FlowTest.
> Versión de referencia: **5.14.1** · última revisión: 2026-09-08.
> Contacto de seguridad: **security@flowtest.es** (ver [Divulgación responsable](#divulgación-responsable)).

FlowTest es una herramienta visual + CLI para componer, ejecutar y documentar flujos de peticiones
HTTP/SQL. Se usa de dos formas, con modelos de seguridad distintos:

- **Autoalojada** (imagen Docker o npm): corre en tu propia infraestructura, contra tus APIs. Tú
  controlas el entorno; FlowTest no envía tus datos a ningún sitio salvo la validación de licencia.
- **Cloud** (`app.flowtest.es`): SaaS multiinquilino gestionado por nosotros, con un **contenedor
  aislado por organización**.

Este documento describe los controles de seguridad de ambos y el endurecimiento realizado en la 5.14.1.

---

## Resumen ejecutivo

| Área | Control |
|------|---------|
| Aislamiento multiinquilino | Un contenedor Docker por organización; datos y proceso separados. El gateway es el único punto de entrada y siempre autentica la sesión. |
| Autenticación | Contraseñas con *scrypt* + sal por usuario; sesiones con JWT firmado en cookies `HttpOnly`, `SameSite=Lax`, `Secure`; rotación de *refresh token* con detección de reuso; 2FA TOTP opcional; SSO OIDC (Business). |
| Autorización | Todo acceso se acota a la organización y al usuario de la sesión; no se confía en identificadores enviados por el cliente. |
| Cifrado de credenciales | Las credenciales de los flujos (`{{secret:…}}`) y las claves de IA se cifran con AES-256-GCM y **nunca** llegan al navegador. |
| Ejecución de código de usuario | Los scripts de nota se ejecutan en el **servidor dentro de un sandbox aislado** (sin acceso a variables de entorno, red interna ni módulos del sistema). |
| Salida de red | En el cloud, un **cortafuegos de salida** impide que un flujo alcance servicios internos, otras organizaciones o direcciones privadas (anti-SSRF). |
| Integridad de facturación | El plan contratado se deriva del producto realmente comprado (verificado por firma), no de datos que controle el comprador. |
| Aislamiento de secretos | Ningún secreto se registra en logs ni se devuelve en respuestas de API. |

---

## Arquitectura y aislamiento multiinquilino (cloud)

- **Un contenedor por organización.** Cada organización (`org`) tiene su propio contenedor
  `flow-app` con su propio almacenamiento de flujos montado desde un directorio dedicado. No comparten
  proceso ni sistema de ficheros con otras organizaciones.
- **Gateway como único punto de entrada.** El acceso a cada organización pasa por
  `https://<org>.app.flowtest.es`. El gateway:
  - exige una **sesión válida** (cookie firmada) cuya organización coincide con el subdominio;
  - **inyecta la identidad del miembro** (`x-flow-user`) a partir de la sesión del servidor y
    **elimina siempre** cualquier cabecera de identidad que envíe el cliente;
  - corta el acceso si la suscripción no está activa.
- **Sin acceso cruzado entre organizaciones.** Una sesión de la organización A no puede alcanzar el
  contenedor de la B: la cookie está acotada al host y se verifica contra el subdominio.
- **Carpetas privadas por miembro.** Dentro de una organización, cada miembro tiene una carpeta
  privada; el servidor deniega el acceso a la de otro miembro. En el cloud, una petición **sin
  identidad de miembro no obtiene contenido privado** (falla cerrado).

**Recomendación de despliegue (defensa en profundidad):** aislar por red los contenedores de cada
organización entre sí (red por organización o reglas de firewall), de modo que solo el gateway pueda
hablar con cada contenedor. Ver [Despliegue seguro](#despliegue-seguro-operadores-del-cloud).

---

## Autenticación y sesiones

- **Contraseñas:** *scrypt* con sal única por usuario y comparación en tiempo constante. Nunca se
  registran ni se devuelven; el hash no sale de la base de datos.
- **Sesiones:** JWT firmado (HS256) en cookies `HttpOnly` + `SameSite=Lax` + `Secure` (en https).
  El *access token* es de vida corta; el *refresh token* rota en cada uso y, si se detecta reuso,
  se revocan todas las sesiones del usuario.
- **Fuerza bruta:** límite de intentos de login por IP/email; el panel de administración añade
  bloqueo de IP tras varios tokens inválidos. El cálculo de IP usa la entrada de confianza del
  proxy inverso, no un valor que el cliente pueda falsear.
- **Restablecer contraseña / verificar email:** enlaces con token aleatorio de alta entropía,
  caducidad corta y **un solo uso atómico**; restablecer la contraseña revoca todas las sesiones.
  Las peticiones de restablecimiento/alta están limitadas por IP (anti email-bombing).
- **2FA (TOTP)** opcional; **SSO OIDC** para el plan Business (el *issuer* se valida y se restringe
  a `https` con host público, anti-SSRF).

---

## Protección de datos y secretos

- **Credenciales de los flujos.** Se referencian como `{{secret:NOMBRE}}` y se guardan cifradas con
  **AES-256-GCM** por espacio de trabajo (clave por instalación, con permisos 0600). Solo se
  descifran **en el servidor** en el momento de ejecutar la petición; el navegador nunca ve el valor
  y el fichero `.flow.json` solo guarda el marcador. Compartir o versionar un flujo no filtra el
  secreto. Existen credenciales compartidas (del equipo) y **privadas por miembro**.
- **Claves de IA (BYOK).** Si conectas tu propia clave de Claude/OpenAI, se cifra con AES-256-GCM,
  nunca se envía al navegador y puede ser privada por miembro. Lo que se envía al modelo pasa por una
  **redacción** que elimina cabeceras de autorización, *tokens* y patrones de secreto.
- **Sin secretos en logs ni respuestas.** El servidor no registra las peticiones con los secretos ya
  descifrados, y el buffer de registro que consulta la consola de la app **redacta** valores tipo
  `Bearer …`, `sk-…`, contraseñas y *tokens*. Las respuestas de la API exponen, como mucho, nombres
  de credenciales o una pista de los últimos caracteres de una clave, nunca el valor.
- **Imagen Docker limpia.** La imagen pública **no** empaqueta flujos del proyecto: un contenedor
  nuevo arranca con un único flujo de bienvenida sin credenciales.
- **Enlaces compartidos.** «Compartir enlace» publica una **foto estática** del canvas (sin la
  lógica del flujo ni credenciales). La página se sirve saneada (sin scripts embebidos) y con una
  política `Content-Security-Policy` de aislamiento (*sandbox*).

---

## Ejecución de flujos y código de usuario

Un flujo (`.flow.json`) puede incluir **scripts JavaScript** en sus notas para calcular valores
(ids únicos, fechas, transformaciones de la respuesta). Esto es potente, así que su ejecución está
delimitada según dónde ocurre:

- **En el servidor** (ejecución móvil, *webhooks* y monitores, que corren dentro del contenedor de
  la organización): los scripts se ejecutan en un **sandbox aislado**. No tienen acceso a las
  variables de entorno del proceso, ni a la red, ni a módulos del sistema, ni al objeto de proceso;
  solo pueden calcular y devolver un valor, con un **tope de tiempo**. Un script no puede leer
  secretos del entorno, llamar a servicios internos ni ejecutar comandos.
- **En el navegador** (al pulsar «Run Flow» en el escritorio): los scripts corren en la sesión del
  propio usuario, con las mismas capacidades que cualquier página web que ese usuario abra. Esto es
  intencional: permite que un flujo dibuje ayudas visuales sobre el lienzo. **Modelo de confianza:**
  un `.flow.json` es, en la práctica, código; **ejecuta solo flujos en los que confíes**, igual que
  no ejecutarías un script de origen desconocido. Al compartir flujos con terceros, considera
  quitar los scripts o mantener una copia sin ellos.

---

## Salida de red controlada (anti-SSRF)

En el cloud, un flujo solo debe alcanzar APIs públicas de Internet, nunca la infraestructura interna.
Un **cortafuegos de salida** bloquea, en todos los caminos que hacen peticiones de red del lado del
servidor (proxy de peticiones, SQL, captura de pantalla, previsualización de web, sesión de
navegación en vivo, notificaciones de monitor), cualquier destino que sea:

- un servicio interno del cloud o el contenedor de **otra** organización;
- una **dirección IP privada o reservada** (incluye la comprobación de a qué resuelve un nombre de
  dominio, y la revalidación en cada **redirección**);
- la dirección de metadatos del proveedor de infraestructura.

En instalaciones **autoalojadas** esto no se aplica (apuntar a tu red local es el caso de uso
normal); es un control específico del entorno multiinquilino.

---

## Integridad de facturación

- El **webhook de pagos** (Lemon Squeezy) se verifica por **firma HMAC** sobre el cuerpo crudo,
  con comparación en tiempo constante y deduplicación anti-replay, **antes** de confiar en nada.
- El **plan** (Pro/Business) y el número de asientos se derivan del **producto realmente comprado**
  (dato que fija la pasarela de pago), nunca de parámetros que controle el comprador. No es posible
  pagar un plan y auto-asignarse uno superior.

---

## Prácticas de desarrollo

- Consultas a base de datos **parametrizadas** en su totalidad (sin construir SQL por concatenación).
- Rutas de fichero con protección contra *path traversal* (normalización, rechazo de `..`, ficheros
  ocultos y rutas absolutas; contención al directorio del proyecto) y de subida de recursos con
  nombres saneados.
- Límites de tamaño en los cuerpos de las peticiones y de concurrencia en las ejecuciones para
  resistir agotamiento de recursos.
- Dependencias mínimas y ancladas; la imagen no incluye componentes con licencias restrictivas.

---

## Endurecimiento de la versión 5.14.1

En agosto de 2026 realizamos una revisión de seguridad interna (dos auditorías independientes del
plano de control y del contenedor de aplicación) y aplicamos, entre otros, estos controles:

| Control | Detalle |
|---------|---------|
| Sandbox de scripts de nota (servidor) | Aislamiento total del entorno de ejecución en el servidor. |
| Cortafuegos de salida (SSRF) | Cobertura de proxy, SQL, captura, previsualización, navegación en vivo y notificaciones; revalidación en redirecciones. |
| Aislamiento de secretos en logs | Eliminación de registro de peticiones resueltas y redacción del buffer de consola. |
| Carpetas privadas «falla cerrado» | En el cloud, sin identidad de miembro no se sirve contenido privado. |
| Entitlement por producto comprado | El plan se deriva del pago verificado, no de datos del comprador. |
| Cookies `Secure`, XFF de confianza | Cookies solo por https; IP de cliente tomada del proxy de confianza. |
| Comparaciones en tiempo constante | Secretos compartidos y tokens comparados con `timingSafeEqual`. |
| Tokens de un solo uso atómicos | Restablecer/verificar/acceso consumidos sin condiciones de carrera. |
| Límites anti-DoS | Tope de tamaño de cuerpo y de ejecuciones concurrentes. |
| CSP y saneado en enlaces compartidos | Página de solo lectura, sin scripts, en aislamiento. |

---

## Despliegue seguro (operadores del cloud)

Recomendaciones para desplegar el modo multiinquilino:

1. **Segmentación de red entre inquilinos.** Colocar cada contenedor de organización en una red que
   comparta únicamente con el gateway, de modo que los contenedores no puedan alcanzarse entre sí ni
   a otros servicios internos por nombre o IP. Es la defensa principal frente a un contenedor
   comprometido, complementaria al cortafuegos de salida de la aplicación.
2. **`ADMIN_TOKEN` obligatorio** en el panel de administración (nunca vacío en producción).
3. **HTTPS de extremo a extremo** con HSTS; las cookies ya se marcan `Secure`.
4. **Secretos fuera de la imagen y del control de versiones**; rotación periódica de la clave de
   firma de sesión y del secreto del puente.
5. **Proxy del socket de Docker** restringido a operaciones de solo lectura, si el panel lo necesita.
6. **Correo en modo producción** (SMTP), nunca en modo consola.

---

## Divulgación responsable

Si crees haber encontrado un problema de seguridad:

- Escribe a **security@flowtest.es** con los detalles y los pasos para reproducirlo.
- Te confirmamos la recepción y trabajamos contigo en la corrección.
- Por favor, no divulgues públicamente el problema hasta que haya una corrección disponible.

Agradecemos el trabajo de la comunidad de seguridad y reconoceremos las aportaciones válidas.

---

## Registro de cambios de seguridad

- **5.14.1 (2026-09-08):** revisión de seguridad y endurecimiento (sandbox de scripts en el servidor,
  cortafuegos de salida anti-SSRF, aislamiento de secretos en logs, carpetas privadas «falla cerrado»,
  integridad de facturación, cookies `Secure`, comparaciones en tiempo constante, tokens de un solo
  uso atómicos, límites anti-DoS, CSP en enlaces compartidos).
- **5.6.0–5.8.0:** credenciales cifradas con ámbito (compartidas/privadas), carpetas privadas por
  miembro, tokens MCP por usuario.
- **5.1.0:** validación de licencia en línea con lista de revocación.
- **5.0.1:** la imagen deja de empaquetar flujos del proyecto (corrección de exposición de datos).
