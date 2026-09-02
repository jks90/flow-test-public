# Ejemplos

## 🗂️ La galería (`workspace/`) — móntala y ejecuta

Un **workspace completo** listo para usar como tu carpeta `flows/`: 16 flows organizados por
carpetas, con un tour guiado («00 EMPIEZA AQUÍ.md») cuyos enlaces ejecutan cada flow con un ▶.

```bash
docker run -d --add-host=host.docker.internal:host-gateway \
  -p 9998:3001 --name flow \
  -v "$(pwd)/workspace:/app/flows" \
  juankanh/flow-app:latest
# http://localhost:9998 → icono «Proyecto» → «00 EMPIEZA AQUÍ.md»
```

| Carpeta | Qué hay dentro |
|---------|----------------|
| `aprende/` | Un concepto por flow, en orden: **variables y cadenas**, **asserts** (tu primer test), **entornos** pre/prod, **notas con scripts JS**, **SQL + HTTP**, **data-driven** con CSV |
| `funciones/` | Las que trabajan solas: **Monitor 24x7** (el servidor lo ejecuta cada 15 min y avisa al fallar), **Webhook desde CI** (`POST /hook/<token>` desde GitHub Actions), **Correo OTP** (el SMTP embebido captura el email y extrae el código), **Puente a local** |
| `flowtest-por-dentro/` | Nuestra plataforma probada con nuestra herramienta, contra los endpoints reales de app.flowtest.es: **Solicitar espacio cloud** (⚠️ crea cuentas de verdad — lee su LEEME), **Login y sesión**, **Comprar licencia** (nuestro funnel de venta, como flow) y **Salud de la plataforma** |
| `paneles/` | Los de enseñar en pantalla grande: **Crypto portfolio** (Ethplorer + Blockscout + CoinGecko encadenados) y **Mapa mundial** (nodos sobre una imagen de fondo) |

Todos los flows llevan asserts y los ejecutables se verificaron con el CLI (exit 0) antes de
publicarse. El tour `.md` se abre en la app desde la **5.5.0** (en versiones anteriores, léelo aquí
en GitHub). El ejemplo XXL sigue siendo [`economia-global-bundle/`](economia-global-bundle/).

## Ficheros sueltos

| Fichero | Qué demuestra |
|---------|---------------|
| `api-login-cadena.flow.json` | Cadena HTTP clásica: login → extraer `{{token}}` → llamada autenticada → detalle con id extraído |
| `sql-verificacion.flow.json` | Nodo SQL que extrae `userId`/`email` de la BBDD y alimenta una verificación HTTP |
| `sql-connections.example.json` | Perfiles de conexión SQL (cópialo como `sql-connections.json` junto a tus flows) |
| `nota-mermaid.flow.json` | Flow documentado con un **diagrama Mermaid** en el canvas (`infoNodes` con `renderMode: "mermaid"`, desde la 4.3.0): el esquema de qué llama a qué se renderiza en vivo junto al flow. Con el botón **Maximizar** (4.24) se ve a pantalla completa |
| `pizarra-anotada.flow.json` | 🆕 4.24 — La cadena de login **anotada con la pizarra** (`drawings`: marco, título, flecha de aviso y elipse destacada), una caja **📌 fijada**, nº de **orden** en cada nodo y una nota con **script JS** que genera un email único en cada Run Flow (4.23). Ábrelo con *Abrir* en la web |

## Probarlos

```bash
# Copiar al contenedor y ejecutar (ajusta variables con --var)
docker cp api-login-cadena.flow.json flow:/tmp/
docker exec flow node cli/run-flow.js --flow /tmp/api-login-cadena.flow.json \
  --var apiBase=http://host.docker.internal:8080 --no-report
```

O cárgalos en el canvas vía MCP: `flow_file_read` + `flow_overwrite` (si están en `flows/`),
o pídele a tu agente que los recree con `node_add_request`/`nodes_connect`.
