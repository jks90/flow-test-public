# 📘 Flow-test — Manual de uso

Manual completo del **Flow Builder** (flow-test): la herramienta visual para componer, ejecutar y **documentar** flujos de peticiones HTTP, SQL y navegación web. Todas las capturas son de la aplicación real.

> [!NOTE]
> Manual de **flow-test v4.4.0** (imagen `juankanh/flow-app:4.4.0`). Lo marcado con 🆕 necesita un **Chrome local**: funciona desde el código fuente (`npm run dev`), no dentro del contenedor.

## Índice

1. [01 La pantalla principal](01-pantalla-principal.md) — toolbar, pestañas, canvas, minimapa, split
2. [02 El nodo Request](02-nodo-request.md) — las cajitas curl: ejecutar, respuesta, extracciones
3. [03 Conexiones y variables](03-conexiones-y-variables.md) — encadenar nodos y mover datos entre ellos
4. [04 Notas, Mermaid y Capturas](04-notas-mermaid-capturas.md) — documentación dentro del canvas 🆕
5. [05 El nodo SQL](05-nodo-sql.md) — consultas Postgres/MySQL/Oracle en el flujo
6. [06 El nodo Web y el modo Live](06-nodo-web-live.md) — navegar una web embebida y capturar pasos 🆕
7. [07 Documentar una web — flow-explore](07-flow-explore.md) — el CLI que genera documentación sola 🆕
8. [08 Paneles y utilidades](08-paneles.md) — historial, consola, batch, GitHub, export
9. [09 El CLI flow-run](09-cli-flow-run.md) — ejecutar flows desde terminal / CI
10. [10 El MCP embebido](10-mcp.md) — que una IA construya flows en tu canvas
11. [11 Docker y despliegue](11-docker.md)

## Arrancar la aplicación

**En desarrollo** (con todas las funciones, incluidas las 🆕):

```bash
git clone https://github.com/jks90/flow-test && cd flow-test && npm install
npm run dev        # web en http://localhost:5173 + server en :3001
```

**Con Docker** (todo salvo las funciones 🆕 que necesitan Chrome local):

```bash
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 --name flow juankanh/flow-app:4.4.0
# web en http://localhost:9998
```

## La idea en 30 segundos

- Cada **cajita** del canvas es un paso: una petición HTTP (curl), una consulta SQL, una nota, una captura de pantalla o una web.
- Las **flechas** definen el orden de ejecución; las **variables** `{{asi}}` mueven datos de una cajita a la siguiente.
- Un flujo se guarda como un `.flow.json` que se puede versionar, compartir, y ejecutar **sin abrir el navegador** con el CLI `flow-run`.
- 🆕 Además de probar APIs, ahora sirve para **documentarlas**: screenshots de cada pantalla de una web con sus llamadas HTTP debajo, generados navegando (modo Live) o con el explorador automático (`flow-explore`).
