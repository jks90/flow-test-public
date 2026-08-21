# El CLI — ejecutar flows por terminal

El runner ejecuta los mismos `.flow.json` que la web, sin navegador: perfecto para CI, cron
o verificar un endpoint recién programado.

## `flow-run` en tu máquina (recomendado, sin código fuente)

El wrapper [`bin/flow-run`](../bin/flow-run) ejecuta el runner **de la imagen** montando tu
directorio actual: los flows se leen y los reports se escriben **en tu disco**.

```bash
cp bin/flow-run ~/.local/bin/ && chmod +x ~/.local/bin/flow-run

cd ~/mi-proyecto            # aquí viven tus flows/
flow-run --dir flows
flow-run --flow flows/login.flow.json --var apiBase=http://localhost:8080
cat resumen/*/report.md     # el report queda en tu máquina
```

- Usa rutas **relativas** al directorio desde el que lo lanzas (es lo que se monta).
- Cambia de versión con `FLOW_IMAGE=juankanh/flow-app:4.28.1 flow-run …`.
- Los curl a `http://localhost:PUERTO` llegan a **tu máquina** automáticamente.

## Dentro del contenedor que ya corre

```bash
docker exec flow node cli/run-flow.js --flow flows/mi-flow.flow.json
docker exec flow node cli/run-flow.js --dir flows          # batería recursiva
```

Devuelve **exit code 1 si algún flow falla** (y 2 si hay error de uso), así que se puede usar
directamente como paso de CI.

## Flags

| Flag | Qué hace |
|------|----------|
| `--flow <fichero>` | Ejecuta un solo `.flow.json` |
| `--dir <carpeta>` | Ejecuta todos los `.flow.json` (recursivo, orden alfabético) |
| `--var clave=valor` | Sobrescribe/añade una variable (repetible) |
| `--timeout <ms>` | Timeout por petición (por defecto 180000) |
| `--sql-connections <fichero>` | Fichero de perfiles SQL (por defecto: el `sql-connections.json` más cercano al flow, buscando hacia arriba) |
| `--skip-sql-nodes` | Ignora los nodos SQL (comportamiento pre-4.1) |
| `--out <fichero-o-dir>` | Log JSON del run |
| `--report-root <dir>` | Carpeta raíz de los reports (por defecto `resumen`) |
| `--no-report` | Sin report con timestamp |
| `--continue-on-error` | Sigue ejecutando tras un fallo |
| `--no-localhost-rewrite` | No reescribir `localhost` → `host.docker.internal` |

## Ejemplos

```bash
# Un flow con variables inyectadas
docker exec flow node cli/run-flow.js \
  --flow flows/store.flow.json \
  --var apiBase=http://host.docker.internal:8080 \
  --var adminEmail=admin@example.com

# Batería completa con report fuera del contenedor
docker exec flow node cli/run-flow.js --dir flows --report-root /tmp/resumen
docker cp flow:/tmp/resumen ./resumen

# Ejecutar un flow de TU máquina (sin copiarlo al contenedor)
docker cp ./mi-flow.flow.json flow:/tmp/
docker exec flow node cli/run-flow.js --flow /tmp/mi-flow.flow.json --no-report
```

## El report

Cada ejecución (salvo `--no-report`) crea `resumen/<fecha-hora>/` con:

| Fichero | Contenido |
|---------|-----------|
| `report.md` | Resumen legible: PASS/FAIL por flow, fallos y timeline de cada nodo |
| `summary.json` | Métricas de la ejecución (machine-readable) |
| `all-runs.json` | Todas las ejecuciones con request/response completos |
| `flows/*.run.json` | Log completo por flow |
| `debug/*.debug.md` | Debug por flow: curl interpolado, headers, bodies, query SQL interpolada, extracto del resultado y variables extraídas |

Las **passwords se enmascaran** en los reports (JDBC incluidas).

## Nodos SQL

Desde la 4.1 el CLI ejecuta los `sqlNodes` (Postgres, MySQL/MariaDB y Oracle en modo thin)
igual que la web: en el mismo orden topológico que los nodos HTTP, con `{{variables}}`
interpoladas en la query y la conexión, y **extracciones por columna** que alimentan a los
nodos siguientes.

- **Perfiles**: un nodo con `connectionProfileId` se resuelve contra `sql-connections.json`
  (junto al flow, o `--sql-connections <fichero>`):

```json
[
  {
    "id": "mi-mysql",
    "name": "MySQL local",
    "dbType": "mysql",
    "host": "host.docker.internal",
    "port": "3306",
    "database": "mi_bbdd",
    "username": "usuario",
    "password": "secreto"
  }
]
```

- Si el perfil no aparece, se usan los campos de conexión inline del nodo (con un aviso).
- Postgres admite `jdbcUrl` tipo `jdbc:postgresql://host:5432/db?sslmode=disable&user=u&password=p`
  (`sslmode=disable` se respeta; sin él se fuerza SSL, como piden AWS RDS/Supabase).

> ⚠️ Los nodos SQL **ejecutan de verdad** (incluidos INSERT/UPDATE). Si una batería antigua
> contaba con que se ignoraban, usa `--skip-sql-nodes`. Y recuerda: desde el contenedor,
> la BBDD se alcanza por nombre de red de Docker o `host.docker.internal`, no `localhost`.

## Scripts JS de las notas (4.23.0)

Los `infoNodes` con `scripts` se ejecutan **antes de la primera petición**, igual que al pulsar
«Run Flow» en la web: cada script hace `return` de un valor que queda como `{{varName}}`.
Precedencia: `envVariables` < scripts < `--var`. Con `--skip-info-scripts` se saltan. Los
dibujos de la pizarra (`drawings`) se ignoran.

## flow-explore — documentar una web navegándola (4.4.0)

Además del runner, la 4.4.0 añade `flow-explore`: navega una web con Chrome (crawl automático
o modo `--manual` donde navegas tú, con sesión/login), hace **screenshot de cada pantalla**,
captura sus **llamadas HTTP** y genera un `.flow.json` documental (pantalla → cajitas curl
re-ejecutables) + informe markdown.

```bash
git clone https://github.com/jks90/flow-test && cd flow-test && npm install
npm run flow:explore -- https://mi-web.com --manual --filter /api/
```

> Necesita un **Chrome/Chromium local** (usa `puppeteer-core`), por lo que se ejecuta desde el
> código fuente, no desde la imagen. Guía completa: [manual/07-flow-explore.md](manual/07-flow-explore.md).
