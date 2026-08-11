---
name: flows
description: Crear, ejecutar y verificar flows de flow-test (.flow.json) contra cualquier API REST — en vivo por MCP sobre el canvas del usuario, o como ficheros ejecutados con el CLI. Úsala cuando el usuario pida crear un flow, probar endpoints recién desarrollados o montar una batería de verificación.
---

# Flows — verificación de APIs con flow-test

Eres un agente que usa **Flow** (flow-test) para verificar APIs: compones flujos de
peticiones HTTP y consultas SQL encadenadas, los ejecutas y lees los resultados.

> Copia esta carpeta a `.claude/skills/flows/` de tu proyecto para activarla en Claude Code.

## Paso 0 — Contexto (pregunta lo que falte, de una vez)

1. **¿Dónde corre la API objetivo?** (host y puerto, p. ej. `http://localhost:8080`).
2. **¿Autenticación?** (ninguna / bearer estático / login que devuelve token / basic).
3. **¿Dónde corre flow-test?** — por defecto contenedor en `http://localhost:9998`
   (MCP en `/mcp`, CLI vía `docker exec flow …`).

## Elige el modo de trabajo

### Modo A — MCP en vivo (preferido si el servidor MCP `flow-test` está conectado)

El usuario **ve el canvas** mientras trabajas. Reglas:

1. `bridge_status` primero: si no hay pestaña web conectada, pide al usuario que abra la web
   y reintenta (las tools de disco funcionan igualmente).
2. `flow_create` con un nombre descriptivo → guarda el `tabId` y úsalo en TODAS las llamadas.
3. Construye por pasos: `node_add_request` / `node_add_sql` → `nodes_connect` (behavior
   `next` define el orden). Las respuestas te dan los `nodeId`.
4. Variables: `variables_set` para la base (`apiBase`, credenciales de prueba); extracciones
   en cada nodo para encadenar (`jsonPath` en HTTP, `column`+`rowIndex` en SQL).
5. Ejecuta con `flow_run` (grafo HTTP) o `node_run` sobre el nodo raíz de una cadena SQL.
   **Espera al resultado y léelo**: status por nodo, `runtimeVariables`, bodies.
   Si `finished:false`, lee el `reason` — nunca asumas que corrió.
6. Itera: `node_update` para corregir, reejecuta, y `console_read`/`runs_read` para
   diagnosticar.
7. Al terminar, `flow_save` para dejar el `.flow.json` en `flows/` (ejecutable en CI con el
   CLI) y resume al usuario: qué se probó, qué pasó, qué variables se extrajeron.

### Modo B — Fichero + CLI (sin MCP, o para CI)

1. Escribe el `.flow.json` siguiendo el formato (nodos, conexiones `next`, extracciones);
   referencia: `docs/flows-formato.md` del repo flow-test-public.
2. Ejecútalo:
   ```bash
   docker exec flow node cli/run-flow.js --flow /ruta/al/flow.flow.json --report-root /tmp/resumen
   ```
   (o `node cli/run-flow.js` si flow-test corre local). `--var clave=valor` para inyectar
   credenciales sin escribirlas en el fichero.
3. Lee el report (`report.md` y `debug/*.debug.md`) y presenta el resultado con el
   status real de cada nodo.
4. Exit code 1 = algún nodo falló → investiga en el debug antes de concluir.

## Diseño de un buen flow de verificación

- **Un flow por caso de uso** (login+listado, alta+consulta, compra completa…), nombres de
  nodo descriptivos.
- **Encadena por variables**, no copies valores: login extrae `token` → los demás nodos usan
  `-H "Authorization: Bearer {{token}}"`.
- **Verifica efectos, no solo status**: tras un POST que crea algo, añade un GET (o un nodo
  SQL) que confirme que existe de verdad.
- **Nodos SQL**: siembra datos de prueba o comprueba el efecto real en la BBDD; extrae ids
  reales para alimentar las llamadas HTTP. Recuerda que ejecutan de verdad: solo entornos
  de prueba.
- **Endpoints destructivos**: pregunta antes de incluirlos, y márcalo en el resumen.

## Qué NO hacer

- No des por buena una ejecución sin leer el resultado real (status por nodo + extracciones).
- No apuntes a producción ni uses credenciales reales sin que el usuario lo pida
  explícitamente.
- No dupliques flows: si ya existe uno para ese endpoint (`flow_files_list`), evoluciónalo.
