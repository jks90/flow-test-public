# 🧰 08 · Paneles y utilidades

> [!TIP]
> **¿Dónde está cada cosa?** Historial, Consola, Batch y GitHub se abren desde **Config**; Variables, Global, SQL Conns, **Pizarra** y **Guía de nodos** desde la **barra lateral de iconos** (un panel a la vez); y las utilidades de organización desde el desplegable **Layout**. Ver [01 La pantalla principal](01-pantalla-principal.md).

## Pizarra 🆕 (4.24)

![](assets/flowtest-30-pizarra.png)

Una capa de dibujo libre **sobre el lienzo**, al estilo Excalidraw: rodea grupos de cajas, escribe títulos y avisos, dibuja tus propias flechas. Se abre con el botón morado **Pizarra** de la toolbar o desde la barra lateral.

| Sección del panel | Qué hay |
|-------------------|---------|
| **Herramientas** | Seleccionar (`V`), Mano (`H`, mueve el lienzo), Lápiz (`P`), Línea (`L`), Flecha (`A`), Rectángulo (`R`), Elipse (`O`), Texto (`T`), Borrador (`E`). El candado **Fijar** mantiene la herramienta tras dibujar (por defecto las formas vuelven a Seleccionar; el lápiz se queda) |
| **Acciones** | Deshacer / Rehacer (también `Ctrl+Z` / `Ctrl+Shift+Z`), Ocultar/Mostrar los dibujos, Vaciar la pizarra |
| **Selección** | Con algo seleccionado: Duplicar (`Ctrl+D`), Eliminar (`Supr`), Traer al frente, Enviar al fondo |
| **Estilo** | 8 colores + personalizado, grosor, línea continua/discontinua/punteada, trazo **Boceto** (a mano alzada) o **Limpio**, relleno para rectángulos y elipses, fuente (manuscrita / normal / código) y tamaño para los textos, opacidad. Con una selección, el cambio se aplica a ella; si no, al siguiente dibujo |

Cómo se usa:

1. Elige una herramienta y **arrastra** sobre el lienzo. `Shift` restringe a cuadrado/círculo/ángulos de 45°.
2. **Texto**: clic donde quieras escribir, teclea (Enter hace salto de línea) y `Esc` o clic fuera para terminar. Doble clic sobre un texto existente lo edita.
3. **Seleccionar**: clic sobre un trazo (en formas sin relleno, sobre el borde) para moverlo o cambiarlo con los tiradores; `Shift` + clic suma a la selección.
4. `Esc` vuelve a Seleccionar y quita la selección.

Los dibujos comparten coordenadas y zoom con las cajas, se ven en el minimapa y **se guardan con el flow** (campo `drawings` del `.flow.json`, solo presente si hay dibujos). Viajan con export/import, GitHub Flows y MCP; el CLI los ignora. Con el panel cerrado, o en modo Seleccionar, las cajas funcionan exactamente igual que siempre (los dibujos quedan debajo de ellas).

> Ejemplo listo: [`examples/pizarra-anotada.flow.json`](../../examples/pizarra-anotada.flow.json).

## Guía de nodos 🆕 (4.24)

![](assets/flowtest-31-guia-nodos.png)

**Layout ▸ Guía de nodos** (o el icono de lista de la barra lateral) abre un índice de **todas las cajas del flow** agrupadas por tipo — Peticiones HTTP (método + URL), Consultas SQL (motor + query), Notas (texto / Mermaid / Captura) y Web (URL) — con su punto de estado, nº de orden y 📌. Tiene filtro de texto por nombre, URL o query. **Clic en un nodo → el lienzo se centra en él y la caja se resalta** un par de segundos. Imprescindible en flows de 20+ cajas.

## Ocultar conectores 🆕 (4.24)

**Layout ▸ Ocultar conectores** quita las líneas de conexión del lienzo para una vista limpia (o para dibujar tus propias flechas en la pizarra). Las conexiones siguen existiendo y la línea provisional al conectar dos cajas se sigue viendo; el mismo item pasa a **Mostrar conectores** para volver. Es un ajuste de vista: no se guarda en el flow ni afecta a la ejecución.

## Variables usadas en la caja (4.23)

Cada caja **request** y **SQL** tiene una franja plegable **VARIABLES USADAS** que lista las `{{variables}}` de su curl, query o conexión con su valor actual, **editable in situ**: lo que escribes va a las variables de entorno del flow (y pisa el valor runtime si lo había). La cabecera avisa de las que **no tienen valor** y oculta las que parecen secretos.

## 📌 Fijar posición (4.23)

Cada caja (request, SQL, nota, web) tiene un candado en su cabecera: fijada, **nada la mueve** — ni colapsar/expandir (resolución de solapes), ni Collapse/Expand All, ni alinear, ni el auto-layout, ni el arrastre. Se guarda en el `.flow.json` (`pinned`).

## Historial

Cada **Run Flow** queda registrado: por nodo, la petición enviada (método, URL, headers, body) y la respuesta completa con duración. Es tu caja negra para «¿qué se envió exactamente hace 10 minutos?».

![](assets/flowtest-08-historial.png)

## Consola

El log de la propia aplicación: nodos añadidos, ejecuciones, capturas, errores de parseo, comandos del MCP… Con niveles (info/success/warn/error) y filtro. Si algo «no hace nada», aquí suele estar el porqué.

![](assets/flowtest-09-consola.png)

## Batch Run

Ejecuta **varios flows en lote** (los seleccionas de tus pestañas o de archivos) y muestra el resultado de cada uno. La versión CI de esto es `flow-run --dir` ([09 El CLI flow-run](09-cli-flow-run.md)).

![](assets/flowtest-10-batch.png)

## GitHub Flows

Conecta con un repo de GitHub (token + owner/repo/rama/carpeta) para **guardar y cargar flows directamente del repo** — el equipo comparte flows sin pasarse archivos.

![](assets/flowtest-12-github.png)

## Export / Abrir / guardado

- Todo se **autoguarda en localStorage** del navegador al momento: cierra y abre tranquilo.
- **Export** descarga el `.flow.json` del flow activo (estado limpio: sin respuestas ni contraseñas de runtime). Es el formato que consumen el CLI, el Batch, GitHub Flows y el MCP.
- **Abrir** (en la barra de pestañas) importa un `.flow.json` en una pestaña nueva.
- El indicador ámbar **«Sin guardar»** junto al nombre avisa de cambios no exportados (los dibujos de la pizarra también cuentan).

## Collapse All · Expand All · Auto Layout · Alinear

![](assets/flowtest-28-collapse-compacto.png)

- **Reset**: limpia respuestas y estados (no borra nodos ni conexiones).
- **Collapse All**: colapsa todas las cajitas **y las acerca** — escala las distancias al ritmo al que encogen las cajas, **conservando tu disposición** (no reordena).
- **Expand All**: expande todo y **restaura las posiciones exactas** previas al Collapse All. Si expandes una cajita suelta, las de debajo se **empujan** para no pisarse.
- **Auto Layout**: reordena por el grafo — la cadena `next` en columnas, y los nodos colgados con conexión informativa (`none`) como satélites bajo su padre. Ordena **todos** los tipos de nodo.
- **Alinear en fila / en columna**: coloca los nodos según su **nº de orden** (campo `#` de cada cabecera; los que no tienen número van al final, en orden de lectura).
- Las cajas **📌 fijadas** no se mueven con ninguna de estas acciones.

## Cron

Los nodos Request, SQL, Web y Nota tienen pestaña **Cron**: ejecución periódica del nodo (cada X min/h) mientras la pestaña esté abierta. El badge con cuenta atrás aparece en la cabecera del nodo.
