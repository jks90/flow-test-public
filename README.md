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
  juankanh/flow-app:5.29.0
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
| **5.29.0** (recomendada) | 🗒️ **Sticky notes en la pizarra**: notas adhesivas con la fecha de creación y un color a elegir entre 6 pasteles (herramienta `N`, doble clic para editar) |
| **5.28.0** | 📊 **Ver ficheros CSV y TSV**: el panel Proyecto abre los `.csv`/`.tsv` de tu carpeta como tabla (cabecera fija, de 1000 en 1000 filas) o como texto, con separador `,`, `;` o tabulador |
| **5.27.1** | ⚙️ **Ajustes de AgentOffice más cómodos**: el panel ocupa casi toda la ventana, agrupa las secciones (Empresa, Proyecto, Guía y voz, Seguridad), marca las que tienen cambios y deja **Guardar** siempre visible; guardar no cierra el panel y, si cierras con cambios, pregunta. Arregla que «Proyecto» saliera pegada debajo de todas |
| **5.27.0** | 🔑 **Conexión del asistente de IA clara**: Claude y OpenAI muestran el mismo estado (sin configurar, conectado, error, caducado) y el modo activo, con **Probar conexión** (dice el error concreto: clave inválida, caducada, sin red, sin cuota…), **Reconfigurar** y **Desconectar**; el panel de AgentOffice sigue abierto al recargar y el CLI envía el `Content-Type` de `-F 'campo=…;type=…'` |
| **5.26.0** | 🔌 **Tantas ventanas como quieras**: las conexiones en vivo (puente MCP, colaboración y AgentOffice) van por WebSocket, así que con varias ventanas de FlowTest abiertas el Guía y los botones (p. ej. contestar una 🛡) ya no se quedan colgados; si un proxy no admite WebSocket, vuelve sola a la conexión clásica |
| **5.25.1** | 🔌 con varias pestañas de flow-test abiertas, el Guía de AgentOffice ya no se queda en «trabajando…» (las pestañas en segundo plano liberan conexiones) y el proxy de Agentes corta limpio si AgentOffice se reinicia |
| **5.25.0** | 🧩 **Plugins que se instalan y actualizan desde la app**: Config ▸ Plugins detecta si AgentOffice no está en tu PC (guía con los comandos listos para copiar) y, si va por detrás, lo actualiza con un clic · 🛒 Marketplace de AgentOffice con pestaña 🔒 Privado (banquillo y roles) y roles de serie publicables |
| **5.24.1** | ☁️ el panel Proyecto espera a que despierte tu espacio del cloud en vez de mostrar «El contenedor está despertando» · 🛒 el marketplace de AgentOffice funciona con flow-test en Docker |
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

Las versiones anteriores (5.12 y antes, desde la 4.x) están en la [tabla completa del manual](docs/manual/11-docker.md#versiones).

```bash
docker pull juankanh/flow-app:5.29.0
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
