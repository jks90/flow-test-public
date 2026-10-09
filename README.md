# Flow — Visual API Orchestrator (edición Docker)

**Flow** (flow-test) es una herramienta visual + CLI + MCP para componer, ejecutar y verificar
flujos de peticiones HTTP y consultas SQL encadenadas: importas comandos `curl`, conectas
nodos, extraes datos de las respuestas (JSONPath / columnas SQL) y los reutilizas en los
siguientes pasos con `{{variables}}`.

Este repositorio es **solo documentación**: todo lo necesario para usar Flow al máximo desde
la **imagen Docker oficial**, sin código fuente.

```
┌─────────────┐   comandos MCP    ┌──────────────────────────────────────┐
│  IA (Claude) ├──────────────────►  contenedor juankanh/flow-app        │
│  claude mcp  ◄──────────────────┤  · Web (canvas en :3001)             │
└─────────────┘  estado/resultados│  · CLI  (flow runner)                │
                                  │  · MCP  (/mcp, 39 tools)             │
       tú miras el canvas ────────►  · SQL  (postgres/mysql/oracle)      │
                                  └──────────────────────────────────────┘
```

## Arranque rápido

```bash
docker run -d \
  --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 \
  -p 1025:1025 \
  --name flow \
  juankanh/flow-app:5.24.1
```

- **Web**: http://localhost:9998 — el canvas visual.
- **CLI**: `docker exec flow node cli/run-flow.js --dir flows` — ejecuta baterías de flows.
- **Correo de prueba** 🆕 4.36: **Config → Correo** — SMTP en `localhost:1025` que recibe los correos de tus servicios y los muestra (buzones `@midominiotest.com`, sin cuentas reales); desde la 4.37 la IA lo maneja por MCP (`mail_*`).
- **MCP** (agentes IA): `claude mcp add --transport http flow-test http://localhost:9998/mcp`
  — la IA construye y ejecuta flows **en tu canvas, mientras lo ves**.

## 🗂️ Galería: arranca con 21 flows de verdad

¿Prefieres tocar antes que leer? Monta la **galería de ejemplos** como tu workspace y la app abre
con un tour guiado dentro del propio producto — cada ejemplo se ejecuta con un clic:

```bash
git clone https://github.com/jks90/flow-test-public.git
docker run -d \
  --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 --name flow \
  -v "$(pwd)/flow-test-public/examples/workspace:/app/flows" \
  juankanh/flow-app:latest
# abre http://localhost:9998 → icono «Proyecto» → «00 EMPIEZA AQUÍ.md»
```

Dentro: `aprende/` (variables, asserts, entornos, scripts, SQL, data-driven — un concepto por flow,
5 minutos cada uno), `funciones/` (monitor 24x7, webhook desde CI, correo OTP, puente a local),
`flowtest-por-dentro/` (nuestra propia plataforma probada con nuestra propia herramienta: alta,
login, **compra de licencia** y salud — el funnel real, documentado como flow) `paneles/` (los
vistosos para enseñar en una demo) y `scripts/` (🆕 5.14: señales de mercado, semáforo del tiempo y
verificación de pedidos calculados con **scripts JS dentro del flow**, gráficos Mermaid incluidos, y dos flows cuyos
scripts dibujan **botones, páginas y acciones** encima de FlowTest). Todos con asserts, verificados con el CLI antes de publicarse.
Detalle en [examples/README.md](examples/README.md). *(El tour en .md se abre desde la 5.5.0.)*

## Versiones de la imagen

| Versión | Qué trae |
|---------|----------|
| **5.24.1** (recomendada) | ☁️ el panel Proyecto espera a que despierte tu espacio del cloud en vez de mostrar «El contenedor está despertando» · 🛒 el marketplace de AgentOffice funciona con flow-test en Docker |
| **5.24.0** | 🔐 **Credenciales en claro bajo control**: al subir un flow al cloud o compartir un enlace, detecta contraseñas y tokens escritos tal cual (SQL, cabeceras, `-u`, cuerpo, URL) y los convierte con un clic a `{{secret:…}}` cifrados · ☁️ el panel Proyecto gestiona carpetas y ficheros del cloud como en local (y el MCP con `source:"cloud"`) |
| **5.23.0** | 🛒 **Marketplace FlowTest**: comparte con tu organización roles, skills y agentes de AgentOffice (con su memoria) o publica especificaciones en el marketplace público (sin memoria, moderado); flow-test hace de puerta segura hacia tu cuenta · 🧪 tests unitarios del servidor (`npm run test:unit`) |
| **5.22.0** | ⬆️ **Avisa de versiones nuevas** («⬆ x.y.z» junto a Config → novedades y cómo actualizar) y, si arrancas el contenedor con `-v /var/run/docker.sock:/var/run/docker.sock -e FLOW_SELF_UPDATE=on`, **se actualiza con un clic** (y vuelve sola a la anterior si la nueva no arranca) · ⚙️ **Configuración con menú lateral** · 🎤 Ctrl+Espacio habla con el Guía de AgentOffice — [11](docs/manual/11-docker.md#actualizaciones--522) |
| **5.21.3** | 🖥️ Abrir flow-test por la **IP de un servidor** (`http://192.168.x.x:9998`) ya no deja la página en blanco, y los botones «Copiar» funcionan por HTTP |
| **5.21.2** | 📬 Mejoras internas del correo de prueba (las tools MCP `mail_*` consultan los buzones remotos antes de leer) |
| **5.21.1** | 💾 Botón **Guardar** siempre visible en la barra y el diálogo «Guardar como…» ya no esconde su botón con muchas carpetas (reporte de usuario) — [01](https://github.com/jks90/flow-test-public/blob/main/docs/manual/01-pantalla-principal.md) |
| **5.21.0** | 🗂️ **Panel Proyecto a medida**: los **scripts** (`.js`, `.mjs`, `.ts`, `.py`, `.sh`, `.sql`) se ven y se abren como código, las carpetas **enlazadas** muestran a dónde apuntan, el panel se **ensancha** arrastrando su borde y puedes **ocultar carpetas** — [08](https://github.com/jks90/flow-test-public/blob/main/docs/manual/08-paneles.md#panel-proyecto-a-medida--521) |
| **5.20.2** | 🗑️ El borrado de documentos `.md`/`.pdf` va por su propia ruta (`DELETE /workspace/file`) y la pestaña solo se cierra si el documento era del disco local |
| **5.20.1** | 🗂️ **Ordenar el proyecto desde el panel**: crear carpetas, **arrastrar** flows, documentos y carpetas a otra carpeta, ✏️ mover o renombrar, borrar carpetas vacías y **borrar documentos `.md` y `.pdf`** (un enlace se quita sin tocar el original). Las pestañas abiertas siguen a su fichero — [08](https://github.com/jks90/flow-test-public/blob/main/docs/manual/08-paneles.md#ordenar-el-proyecto--520) |
| **5.19.4** | Incluye lo de las 5.19.0–5.19.3 (no publicadas sueltas): **asserts de errores esperados** (`404`/`4xx` en verde si se cumplen) y con `{{variables}}`, **contexto para el plugin Agentes** (AgentOffice sabe qué flow y nodo miras) y sus **Ajustes tipo Cmd** · 🗂️ El panel Proyecto y `flow_files_list` **ya no repiten los flows** de las copias de trabajo de los agentes (git worktrees dentro de un repo enlazado en el workspace) — [08](https://github.com/jks90/flow-test-public/blob/main/docs/manual/08-paneles.md#carpetas-enlazadas-y-copias-de-los-agentes--5194) |
| **5.19.0–5.19.3** | ✅ **Asserts de errores esperados**: un assert de status `404`/`4xx` deja el nodo en verde si se cumple, y los asserts aceptan `{{variables}}` ([02](https://github.com/jks90/flow-test-public/blob/main/docs/manual/02-nodo-request.md#errores-esperados-y-variables--519)). Plugin Agentes: recibe lo que estás viendo, panel de Ajustes propio y escucha por voz; `flow_open` recarga del disco y `flow_save` no pisa cambios ajenos |
| **5.18.0** | 🧩 **Plugins** (Config ▸ Plugins) con **AgentOffice** como primero: una oficina de agentes de IA que trabaja en tus repos, embebida en FlowTest (Pro). El workspace **sigue enlaces simbólicos**: una carpeta de proyecto puede reunir los `flows/` de varios repos — [08](https://github.com/jks90/flow-test-public/blob/main/docs/manual/08-paneles.md#plugins-y-agentoffice--518) |
| **5.17.0** | 🔔 **Vigilancia en el servidor**: al monitor programado se le añaden **reglas** sobre los datos que extrae el flow (`precio > 2700`, `senal cambia`, `estado contiene ERROR`…). El servidor lo ejecuta cada N min **sin navegador** y avisa por webhook (Slack, Telegram, ntfy, n8n…) **solo cuando una regla se dispara o se recupera**; `notifyUrl` admite `{{secret:X}}`; tool MCP `flow_monitor` (39 tools) — [08](https://github.com/jks90/flow-test-public/blob/main/docs/manual/08-paneles.md) |
| **5.16.0** | 🔎 **Notas a pantalla completa**: botón **Maximizar** junto a «Copiar» en la vista previa de cada nota (y «Maximizar nota» en el menú de la caja) — variables resueltas, enlaces vivos, tamaño de letra con Ctrl + rueda, Esc para cerrar ([04](https://github.com/jks90/flow-test-public/blob/main/docs/manual/04-notas-mermaid-capturas.md)). El panel Proyecto **recuerda** si estabas en 💻 Local o ☁️ Cloud al recargar |
| **5.15.1** | 🛡️ **Sandbox de los scripts de nota en el navegador**: un flow que no es tuyo (abierto de disco o del cloud, importado, de la galería) ejecuta sus scripts JS en un sandbox sin página, sin sesión y sin red; si un script necesita dibujar interfaz o llamar al MCP, la app **pregunta antes de ejecutarlo** y puedes marcar el flow como **de confianza** (chip 🔒 / 🛡️ en la barra; se recuerda por fichero en tu navegador). Imagen con parches de Alpine y sin `npm` en el contenedor |
| **5.15.0** | 🛡️ Primera versión del sandbox en el navegador y de los flows de confianza (la 5.15.1 pregunta antes de ejecutar en vez de tras un fallo silencioso) |
| **5.14.3** | 🔒 Dependencias sin vulnerabilidades conocidas (`npm audit` a cero) y tope de memoria de los runners del servidor (`FLOW_RUNNER_HEAP_MB`, 512 MB) |
| **5.14.2** | 🖼️ El visor de documentos `.md` del proyecto muestra **cualquier imagen del proyecto** (rutas relativas al documento), no solo las de `flows/assets/` |
| **5.14.1** | 🔒 **Seguridad**: los scripts JS de las notas se ejecutan en el servidor dentro de un **sandbox** (sin acceso a variables de entorno, red interna ni módulos del sistema) y, en el cloud, un **cortafuegos de salida** impide que un flow alcance servicios internos u otras organizaciones. Ver el informe de seguridad ([docs/seguridad.md](docs/seguridad.md)) |
| **5.14.0** | 📮 **Reportar un problema** desde la app (Config ▸ Reportar un problema, o 💬 en el móvil): error, idea o pregunta con captura del canvas, imagen pegada y datos técnicos opcionales (secretos tapados); bandeja del equipo, respuesta por email y «Mis reportes» en la app y en la cuenta. Pro/Business; sin red se guarda y se reintenta. El asistente de IA lleva la **skill de flows de serie** y un editor de instrucciones del proyecto; galería `scripts/` con 5 flows |
| **5.13.1** | 🤖 **IA también en el móvil** (pestaña «✨ IA» en `/m`: explica, revisa y propone sobre el flow; con el escritorio abierto edita ese canvas), **«Entrar con Anthropic»** (login OAuth sin claves), conexión directa desde el navegador, **Ctrl+V de capturas** en el chat y **«Guardar como…» ▸ ☁️ Cloud** |
| **5.13.0** | 🤖 **Asistente de IA**: un chat dentro de la app que construye, explica, prueba y arregla flows en tu canvas en directo con tu clave de Claude u OpenAI; ves cada tool con su input mientras se escribe, pide permiso antes de borrar o ejecutar, se para y se deshace por turno; comandos `/explain` `/test` `/fix` `/docs`, adjuntos (OpenAPI, HAR, PDF, capturas), coste por turno y tope mensual |
| **5.12.0** | 🔗 **Compartir enlace**: publica una foto del canvas en una URL pública que abre cualquiera sin la app ni cuenta — se ve como el PDF, en el navegador. Con el Docker vinculado se publica en tu espacio del cloud (`https://<org>.app.flowtest.es/share/…`); caducidad opcional, visitas, y gestión desde el modal, tu cuenta y el admin |
| **5.11.0** | 📱 **Versión móvil**: entra en `/m` desde el teléfono para **ejecutar flows y ver el resultado** paso a paso, y leerlos como documentación. App aparte y ligera, instalable en la pantalla de inicio |
| **5.10.0** | 🗂️ **El panel Proyecto apunta a tu cloud**: con el Docker vinculado, lista y guarda los flows de tu espacio del cloud; el selector 💻 Local / ☁️ Cloud arranca en el entorno que gobierna |
| **5.9.1** | 🔧 **Modo vinculado más limpio**: chip discreto «☁️ org ● plan» en la barra superior en vez de la franja de aviso, y el panel Proyecto / los chips PRO se actualizan al instante al conmutar licencia ↔ cuenta cloud |
| **5.9.0** | ☁️ **Sincroniza tus flows local ↔ cloud**: si tu Docker está vinculado a tu cuenta, trae o sube flows entre tu disco y tu espacio del cloud desde el botón ☁️ del panel Proyecto (respeta carpetas privadas; el token se queda en el servidor) |
| **5.8.1** | 🔁 **Interruptor licencia ↔ cuenta cloud**: si tienes clave **y** cuenta vinculada, un selector elige cuál gobierna la instalación (sin mezclar); pausar la vinculación vuelve a la licencia con un clic |
| **5.8.0** | 🤖 **Conecta tu IA al cloud** (token MCP por usuario) · 🐳 **vincula tu Docker a la cuenta** (hereda el plan de tu suscripción, sin claves) · credenciales con ámbito 🔒 privado / 👥 del equipo |
| **5.7.0** | 🔒 **Credenciales cifradas** (`{{secret:X}}`, AES-256 server-side) · **carpetas privadas por miembro** en el cloud (`privado/<tú>/`) con **guardar Compartido/🔒 Privado** · **Business = 1.000 flows** (10× Pro, no ilimitado) · fixes de colaboración (roster sin usuarios fantasma) |
| **5.6.0** | 🔒 Primera versión de las **credenciales cifradas** `{{secret:X}}` (las resuelve solo el servidor) — ampliadas en la 5.7.0 |
| **5.5.0** | 👥 **Colaboración en vivo (Business)**: presencia + edición del mismo flow sincronizada por nodos entre navegadores, con aviso de pisada; **Ctrl+Z/Ctrl+Y del canvas** (todos los planes); **galería montable** con tour ▶; panel Proyecto compacto estilo árbol; leer `.md` libre en trial; imagen **sin componentes GPL** |
| **5.4.0** | **Monitores programados**: el servidor ejecuta el flow cada N min sin navegador, historial de 100 runs y aviso por POST al fallar (modal «Automatización…») |
| **5.3.0** | **CI + specs**: asserts por nodo (CLI exit ≠ 0), entornos con nombre (`--env`), **webhook entrante** (`POST /hook/<token>`), data-driven (`--data`) e **import OpenAPI/Swagger** |
| **5.1.0** | **Prueba 14 días + licencia online** (validada y revocable); ver/editar nunca se bloquea |
| **5.0.1** | **Fix de seguridad**: la imagen ya no incluye flows del proyecto — solo el flow de bienvenida limpio. Usa siempre ≥ 5.0.1 |
| **5.0.0** | **Licencias por token (JWT)**: uso personal gratis; comercial/equipo/nube con licencia. **Config ▸ Licencia** activa la clave (RS256 offline) |
| **4.51.3** | **Licencia propietaria**: uso personal y no comercial gratis con todas las funciones (self-hosted); uso comercial / equipo / nube con licencia de pago. La imagen incluye el fichero `LICENSE`. |
| **4.51.2** | **Scripts «después» con `vars` y variables vivas en la pizarra** · 4.51.2: el PDF exportado incluye la imagen de fondo y las imágenes de la pizarra también desde Docker (4.51.1: el MCP conserva `when` al crear notas con `node_add_info`): cada script JS de una nota puede correr antes de la primera petición o **al terminar Run Flow con todas las extracciones** (`return (vars.wti - vars.wtiPrev).toFixed(2)`, semáforos `🔴/🟢`…); todo script recibe `vars`; los textos e imágenes de la pizarra resuelven `{{variables}}` al pintarse. Web / CLI / MCP (`when`) |
| **4.50.0** | **Imágenes y GIF en la pizarra**: sección «Imagen / GIF» del panel Pizarra (fichero subido al proyecto o URL) y **Ctrl+V de una imagen del portapapeles**; los GIF se animan; se mueven, redimensionan (proporción fija, Shift la libera), duplican y ordenan como cualquier dibujo, viajan en el `.flow.json` y el MCP las añade con `whiteboard_update` (`type: image`) |
| 4.23 – 4.41 | Scripts JS de las notas al Run Flow y 📌 fijar cajas, **Pizarra** estilo Excalidraw, `flows/` como proyecto (panel Proyecto + Ctrl+S), Config ▸ Vista (tamaño de nodos, modo compacto), campo `#` con celda y Alinear en cuadrícula, cabecera de cajas simplificada, activar/desactivar nodos, extracciones plegables con ↻, MCP con control total (33–37 tools, correo incluido), cadenas entre cualquier tipo de nodo, 10 fuentes en la pizarra, **Correo de prueba** (SMTP embebido `:1025` + mail.tm + código OTP), documentos `.md` como pestañas y árbol de carpetas — historial completo en el manual ([11](docs/manual/11-docker.md)) |
| 4.3.0 | Nodos de nota en **modo Mermaid** («Add Mermaid»): diagramas renderizados en vivo en el canvas, con interpolación `{{variable}}` — esquematiza qué llama a qué junto al propio flow |
| 4.2.0 | **MCP embebido** (`/mcp`, 18 tools: la IA construye/ejecuta flows en la web en directo) + puente AI↔web por SSE + typecheck del frontend saneado |
| 4.1.x | El CLI ejecuta **sqlNodes** (Postgres/MySQL/Oracle) con paridad con la web: perfiles de conexión, `{{variables}}` en queries, extracciones por columna. ⚠️ Desde aquí `--dir flows` toca BBDD reales (`--skip-sql-nodes` para el comportamiento antiguo) |
| 4.0.16 | Web + CLI HTTP: curl import, extracciones JSONPath, reports en `resumen/`, batch, cron, multi-pestaña |
| 4.4 – 4.22 | Nodo Web + modo **Live** (login real, capturas con sus llamadas HTTP), `flow-explore`, Chromium en la imagen, CAs corporativas en `/certs`, teclado directo en Live, checks LNA desactivados, nº de orden y alineado — historial completo en el manual |

```bash
docker pull juankanh/flow-app:5.24.1
```

## Documentación

| Guía | Contenido |
|------|-----------|
| [docs/manual/](docs/manual/README.md) | 📘 **Manual de uso completo con capturas de pantalla anotadas**: la pantalla principal, cada tipo de nodo, variables, **Vista (tamaño de nodos, modo compacto, separación)**, **proyecto (flows/ + Ctrl+S)**, **pizarra**, **correo de prueba (SMTP)**, guía de nodos, modo Live, flow-explore, paneles, CLI, MCP y Docker |
| [docs/instalacion-docker.md](docs/instalacion-docker.md) | Montar la imagen: puertos (web y SMTP de prueba), redes, variables de entorno, persistencia de flows, actualizar de versión, troubleshooting |
| [docs/cli.md](docs/cli.md) | El runner por terminal: flags, baterías, reports, exit codes para CI, nodos SQL y perfiles de conexión |
| [docs/agentoffice.md](docs/agentoffice.md) | 🏢 **AgentOffice**: instalar la oficina de agentes de IA en tu máquina, conectarla a FlowTest, proyectos con enlaces a tus repos, arranque automático, actualizaciones, avisos por Telegram, revisión automática y problemas frecuentes |
| [docs/mcp.md](docs/mcp.md) | Conectar una IA: Claude Code y Claude Desktop, las 39 tools (incl. correo de prueba y vigilancia), seguridad, flujos de trabajo típicos |
| [docs/flows-formato.md](docs/flows-formato.md) | El formato `.flow.json` a fondo: nodos HTTP y SQL, conexiones, variables y extracciones — para escribir flows a mano o con IA |
| [skills/flows/](skills/flows/SKILL.md) | **Skill para agentes IA** (Claude Code): cómo trabajar con Flow + [schema de autoría](skills/flows/references/flow-schema.md) — cópiala a tu proyecto |
| [examples/](examples/) | Flows de ejemplo listos para cargar o ejecutar |

## Ficheros listos para usar

| Fichero | Para qué |
|---------|----------|
| [bin/flow-run](bin/flow-run) | **El CLI en tu máquina sin código fuente**: wrapper que ejecuta el runner de la imagen montando tu directorio actual — flows y reports quedan en tu disco. `cp bin/flow-run ~/.local/bin/ && chmod +x ~/.local/bin/flow-run` |
| [docker-compose.example.yml](docker-compose.example.yml) | Compose de referencia: puertos, volumen de flows, env vars del MCP y red de tus APIs |
| [.mcp.json.example](.mcp.json.example) | Config MCP por proyecto para Claude Code (transporte HTTP) |
| [mcp-config.stdio.example.json](mcp-config.stdio.example.json) | Config para clientes MCP **solo-stdio** (vía [`mcp-remote`](https://www.npmjs.com/package/mcp-remote)) |

## Lo esencial en 4 recetas

**1. Ver la web y crear un flow a mano** → abre http://localhost:9998, pulsa *Add Request*,
pega un `curl`, conecta nodos y *Run Flow*.

**2. Ejecutar una batería por terminal (CI)**:

```bash
docker exec flow node cli/run-flow.js --dir flows --report-root /tmp/resumen
docker cp flow:/tmp/resumen ./resumen   # informe completo (report.md, debug por flow…)
```

**3. Que una IA construya el flow mientras lo miras**:

```bash
claude mcp add --transport http flow-test http://localhost:9998/mcp
# abre http://localhost:9998 en el navegador y dile a Claude:
#   «crea un flow que haga login en mi API y liste los usuarios con el token»
```

**4. Guardar lo que la IA construyó y ejecutarlo en CI**: la tool `flow_save` deja el
`.flow.json` en el contenedor; `docker exec flow node cli/run-flow.js --flow flows/mi-flow.flow.json`
lo ejecuta igual que la web.

**5. Trabajar sobre tu carpeta de flows como un proyecto** (4.25): monta `-v /tu/carpeta:/app/flows`
(o `-v /tu/carpeta:/data/flows -e FLOW_FLOWS_DIR=/data/flows`), abre el panel **Proyecto** en la web y
edita/guarda con **Ctrl+S** directamente en tus ficheros — versionables en git y ejecutables por el CLI
sin pasos intermedios.

## Requisitos

- Docker (la imagen es multi-arquitectura estándar `node:20-alpine`).
- Para el MCP: [Claude Code](https://claude.com/claude-code) u otro cliente MCP con
  transporte *Streamable HTTP*.

> La imagen se publica en Docker Hub como
> [`juankanh/flow-app`](https://hub.docker.com/r/juankanh/flow-app).

## Licencia y planes

> Componentes de terceros incluidos en la imagen y sus licencias: [LICENCIAS.md](LICENCIAS.md).

**flow-test es software propietario** ([`LICENSE`](LICENSE)):

- **Gratis** — uso **personal y no comercial** por una persona, con **todas las funciones**, self-hosted en tu máquina.
- **De pago** — el **uso comercial, en equipo o en la nube** requiere una **licencia de pago**; algunas funciones avanzadas se habilitan con una clave de licencia.

Para una licencia comercial, contacta con el autor a través de este repositorio.

## Activar una licencia

flow-test funciona **gratis para uso personal**. Para uso comercial, en equipo o en la nube necesitas una licencia (ver [`LICENSE`](LICENSE)). Una vez la tengas:

1. En la app: **Config ▸ Licencia** → pega tu clave → **Activar**. Se guarda en `flows/.license` y se verifica al momento.
2. O por entorno: arranca el contenedor con `-e FLOW_LICENSE="<tu-clave>"`.

La verificación es **offline** (tu clave nunca sale de tu máquina): el servidor comprueba la firma **RS256** con una clave pública empaquetada. Para conseguir una licencia comercial, contacta con el autor a través de este repositorio.
