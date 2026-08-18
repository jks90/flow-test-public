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

## Versiones

| Versión | Qué trae |
|---------|----------|
| 4.2.0 | MCP embebido (18 tools) |
| 4.3.0 | Nodo Mermaid |
| 4.4.0 | Documentar webs: nodo Captura, modo Live del nodo Web, CLI flow-explore, Auto Layout para todos los nodos, maximizar el nodo Web (Chrome solo desde el código fuente) |
| **4.5.0** | Chromium dentro de la imagen 🆕: Captura, Live y flow-explore (crawl) funcionan en Docker; credenciales para webs tras SSO con `FLOW_CAPTURE_COOKIES`/`FLOW_CAPTURE_HEADERS` |
