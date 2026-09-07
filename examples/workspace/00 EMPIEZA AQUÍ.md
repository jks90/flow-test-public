# 🚀 Empieza aquí — la galería de FlowTest

Esto no es una documentación para leer: es un workspace para **ejecutar**. Cada enlace de esta página lleva un ▶ al lado — púlsalo y el flow entero corre delante de ti, con sus variables, sus extracciones y sus asserts en verde (o en rojo, que también se aprende). Abre el que te llame la atención y mira qué pasa en el canvas.

## 🎓 Aprende (5 minutos cada uno)

En orden, cada uno enseña una sola cosa y la enseña ejecutándose:

- [[01 · Variables y cadenas]] — la idea central de FlowTest: una petición extrae un dato con JSONPath y la siguiente lo usa con `{{variable}}`, sin copiar y pegar nada.
- [[02 · Asserts — tu primer test]] — un 200 no es un test: aquí compruebas status, contenido del JSON y tiempo de respuesta, y ves el flow ponerse rojo cuando la API no cumple.
- [[03 · Entornos pre y prod]] — el mismo flow apuntando a pre o a prod cambiando un selector (o un `--env` en el CLI), sin duplicar nada.
- [[04 · Notas con scripts]] — una nota con JavaScript genera datos únicos por ejecución (emails, timestamps, ids) antes de la primera petición.
- [[05 · SQL más HTTP]] — llamas a la API y verificas directamente en la base de datos, en el mismo grafo y con las mismas variables.
- [[06 · Data-driven]] — una sola cadena, un CSV de casos: el CLI ejecuta el flow una vez por fila y te dice cuáles pasaron.
- [[07 · Errores y reintentos]] — retry con backoff por nodo, pausas en el conector (el badge ⏱) y una rama on_error de plan B: resiliencia que se ve.
- [[08 · Paralelo y tiempos]] — el grafo ES el orden: la fila entera corre a la vez y el cierre espera a todos; en el CLI, exactamente igual.

## ⚙️ Funciones que pagan el café

Lo que convierte un juguete visual en una herramienta que trabaja mientras duermes:

- [[Monitor 24x7]] — el servidor ejecuta tu flow cada N minutos sin navegador y te avisa por webhook cuando algo falla: enterarte tú antes que tus usuarios.
- [[Webhook desde CI]] — tu pipeline lanza el flow con un simple `POST /hook/<token>` y recibe el resultado por nodo: tests de API en Jenkins o GitHub Actions sin instalar nada más.
- [[Correo OTP]] — probar un registro con código de verificación sin cuentas de correo reales: el SMTP embebido captura el email y te extrae el OTP como variable.
- [[Puente a local]] — tu API vive en `localhost` y el flow corre en Docker: el puente `host.docker.internal` la alcanza igual, y aquí lo ves funcionando.
- [[Documentar una web]] — el nodo Web carga flowtest.es EN VIVO dentro del canvas, junto a tres capturas que hizo el propio producto (POST /capture, Chromium del servidor) — no están pegadas a mano.

## 🏠 FlowTest por dentro

Nuestra propia plataforma, probada con nuestra propia herramienta — estos cuatro flows atacan los endpoints reales de flowtest.es y app.flowtest.es, y el de compra es literalmente nuestro funnel de venta:

- [[Solicitar espacio cloud]] — el alta de una organización cloud, de verdad (lee antes el LEEME de la carpeta: crea cuentas reales).
- [[Login y sesión]] — credenciales → token → perfil → logout contra el servicio de cuentas.
- [[Comprar licencia]] — planes y checkout: el mismo camino que recorre un cliente que paga.
- [[Salud de la plataforma]] — health checks de la web, las cuentas y el TLS; el flow que nosotros mismos monitorizamos.
- [[Automatiza tu FlowTest]] — tu instalación también tiene API: /monitors, /hook/<token> y /access — vigila tu FlowTest CON FlowTest.
- [[Puente cloud (flow-bridge)]] — el endpoint auténtico del puente cloud→local, rechazando un token falso como debe (401), con el viaje de la petición en un diagrama.

Más detalle (y el aviso importante) en `flowtest-por-dentro/00 LEEME.md`.

## 🗺️ Paneles para enseñar en una demo

Para cuando quieras impresionar en pantalla grande:

- [[Crypto portfolio]] — precios en vivo encadenados y calculados sobre el canvas.
- [[Mapa mundial]] — nodos colocados sobre una imagen de fondo: APIs de países y divisas repartidas por el mapa.
- [[Arquitectura FlowTest]] — el mapa del sistema en un diagrama Mermaid vivo + cuatro cajitas tocando los endpoints reales del cloud, ahora mismo.

Y si quieres el ejemplo XXL, el bundle `economia-global-bundle` del repo (en `../economia-global-bundle`) es un workspace completo de economía global con decenas de nodos, SQL y documentación enlazada.

## 🧮 Scripts: lo que se calcula dentro del flow

Tres flows que enseñan hasta dónde llegan los **scripts JS de las notas** — sin tocar la app, todo dentro del `.flow.json`:

- [[Scripts 01 · Cripto: señal calculada en el flow]] — dos peticiones a CoinGecko traen 90 días de precios y un script «después» calcula SMA20/50, RSI14, variaciones y una señal por activo; otro script genera un **gráfico Mermaid** que la nota de al lado pinta en vivo.
- [[Scripts 02 · Tiempo: semáforo y gráfico por script]] — una petición a Open-Meteo y los scripts la convierten en un semáforo por día 🟢🟡🔴, un aviso y un gráfico de máximas y mínimas; los umbrales y la ciudad se cambian en Variables.
- [[Scripts 03 · Pedido: datos de prueba y verificación por script]] — scripts «antes» que fabrican un pedido distinto en cada run, un POST con asserts, y scripts «después» que comparan enviado vs recibido, validan el email y calculan base + IVA (con un pie de Mermaid).

- [[Scripts 04 · Divisas: botones, páginas y acciones]] — el script DIBUJA interfaz: una barra de botones abajo a la izquierda, una «página» (diálogo con pestañas y un conversor que recalcula al escribir), un informe en una pestaña nueva del navegador, descarga CSV, copiar, y ▶ que vuelve a ejecutar el flow hablando con el MCP de tu instalación.
- [[Scripts 05 · Tiempo: botones que cambian el flow]] — botones que **cambian el flow**: cada ciudad escribe sus coordenadas en Variables (MCP `variables_set`) y relanza el flow (`flow_run`); al terminar, el script se redibuja con la previsión nueva.

Los tres primeros siguen la regla de oro: un script hace `return` de un valor y ese valor pasa a ser una `{{variable}}`. Los dos últimos enseñan que un script también puede dibujar botones y páginas encima de FlowTest (corre dentro de la página, con todos sus permisos) — con higiene: una sola instancia, Shadow DOM, nada en localStorage, sin bucles ni notificaciones, cada acción es un clic tuyo. Ejecuta solo flows en los que confíes.

## ⌨️ Y todo esto sin abrir el navegador

```bash
flow-run --dir flows                              # toda la carpeta en tu CI
flow-run --flow mi-flow.flow.json --data casos.csv # un run por fila del CSV
flow-run --flow mi-flow.flow.json --env prod       # mismo flow, entorno prod
# exit ≠ 0 si falla un solo assert → el build se cae solo
```

---

¿Quieres el detalle de cada cosa? La documentación vive en [flowtest.es/docs](https://flowtest.es/docs), y el manual completo del formato está en la carpeta `docs/` de este mismo repo (`docs/flows-formato.md`).
