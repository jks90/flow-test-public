# 📱 12 · La vista móvil

Desde la **5.11**, FlowTest trae una segunda cara pensada para el teléfono: entra en
**`http://tu-servidor:9998/m`** y tienes tus flows para **ejecutarlos, ver el resultado y leerlos como
documentación**. Editar sigue siendo cosa del escritorio.

No es la aplicación de siempre encogida: es una **app aparte**. El editor (canvas, pizarra, arrastrar
cajas) no se descarga en el móvil, así que abre rápido incluso con datos.

> Si entras a `http://tu-servidor:9998` desde un teléfono, te lleva solo a `/m` **antes** de descargar
> el editor. Para ver el escritorio en un móvil o una tablet, usa `?desktop=1` (se recuerda) o el
> enlace «Abrir la versión de escritorio» del pie.

## Llegar desde el teléfono

El móvil tiene que poder ver la máquina donde corre FlowTest. En una red doméstica u oficina basta la
IP del servidor:

```bash
# en la máquina del contenedor
hostname -I | awk '{print $1}'   # p. ej. 192.168.1.50
# en el teléfono: http://192.168.1.50:9998/m
```

Añádela a la **pantalla de inicio** y se abre como una aplicación, sin barras del navegador.

## La lista

![Lista de flows en el móvil](assets/flowtest-69-movil-lista.png)

Los flows del proyecto, en un **árbol de carpetas** y con buscador. Cada fila dice cuántos pasos tiene, si
lleva notas y cuándo se tocó por última vez.

**Desde la 5.30.2** cada carpeta muestra solo su nombre, con sangría dentro de la suya, y a la derecha
el total de flows que hay en ella y en sus subcarpetas. De entrada todas están plegadas, para ver el
proyecto de un vistazo; las que abres se recuerdan. Una carpeta que solo contiene otra se junta con
ella en una fila («almadance / flows»). Al buscar se abre todo para que salgan los resultados.

**Desde la 5.30.3** la lista **no enseña las carpetas que ocultaste** en el panel Proyecto del
escritorio ([08](08-paneles.md#panel-proyecto-a-medida--521)): esa lista se guarda en el servidor y
la comparten los dos. Al final aparece «N carpetas ocultas en el escritorio · mostrar» para verlas
atenuadas un momento.

**Desde la 5.30.4** la lista enseña también los **documentos** del proyecto, igual que el panel
Proyecto: `.md`, `.pdf`, scripts (`.js`, `.ts`, `.py`, `.sh`, `.sql`…) y datos `.csv`/`.tsv`, después de
los flows de cada carpeta y con su icono. Al tocarlos se abren en solo lectura con los mismos visores
del escritorio: el Markdown renderizado (sus `[[enlaces]]` abren el flow), el script con su código y el
CSV como tabla. El **PDF** se abre con el visor del teléfono (botón «Abrir el PDF», función Pro).

Si tu instalación está **vinculada a una cuenta cloud**, arriba aparece el selector **💻 Local /
☁️ Cloud** y la lista arranca en el entorno que gobierna la instalación, igual que el panel Proyecto
del escritorio.

## Documentación de un flow

![Documentación de un flow en el móvil](assets/flowtest-70-movil-documentacion.png)

La pestaña **Documentación** enseña el flow como se lee, no como se dibuja:

- **Notas** con su texto, sus diagramas Mermaid o sus capturas. En un flow con muchas notas llegan
  plegadas y se abren al tocarlas.
- **Pasos en orden de ejecución**, numerados, con su método, su dirección y los asserts que tiene cada
  uno. Es el mismo orden en que los va a lanzar el runner.
- **Variables** y el entorno activo.

Sirve para consultar en una reunión qué comprueba un flujo, o para enseñárselo a alguien sin abrir el
portátil.

## Ejecutar y ver el resultado

![Resultado de una ejecución en el móvil](assets/flowtest-71-movil-resultado.png)

La pestaña **Ejecución** lo lanza en el servidor, con el mismo motor que usan el CLI, los webhooks y
los monitores. Antes de ejecutar puedes:

- elegir **entorno** (dev, pre, pro… los que tenga el flow),
- **cambiar variables solo para esa ejecución**, sin tocar el fichero.

Mientras corre ves una **barra de progreso**, la **salida del runner en directo** y los pasos que ya
han terminado marcados en verde o en rojo. Al acabar, cada nodo muestra su código, el tiempo, los
asserts, las variables extraídas y el cuerpo de la respuesta (plegado, con botón de copiar). El botón
**Resumen** copia el resultado en texto para pegarlo donde haga falta.

Si te sales de la pantalla a mitad, la ejecución se corta en el servidor: no quedan procesos sueltos.

## Lo que no hace

- **No se edita.** Para tocar un flow, el escritorio.
- **Ejecutar requiere plan Pro**, igual que el CLI y los webhooks. Durante la prueba de 14 días la
  documentación se ve entera y el botón de ejecutar queda desactivado.
- Un flow que vive **en tu cloud** se puede leer, pero para ejecutarlo con el motor de esta
  instalación el botón te ofrece **traerlo** a su disco primero.
- Los documentos `.md` y `.pdf` del proyecto, y el borrado de ficheros, siguen siendo del escritorio.


## AgentOffice en la app Android 🆕 (5.30.2)

La app Android muestra un **botón redondo 🤖** abajo a la derecha. Al tocarlo abre **AgentOffice** del
mismo servidor en una vista pensada para el teléfono, y el botón pasa a **⚡** para volver a FlowTest.
También se abre manteniendo pulsado el icono de la app ▸ **Agentes**.

- **Resumen:** tareas en curso, en revisión y por hacer; el equipo del proyecto con lo que hace cada
  uno; la cuota de Claude y Codex; y los demás proyectos.
- **Bandeja:** las preguntas de los agentes (con sus botones de respuesta), las tareas para revisar
  con **✓ Aprobar** / **↩ Devolver** y las propuestas. La pestaña lleva un número con lo pendiente.
- **Tareas:** filtros (activas, en curso, revisión, fallidas, backlog, hechas), detalle de cada una
  con su resumen y sus cambios, **Reintentar** las fallidas y **＋ Nueva tarea**.
- **Guía:** el chat con el Guía. Con el **🎤** le hablas y lo dictado se envía.

**🎤 Dictado (5.30.4):** en la app, los campos de AgentOffice llevan un **🎤** que abre el
reconocedor de voz del teléfono (en español) y escribe lo que dices. En **＋ Nueva tarea**,
**«🎤 Dictar la tarea»** te deja contarla con tus palabras y la pasa por **✨ Redactar con IA**, que
propone título, descripción y quién la hace; la revisas y pulsas **Crear**. También hay 🎤 al
devolver una tarea y al responder una pregunta. (En un navegador por HTTP no hay micro: ahí se dicta
con el 🎤 del teclado.)

Funciona con el FlowTest de tu red (el PC o un servidor) que tenga AgentOffice como plugin
([08](08-paneles.md#plugins-y-agentoffice--518)); en el cloud no hay AgentOffice. Si el AgentOffice
es anterior y no tiene la vista móvil, se abre su web completa. Mantener pulsado el botón y
arrastrarlo lo cambia de altura.

### ＋ Nuevo proyecto 🆕 (5.31.0)

Ya no hace falta ir al escritorio para crear un proyecto de AgentOffice:

- **En el escritorio** el botón **＋ Nuevo proyecto** está siempre visible, con texto, junto al
  selector de proyectos.
- **En el móvil** (vista de AgentOffice) lo tienes en el **Resumen** y como última opción del
  selector de proyectos de arriba (**＋ Nuevo proyecto…**).

La hoja del móvil pide solo dos cosas: el **nombre** y la **carpeta del workspace** (se propone a
partir del nombre y se crea si no existe). Se crea sin repositorio y con el equipo y el
coordinador de serie; el repositorio y el resto de ajustes se añaden después desde la web. Avisa si
el nombre está vacío o ya existe, o si falta la carpeta; al crear, el proyecto queda seleccionado y
se abre su Resumen.

*(Sin captura: la pantalla requiere un entorno de demo con datos inventados.)*

## La pestaña ✨ IA 🆕 (5.13.1)

El detalle de un flow tiene una tercera pestaña, **✨ IA**: el mismo asistente del escritorio ([13](13-asistente-ia.md)) desde el teléfono. Con cada mensaje la IA recibe un resumen del flow que estás leyendo (pasos en orden de ejecución, variables, asserts), así puedes pedirle que te lo explique, que te diga qué mejorarías o que revise el último fallo. Sin el escritorio abierto **no edita nada** (no hay canvas): explica y propone. Si tienes el escritorio abierto en el mismo servidor, las herramientas actúan sobre esa pestaña y lo ves llegar allí, con las mismas confirmaciones (Sí/No) y el botón de parar. La clave, el proveedor y el modelo son los del servidor: se configuran en el escritorio (panel ✨ ▸ ⚙). Función Pro.

## Reportar un problema desde el móvil 🆕 (5.14)

El botón 💬 de la barra superior (en la lista y dentro de un flow) abre el mismo formulario que en el escritorio: tipo (error / idea / pregunta), título, descripción, **foto de la cámara o de la galería** y datos técnicos opcionales (versión, teléfono, flow abierto). La pestaña «Mis reportes» muestra el estado y la respuesta del equipo. Es una función Pro/Business; en la prueba el formulario se ve pero no envía. Detalles en [14](14-reportar-problemas.md).

![](assets/flowtest-78-reportar-movil.png)
