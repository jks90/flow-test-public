# Instalación con Docker

Todo Flow vive en una sola imagen: web, CLI, motor SQL y servidor MCP.

> ¿Prefieres Compose? Copia [`docker-compose.example.yml`](../docker-compose.example.yml)
> como `docker-compose.yml` y `docker compose up -d` — trae puertos, volumen de flows y las
> env vars del MCP comentadas.

## Arranque básico

```bash
docker run -d \
  --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 \
  --name flow \
  juankanh/flow-app:4.32.0
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
  juankanh/flow-app:4.32.0
```

## Variables de entorno

| Variable | Por defecto | Qué hace |
|----------|-------------|----------|
| `PORT` | `3001` | Puerto interno del servidor |
| `FLOW_FLOWS_DIR` | `/app/flows` | Carpeta del **proyecto** (los `.flow.json` que ve el panel Proyecto, el MCP, el CLI y donde van las capturas). Normalmente no se toca: se monta tu carpeta sobre `/app/flows`. Úsala si prefieres montarla en otra ruta (`-v /tu/carpeta:/data/flows -e FLOW_FLOWS_DIR=/data/flows`) |
| `FLOW_REWRITE_LOCALHOST` | `true` en la imagen | Reescribe `localhost`/`127.0.0.1` → `host.docker.internal` en las peticiones de los flows (web y CLI). Ponla a `false` si tu objetivo es un servicio **dentro** del propio contenedor |
| `FLOW_MCP_TOKEN` | *(sin definir)* | Si la defines, el endpoint `/mcp` exige `Authorization: Bearer <token>` — necesario para usar el MCP desde **otra máquina** de forma segura |
| `FLOW_MCP_ALLOW_LAN` | *(sin definir)* | `true` abre `/mcp` a la LAN **sin** auth (solo redes de confianza). Por defecto `/mcp` solo acepta clientes locales |
| `FLOW_VIEWPORT` | `1366x850` | Resolución de la sesión Live, por ejemplo `1920x1080`; la vista y los clics se adaptan a la resolución efectiva |
| `FLOW_FAKE_WEBCAM` | *(sin definir)* | Ruta a un MJPEG/Y4M que Chromium servirá como webcam. Solo QA |
| `FLOW_VIDEOMOCK_MOLDS` | *(sin definir)* | Carpeta privada con las plantillas necesarias para `POST /videomock/dni` |

Ejemplo con MCP protegido por token:

```bash
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 -e FLOW_MCP_TOKEN=mi-secreto \
  --name flow juankanh/flow-app:4.32.0
```

## Webcam falsa y videomock de DNI (solo QA)

Para probar un onboarding de vídeo-identificación, monta el vídeo y las siete plantillas privadas
dentro del volumen de flows:

```bash
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 --name flow \
  -e FLOW_FAKE_WEBCAM=/app/flows/webcam/DNIWebcamMock.mjpeg \
  -e FLOW_VIDEOMOCK_MOLDS=/app/flows/molds \
  -e FLOW_VIEWPORT=1920x1080 \
  -v "$(pwd)/flows:/app/flows" \
  juankanh/flow-app:4.32.0
```

Genera el vídeo antes de abrir la sesión Live:

```bash
curl -X POST http://localhost:9998/videomock/dni \
  -H 'content-type: application/json' \
  -d '{
    "personalNumber":"12345678",
    "primaryName":"ANA",
    "secondaryName":"PRUEBA TEST",
    "birthDate":"1980-12-12",
    "expirationDate":"2031-08-05",
    "sex":"F",
    "father":"PADRE",
    "mother":"MADRE",
    "birthPlace":"MADRID",
    "supportNumber":"ABC123456",
    "addresses":{"streetName":"CALLE QA","streetNumber":"1","postalCode":"28001","city":"MADRID"}
  }'
```

`primaryName` es el nombre y `secondaryName` debe contener exactamente dos apellidos separados
por espacio y menos de 26 caracteres. El endpoint responde `201` y sustituye el vídeo de forma
atómica. Las plantillas incluyen imágenes de identidad: no se distribuyen en la imagen, no deben
subirse a Git y solo deben utilizarse en entornos de QA.

## Apuntar el proyecto a una carpeta tuya

El panel **Proyecto** de la web (4.25) trabaja sobre la carpeta `flows/` del servidor — en la imagen,
`/app/flows`. Para que sea **tu** carpeta local (la que versionas en git), móntala:

```bash
docker run -d --add-host=host.docker.internal:host-gateway -p 9998:3001 \
  -v /home/yo/mis-flows:/app/flows \
  --name flow juankanh/flow-app:4.32.0
```

Si prefieres otra ruta dentro del contenedor, indícasela con `FLOW_FLOWS_DIR`:

```bash
  -v /home/yo/mis-flows:/data/flows -e FLOW_FLOWS_DIR=/data/flows
```

El panel muestra la ruta efectiva (la de dentro del contenedor) y **Ctrl+S** escribe directamente en
tus ficheros, que conservan tu usuario como dueño aunque el proceso del contenedor sea root. Las
capturas (`/flow-assets`), el perfil de Chromium del modo Live y `flow-explore` usan la misma carpeta.

## Persistir tus flows

Los `.flow.json` guardados desde el MCP (`flow_save`) o copiados a mano viven en `/app/flows`
del contenedor. Para que sobrevivan a recrear el contenedor, monta un volumen:

```bash
mkdir -p ./flows
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 \
  -v "$(pwd)/flows:/app/flows" \
  --name flow juankanh/flow-app:4.32.0
```

> Con el volumen montado, los flows de ejemplo que trae la imagen quedan ocultos: tu carpeta
> manda. Copia dentro también tu `sql-connections.json` si usas nodos SQL (ver
> [cli.md](cli.md#nodos-sql)).
>
> Nota: los flows que editas en la **web** se autoguardan en el `localStorage` de tu navegador;
> en disco persiste lo que guardas con **Ctrl+S** desde el panel **Proyecto** (4.25 — escribe
> directamente en `/app/flows`, o sea en tu `./flows`), lo que exportas (*Export*), lo que guarda el
> MCP (`flow_save`) o lo que copias a `/app/flows`. Los ficheros escritos por el contenedor
> conservan el dueño del host.

## Actualizar de versión

```bash
docker pull juankanh/flow-app:4.32.0
docker stop flow && docker rm flow
docker run -d ... juankanh/flow-app:4.32.0   # mismo run de siempre
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
