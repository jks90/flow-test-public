# 🐳 11 · Docker y despliegue

## Ejecutar

```bash
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 -p 1025:1025 --name flow juankanh/flow-app:4.37.0
# web + API + MCP en http://localhost:9998 · 1025 = SMTP de prueba (Config ▸ Correo, 4.36)
```


## Construir

```bash
cd flow-test   # repo con código fuente
docker build -t juankanh/flow-app:<version> .
```

## Peculiaridades dentro del contenedor

- **localhost se reescribe** a `host.docker.internal` automáticamente (flows que apuntan a APIs de tu máquina siguen funcionando). Desactivable con `FLOW_REWRITE_LOCALHOST=false` o `--no-localhost-rewrite` en el CLI.
- El **CLI va dentro**: `docker exec flow node cli/run-flow.js --dir flows`.
- El **MCP va dentro**: `claude mcp add --transport http flow-test http://localhost:9998/mcp`.
- **`flows/` es el proyecto** (4.25): con `-v /tu/carpeta:/app/flows` (o `FLOW_FLOWS_DIR` para otra ruta), lo que guardas con Ctrl+S desde la web queda en tu disco, con tu usuario como dueño, listo para git y para el CLI.

## Chrome dentro de la imagen (desde la 4.5.0)

La imagen trae **Chromium + puppeteer-core**: el botón **Capturar** del nodo Captura, el
**modo Live** del nodo Web y el crawl de `flow-explore`
(`docker exec flow node cli/explore-web.js https://mi-web.com`) funcionan dentro del
contenedor sin instalar nada. La única excepción es `flow-explore --manual`/`--headful`,
que abre un Chrome visible → necesita pantalla, así que se ejecuta desde el repo local.

### Webs detrás de un login/SSO (Cloudflare Access, etc.)

El modo directo (iframe) no puede renderizar webs que responden `X-Frame-Options: DENY` —
eso es el navegador, no Flow. El modo Captura/Live sí, pero el Chrome del server necesita
tus credenciales; pásalas por entorno:

```yaml
    environment:
      # Cookie de sesión (p. ej. CF_Authorization de Cloudflare Access)
      - FLOW_CAPTURE_COOKIES=CF_Authorization=<valor>
      # O cabeceras (p. ej. un service token de Cloudflare Access)
      - FLOW_CAPTURE_HEADERS={"CF-Access-Client-Id":"<id>","CF-Access-Client-Secret":"<secreto>"}
```

También se aceptan por petición (`POST /capture` y `POST /capture-session/start` admiten
`{ "headers": {...}, "cookies": "k=v" }`). Las llamadas registradas en los flows siguen
enmascarando `authorization`/`cookie`: las credenciales nunca se escriben. Extras:
`FLOW_CHROME` (ruta del binario) y `FLOW_CHROME_ARGS` (flags).

**Escape hatch sin montar CAs** (ya desde la 4.5.0): `FLOW_CHROME_ARGS=--ignore-certificate-errors`
desactiva la validación TLS de **todas** las capturas — aceptable en un entorno de INT contra un
MITM corporativo conocido, no como default. Nota técnica: `update-ca-certificates` a secas NO
arregla Chromium (usa su BBDD NSS, no el store del sistema); por eso el entrypoint carga `/certs`
en ambos y loguea `[certs] CA corporativa cargada (system + NSS): <CN>`.

Desde la **4.6.0** suele bastar con loguearte una vez en el propio Live (OTP incluido): el
**perfil de Chromium persiste** en `flows/.chrome-profile` y el login sobrevive al cierre por
inactividad y a restarts. `FLOW_CHROME_PROFILE=off` lo desactiva; `FLOW_SESSION_IDLE_MS` ajusta el cierre por inactividad
(el perfil no se borra al expirar) y `POST /capture-session/logout` lo borra del todo — el
logout real del SSO. ⚠ Con el volumen
`./flows:/app/flows`, ese perfil (cookies vivas) queda en tu disco: no lo compartas.

### Dominios internos y Local Network Access (desde la 4.9.0)

Dos trampas de red corporativa que producían **fallos mudos**:

- **Local Network Access (Chromium ≥138)**: una web pública que llama a su API en la red
  privada (p. ej. `app.empresa.es` → `api.internal.empresa.es`) queda bloqueada **sin prompt**
  en headless: la app carga, el click llega, y el fetch muere con
  `net::ERR_BLOCKED_BY_LOCAL_NETWORK_ACCESS_CHECKS` (visible en la pestaña Network). Desde la
  4.9.0 esos checks van **desactivados** en el navegador de captura; `FLOW_CHROME_LNA_CHECKS=on`
  restaura el comportamiento estándar de Chromium.
- **DNS interno**: si el DNS corporativo vive en la VPN del host (resolv.conf apuntando a
  127.0.0.x), Docker cae a DNS público y los dominios internos no resuelven dentro del
  contenedor. Solución:

```yaml
    dns:
      - 10.0.0.53   # el DNS corporativo que resuelve *.internal.empresa.es
```

### CAs corporativas (desde la 4.6.0)

Si tu red intercepta TLS con una CA propia (Zscaler, WARP…), monta tus certificados:

```yaml
    volumes:
      - /usr/local/share/ca-certificates:/certs:ro
```

Al arrancar, el contenedor los carga en el store del sistema, en `NODE_EXTRA_CA_CERTS`
(peticiones del server/CLI y drivers SQL) y en la BBDD NSS de Chromium (captura/Live).
Monta con volumen, **no** con `docker cp`: los `.crt` que son symlinks (p. ej.
`managed-warp.crt → managed-warp.pem`) llegarían rotos — se ignoran con un aviso en los logs.

## Versiones

| Versión | Qué trae |
|---------|----------|
| 4.2.0 | MCP embebido (18 tools) |
| 4.3.0 | Nodo Mermaid |
| 4.4.0 | Documentar webs: nodo Captura, modo Live del nodo Web, CLI flow-explore, Auto Layout para todos los nodos, maximizar el nodo Web (Chrome solo desde el código fuente) |
| 4.5.0 | Chromium dentro de la imagen: Captura, Live y flow-explore (crawl) funcionan en Docker; credenciales para webs tras SSO con `FLOW_CAPTURE_COOKIES`/`FLOW_CAPTURE_HEADERS` |
| 4.6.0 | Perfil de Chromium persistente (el login del Live sobrevive a idle/restart), CAs corporativas desde `/certs`, fallos de navegación visibles en el canvas, sonda de iframes que nombra al host bloqueante y ofrece Live, barra de URL + pegar en Live |
| 4.7.0 | Teclado directo sobre la vista Live (Ctrl+V pega el OTP), errores de certificado explicados en pantalla, sesiones de perfil concurrentes, `FLOW_SESSION_IDLE_MS`, `POST /capture-session/logout` borra el perfil |
| 4.8.0 | Pestañas **Consola** y **Network** en el nodo Web: la sesión Live enseña los `console.*`/errores de la página y todo su tráfico (sin headers ni cuerpos) |
| 4.9.0 | Local Network Access desactivado en el navegador de captura (apps internas: el fetch a la API privada ya no muere mudo), aviso LNA/PNA en pantalla y pista de DNS corporativo |
| 4.12.1 | Webcam falsa y generación de videomock de DNI para QA; viewport Live configurable y sincronizado con la vista y los clics |
| 4.13 – 4.21 | Toolbar compacta (Add Node ▾, Layout ▾, modal Config), Expand All, cajas colapsadas compactas, Collapse All sin reordenar, anti-solapes, Vista previa/Texto en notas, minimapa oculto por defecto |
| 4.22.0 | Nº de orden por nodo + Alinear en fila/columna |
| 4.23.0 | Scripts JS de las notas ejecutados al Run Flow (web, MCP y CLI; `--skip-info-scripts`), 📌 fijar cajas, Variables usadas en cada caja, tool MCP `tab_close` |
| 4.24.0 | **Pizarra** estilo Excalidraw sobre el lienzo (`drawings` en el flow), barra lateral de iconos con un panel a la vez, **Guía de nodos**, Ocultar conectores, Maximizar Mermaid, tool MCP `node_add_info` (20 tools) |
| **4.37.0** | **El MCP controla el correo de prueba (37 tools)**: `mail_state`, `mail_server`, `mail_address`, `mail_messages` — arrancar el SMTP, buzones y verificar correos (OTP/enlace) sin pestaña web; guía de red cuando el servicio que envía corre en otra máquina (LAN, red Docker, túnel `ssh -R`) |
| 4.36.0 | **Correo de prueba (Config → Correo)**: SMTP embebido en el puerto `1025` (sin TLS, auth opcional) con buzones `@midominiotest.com` para capturar los correos que envían tus servicios sin cuentas reales; panel tipo Consola con HTML/texto/origen/adjuntos, búsqueda y filtro por buzón; persistencia en `flows/.mail/`; `GET /mail/messages/latest?to=` para verificarlo desde un flow. `-p 1025:1025`, `FLOW_MAIL_PORT`, `FLOW_MAIL_AUTOSTART`, `FLOW_MAIL_DOMAIN` |
| 4.35.0 | **10 fuentes para el texto de la pizarra** (8 manuscritas empaquetadas: Manuscrita, Boceto, **Excalidraw**/Excalifont, **Indie** Flower, Rotulador, Esbozo, Arquitecto, Nota + Normal y Código) con cuadrícula y vista previa; MCP `whiteboard_update` las acepta y mide los textos sin `w/h`. **Las filas bajan en bloque cuando una card crece** al ejecutarse (Vista ▸ «Evitar solapes al soltar o al crecer una caja») |
| 4.34.0 | **Cadenas entre cualquier tipo de nodo** (▶ sigue next/on_error/parallel hacia request, SQL, nota o web), **Run Flow ejecuta también los SQL**, **pausa por conector** (`delayMs`, popover en la flecha; web, CLI y MCP); fix caja SQL colapsada |
| 4.33.0 | **MCP con control total** (33 tools): `view_settings`, `global_variables_set`, `tab_rename`, `connection_update`, `sql_connections_list`, `canvas_layout separate`; `flow_state` con globales, perfiles SQL y vista |
| 4.32.0 | **Variable extractions** plegables con el diseño de «Variables usadas», estado `n/n ✓` / `n sin valor` y botón **↻ re-extraer** de la última respuesta sin repetir la llamada |
| 4.31.0 | **Activar / desactivar nodos** desde el menú del icono (se saltan en Run Flow, cadenas, cron, MCP y CLI; conexiones recorridas; `disabled` en el flow) |
| 4.30.0 | **Cabecera de las cajas simplificada** (icono-menú · plegar · estado · título · ▶; el resto en el menú del icono); fix MCP `node_add_sql` (`order`/`pinned`/`cell`) |
| 4.29.0 | Botón **Copiar** en la vista previa de las notas, **Layout ▸ Pin All / Unpin All**, **guía de nodos plegable** (MCP `canvas_layout pin_all|unpin_all`) |
| 4.28.2 | Campo **`#` = `columna,fila`** (1,1 arriba a la izquierda) y **Layout ▸ Alinear en cuadrícula**; `cell` en el flow y en el MCP (`canvas_layout grid`); 4.28.1: **Auto Layout respeta las celdas**; 4.28.2: **fix Run Flow** (flechas `next` desde notas/SQL/web ya no bloquean el request destino, ciclos al final, scripts de notas en el Historial) |
| 4.27.0 | **Config ▸ Vista**: tamaño de los nodos, **modo compacto** (caja con icono + título con estado + ▶, tooltip al pasar el ratón, clic abre la card) y **separación entre nodos** (mínima configurable, apartar vecinas al soltar, «Separar nodos solapados ahora») |
| 4.26.0 | **`flows/` como proyecto**: panel Proyecto, abrir/guardar con **Ctrl+S** en el fichero, «Guardar como…», detección de cambios en disco, ficheros con el dueño del host desde Docker; **enlaces `[[flow#nodo]]`** entre flows en las notas; `FLOW_FLOWS_DIR` para apuntar el proyecto a otra carpeta |
