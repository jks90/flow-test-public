# 🏠 FlowTest por dentro — léeme antes de ejecutar

La plataforma tiene tres piezas, y estos flows tocan las tres:

| Pieza | Qué es |
|-------|--------|
| **flowtest.es** | La web pública y la documentación |
| **app.flowtest.es** | Cuentas, organizaciones y espacios cloud (el servicio de cuentas) |
| **Tu instalación** | La imagen Docker `juankanh/flow-app` + el CLI `flow-run` — donde estás ahora |

> ⚠️ **AVISO:** [[Solicitar espacio cloud]] hace un `POST /api/register` **real** contra app.flowtest.es: crea una cuenta y una organización de verdad, con verificación por correo incluida. Sustituye antes los placeholders (`tu-email@ejemplo.com`) por un email tuyo, ejecútalo una sola vez y no lo metas en un monitor ni en un data-driven. El resto de flows de la carpeta son de solo lectura y puedes lanzarlos cuantas veces quieras.

## Los 4 flows de esta carpeta

| Flow | Endpoints que toca | Qué demuestra |
|------|--------------------|----------------|
| [[Solicitar espacio cloud]] | `POST /api/register` (+ verificación por correo) | El alta real de una organización cloud |
| [[Login y sesión]] | `POST /api/login` → `GET /api/me` → `POST /api/logout` | Credenciales → token → sesión, encadenado con extracciones |
| [[Comprar licencia]] | `GET /api/plans` y el checkout | Nuestro funnel de compra real, convertido en test |
| [[Salud de la plataforma]] | `GET /health` de cuentas, la web y `GET /api/tls-check` | El health check que nosotros mismos monitorizamos |

Nuestra propia plataforma, probada con nuestra propia herramienta: si un día [[Salud de la plataforma]] se pone rojo, nos hemos enterado antes que tú.

## Los dos flows nuevos de la carpeta

| Flow | Qué toca |
|------|----------|
| [[Automatiza tu FlowTest]] | `GET /monitors` y `GET /access` de TU instalación (localhost:9998) — la API local |
| [[Puente cloud (flow-bridge)]] | `GET app.flowtest.es/api/bridge/poll` — el endpoint real del puente cloud→local (401 con token falso, como debe ser) |
