# Licencias del software incluido en la imagen `juankanh/flow-app`

Flow es **software propietario** (ver [LICENSE](LICENSE)). La imagen Docker incluye además
componentes de terceros, todos bajo licencias permisivas:

- **Dependencias de Node.js** (runtime del servidor y del CLI): MIT, ISC, Apache-2.0,
  BSD-2/3-Clause y equivalentes. Cada paquete conserva su fichero de licencia dentro de
  `node_modules/` en la propia imagen.
- **Fuentes tipográficas de la pizarra**: SIL Open Font License 1.1 (Excalifont — © Excalidraw —,
  Patrick Hand, Caveat, Indie Flower, Cabin Sketch, Architects Daughter, Gloria Hallelujah) y
  Apache-2.0 (Permanent Marker). El texto completo de los avisos se sirve en la propia app:
  `http://localhost:9998/licencias-fuentes.txt`.
- **Chromium** (BSD-3-Clause y licencias de sus componentes) y **Node.js** (MIT), instalados desde
  los repositorios de la imagen base.

## Nota histórica: ffmpeg en imágenes ≤ 5.4.0

Las imágenes hasta la **5.4.0** incluían un binario estático de **ffmpeg** (GPL-3.0) a través del
paquete `ffmpeg-static`, usado por una función interna de QA retirada en la 5.5.0. En cumplimiento
de la GPL, el código fuente completo correspondiente a ese binario está disponible en:

- Paquete: <https://github.com/eugeneware/ffmpeg-static> (builds de <https://johnvansickle.com/ffmpeg/>)
- Código fuente de ffmpeg: <https://ffmpeg.org/download.html#get-sources>

La GPL aplicaba **solo a ese binario** (se invocaba como programa externo, sin enlazar con el código
de Flow): no afecta a la licencia propietaria de Flow ni a nada más de la imagen. **Desde la 5.5.0
la imagen no contiene ningún componente GPL/LGPL/AGPL.**
