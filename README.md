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
                                  │  · MCP  (/mcp, 37 tools)             │
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
  juankanh/flow-app:5.7.0
```

- **Web**: http://localhost:9998 — el canvas visual.
- **CLI**: `docker exec flow node cli/run-flow.js --dir flows` — ejecuta baterías de flows.
- **Correo de prueba** 🆕 4.36: **Config → Correo** — SMTP en `localhost:1025` que recibe los correos de tus servicios y los muestra (buzones `@midominiotest.com`, sin cuentas reales); desde la 4.37 la IA lo maneja por MCP (`mail_*`).
- **MCP** (agentes IA): `claude mcp add --transport http flow-test http://localhost:9998/mcp`
  — la IA construye y ejecuta flows **en tu canvas, mientras lo ves**.

## 🗂️ Galería: arranca con 16 flows de verdad

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
login, **compra de licencia** y salud — el funnel real, documentado como flow) y `paneles/` (los
vistosos para enseñar en una demo). Todos con asserts, verificados con el CLI antes de publicarse.
Detalle en [examples/README.md](examples/README.md). *(El tour en .md se abre desde la 5.5.0.)*

## Versiones de la imagen

| Versión | Qué trae |
|---------|----------|
| **5.7.0** (recomendada) | 🔒 **Credenciales cifradas** (`{{secret:X}}`, AES-256 server-side) · **carpetas privadas por miembro** en el cloud (`privado/<tú>/`) con **guardar Compartido/🔒 Privado** · **Business = 1.000 flows** (10× Pro, no ilimitado) · fixes de colaboración (roster sin usuarios fantasma) |
| **5.6.0** | 🔒 **Credenciales cifradas** (`{{secret:X}}` — AES-256 en el servidor, el navegador nunca ve el valor, flows compartibles sin filtrar nada) y **carpetas privadas por miembro** en el cloud (`privado/<tú>/`, invisible e intocable para el resto del equipo) |
| **5.5.0** | 👥 **Colaboración en vivo (Business)**: presencia + edición del mismo flow sincronizada por nodos entre navegadores, con aviso de pisada; **Ctrl+Z/Ctrl+Y del canvas** (todos los planes); **galería montable** con tour ▶; panel Proyecto compacto estilo árbol; leer `.md` libre en trial; imagen **sin componentes GPL** |
| **5.4.0** | **Monitores programados**: el servidor ejecuta el flow cada N min sin navegador, historial de 100 runs y aviso por POST al fallar (modal «Automatización…») |
| **5.3.0** | **CI + specs**: asserts por nodo (CLI exit ≠ 0), entornos con nombre (`--env`), **webhook entrante** (`POST /hook/<token>`), data-driven (`--data`) e **import OpenAPI/Swagger** |
| **5.1.0** | **Prueba 14 días + licencia online** (validada y revocable); ver/editar nunca se bloquea |
| **5.0.1** | **Fix de seguridad**: la imagen ya no incluye flows del proyecto — solo el flow de bienvenida limpio. Usa siempre ≥ 5.0.1 |
| **5.0.0** | **Licencias por token (JWT)**: uso personal gratis; comercial/equipo/nube con licencia. **Config ▸ Licencia** activa la clave (RS256 offline) |
| **4.51.3** | **Licencia propietaria**: uso personal y no comercial gratis con todas las funciones (self-hosted); uso comercial / equipo / nube con licencia de pago. La imagen incluye el fichero `LICENSE`. |
| **4.51.2** | **Scripts «después» con `vars` y variables vivas en la pizarra** · 4.51.2: el PDF exportado incluye la imagen de fondo y las imágenes de la pizarra también desde Docker (4.51.1: el MCP conserva `when` al crear notas con `node_add_info`): cada script JS de una nota puede correr antes de la primera petición o **al terminar Run Flow con todas las extracciones** (`return (vars.wti - vars.wtiPrev).toFixed(2)`, semáforos `🔴/🟢`…); todo script recibe `vars`; los textos e imágenes de la pizarra resuelven `{{variables}}` al pintarse. Web / CLI / MCP (`when`) |
| **4.50.0** | **Imágenes y GIF en la pizarra**: sección «Imagen / GIF» del panel Pizarra (fichero subido al proyecto o URL) y **Ctrl+V de una imagen del portapapeles**; los GIF se animan; se mueven, redimensionan (proporción fija, Shift la libera), duplican y ordenan como cualquier dibujo, viajan en el `.flow.json` y el MCP las añade con `whiteboard_update` (`type: image`) |
| **4.49.1** | **Imagen de fondo del flow** (4.49.1: también se ve en el minimapa): Config → Vista → «Fondo del flow» pone una imagen (un mapa del mundo, un plano, un diagrama…) detrás del lienzo — subida al proyecto o por URL, con posición, ancho (alto automático) y opacidad — en las mismas coordenadas que los nodos, así que puedes ordenar nodos y pizarra encima; se guarda en el flow (`settings.background`), sale en el PDF y el MCP la controla con `flow_background` |
| **4.48.0** | **La configuración vive en cada flow**: Config ▸ Vista (tamaño de nodos, modo compacto, separación…), Ocultar conectores, el estilo de la pizarra y el zoom/posición del lienzo se guardan dentro del `.flow.json` (`settings`) y cambian al cambiar de pestaña — cada flow se abre tal y como lo dejaste, también en otro navegador u ordenador; el zoom no marca la pestaña como modificada y el navegador solo guarda el valor por defecto para flows sin la suya |
| **4.47.0** | **Copiar y pegar en la pizarra, y mover el grupo desde dentro**: Ctrl+C/X/V (y botones Copiar/Pegar en el panel) duplican la selección donde quieras, con desplazamiento acumulado en cada pegado; y arrastrar desde el hueco **dentro** de una selección (p. ej. entre los trazos de una figura) mueve el grupo entero en vez de empezar un rectángulo nuevo y perder la selección |
| **4.46.0** | **Selección múltiple en la pizarra**: con la herramienta Seleccionar, arrastra sobre el fondo del lienzo para dibujar un rectángulo que selecciona todos los dibujos que toca (Shift acumula) — y mueve, cambia de estilo, duplica o borra el grupo entero de una vez |
| 4.41.0 | **Árbol de carpetas en el panel Proyecto**: las subcarpetas de `flows/` se anidan dentro de su padre (nombre corto e indentación por nivel, sin rutas planas repetidas tipo `flows/a/b/`); plegar una carpeta oculta **todo su subárbol**, su contador suma todos los ficheros que contiene y el plegado se recuerda |
| 4.40.0 | **Los `.md` se abren como pestañas**: los documentos Markdown del panel Proyecto ya no se abren en un modal sino como **una pestaña más** (chip **MD**) — el documento ocupa el área del canvas y convive con los flows; los enlaces `[[flow]]` y su ▶ funcionan igual; solo lectura (Ctrl+S avisa y nunca pisa el fichero); las filas del panel saben si el documento está abierto (ir a la pestaña / cerrar) |
| 4.39.0 | **▶ Ejecutar un flow desde un documento**: cada enlace `[[flow]]` del visor Markdown lleva adosado un botón verde **▶** que abre ese flow del proyecto y lanza su **Run Flow** completo (scripts de notas + nodos HTTP y SQL), cerrando el visor para ver la ejecución en el canvas — un `.md` con enlaces se convierte en un lanzador de baterías de prueba |
| 4.38.0 | **Documentos Markdown en el panel Proyecto**: los `.md` de `flows/` (tus notas y los informes que genera `flow-explore` en `assets/…`) se listan con su título y un chip **MD**, y se abren en un **visor de solo lectura** — encabezados, listas, tablas, bloques de código, desplegables `<details>`, imágenes de `assets/` servidas solas y **enlaces `[[flow]]` / `[[flow#Nodo]]`** que cierran el visor y llevan a ese flow en el canvas; botones ver fuente / recargar / maximizar. Nueva ruta `GET /workspace/file?path=…`; el MCP y la CLI siguen viendo solo `.flow.json` |
| 4.37.0 | **El MCP controla el correo de prueba (37 tools)**: `mail_state`, `mail_server` (start/stop/configure), `mail_address` (add/delete) y `mail_messages` (list/latest/read/delete/clear) — la IA arranca el SMTP, crea buzones y **verifica que tu servicio envió el correo** (`latest` → OTP/enlace) sin pestaña web. Docs de red para cuando el servicio que envía corre en **otra máquina** (LAN, red Docker, túnel `ssh -R`/ngrok, firewall) |
| 4.36.0 | **Correo de prueba (Config → Correo)**: un **servidor SMTP embebido** (puerto `1025`, sin TLS, auth opcional) para probar los correos que envían tus servicios **sin cuentas reales** — crea buzones `nombre@midominiotest.com` (o acepta cualquier destinatario y aparecen solos), apunta tu servicio a `host:1025` y los correos se ven al instante en un modal como la Consola: HTML renderizado, texto, origen, adjuntos, búsqueda y filtro por buzón; se guardan en `flows/.mail/` y sobreviven al reinicio. Desde un flow, `GET /mail/messages/latest?to=buzón` devuelve el último correo (404 si no hay) para **verificar que tu servicio lo envió** y extraer el código/enlace con JSONPath. Publica el puerto con `-p 1025:1025` |
| 4.35.0 | **10 fuentes para el texto de la pizarra** (8 manuscritas empaquetadas con la app, sin red ni fuentes del sistema — Manuscrita, Boceto, **Excalidraw** (Excalifont), **Indie** (Indie Flower), Rotulador, Esbozo, Arquitecto, Nota — más Normal y Código) elegidas desde una cuadrícula con vista previa; el MCP (`whiteboard_update`) las acepta y mide los textos sin `w/h`. **Las filas bajan en bloque cuando una card crece** (p. ej. al pintarse la respuesta tras ejecutarla), así no queda tapada por la fila siguiente y la fila sigue alineada (ajuste Vista ▸ «Evitar solapes al soltar o al crecer una caja»; la que crece y las 📌 no se mueven) |
| 4.34.0 | **Cadenas entre cualquier tipo de nodo**: el ▶ de una request, SQL, nota o web ejecuta el nodo y sigue sus flechas hacia nodos de cualquier tipo (HTTP → SQL, nota → request…): `next` si fue bien, `on_error` si falló, `parallel` siempre. **Run Flow ejecuta también los SQL** en el mismo orden topológico que las requests (paridad con el CLI). **Pausa por conector**: clic en el círculo de la flecha → comportamiento + pausa (1/2/5/10 s o a mano) antes de lanzar el destino; la respetan web, CLI y MCP (`delayMs`). Fix: la caja SQL colapsada con resultado volvía a pintar bien el resultado |
| 4.33.0 | **MCP con control total (33 tools)**: `view_settings` (la vista del usuario: tamaño de nodos, modo compacto, separación), `global_variables_set`, `tab_rename`, `connection_update` (behavior de una conexión), `sql_connections_list`, `canvas_layout separate`; `flow_state` devuelve además globales, perfiles SQL y vista. Skill `flows` y `flows-formato` documentan la colocación por celdas (`cell`), 📌 y `disabled` |
| 4.32.0 | **Variable extractions con el diseño de «Variables usadas»** en las cajas request: franja plegable (plegada por defecto) cuya cabecera dice cuántas extracciones resuelven en la última respuesta (`2/2 ✓` / `1 sin valor`), botón **Add** y botón **↻** que **vuelve a extraer** de la respuesta guardada sin repetir la llamada (corrige el JSONPath y recupera la variable); cada fila muestra el valor que resuelve |
| 4.31.0 | **Activar / desactivar nodos** desde el menú del icono de cada caja: un nodo desactivado se **salta** al ejecutar (Run Flow, cadenas, cron, MCP `flow_run`, CLI) pero sus conexiones se siguen recorriendo; se ve atenuado con un ⏻ rojo en el icono y «OFF» en la guía de nodos; el ▶ de la propia caja sí lo ejecuta. Campo `disabled` en el flow (`node_update` por MCP) |
| 4.30.0 | **Cabecera de las cajas simplificada**: en request, SQL, nota y web solo queda **icono del tipo (= menú) · plegar · ● estado · título · ▶**; fijar 📌, orden/celda `#`, conectar, cron, conexión SQL, copiar a otro flow, maximizar, eliminar y la información (duración, respuesta, URL, resultado) viven en el **menú del icono**. Las cajitas colapsadas son una sola fila. Fix MCP: `node_add_sql` ya respeta `order`/`pinned`/`cell` |
| 4.29.0 | **Copiar la vista previa de una nota** (botón Copiar: texto con las `{{variables}}` resueltas + HTML con enlaces), **Layout ▸ Pin All / Unpin All** (fija o libera todas las cajas del flow; MCP `canvas_layout pin_all|unpin_all`) y **Guía de nodos plegable** (grupos que se pliegan con un clic, Alt/doble clic deja solo uno, plegar/desplegar todo, recordado entre sesiones) |
| 4.28.2 | **Campo `#` con celda `columna,fila`** y **Layout ▸ Alinear en cuadrícula**: además del nº de orden, en el `#` de cada cabecera puedes escribir `1,1` (arriba a la izquierda), `3,4` (columna 3, fila 4)… y la acción de cuadrícula coloca cada caja en su celda — cada columna tan ancha como su caja más ancha, cada fila tan alta como la más alta; las cajas sin celda y las 📌 no se mueven. La celda se ve en la guía de nodos y en la caja compacta, se guarda en el flow (`cell`) y el MCP la acepta (`cell` en `node_add_*`/`node_update`, `canvas_layout mode=grid`). 4.28.1: **Auto Layout respeta las celdas** (cuadrícula primero; el grafo solo ordena las cajas sin celda, debajo). 4.28.2: **fix Run Flow** — una flecha ▶ `next` que salía de una nota/SQL/web dejaba su request destino sin ejecutar (en silencio); los ciclos se ejecutan al final; los scripts de las notas aparecen en el Historial como primer paso y la pestaña «Scripts JS» marca ✓/⚠ |
| 4.27.0 | **Config ▸ Vista**: **tamaño de los nodos** (30–150 %, sin tocar posiciones ni zoom), **modo compacto** Auto/Siempre/Nunca — al verse pequeños, cada nodo pasa a ser una **caja con el icono de su tipo**, el título encima con la redondita de estado, sus 4 conectores y un ▶ para ejecutarlo; tooltip con método/URL, query, estado y extracciones al pasar el ratón y **clic para abrir la card completa** —, y **separación entre nodos**: separación mínima, «Evitar solapes al soltar una caja» (las vecinas se apartan; la soltada y las 📌 no se mueven) y botón «Separar nodos solapados ahora». Ajustes guardados en el navegador, no en el flow |
| 4.26.0 | **`flows/` como proyecto**: el panel **Proyecto** lista los `.flow.json` del contenedor (`/app/flows`, móntalo con `-v ./flows:/app/flows`) por carpeta, marca cuál está abierto, cuál tiene cambios (●) y cuál cambió en disco; abres cualquiera en una pestaña y **Ctrl+S** (o el botón **Guardar**) escribe la pestaña en su fichero — se acabó exportar y sobrescribir a mano. Pestañas nuevas → «Guardar como…» (nombre + carpeta); si el fichero cambió en disco (CLI, MCP, git) avisa antes de pisarlo y permite recargarlo. Los ficheros conservan el dueño del host aunque el contenedor corra como root. `FLOW_FLOWS_DIR` apunta el proyecto a cualquier carpeta (el panel muestra la ruta real). **Enlaces entre flows** en las notas: `[[otro-flow]]`, `[[otro-flow|texto]]`, `[[otro-flow#Nombre de nodo]]` abren ese flow y centran el nodo; las URLs http(s) son clicables |
| 4.24.0 | **Pizarra estilo Excalidraw** sobre el lienzo (lápiz, línea, flecha, rectángulo, elipse, texto, borrador; trazo boceto o limpio, colores, relleno, deshacer/rehacer; los dibujos se guardan en el flow como `drawings` y se ven en el minimapa), **barra lateral de iconos** con un panel visible a la vez (Variables, Global, SQL, GitHub, Pizarra, Guía de nodos), **Guía de nodos** (índice por tipo/nombre con foco al clic) y **Ocultar conectores** en el menú Layout, botón **Maximizar** en la vista previa Mermaid (pantalla completa con zoom), tool MCP `node_add_info` (notas/diagramas/capturas desde la IA; 20 tools) |
| 4.23.0 | Los **scripts JS de las notas se ejecutan al pulsar Run Flow** (web, MCP `flow_run` y CLI) y sus valores entran como variables antes de la primera petición (`--skip-info-scripts` en CLI); **📌 fijar** cajas (ningún relayout ni arrastre las mueve); franja **Variables usadas** en cada caja request/SQL con edición in situ; tool MCP `tab_close` |
| 4.3.0 | Nodos de nota en **modo Mermaid** («Add Mermaid»): diagramas renderizados en vivo en el canvas, con interpolación `{{variable}}` — esquematiza qué llama a qué junto al propio flow |
| 4.2.0 | **MCP embebido** (`/mcp`, 18 tools: la IA construye/ejecuta flows en la web en directo) + puente AI↔web por SSE + typecheck del frontend saneado |
| 4.1.x | El CLI ejecuta **sqlNodes** (Postgres/MySQL/Oracle) con paridad con la web: perfiles de conexión, `{{variables}}` en queries, extracciones por columna. ⚠️ Desde aquí `--dir flows` toca BBDD reales (`--skip-sql-nodes` para el comportamiento antiguo) |
| 4.0.16 | Web + CLI HTTP: curl import, extracciones JSONPath, reports en `resumen/`, batch, cron, multi-pestaña |
| 4.4 – 4.22 | Nodo Web + modo **Live** (login real, capturas con sus llamadas HTTP), `flow-explore`, Chromium en la imagen, CAs corporativas en `/certs`, teclado directo en Live, checks LNA desactivados, nº de orden y alineado — historial completo en el manual |

```bash
docker pull juankanh/flow-app:5.7.0
```

## Documentación

| Guía | Contenido |
|------|-----------|
| [docs/manual/](docs/manual/README.md) | 📘 **Manual de uso completo con capturas de pantalla anotadas**: la pantalla principal, cada tipo de nodo, variables, **Vista (tamaño de nodos, modo compacto, separación)**, **proyecto (flows/ + Ctrl+S)**, **pizarra**, **correo de prueba (SMTP)**, guía de nodos, modo Live, flow-explore, paneles, CLI, MCP y Docker |
| [docs/instalacion-docker.md](docs/instalacion-docker.md) | Montar la imagen: puertos (web y SMTP de prueba), redes, variables de entorno, persistencia de flows, actualizar de versión, troubleshooting |
| [docs/cli.md](docs/cli.md) | El runner por terminal: flags, baterías, reports, exit codes para CI, nodos SQL y perfiles de conexión |
| [docs/mcp.md](docs/mcp.md) | Conectar una IA: Claude Code y Claude Desktop, las 37 tools (incl. correo de prueba), seguridad, flujos de trabajo típicos |
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
