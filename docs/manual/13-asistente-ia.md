# 13 · El asistente de IA 🆕 (5.13)

Un chat dentro de la app que **construye, explica, prueba y arregla flows en tu canvas en directo**, con tu propia clave de **Claude** (Anthropic) u **OpenAI**. No es un chat al lado del producto: cada cosa que la IA decide hacer pasa por las mismas herramientas que usa el MCP embebido, así que ves los nodos aparecer, las conexiones dibujarse y las respuestas llegar mientras habla. Tú puedes pararla, corregirla o deshacer su turno.

![](assets/flowtest-74-ia-panel.png)

Se abre con el icono ✨ de la barra lateral izquierda. Es una función **Pro**; en la prueba de 14 días el panel se ve pero el chat pide pasar a Pro.

## Configurar: tu clave, tu proveedor

⚙ **Ajustes** dentro del panel:

- **Proveedor**: Claude u OpenAI. Cada uno con su clave; puedes tener las dos y cambiar cuando quieras (cada conversación se queda con el proveedor con el que empezó).
- **Clave de API**: se guarda **cifrada en el servidor** (AES-256-GCM, la misma clave maestra de las credenciales) y **nunca llega al navegador**. La pagas tú a tu proveedor: flow-test no cobra por el uso de la IA. «Probar conexión» valida la clave y trae la lista real de modelos de tu cuenta.
- **Modelo**: sugeridos `claude-opus-5` (el mejor construyendo flows), `claude-sonnet-5` (más barato), `claude-haiku-4-5` (para charlar) y los `gpt-5` / `gpt-4.1` de OpenAI; o cualquiera de tu lista.
- **Esfuerzo** por defecto (rápido / normal / profundo / máximo): cuánto piensa el modelo antes de actuar. Se puede cambiar por mensaje desde el chat.
- **Preguntar antes de ejecutar**: pedir confirmación antes de `flow_run` / `node_run`. Borrar o sustituir se pregunta siempre.
- **Tope mensual** en dólares: al alcanzarlo el chat se para hasta que lo subas o cambie el mes.

En el **cloud**, un miembro puede usar la **clave de la org** (la comparte todo el equipo) o marcar **«solo mía»**: su clave privada manda sobre la compartida y solo la ve él.

## Cómo trabaja

En cada mensaje la IA recibe una **foto compacta del flow** (nodos con nombre, método, URL, celda y estado; conexiones; nombres de variables) y la lista de **lo que has cambiado tú a mano desde su último paso**. Por eso puedes editar el canvas mientras trabaja: en el siguiente paso lo ve y no lo pisa.

Después actúa con las **mismas 38 herramientas del MCP** (`node_add_request`, `nodes_connect`, `variables_set`, `flow_run`, `canvas_layout`…). Todo se retransmite en streaming al panel:

- el **razonamiento resumido** (plegado, «razonamiento») y el texto a medida que lo escribe;
- cada **tool** como una tarjeta: nombre, el input **escribiéndose en directo**, estado (escribiendo → ejecutando → hecho/error) y el resultado plegable; los ids de los nodos tocados salen como chips: clic para centrar el canvas en ese nodo;
- sobre el nodo, una marca **«🤖 IA»** unos segundos, igual que las marcas de colaboración;
- al final del turno, una línea con **llamadas, tokens (entrada, salida, caché) y coste** del turno.

![](assets/flowtest-75-ia-confirmacion.png)

**Confirmaciones.** Antes de borrar o sustituir (`node_delete`, `connection_delete`, `flow_overwrite`, cerrar pestaña, `flow_reset`, limpiar la pizarra, borrar correos…) aparece una tarjeta ámbar con la acción exacta y dos botones. Con «Preguntar antes de ejecutar» activo, lo mismo antes de lanzar peticiones o SQL de verdad. Si dices que no, la IA lo sabe («el usuario ha rechazado esta acción») y te pregunta qué prefieres. Si no contestas en 5 minutos o cierras el panel, cuenta como no.

**Parar** (■) corta la respuesta en cuanto termina la tool en curso; lo ya hecho en el canvas se queda. **Deshacer turno** restaura el canvas a como estaba antes del último mensaje (entra en la pila de Ctrl+Z, así que Ctrl+Y lo rehace).

![](assets/flowtest-76-ia-turno.png)

## Comandos rápidos

Escribe `/` en el chat:

| Comando | Qué hace |
|---|---|
| `/explain` | Explica el flow paso a paso sin tocar nada |
| `/docs` | Añade al canvas una nota Mermaid con el esquema y una nota con objetivo, variables y cómo ejecutarlo |
| `/test` | Convierte el flow en un test: asserts por request y extracciones para encadenar |
| `/fix` | Lee el último run y la consola, explica el fallo, corrige el nodo culpable y, con tu permiso, lo vuelve a ejecutar |
| `/run` | Ejecuta el flow y cuenta el resultado real nodo a nodo |
| `/clean` | Ordena el canvas por celdas y separa solapes |

Puedes añadir detalle detrás: `/fix el login devuelve 401`.

## Adjuntos y contexto

Arrastra ficheros al panel (o 📎): **OpenAPI/Swagger** (`.json`, `.yaml`), **HAR**, `.md`, `.txt`, `.csv`, `.curl`, **PDF** (Claude lo lee nativo, con citas) e **imágenes** (una captura de Postman, un error en pantalla). Hasta 20 MB por mensaje. Pídele «monta el flow de esta spec» o «este HAR es el login, reprodúcelo».

Crea `flows/IA.md` con las normas de tu equipo (entornos, nombres, qué no tocar, cómo se autentica la API) y la IA lo lee en todas las conversaciones.

## Seguridad y coste

- A la IA **nunca le llegan credenciales en claro**: las cabeceras `Authorization`/`Cookie`/`x-api-key`, los `Bearer …` y las claves con pinta de `sk-…` de los resultados se sustituyen por «oculto» antes de enviarse; los `{{secret:X}}` y `{{variables}}` viajan como placeholders. Cuando necesite un token te pedirá que lo crees en Variables ▸ Credenciales.
- Las conversaciones, los adjuntos, el uso (`usage.jsonl`) y la auditoría (`audit.jsonl`: quién ejecutó qué tool) viven en `flows/.ai/` del servidor. En el cloud, **Mi cuenta ▸ 🤖 Asistente de IA** muestra tu consumo del mes y el del equipo, y el administrador ve por org el uso por persona y modelo y las últimas acciones; ni las claves ni los chats salen del contenedor.
- Cada llamada anota tokens y coste (precios de Anthropic por modelo; OpenAI muestra tokens y «—» en coste). El tope mensual corta el chat, no el resto de la app.

## Límites y consejos

- Máximo 40 pasos (llamadas) por mensaje; si se queda corto, dile «continúa».
- Con la app cerrada no hay canvas: el chat funciona pero las tools fallan con aviso. Ten la pestaña abierta.
- Un flow grande consume más tokens por mensaje (la foto del flow va en cada uno); la caché del proveedor abarata las repeticiones.
- El asistente usa el mismo puente que el MCP: si tienes Claude Code conectado al `/mcp` a la vez, los dos mandan sobre la misma pestaña.
