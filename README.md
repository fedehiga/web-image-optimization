# web-image-optimization

## webp

Una imagen webp puede tener las siguientes características:
- *Lossless*: sin pérdida de calidad
- *Lossy*: con pérdida de calidad
- *Alpha*: soporta canal alpha para transparencias (como los PNG 24 bits)
- *Animated*: soporta animación (como los .GIF)

### SOPORTE

Soporte de browsers de webp lossless, lossy, alpha y animated:

#### Desktop

- **Chrome**: desde la versión *23* (animated desde la v. *32*)
- **Firefox**: desde la versión *65*
- **Safari**: desde la versión *14* (y sólo a partir de OS Big Sur) (soporte reciente)
- **IE**: No
- **Edge**: desde la versión 18
- **Opera**: desde la versión 12.1 (animated desde la v. *19*)

#### Mobile

- **Android Browser**: desde la versión *4* (animated desde la v. *4.2*)
- **Chrome for Android**: desde la versión *23*
- **Safari iOS**: desde la versión *14* (soporte reciente)
- **Samgung Internet**: desde la versión 4

[Fuente](https://caniuse.com/webp)

### IMPLEMENTACIÓN

#### Via HTML

```html
<picture>
    <source srcset="images/mypicture.webp" type="image/webp">
    <source srcset="images/mypicture.jpg" type="image/jpeg">
    <img src="images/mypicture.jpg" alt="...">
</picture>
```

Fallback: si el browser no soporta webp ni srcset, el termina descargando la última línea ``<img src="...">``.

#### Via CSS

Lo primero es detectar si el browser soporta webp. Si no lo soporta, la idea es servirle la imagen jpg o png original. La detección puede hacerse de varias maneras.

##### DETECCIÓN vía Modernizr JS

Esta lib se utiliza para saber qué tipo de características soporta (o no) el browser del usuario.

- Ir al [website](https://modernizr.com/download?webp-setclasses&q=webp)
- Elegir las características *Webp* y *Webp Lossless* (y *Webp Animation y Webp Alpha* opcionalmente), luego "Build" y download. Se descarga *modernizr-custom.js*.
- Luego de incluir ese script en una página, el resultado es una clase html agregada en el tag ``<html>``: ``webp``en el caso de que el browser soporte webp, y ``no-webp`` en el caso contrario. Entonces:

```css
.no-webp .element {
  background-image: url("image.jpg");
}

.webp .element {
  background-image: url("image.webp");
}
```

Opcionalmente por JS:

```js
 Modernizr.on('webp', function (result) {
  if (result) {
    // supported
  } else {
    // not-supported
  }
});
```

#### DETECCIÓN vía JS propio

La idea es intentar decodificar una pequeñísima imagen webp, y chequear e éxito de ese intento:

```js
// check_webp_feature:
//   'feature' can be one of 'lossy', 'lossless', 'alpha' or 'animation'.
//   'callback(feature, result)' will be passed back the detection result (in an asynchronous way!)
function check_webp_feature(feature, callback) {
    var kTestImages = {
        lossy: "UklGRiIAAABXRUJQVlA4IBYAAAAwAQCdASoBAAEADsD+JaQAA3AAAAAA",
        lossless: "UklGRhoAAABXRUJQVlA4TA0AAAAvAAAAEAcQERGIiP4HAA==",
        alpha: "UklGRkoAAABXRUJQVlA4WAoAAAAQAAAAAAAAAAAAQUxQSAwAAAARBxAR/Q9ERP8DAABWUDggGAAAABQBAJ0BKgEAAQAAAP4AAA3AAP7mtQAAAA==",
        animation: "UklGRlIAAABXRUJQVlA4WAoAAAASAAAAAAAAAAAAQU5JTQYAAAD/////AABBTk1GJgAAAAAAAAAAAAAAAAAAAGQAAABWUDhMDQAAAC8AAAAQBxAREYiI/gcA"
    };
    var img = new Image();
    img.onload = function () {
        var result = (img.width > 0) && (img.height > 0);
        callback(feature, result);
    };
    img.onerror = function () {
        callback(feature, false);
    };
    img.src = "data:image/webp;base64," + kTestImages[feature];
}
```

Nota: la carga es non-blocking y asynchronous, cualquier código que dependa del soporte de webp debe ubicarse preferentemente en la callback function.

### CONVERSIÓN a formato .webp

#### Via Photoshop plugin
[Photoshop plugin](http://telegraphics.com.au/sw/product/WebPFormat#webpformat)

#### Via cwebp command line tool

[Google cwebp command line tool](https://developers.google.com/speed/webp/docs/using)
[Download repository](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/index.html)

Ej:

```bash
cwebp -q 80 image.png -o image.webp
```
Chequear la [doc](https://developers.google.com/speed/webp/docs/cwebp).

#### Via Node JS

Usando el package [imagemin](https://www.npmjs.com/package/imagemin) y el plugin [imagemin-webp](https://www.npmjs.com/package/imagemin-webp):

```bash
  npm install imagemin imagemin-webp
```

Luego creamos un archivo *webp.js*:

```js
const imagemin = require('imagemin'),
  webp = require('imagemin-webp')
const outputFolder = './images/webp'
const produceWebP = async () => {
  await imagemin(['images/*.png'], {
    destination: outputFolder,
    plugins: [
      webp({
        lossless: true
      })
    ]
  })
  console.log('PNGs processed')
  await imagemin(['images/*.{jpg,jpeg}'], {
    destination: outputFolder,
    plugins: [
      webp({
        quality: 65
      })
    ]
  })
  console.log('JPGs and JPEGs processed')
}
produceWebP()
```

Y ejecutamos el script:

```bash
node webp.js
```

El script procesa todas las imágenes  en la carpeta ``img`` y las convierte a webp.
Cuando convertimos PNG, seteamos ``lossless: true``.
Cuando convertimos JPG, lo hacemos con ``quality: 65``.
Experimentar con más opciones documentadas en la [página del plugin](https://www.npmjs.com/package/imagemin-webp).



