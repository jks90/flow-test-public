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
| `connections` | Aristas del grafo: qué se ejecuta después de qué |
| `envVariables` | Variables iniciales del flow (`{{clave}}`) |

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

## Conexiones (`connections[]`)

```json
{ "id": "c1", "sourceId": "login", "targetId": "listar", "behavior": "next" }
```

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

## Ejemplo completo

[`examples/api-login-cadena.flow.json`](../examples/api-login-cadena.flow.json) — login →
extraer token → llamada autenticada; y
[`examples/sql-verificacion.flow.json`](../examples/sql-verificacion.flow.json) — consulta
SQL → variable → verificación HTTP con esa variable.

## Consejos para generarlos con IA

- Ids: cualquier string único vale (al importarse por la web/MCP se regeneran).
- `position` es solo visual; si generas a máquina, escalona (`x: 100 + i*360`).
- Riesgo: los flows **ejecutan de verdad** (HTTP con efectos y SQL con INSERT/UPDATE).
  Apunta a entornos de prueba.
- La forma más cómoda: deja que la IA use el **MCP** ([mcp.md](mcp.md)) y guarde con
  `flow_save` — el JSON sale siempre bien formado.
