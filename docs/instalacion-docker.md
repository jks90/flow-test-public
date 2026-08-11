# Instalación con Docker

Todo Flow vive en una sola imagen: web, CLI, motor SQL y servidor MCP.

## Arranque básico

```bash
docker run -d \
  --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 \
  --name flow \
  juankanh/flow-app:4.2.0
```

| Pieza | Dónde queda |
|-------|-------------|
| Web (canvas) | http://localhost:9998 |
| CLI | `docker exec flow node cli/run-flow.js …` |
| MCP | `http://localhost:9998/mcp` |
| Flows incluidos | `/app/flows` dentro del contenedor |

`--add-host=host.docker.internal:host-gateway` es lo que permite que los flows que apuntan a
`http://localhost:PUERTO` lleguen a los servicios de **tu máquina** (ver «Reescritura de
localhost» abajo). En Docker Desktop (Mac/Windows) ese alias ya existe y puedes omitirlo.

## Con tus APIs en una red de Docker

Si tu API corre en otro contenedor, mete a Flow en la misma red y usa el nombre del servicio
en los curl (`http://mi-api:8080/...`):

```bash
docker run -d \
  --network mi_red \
  --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 \
  --name flow \
  juankanh/flow-app:4.2.0
```

## Variables de entorno

| Variable | Por defecto | Qué hace |
|----------|-------------|----------|
| `PORT` | `3001` | Puerto interno del servidor |
| `FLOW_REWRITE_LOCALHOST` | `true` en la imagen | Reescribe `localhost`/`127.0.0.1` → `host.docker.internal` en las peticiones de los flows (web y CLI). Ponla a `false` si tu objetivo es un servicio **dentro** del propio contenedor |
| `FLOW_MCP_TOKEN` | *(sin definir)* | Si la defines, el endpoint `/mcp` exige `Authorization: Bearer <token>` — necesario para usar el MCP desde **otra máquina** de forma segura |
| `FLOW_MCP_ALLOW_LAN` | *(sin definir)* | `true` abre `/mcp` a la LAN **sin** auth (solo redes de confianza). Por defecto `/mcp` solo acepta clientes locales |

Ejemplo con MCP protegido por token:

```bash
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 -e FLOW_MCP_TOKEN=mi-secreto \
  --name flow juankanh/flow-app:4.2.0
```

## Persistir tus flows

Los `.flow.json` guardados desde el MCP (`flow_save`) o copiados a mano viven en `/app/flows`
del contenedor. Para que sobrevivan a recrear el contenedor, monta un volumen:

```bash
mkdir -p ./flows
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 \
  -v "$(pwd)/flows:/app/flows" \
  --name flow juankanh/flow-app:4.2.0
```

> Con el volumen montado, los flows de ejemplo que trae la imagen quedan ocultos: tu carpeta
> manda. Copia dentro también tu `sql-connections.json` si usas nodos SQL (ver
> [cli.md](cli.md#nodos-sql)).
>
> Nota: los flows que editas en la **web** se guardan en el `localStorage` de tu navegador;
> lo que persiste en disco es lo que exportas (botón *Export*), guardas por MCP (`flow_save`)
> o copias a `/app/flows`.

## Actualizar de versión

```bash
docker pull juankanh/flow-app:4.2.0
docker stop flow && docker rm flow
docker run -d ... juankanh/flow-app:4.2.0   # mismo run de siempre
```

Los flows en volumen (y los del navegador) no se pierden.

## Troubleshooting

| Síntoma | Causa y arreglo |
|---------|-----------------|
| Los flows a `http://localhost:XXXX` fallan con `ECONNREFUSED` | Falta `--add-host=host.docker.internal:host-gateway` en el `docker run` |
| Quiero llamar a un servicio del **propio contenedor** y me redirige al host | La reescritura de localhost está activa: arranca con `-e FLOW_REWRITE_LOCALHOST=false` (o usa `--no-localhost-rewrite` en el CLI) |
| `/mcp` responde 403 desde otra máquina | Comportamiento por defecto (solo clientes locales). Define `FLOW_MCP_TOKEN` o `FLOW_MCP_ALLOW_LAN=true` |
| La web no “ve” al agente / las tools dicen *no web tab connected* | Abre http://localhost:9998 en un navegador: la pestaña se conecta sola al puente. Manda la **última** pestaña abierta |
| Un nodo SQL falla | Revisa el perfil en `flows/sql-connections.json` y que la BBDD sea alcanzable **desde el contenedor** (nombres de red de Docker, no `localhost`) |
