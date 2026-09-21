# El formato `.flow.json`

Un flow es un JSON autocontenido que la web, el CLI y el MCP entienden por igual. Conocer el
formato permite escribir flows a mano, generarlos con una IA o versionarlos en git junto a tu
API.

## Esqueleto

```json
{
  "version": "1.0",
  "name": "Mi flow",
  "nodes": [],
  "sqlNodes": [],
  "connections": [],
  "envVariables": {}
}
```

| Clave | Qué es |
|-------|--------|
| `nodes` | Nodos HTTP (curl) |
| `sqlNodes` | Nodos SQL (opcional) |
| `infoNodes` | Notas, diagramas Mermaid y capturas (opcional) |
| `webNodes` | Nodos Web (opcional; solo en la web) |
| `connections` | Aristas del grafo: qué se ejecuta después de qué |
| `envVariables` | Variables iniciales del flow (`{{clave}}`) |
| `environments` / `activeEnvironment` | Entornos con nombre (opcional, desde la **5.3.0**): `{"pre": {...}, "prod": {...}}` — overrides sobre `envVariables` al ejecutar; `activeEnvironment` fija el activo |
| `settings` | Configuración del flow (opcional, desde la **4.48.0**): vista, webhook entrante, monitor… (ver abajo) |
| `drawings` | Dibujos de la pizarra (opcional, desde la **4.24.0**; solo se escribe si hay alguno) |

Todos los nodos admiten además dos campos opcionales: `order` (nº de orden que usan *Alinear en
fila/columna* y la guía de nodos) y `pinned` (`true` = la caja no la mueve ningún relayout ni el
arrastre).

## Nodos HTTP (`nodes[]`)

```json
{
  "id": "login",
  "name": "Login",
  "curl": "curl -X POST \"{{apiBase}}/auth/login\" -H \"Content-Type: application/json\" -d '{\"email\":\"{{email}}\",\"password\":\"{{password}}\"}'",
  "position": { "x": 100, "y": 100 },
  "status": "idle",
  "collapsed": false,
  "extractions": [
    { "id": "e1", "varName": "token", "jsonPath": "$.data.accessToken" }
  ]
}
```

- **`curl`**: un comando curl normal. Soporta `-X`, `-H`, `-d/--data-raw`, `--data-urlencode`,
  `-u` (basic auth) y `-F` (multipart). Cualquier parte puede llevar `{{variables}}`.
- **`extractions`**: tras la respuesta, cada extracción evalúa un **JSONPath** sobre el body
  y guarda el valor como variable para los nodos siguientes. Rutas tipo `$.data.items[0].id`.
- Un nodo se considera OK con status HTTP 2xx.
- **`asserts`** (opcional, desde la **5.3.0**): condiciones que deben cumplirse para dar el nodo
  por bueno **aunque el HTTP sea 2xx** — web, CLI (exit ≠ 0) y MCP las evalúan igual:

  ```json
  "asserts": [
    { "id": "a1", "kind": "status", "expected": "2xx" },
    { "id": "a2", "kind": "jsonpath", "path": "$.data.id", "op": "exists" },
    { "id": "a3", "kind": "jsonpath", "path": "$.status", "op": "equals", "value": "active" },
    { "id": "a4", "kind": "time", "maxMs": 2000 }
  ]
  ```

  `kind: "status"` admite `2xx`, rangos `200-204` o listas `200,201`; `jsonpath` admite
  `op`: `exists` | `equals` | `not-equals` | `contains` | `gt` | `lt` (con `value`);
  `time` falla si la respuesta tarda más de `maxMs`.

## Nodos SQL (`sqlNodes[]`)

```json
{
  "id": "buscar-usuario",
  "name": "Buscar usuario",
  "position": { "x": 100, "y": 300 },
  "collapsed": false,
  "dbType": "mysql",
  "connectionProfileId": "mi-mysql",
  "host": "", "port": "", "database": "", "username": "", "password": "",
  "query": "SELECT id, email FROM users WHERE email = '{{email}}' LIMIT 1",
  "displayMode": "table",
  "status": "idle",
  "extractions": [
    { "id": "x1", "varName": "userId", "column": "id", "rowIndex": 0 }
  ]
}
```

- **`dbType`**: `postgres` | `mysql` | `oracle` (Oracle en modo thin, sin cliente nativo).
- **Conexión**: por `connectionProfileId` (resuelto contra `sql-connections.json`) **o** con
  los campos inline (`host`, `port`, `database`, `username`, `password`). Postgres admite
  además `jdbcUrl` (`jdbc:postgresql://…?sslmode=disable&user=u&password=p`).
- **`query`**: SQL con `{{variables}}` interpoladas.
- **`extractions`**: `column` + `rowIndex` → el valor de esa celda del resultado pasa a ser
  una variable. `rowIndex` es obligatorio (0 = primera fila).

## Notas y diagramas (`infoNodes[]`)

Cajitas de documentación dentro del canvas. No hacen peticiones, pero **sus `scripts` sí se
ejecutan** (desde la **4.23.0**) al pulsar Run Flow, con `flow_run` por MCP y en el CLI: sus
valores entran como variables antes de la primera petición.

```json
{
  "id": "esquema",
  "name": "Diagrama",
  "content": "graph TD\n  Cliente --> API\n  API --> BBDD[(BBDD)]",
  "position": { "x": 520, "y": 60 },
  "collapsed": false,
  "scripts": [],
  "renderMode": "mermaid"
}
```

- Sin `renderMode` (o `"text"`) es una **nota de texto** libre.
- Con `renderMode: "mermaid"` (desde la **4.3.0**) el `content` es código
  [Mermaid](https://mermaid.js.org) y la web lo renderiza como diagrama en vivo — útil para
  esquematizar qué llama a qué junto al propio flow. Admite `{{variables}}` en el código.
- Con `renderMode: "image"` (desde la **4.4.0**) la nota es un **nodo Captura**: muestra la
  imagen de `imageSrc` (`/flow-assets/…`, `http(s)` o data URI). Es el nodo que generan el
  modo Live y `flow-explore` para documentar pantallas de una web; `content` lleva las notas
  (URL, fecha…).
- `scripts`: lista de `{ "id", "varName", "code" }` — `code` es JavaScript que hace `return` de un
  valor; queda disponible como `{{varName}}`. Se ejecutan en orden antes de la primera petición
  (web, MCP y CLI; en el CLI la precedencia es `envVariables` < scripts < `--var`, y
  `--skip-info-scripts` los desactiva). Útil para ids/emails únicos por ejecución:
  `return 'qa+' + Date.now() + '@example.com'`. Deja `[]` si no los necesitas.
- Los scripts corren en la página web (o en el CLI, sin `document`) y reciben `vars`; con
  `when: "after"` corren al acabar el run con todas las extracciones. Un array extraído llega
  como texto `"v1,v2,…"` y un objeto como `[object Object]`: extrae primitivos y parsea. Un
  script «after» también puede dibujar botones y paneles sobre FlowTest y llamar a `/mcp`
  desde la página (ver el manual, 04 ▸ «Scripts que dibujan botones, páginas y acciones» y su contrato de higiene). Desde la 5.15 eso solo
  ocurre en flows **de confianza**: los abiertos de fichero, cloud o import corren en un sandbox sin
  página hasta que el usuario confía (chip de la barra).

## Pizarra (`drawings[]`) — desde la 4.24.0

Anotaciones dibujadas sobre el lienzo (rectángulos, elipses, flechas, líneas, trazos de lápiz y
textos). Comparten coordenadas con los nodos; no se ejecutan y el CLI las ignora. Solo aparecen en
el fichero cuando hay alguna.

```json
{
  "id": "marco",
  "type": "rect",
  "x": 60, "y": 100, "w": 1540, "h": 420,
  "stroke": "#f59e0b", "strokeWidth": 3, "strokeStyle": "solid",
  "fill": "transparent", "opacity": 1, "sketchy": true, "seed": 4242
}
```

| Campo | Valores |
|-------|---------|
| `type` | `pen` \| `line` \| `arrow` \| `rect` \| `ellipse` \| `text` |
| `x`, `y`, `w`, `h` | Caja del elemento. En `line`/`arrow` son el punto inicial y el vector hasta el final (`w`/`h` pueden ser negativos) |
| `points` | Solo `pen`: `[{ "x", "y" }, …]` en coordenadas del lienzo |
| `stroke`, `strokeWidth`, `strokeStyle` | Color, grosor y `solid` \| `dashed` \| `dotted` |
| `fill` | Solo `rect`/`ellipse`: color (p. ej. `#3b82f640`) o `transparent` |
| `opacity` | 0–1 |
| `sketchy` | `true` = trazo a mano alzada (estilo boceto); el `seed` fija el temblor para que no cambie entre renders |
| `text`, `fontSize`, `font` | Solo `text`: contenido (admite saltos de línea), tamaño y fuente: manuscritas `hand` (Patrick Hand, por defecto) \| `sketch` (Caveat) \| `excali` (Excalifont, la de Excalidraw) \| `indie` (Indie Flower) \| `marker` (Permanent Marker) \| `draft` (Cabin Sketch) \| `architect` (Architects Daughter) \| `note` (Gloria Hallelujah), o `sans` \| `mono`. Las manuscritas van empaquetadas con la app (desde 4.35.0; antes solo `hand`, `sans`, `mono`) |

Ejemplo completo: [`examples/pizarra-anotada.flow.json`](../examples/pizarra-anotada.flow.json).

## Conexiones (`connections[]`)

```json
{ "id": "c1", "sourceId": "login", "targetId": "listar", "behavior": "next" }
```

| `delayMs` (opcional, 4.34) | Pausa en ms antes de lanzar el destino cuando se recorre la flecha (web, CLI y MCP) |
| `behavior` | Semántica |
|------------|-----------|
| `next` (por defecto) | El destino se ejecuta cuando el origen termina OK — **define el orden topológico** |
| `on_error` | El destino solo se ejecuta si el origen falla (ejecución interactiva en la web) |
| `parallel` | Disparo simultáneo (ejecución interactiva en la web) |
| `none` | Solo informativa, no ejecuta nada |

> El CLI y el «Run Flow» de la web ordenan el grafo **solo con las aristas `next`**. Los
> nodos sin dependencias forman la primera capa y se ejecutan en paralelo.

## Variables

- `envVariables` del flow → disponibles desde el primer nodo.
- Cada extracción (JSONPath o columna SQL) añade/actualiza variables **para las capas
  siguientes**.
- Se interpolan con `{{nombre}}` en cualquier curl, query o campo de conexión.
- En el CLI, `--var clave=valor` sobrescribe cualquier variable.
- Variables automáticas: `runId` y `runTimestamp`.
- **Entornos** (5.3.0): `environments` define juegos de overrides con nombre y
  `activeEnvironment` el activo; al ejecutar, el entorno pisa a `envVariables`
  (precedencia final: `envVariables` < entorno < scripts/extracciones < `--var`).
  En el CLI, `--env <nombre>` elige entorno sin tocar el fichero.

## Configuración del flow (`settings`) — opcional

```json
"settings": {
  "webhook": { "token": "af31…" },
  "monitor": { "intervalMin": 15, "notifyUrl": "https://hooks.slack.com/…", "notify": "fail",
               "rules": [{ "id": "r1", "name": "Oro rompe 2700", "variable": "oro_precio", "op": "gt", "value": "2700" }] }
}
```

- `webhook` (5.3.0): publica `POST /hook/<token>` en el servidor — cualquier sistema lanza el
  flow por HTTP (query params y body JSON plano llegan como variables; `__env` elige entorno).
- `monitor` (5.4.0): el servidor ejecuta el flow solo cada `intervalMin` minutos, sin navegador;
  historial de 100 runs en `flows/.monitors/` (`GET /monitors`) y aviso `POST` a `notifyUrl`
  al fallar/cambiar de estado (`notify: "fail"`) o en cada run (`"always"`).
- `monitor.rules` (5.17.0, opcional): **reglas de vigilancia** sobre las variables con las que termina el
  run — `[{ "id", "name", "variable", "op", "value", "message"? }]` (máx. 20) con `op` = `gt` `gte` `lt`
  `lte` `equals` `not-equals` `contains` `not-contains` `changed` (sin `value`). El servidor las evalúa tras
  cada run del monitor y avisa a `notifyUrl` solo cuando una regla se dispara, se recupera o el valor cambia;
  `message` admite `{{variables}}` del run y `notifyUrl` admite `{{secret:X}}`. El CLI las ignora.
- También puede llevar `view` (tamaño/compacto/separación de nodos), `hideConnections`,
  `whiteboardStyle`, `viewport` y `background` (imagen de fondo del canvas) — todo visual;
  el CLI lo ignora. Ambos bloques de automatización se publican al **guardar** el flow en `flows/`.

## Ejemplo completo

[`examples/api-login-cadena.flow.json`](../examples/api-login-cadena.flow.json) — login →
extraer token → llamada autenticada; y
[`examples/sql-verificacion.flow.json`](../examples/sql-verificacion.flow.json) — consulta
SQL → variable → verificación HTTP con esa variable.

## Colocación y estado de cualquier nodo

Campos opcionales comunes a `nodes[]`, `sqlNodes[]`, `infoNodes[]` y `webNodes[]`:

| Campo | Tipo | Para qué |
|-------|------|----------|
| `position` | `{x, y}` | Esquina superior izquierda (px). Solo visual |
| `collapsed` | bool | Caja plegada |
| `order` | entero | Nº de orden para *Alinear en fila/columna* y la guía de nodos |
| `cell` | `{col, row}` | **Celda de cuadrícula** (4.28): `{col:1,row:1}` arriba a la izquierda; *Alinear en cuadrícula* y *Auto Layout* colocan cada caja en su celda — la forma cómoda de diseñar la disposición a mano: filas = capas (notas / HTTP / SQL / web), columnas = pasos |
| `pinned` | bool | 📌 Fijado: ningún relayout ni el arrastre lo mueven |
| `disabled` | bool | Desactivado (4.31): Run Flow, cadenas, cron, MCP y CLI lo saltan (sus conexiones se siguen); el ▶ de la caja sí lo ejecuta |

## Consejos para generarlos con IA

- Ids: cualquier string único vale (al importarse por la web/MCP se regeneran).
- `position` es solo visual; mejor que calcular coordenadas, da `cell` `{col, row}` a cada nodo y deja que *Alinear en cuadrícula* (o `canvas_layout grid` por MCP) calcule el sitio. Si generas `position` a máquina, escalona ≥ 560 px en X y ≥ 520 px en Y.
- Riesgo: los flows **ejecutan de verdad** (HTTP con efectos y SQL con INSERT/UPDATE).
  Apunta a entornos de prueba.
- La forma más cómoda: deja que la IA use el **MCP** ([mcp.md](mcp.md)) y guarde con
  `flow_save` — el JSON sale siempre bien formado.
