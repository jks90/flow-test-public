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

## ⚙️ Funciones que pagan el café

Lo que convierte un juguete visual en una herramienta que trabaja mientras duermes:

- [[Monitor 24x7]] — el servidor ejecuta tu flow cada N minutos sin navegador y te avisa por webhook cuando algo falla: enterarte tú antes que tus usuarios.
- [[Webhook desde CI]] — tu pipeline lanza el flow con un simple `POST /hook/<token>` y recibe el resultado por nodo: tests de API en Jenkins o GitHub Actions sin instalar nada más.
- [[Correo OTP]] — probar un registro con código de verificación sin cuentas de correo reales: el SMTP embebido captura el email y te extrae el OTP como variable.
- [[Puente a local]] — tu API vive en `localhost` y el flow corre en Docker: el puente `host.docker.internal` la alcanza igual, y aquí lo ves funcionando.

## 🏠 FlowTest por dentro

Nuestra propia plataforma, probada con nuestra propia herramienta — estos cuatro flows atacan los endpoints reales de flowtest.es y app.flowtest.es, y el de compra es literalmente nuestro funnel de venta:

- [[Solicitar espacio cloud]] — el alta de una organización cloud, de verdad (lee antes el LEEME de la carpeta: crea cuentas reales).
- [[Login y sesión]] — credenciales → token → perfil → logout contra el servicio de cuentas.
- [[Comprar licencia]] — planes y checkout: el mismo camino que recorre un cliente que paga.
- [[Salud de la plataforma]] — health checks de la web, las cuentas y el TLS; el flow que nosotros mismos monitorizamos.

Más detalle (y el aviso importante) en `flowtest-por-dentro/00 LEEME.md`.

## 🗺️ Paneles para enseñar en una demo

Para cuando quieras impresionar en pantalla grande:

- [[Crypto portfolio]] — precios en vivo encadenados y calculados sobre el canvas.
- [[Mapa mundial]] — nodos colocados sobre una imagen de fondo: APIs de países y divisas repartidas por el mapa.

Y si quieres el ejemplo XXL, el bundle `economia-global-bundle` del repo (en `../economia-global-bundle`) es un workspace completo de economía global con decenas de nodos, SQL y documentación enlazada.

## ⌨️ Y todo esto sin abrir el navegador

```bash
flow-run --dir flows                              # toda la carpeta en tu CI
flow-run --flow mi-flow.flow.json --data casos.csv # un run por fila del CSV
flow-run --flow mi-flow.flow.json --env prod       # mismo flow, entorno prod
# exit ≠ 0 si falla un solo assert → el build se cae solo
```

---

¿Quieres el detalle de cada cosa? La documentación vive en [flowtest.es/docs](https://flowtest.es/docs), y el manual completo del formato está en la carpeta `docs/` de este mismo repo (`docs/flows-formato.md`).
