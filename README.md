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
                                  │  · MCP  (/mcp, 28 tools)             │
       tú miras el canvas ────────►  · SQL  (postgres/mysql/oracle)      │
                                  └──────────────────────────────────────┘
```

## Arranque rápido

```bash
docker run -d \
  --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 \
  --name flow \
  juankanh/flow-app:4.28.2
```

- **Web**: http://localhost:9998 — el canvas visual.
- **CLI**: `docker exec flow node cli/run-flow.js --dir flows` — ejecuta baterías de flows.
- **MCP** (agentes IA): `claude mcp add --transport http flow-test http://localhost:9998/mcp`
  — la IA construye y ejecuta flows **en tu canvas, mientras lo ves**.

## Versiones de la imagen

| Versión | Qué trae |
|---------|----------|
| **4.28.2** (recomendada) | **Campo `#` con celda `columna,fila`** y **Layout ▸ Alinear en cuadrícula**: además del nº de orden, en el `#` de cada cabecera puedes escribir `1,1` (arriba a la izquierda), `3,4` (columna 3, fila 4)… y la acción de cuadrícula coloca cada caja en su celda — cada columna tan ancha como su caja más ancha, cada fila tan alta como la más alta; las cajas sin celda y las 📌 no se mueven. La celda se ve en la guía de nodos y en la caja compacta, se guarda en el flow (`cell`) y el MCP la acepta (`cell` en `node_add_*`/`node_update`, `canvas_layout mode=grid`). 4.28.1: **Auto Layout respeta las celdas** (cuadrícula primero; el grafo solo ordena las cajas sin celda, debajo). 4.28.2: **fix Run Flow** — una flecha ▶ `next` que salía de una nota/SQL/web dejaba su request destino sin ejecutar (en silencio); los ciclos se ejecutan al final; los scripts de las notas aparecen en el Historial como primer paso y la pestaña «Scripts JS» marca ✓/⚠ |
| 4.27.0 | **Config ▸ Vista**: **tamaño de los nodos** (30–150 %, sin tocar posiciones ni zoom), **modo compacto** Auto/Siempre/Nunca — al verse pequeños, cada nodo pasa a ser una **caja con el icono de su tipo**, el título encima con la redondita de estado, sus 4 conectores y un ▶ para ejecutarlo; tooltip con método/URL, query, estado y extracciones al pasar el ratón y **clic para abrir la card completa** —, y **separación entre nodos**: separación mínima, «Evitar solapes al soltar una caja» (las vecinas se apartan; la soltada y las 📌 no se mueven) y botón «Separar nodos solapados ahora». Ajustes guardados en el navegador, no en el flow |
| 4.26.0 | **`flows/` como proyecto**: el panel **Proyecto** lista los `.flow.json` del contenedor (`/app/flows`, móntalo con `-v ./flows:/app/flows`) por carpeta, marca cuál está abierto, cuál tiene cambios (●) y cuál cambió en disco; abres cualquiera en una pestaña y **Ctrl+S** (o el botón **Guardar**) escribe la pestaña en su fichero — se acabó exportar y sobrescribir a mano. Pestañas nuevas → «Guardar como…» (nombre + carpeta); si el fichero cambió en disco (CLI, MCP, git) avisa antes de pisarlo y permite recargarlo. Los ficheros conservan el dueño del host aunque el contenedor corra como root. `FLOW_FLOWS_DIR` apunta el proyecto a cualquier carpeta (el panel muestra la ruta real). **Enlaces entre flows** en las notas: `[[otro-flow]]`, `[[otro-flow|texto]]`, `[[otro-flow#Nombre de nodo]]` abren ese flow y centran el nodo; las URLs http(s) son clicables |
| 4.24.0 | **Pizarra estilo Excalidraw** sobre el lienzo (lápiz, línea, flecha, rectángulo, elipse, texto, borrador; trazo boceto o limpio, colores, relleno, deshacer/rehacer; los dibujos se guardan en el flow como `drawings` y se ven en el minimapa), **barra lateral de iconos** con un panel visible a la vez (Variables, Global, SQL, GitHub, Pizarra, Guía de nodos), **Guía de nodos** (índice por tipo/nombre con foco al clic) y **Ocultar conectores** en el menú Layout, botón **Maximizar** en la vista previa Mermaid (pantalla completa con zoom), tool MCP `node_add_info` (notas/diagramas/capturas desde la IA; 20 tools) |
| 4.23.0 | Los **scripts JS de las notas se ejecutan al pulsar Run Flow** (web, MCP `flow_run` y CLI) y sus valores entran como variables antes de la primera petición (`--skip-info-scripts` en CLI); **📌 fijar** cajas (ningún relayout ni arrastre las mueve); franja **Variables usadas** en cada caja request/SQL con edición in situ; tool MCP `tab_close` |
| 4.22.0 | **Nº de orden** por nodo (campo `#` en la cabecera, persistido en el flow) y **Alinear en fila / en columna** en el menú Layout |
| 4.13 – 4.21 | **Toolbar compacta**: desplegables Add Node y Layout, modal Config (Historial, Consola, SQL Conns, GitHub, Variables, Global), Expand All; **cajas colapsadas compactas** (ancho por tipo, botones bajo el título); Collapse All acerca las cajas sin reordenar y Expand All restaura posiciones; anti-solapes al expandir; conmutador Vista previa / Texto en las notas; minimapa oculto por defecto |
| 4.12.1 | **Videomock de DNI para QA**: Chromium puede usar un MJPEG/Y4M montado como webcam mediante `FLOW_FAKE_WEBCAM`; `POST /videomock/dni` regenera el DNI con datos del titular usando plantillas privadas. `FLOW_VIEWPORT` configura la sesión Live y el frontend usa sus dimensiones reales para escalar la vista y los clics |
| 4.9.0 | **Apps internas sin fallos mudos**: los checks de Local Network Access de Chromium vienen desactivados en el navegador de captura (una web pública ya puede llamar a su API en red privada; `FLOW_CHROME_LNA_CHECKS=on` los restaura), aviso en pantalla si aparece un bloqueo LNA/PNA y pista de DNS corporativo en `ERR_NAME_NOT_RESOLVED` |
| 4.8.0 | **Pestañas Consola y Network en el nodo Web**: con la sesión Live abierta ves los `console.*`/errores de la página y todo su tráfico (método/URL/status/duración, fallos `net::ERR_*` incluidos — nunca headers ni cuerpos), con contadores en vivo y botón Limpiar |
| 4.7.0 | **Teclear directo sobre la vista Live** (imprimibles, Enter/Tab/flechas, Ctrl+V pega — OTPs y contraseñas sin la caja aparte), errores `ERR_CERT_*` explicados con su pista (CA en `/certs` o `FLOW_CHROME_ARGS=--ignore-certificate-errors`), sesiones de perfil concurrentes (pestañas del mismo Chromium), `FLOW_SESSION_IDLE_MS` y `POST /capture-session/logout` que borra el perfil (logout real del SSO) |
| 4.6.0 | **Login una vez y esquemas honestos**: perfil de Chromium persistente para el modo Live (el OTP/SSO sobrevive al idle y a restarts), CAs corporativas cargadas desde `/certs` al arrancar, fallos de navegación visibles en el canvas (`navigation: {ok, errorCode, finalUrl}`), sonda de iframes que nombra al host bloqueante real (el SSO tras el redirect) y ofrece Live, barra de URL y pegar del portapapeles en Live |
| 4.5.0 | **Toda la funcionalidad web dentro del contenedor**: la imagen trae Chromium — el nodo Captura, el modo Live y el crawl de `flow-explore` funcionan en Docker sin instalar nada. Webs tras SSO (p. ej. Cloudflare Access): credenciales por `FLOW_CAPTURE_COOKIES` / `FLOW_CAPTURE_HEADERS` o por petición |
| 4.4.0 | **Documentar webs**: nodo Captura (screenshot de una web desde el canvas), modo **Live** del nodo Web (navegas la web embebida y cada captura crea la documentación con sus llamadas HTTP) y CLI `flow-explore`. En esta versión las funciones que usan Chrome corrían solo desde el código fuente. Auto Layout para todos los nodos y maximizar el nodo Web |
| 4.3.0 | Nodos de nota en **modo Mermaid** («Add Mermaid»): diagramas renderizados en vivo en el canvas, con interpolación `{{variable}}` — esquematiza qué llama a qué junto al propio flow |
| 4.2.0 | **MCP embebido** (`/mcp`, 18 tools: la IA construye/ejecuta flows en la web en directo) + puente AI↔web por SSE + typecheck del frontend saneado |
| 4.1.x | El CLI ejecuta **sqlNodes** (Postgres/MySQL/Oracle) con paridad con la web: perfiles de conexión, `{{variables}}` en queries, extracciones por columna. ⚠️ Desde aquí `--dir flows` toca BBDD reales (`--skip-sql-nodes` para el comportamiento antiguo) |
| 4.0.16 | Web + CLI HTTP: curl import, extracciones JSONPath, reports en `resumen/`, batch, cron, multi-pestaña |

```bash
docker pull juankanh/flow-app:4.28.2
```

## Documentación

| Guía | Contenido |
|------|-----------|
| [docs/manual/](docs/manual/README.md) | 📘 **Manual de uso completo con capturas de pantalla anotadas**: la pantalla principal, cada tipo de nodo, variables, **Vista (tamaño de nodos, modo compacto, separación)**, **proyecto (flows/ + Ctrl+S)**, **pizarra**, guía de nodos, modo Live, flow-explore, paneles, CLI, MCP y Docker |
| [docs/instalacion-docker.md](docs/instalacion-docker.md) | Montar la imagen: puertos, redes, variables de entorno, persistencia de flows, actualizar de versión, troubleshooting |
| [docs/cli.md](docs/cli.md) | El runner por terminal: flags, baterías, reports, exit codes para CI, nodos SQL y perfiles de conexión |
| [docs/mcp.md](docs/mcp.md) | Conectar una IA: Claude Code y Claude Desktop, las 28 tools, seguridad, flujos de trabajo típicos |
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
