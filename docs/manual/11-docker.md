# 🐳 11 · Docker y despliegue

## Ejecutar

```bash
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 --name flow juankanh/flow-app:4.4.0
# web + API + MCP, todo en http://localhost:9998
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

Desde la **4.6.0** suele bastar con loguearte una vez en el propio Live (OTP incluido): el
**perfil de Chromium persiste** en `flows/.chrome-profile` y el login sobrevive al cierre por
inactividad y a restarts. `FLOW_CHROME_PROFILE=off` lo desactiva. ⚠ Con el volumen
`./flows:/app/flows`, ese perfil (cookies vivas) queda en tu disco: no lo compartas.

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
| **4.6.0** | Perfil de Chromium persistente 🆕 (el login del Live sobrevive a idle/restart), CAs corporativas desde `/certs`, fallos de navegación visibles en el canvas, sonda de iframes que nombra al host bloqueante y ofrece Live, barra de URL + pegar en Live |
