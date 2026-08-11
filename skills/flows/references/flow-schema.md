# Schema `.flow.json` (flow-test 4.x)

Formato exacto que entienden la web, el CLI y el MCP. Un agente debe seguirlo **al pie de la
letra** cuando escriba flows a mano.

## Estructura de primer nivel

```json
{
  "version": "1.0",
  "name": "dominio-caso",
  "nodes": [],
  "infoNodes": [],
  "sqlNodes": [],
  "webNodes": [],
  "connections": [],
  "envVariables": {}
}
```

- Para flows de API usa `nodes` (HTTP), `sqlNodes` (si verificas BBDD), `connections` y
  `envVariables`; deja `infoNodes` y `webNodes` como arrays vacíos.
- `name` corto y descriptivo (`{dominio}-{caso}`), que coincida con el nombre del fichero.

## RequestNode (`nodes[]`)

```json
{
  "id": "<uuid-v4>",
  "name": "Auth - Login",
  "curl": "curl -X POST \"{{apiBase}}/auth/login\" \\\n\n-H \"Content-Type: application/json\" \\\n\n-d '{\n  \"email\": \"{{email}}\",\n  \"password\": \"{{password}}\"\n}'",
  "position": { "x": 60, "y": 60 },
  "status": "idle",
  "extractions": [
    { "id": "<uuid-v4>", "varName": "token", "jsonPath": "$.accessToken" }
  ],
  "collapsed": false
}
```

Reglas:

- `curl` es un comando curl **bash completo**. Soporta `-X`, `-H`, `-d/--data-raw`,
  `--data-urlencode`, `-u` (basic auth) y `-F` (multipart). Las continuaciones de línea se
  escriben como `\\\n\n` (backslash + línea en blanco), como en los flows existentes.
- `{{variable}}` se interpola en cualquier parte del curl (URL, headers, body).
- `extractions[].jsonPath` se evalúa sobre el body JSON de la respuesta; el valor queda en
  `varName` para los nodos **posteriores**. Rutas tipo `$.data.items[0].id`.
- Un nodo pasa con status HTTP 2xx; cualquier otra cosa lo marca FAIL.
- `status` siempre `"idle"` en disco; `position.x` suele incrementarse ~520 por nodo en la
  misma fila (`y` constante).

## SqlNode (`sqlNodes[]`)

```json
{
  "id": "<uuid-v4>",
  "name": "BBDD - Último usuario",
  "position": { "x": 60, "y": 320 },
  "collapsed": false,
  "dbType": "mysql",
  "connectionProfileId": "mi-mysql",
  "host": "", "port": "", "database": "", "username": "", "password": "",
  "query": "SELECT id, email FROM users WHERE email = '{{email}}' LIMIT 1",
  "displayMode": "table",
  "status": "idle",
  "extractions": [
    { "id": "<uuid-v4>", "varName": "userId", "column": "id", "rowIndex": 0 }
  ]
}
```

Reglas:

- `dbType`: `postgres` | `mysql` | `oracle` (Oracle en modo thin).
- Conexión: por `connectionProfileId` (resuelto contra `sql-connections.json`, que se busca
  desde la carpeta del flow hacia arriba) **o** campos inline. Si el perfil no existe, se
  cae a los campos inline con un aviso. Postgres admite `jdbcUrl`
  (`jdbc:postgresql://host:5432/db?sslmode=disable&user=u&password=p`).
- `query` admite `{{variables}}`. **Ejecuta de verdad** (INSERT/UPDATE incluidos): solo
  entornos de prueba.
- `extractions`: `column` (nombre de columna) + `rowIndex` (0 = primera fila,
  **obligatorio**) → variable para los nodos siguientes.
- Los sqlNodes entran en el mismo grafo topológico que los HTTP: pueden ser origen o destino
  de una connection.

## Connection (`connections[]`)

```json
{ "id": "<uuid-v4>", "sourceId": "<node-uuid>", "targetId": "<node-uuid>" }
```

- Definen el **orden de ejecución** (topológico): un nodo corre cuando su origen termina OK,
  y ve las variables extraídas aguas arriba.
- `behavior` opcional: `next` (por defecto — el único que ordena el grafo en CLI y «Run
  Flow»), `on_error`, `parallel`, `none` (interactivos de la web).
- Los nodos sin dependencias forman la primera capa y corren en paralelo.

## envVariables

```json
{
  "apiBase": "http://localhost:8080",
  "email": "qa@example.com",
  "password": "cambia-esto"
}
```

- Todos los valores son **strings**.
- Toda `{{var}}` usada en un curl o query debe existir aquí **o** venir de una extracción
  aguas arriba — si no, se queda sin interpolar tal cual (`{{var}}` literal en la petición).
- En ejecución se sobrescriben con `--var clave=valor` (CLI) o `variables_set` (MCP) — pon
  ahí las credenciales en vez de hardcodearlas en el curl.

## UUIDs

Genera UUIDs v4 reales y **frescos para cada** nodo/conexión/extracción
(`uuidgen`, `python3 -c "import uuid; print(uuid.uuid4())"` o el `crypto.randomUUID()` de
Node). Al importar por web/MCP se regeneran, pero en disco deben ser únicos dentro del flow.

## Ejemplo mínimo completo (login → llamada autenticada)

```json
{
  "version": "1.0",
  "name": "auth-login-listado",
  "nodes": [
    {
      "id": "11111111-1111-4111-8111-111111111111",
      "name": "Auth - Login",
      "curl": "curl -X POST \"{{apiBase}}/auth/login\" \\\n\n-H \"Content-Type: application/json\" \\\n\n-d '{\n  \"email\": \"{{email}}\",\n  \"password\": \"{{password}}\"\n}'",
      "position": { "x": 60, "y": 60 },
      "status": "idle",
      "extractions": [
        { "id": "22222222-2222-4222-8222-222222222222", "varName": "token", "jsonPath": "$.accessToken" }
      ],
      "collapsed": false
    },
    {
      "id": "33333333-3333-4333-8333-333333333333",
      "name": "Users - Listado autenticado",
      "curl": "curl \"{{apiBase}}/users\" \\\n\n-H \"Authorization: Bearer {{token}}\"",
      "position": { "x": 580, "y": 60 },
      "status": "idle",
      "extractions": [],
      "collapsed": false
    }
  ],
  "infoNodes": [],
  "sqlNodes": [],
  "webNodes": [],
  "connections": [
    { "id": "55555555-5555-4555-8555-555555555555", "sourceId": "11111111-1111-4111-8111-111111111111", "targetId": "33333333-3333-4333-8333-333333333333" }
  ],
  "envVariables": {
    "apiBase": "http://localhost:8080",
    "email": "qa@example.com",
    "password": "cambia-esto"
  }
}
```

> Los UUID de arriba son placeholders legibles: genera los tuyos.

## Ejecutar y diagnosticar

```bash
# Contenedor (lo habitual)
docker exec flow node cli/run-flow.js --flow flows/x.flow.json --report-root /tmp/resumen
docker exec flow node cli/run-flow.js --dir flows --report-root /tmp/resumen

# Instalación local del repo flow-test
node cli/run-flow.js --flow flows/x.flow.json --report-root /tmp/resumen

# Sobrescribir variables (credenciales, host del API…)
… --var apiBase=http://host.docker.internal:8080 --var password=otra
```

- **Exit code 1** si algún nodo falla (no-2xx o error SQL); `0` = todo verde.
- Report en `<report-root>/<fecha-hora>/`: `report.md` (resumen), `summary.json`,
  `all-runs.json`, `flows/*.run.json` y `debug/*.debug.md` — **el debug es lo primero que
  hay que leer al diagnosticar**: curl/query interpolados + request/response completos +
  variables extraídas.
- Flags útiles: `--timeout <ms>` (180000 por defecto), `--continue-on-error`, `--no-report`,
  `--skip-sql-nodes`, `--no-localhost-rewrite`, `--sql-connections <fichero>`.
- Desde el contenedor, `localhost` en los curl se reescribe a `host.docker.internal`
  (apaga con `--no-localhost-rewrite` o `FLOW_REWRITE_LOCALHOST=false`).
