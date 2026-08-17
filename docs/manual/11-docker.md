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

## Qué NO funciona en Docker (necesita Chrome local)

| Función | Alternativa |
|---------|-------------|
| Botón **Capturar** del nodo Captura 🆕 | Usarlo contra el dev (`npm run dev`) |
| **Modo Live** del nodo Web 🆕 | Ídem |
| CLI **flow-explore** 🆕 | Ejecutarlo desde el repo (`npm install` + Chrome del sistema) |

El resto (canvas, requests, SQL, CLI flow-run, MCP, notas/mermaid, nodos Captura mostrando imágenes ya generadas) funciona igual en ambos.

## Versiones

| Versión | Qué trae |
|---------|----------|
| 4.2.0 | MCP embebido (18 tools) |
| 4.3.0 | Nodo Mermaid |
| **4.4.0** | Documentar webs 🆕: nodo Captura, modo Live del nodo Web, CLI flow-explore, Auto Layout para todos los nodos, maximizar el nodo Web |
