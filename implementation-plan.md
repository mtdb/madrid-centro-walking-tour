# Plan de implementación — Madrid centro walking tour

## Objetivo

Construir una guía web móvil y autoguiada para recorrer Madrid centro a pie desde el parking de Plaza Mayor y regresar al mismo punto. La experiencia debe seguir el lenguaje editorial y la arquitectura de `../alcala-walking-tour`: una página larga, visual, rápida, usable desde el móvil mientras se camina y capaz de seguir funcionando con conectividad limitada.

La web no será una aplicación de mapas completa. El contenido, las fotografías, las instrucciones y los enlaces a aplicaciones de navegación deben ser útiles por sí mismos; Google Maps, OsmAnd u otra aplicación se abrirán únicamente como apoyo.

## Análisis del proyecto de referencia

### Arquitectura técnica

`../alcala-walking-tour` es una PWA estática en JavaScript nativo:

- `index.html` contiene la carcasa semántica: barra fija, hero, introducción, introducción de la ruta, contenedor dinámico de paradas, pie y dos diálogos.
- `src/app.js` concentra los datos y la lógica. `GLOSSARY` y `STOPS` son las fuentes de contenido; las paradas se renderizan mediante `stopTemplate()`.
- `styles.css` implementa todo el diseño sin framework ni preprocesador.
- `manifest.webmanifest` permite instalar la guía como aplicación.
- `sw.js` precarga los recursos principales y aplica caché de red con fallback a `index.html`.
- `package.json` solo aporta `type: module` y el chequeo sintáctico `node --check src/app.js`; no hay bundler ni dependencia de runtime.
- `tour.md` funciona como documento editorial y fuente de verdad para el contenido, separado de la presentación.
- `IMAGE_CREDITS.md` documenta las atribuciones de imágenes con licencia abierta.

### Lenguaje visual que debe conservarse

- Estética editorial sobria, inspirada en una guía cultural impresa.
- Fondo papel cálido (`#f2ede4`), tinta casi negra (`#25231f`) y acentos terracota, verde, azul y rosa apagados.
- `Playfair Display` para titulares y `DM Sans` para interfaz y texto corrido.
- Hero fotográfico a pantalla completa con velo oscuro y titular grande.
- Barra superior fija translúcida con marca, parada activa, progreso y ajustes.
- Introducción oscura de dos columnas con contexto histórico, datos destacados, curiosidad y línea temporal.
- Cada parada ocupa aproximadamente una pantalla en escritorio y se convierte en una tarjeta vertical en móvil.
- Alternancia cromática de paradas; número grande de baja opacidad; imagen con marco desplazado y parallax muy sutil.
- Bloques diferenciados para “Hitos en el tiempo”, “Fíjate en esto”, contexto y curiosidad.
- Acciones claras: navegar a la siguiente parada, abrir el lugar, copiar coordenadas y marcar como visitada.
- Respeto por `prefers-reduced-motion` y disposición móvil específica a partir de 720 px.

### Funcionalidad que conviene reutilizar

1. Estado local con `localStorage` para paradas visitadas, proveedor de mapas y parada opcional.
2. `IntersectionObserver` para actualizar en la barra superior la parada que está en pantalla.
3. Diálogo de selección de proveedor: OsmAnd~, Google Maps, preguntar siempre o copiar coordenadas.
4. Enlaces de lugar y navegación peatonal con coordenadas.
5. Copia de coordenadas con fallback a `prompt`.
6. Glosario contextual mediante botones accesibles y `dialog` nativo.
7. Descarga de GPX generada en el navegador a partir de los waypoints.
8. Imágenes con `loading="lazy"`, dimensiones explícitas, texto alternativo y atribución visible cuando corresponda.
9. Registro del service worker después de cargar la página.

### Riesgos o puntos a mejorar respecto al referente

- La ruta de Madrid es circular, no lineal. El texto y el botón final deben explicar el regreso al parking y no prometer una “siguiente parada” inexistente.
- El recorrido tiene 11 paradas editoriales frente a las 8 principales de Alcalá. Hay que limitar la cantidad de texto por parada para que el paseo no se convierta en una lectura interminable.
- La navegación actual usa la posición actual del dispositivo como origen. Mantener ese comportamiento para el uso real en la calle, pero ofrecer también un enlace explícito “volver al parking” desde Debod.
- El proveedor por defecto `ask` es correcto para la primera visita, pero la preferencia debe persistir con una clave propia de Madrid para no mezclar datos con una instalación de Alcalá.
- El caché del service worker debe versionarse con un nombre distinto y contener todos los recursos de Madrid; nunca reutilizar `alcala-tour-v3`.
- El proyecto de referencia contiene cambios sin confirmar en su árbol de trabajo. Copiar el patrón, no archivos a ciegas; tomar como fuente de diseño la versión inspeccionada y verificar cada adaptación.

## Alcance de la primera versión

### Recorrido base revisado

La prioridad revisada es que el circuito termine cerca del coche. Debod deja de ser el final y se convierte en una parada intermedia; el tramo final será urbano, descendente y corto desde Sol hasta el Mesón del Champiñón y el parking.

1. Parking y Plaza Mayor.
2. Mercado de San Miguel.
3. Plaza de la Villa y calles del Madrid de los Austrias.
4. Catedral de la Almudena, Plaza de la Armería y Mirador de la Cornisa.
5. Palacio Real.
6. Plaza de Oriente, Teatro Real y Ópera como un único conjunto.
7. Templo de Debod y Parque del Oeste, llegando por Bailén/Cuesta de San Vicente/Paseo de Rosales.
8. Plaza de España y Gran Vía, en dirección este.
9. Callao y Gourmet Experience de El Corte Inglés.
10. Puerta del Sol, llegando por Preciados.
11. Mesón del Champiñón y parking de Plaza Mayor.

El trazado recomendado es:

`Plaza Mayor → Mercado de San Miguel → Plaza de la Villa → Almudena → Palacio Real → Plaza de Oriente/Ópera → Cuesta de San Vicente → Debod → Plaza de España → Gran Vía → Callao → Preciados → Sol → Calle Mayor → Plaza Mayor → Cava de San Miguel`

Así no se reserva Debod para el atardecer ni se obliga al visitante a caminar 30–35 minutos después de la última parada monumental. Desde Sol hasta el Mesón son aproximadamente 10–12 minutos; desde el Mesón, el parking queda a pocos minutos. La puesta de sol en Debod será una posibilidad, no el eje temporal del paseo.

La calle Arenal deja de ser una parada específica: obligaría a volver hacia Ópera o a desviar el circuito. Ópera se conserva integrada en el conjunto Plaza de Oriente–Teatro Real y como conexión hacia Debod.

### Cambios de implementación derivados de la revisión

- `src/app.js`: cambiar el orden visible a `[Plaza Mayor, Mercado, Villa, Almudena, Palacio, Oriente/Ópera, Debod, Gran Vía/Plaza de España, Callao, Sol, Champiñón]` mediante `ROUTE_ORDER`.
- `src/app.js`: convertir la actual parada “Ópera, Arenal y Sol” en “Puerta del Sol” y trasladar Ópera al subtítulo/detalle de Plaza de Oriente; eliminar Arenal como destino de navegación.
- `src/app.js`: cambiar Debod de parada final a parada con siguiente destino Plaza de España; retirar su bloque especial de regreso al parking.
- `src/app.js`: mantener el Mesón como última parada y conservar su acción `Volver al parking`; el GPX debe seguir el nuevo orden.
- `index.html`: sustituir las menciones a Debod como final por “parada intermedia” y explicar que el circuito termina junto al coche.
- `walking-tour.md`: reescribir el itinerario y el horario hacia atrás desde la hora prevista de salida, no desde la puesta de sol; dejar Debod como visita flexible.
- `implementation-plan.md`: conservar la Galería opcional después del Palacio y verificar que no desplace la llegada a Debod/Plaza de España fuera de los horarios de Callao.
- Pruebas: revisar enlaces “siguiente”, progreso, numeración, GPX, paradas visitadas y navegación final con y sin Galería activa.

La distancia, duración, pendientes y tiempos de visita deben presentarse como estimaciones, no como una promesa exacta. El plan editorial debe distinguir entre paseo exterior, comida y visita interior del Palacio Real.

### Parada opcional

Implementar el mismo patrón de “Añadir parada opcional” del referente para la **Galería de las Colecciones Reales**, insertada entre Palacio Real y Plaza de Oriente. Debe advertir que añade tiempo y que sus horarios/entradas deben comprobarse antes de salir.

La visita interior del Palacio Real, la cúpula/museo de la Almudena y la terraza de El Corte Inglés son detalles de sus paradas obligatorias, no paradas opcionales separadas.

## Modelo de contenido

### Estructura de cada objeto `STOPS`

Mantener el esquema del referente y adaptar los campos a Madrid:

```js
{
  id: 1,
  slug: "plaza-mayor",
  name: "Plaza Mayor",
  shortName: "Plaza Mayor",
  subtitle: "Inicio · parking y Madrid de los Austrias",
  coordinates: { lat: 40.4154, lng: -3.7074 },
  duration: "15–20 min",
  price: "Exterior gratuito",
  description: "...",
  curiosity: "...",
  timeline: [{ date: "...", label: "...", text: "..." }],
  details: [{ kind: "look", label: "Fíjate en esto", title: "...", text: "...", extra: "..." }],
  source: "https://...",
  sourceExtra: "https://...",
  sourceExtraLabel: "...",
  image: "images/plaza-mayor.webp",
  alt: "...",
  imageWidth: 1600,
  imageHeight: 1067
}
```

### Contenido que debe preparar la fase editorial

Para cada parada completar:

- Nombre y subtítulo orientados a la acción.
- Texto principal breve, de dos o tres párrafos como máximo.
- Una curiosidad verificable y relevante para quien está allí.
- Uno o varios hitos temporales, diferenciando fecha documentada, periodo aproximado, edificio actual y transformación posterior.
- Un bloque “Fíjate en esto” que dirija la mirada a un elemento visible.
- Información práctica: duración, coste, accesibilidad, baños o restricciones cuando sea pertinente.
- Coordenadas exactas del punto donde conviene situarse, no solo de la plaza completa.
- Fuente oficial principal y, solo si aporta valor, una segunda referencia.

### Glosario inicial

Crear un `GLOSSARY` específico para términos que aparecerán varias veces, por ejemplo:

- Austrias / Habsburgo.
- ZBEDEP.
- Colegiata.
- Cornisa.
- Almudena.
- Real sitio.
- Teatro Real.
- Almadraba.
- Gourmet Experience.

No convertir en botones todas las palabras históricas: el glosario debe resolver dudas reales sin interrumpir la lectura.

## Plan de archivos

### Archivos nuevos

- `index.html`: carcasa de la PWA de Madrid, metadatos SEO y accesibilidad, hero, introducción, ruta, diálogos y pie.
- `styles.css`: copia evolucionada del sistema visual de Alcalá con paleta y posibles tratamientos fotográficos de Madrid.
- `src/app.js`: `GLOSSARY`, `STOPS`, estado, renderizado, navegación, progreso, GPX, glosario y parallax.
- `manifest.webmanifest`: nombre, descripción, colores, icono y `start_url` de Madrid.
- `sw.js`: caché versionada, por ejemplo `madrid-centro-tour-v1`, con todos los recursos locales.
- `icons/compass.svg`: reutilizar el icono si encaja; si la identidad requiere diferenciar ambas guías, crear una variante con licencia y documentarla.
- `images/*.webp`: hero y una imagen por parada; imágenes secundarias solo cuando expliquen una comparación o un detalle arquitectónico.
- `IMAGE_CREDITS.md`: atribución de cada imagen externa, autor, fuente, licencia y transformación realizada.

### Archivos editoriales

- `tour.md`: conservar y ampliar la ruta ya preparada en este directorio, incorporando el relato, las decisiones de orden, los tiempos, el regreso y las fuentes.
- `implementation-plan.md`: este plan; no debe convertirse en contenido mostrado al visitante.

### Configuración

- `package.json`: nombre `madrid-centro-walking-tour`, versión inicial `1.0.0`, `type: module` y script `check` con `node --check src/app.js`.
- No introducir framework, bundler, gestor de estado ni SDK de mapas en la primera versión.

## Estructura de `index.html`

Conservar la jerarquía probada del referente:

1. `header.topbar`
   - Marca “Madrid / paseo a pie”.
   - Etiqueta de parada activa.
   - Progreso `0 / 11` o `0 / 12` cuando se incluye la Galería.
   - Ajustes de proveedor de mapas.
2. `main#inicio`
   - Hero con Plaza Mayor o Templo de Debod al atardecer.
   - Datos: 5–6 km, 6–7 h con comida, circuito, parking ECO.
   - Botón “Empezar la ruta”.
   - Introducción histórica breve: Madrid de los Austrias, capital de los Borbones y eje moderno de Gran Vía.
   - Línea temporal corta, sin convertirla en una clase de historia.
   - Introducción del recorrido circular y aviso de que Debod es una pausa intermedia, no el final obligatorio.
   - `div[data-stops]` para las paradas dinámicas.
3. `footer`
   - Cierre con el regreso al parking.
   - Descargar GPX.
   - Enlace a `tour.md`/notas editoriales si se desea mantenerlo visible.
4. `dialog[data-map-dialog]` y `dialog[data-glossary-dialog]`.

Usar atributos ARIA, `alt` descriptivos, foco visible y botones reales para acciones. No depender de iconos sin etiqueta.

## Lógica de `src/app.js`

### Datos y estado

Definir claves aisladas de Madrid:

```js
const KEYS = {
  visited: "madrid-centro-tour:visited-stops",
  provider: "madrid-centro-tour:map-provider",
  optional: "madrid-centro-tour:include-gallery"
};
```

El estado mínimo será `{ visited, provider, optional, active }`. Al activar la Galería, insertarla en la secuencia después del Palacio Real y recalcular numeración visible, progreso, GPX y destino de “siguiente parada”.

### Navegación

- `place`: abrir el punto actual en Google Maps, OsmAnd o `geo:`/copiar coordenadas.
- `navigate`: abrir navegación peatonal hacia la siguiente parada.
- `return`: acción específica del cierre gastronómico hacia el parking de Plaza Mayor. Debod tendrá navegación normal hacia Plaza de España; ya no debe contener un bloque de regreso al parking.
- Conservar `ask` como opción persistente para que el usuario pueda elegir aplicación la primera vez.
- Verificar que el enlace de Google Maps tenga `travelmode=walking`.
- El GPX debe contener todos los waypoints visibles en el orden real, incluyendo la Galería si está activa.

### Renderizado

Reutilizar `stopTemplate()` con estos ajustes:

- Etiquetas de parada en español y número visible.
- El bloque “Siguiente” debe indicar también distancia o tiempo al siguiente punto cuando esté disponible.
- La última parada debe mostrar “Volver al parking” después del cierre en el Mesón, en vez de “Fin del paseo” sin acción.
- Añadir detalles específicos para: comida en San Miguel, Debod como parada intermedia, acceso a la terraza de Callao y cierre de cerveza y champiñones.
- Sanitizar o mantener estático todo contenido interpolado; no aceptar HTML introducido por el usuario.

### Interacción y rendimiento

- Mantener `IntersectionObserver`, parallax con `requestAnimationFrame` y respeto a `prefers-reduced-motion`.
- Lazy-load para todas las imágenes salvo la del hero.
- Evitar vídeos, mapas embebidos y librerías pesadas.
- Añadir estados de error razonables para portapapeles, `dialog` y service worker.

## Dirección visual específica para Madrid

- Hero recomendado: Plaza Mayor al inicio o Templo de Debod al atardecer si existe una imagen con licencia clara. Elegir una sola narrativa visual y no mezclar demasiados monumentos en el encabezado.
- Paleta base: conservar papel/tinta de Alcalá y usar acentos inspirados en piedra, terracota, verde de jardines y azul de cielo.
- Paradas gastronómicas: usar una variación cálida, no fotografías saturadas de comida que rompan el carácter histórico.
- Callao y Gran Vía: reservar el acento azul/gris o una imagen nocturna únicamente si mantiene suficiente contraste para texto.
- Debod: usar un bloque intermedio de descanso y orientación hacia Plaza de España; el atardecer queda como posibilidad, no como requisito.
- Mantener la alternancia de paradas, pero verificar contraste WCAG en cada combinación de fondo, tinta y acento.

## Fuentes y verificación editorial

Antes de implementar el contenido definitivo, comprobar en fuentes primarias:

- Horarios, cierres y entradas del Palacio Real y de la Galería de las Colecciones Reales.
- Horarios de la Catedral de la Almudena y restricciones durante misas.
- Horarios y puestos vigentes del Mercado de San Miguel.
- Ubicación y horario del Mesón del Champiñón.
- Acceso, horarios y hostelería de Gourmet Experience Callao.
- Condiciones del parking de Plaza Mayor, gálibo y acceso ZBEDEP.
- Hora de puesta de sol, solo como dato calculado para la fecha que el visitante elija; no fijarla permanentemente en el contenido.

Los enlaces oficiales deben vivir en cada parada y las fuentes de fotografías en `IMAGE_CREDITS.md`. No presentar precios u horarios como permanentes sin fecha de comprobación.

## Fases de implementación

### Fase 1 — Preparación y contenido

1. Confirmar el orden final, coordenadas, tiempos y paradas opcionales.
2. Convertir `walking-tour.md` en la fuente editorial completa de Madrid.
3. Recopilar fuentes oficiales y separar hechos confirmados de recomendaciones.
4. Seleccionar hero e imágenes de paradas con licencias documentadas.

### Fase 2 — Scaffold de la aplicación

1. Crear `index.html`, `styles.css`, `src/app.js`, manifest, service worker e iconos.
2. Trasladar la carcasa del referente y renombrar títulos, metadatos, claves y nombres de caché.
3. Verificar que el proyecto funciona sin servidor de desarrollo ni proceso de build.

### Fase 3 — Datos y experiencia de ruta

1. Implementar el array de paradas base.
2. Implementar la Galería como parada opcional.
3. Añadir navegación, copiar coordenadas, GPX, progreso y paradas visitadas.
4. Añadir glosario específico y acción final de regreso al parking.

### Fase 4 — Diseño y contenido visual

1. Ajustar hero y metadatos de Madrid.
2. Cargar imágenes optimizadas en WebP con ancho/alto explícitos.
3. Revisar alternancia de fondos, legibilidad y recortes en móvil.
4. Añadir créditos visibles y `IMAGE_CREDITS.md`.

### Fase 5 — PWA, accesibilidad y validación

1. Versionar y probar el service worker.
2. Ejecutar `npm run check`.
3. Probar en Android en modo vertical, con red lenta y después de instalar la PWA.
4. Probar en escritorio a 1440 px y en móvil alrededor de 320–390 px.
5. Revisar teclado, foco, lectores de pantalla, `dialog`, contraste y movimiento reducido.
6. Validar que todos los enlaces externos abren la parada correcta y que el GPX conserva el orden.

## Criterios de aceptación

### Contenido y ruta

- La ruta visible coincide con el circuito aprobado y no introduce retrocesos innecesarios.
- Plaza Mayor, Mercado, Austrias, Almudena, Palacio, Oriente/Ópera, Debod, Plaza de España, Gran Vía, Callao, Sol y Champiñón aparecen en el orden correcto.
- La última parada está a pocos minutos del parking y permite navegar hasta él; Debod no contiene un regreso largo al coche.
- Los datos de horarios, precios y accesos llevan fuente o advertencia de verificación.

### Producto

- La primera pantalla explica lugar, duración, distancia, coste de la ruta base y punto de inicio.
- Un visitante puede seguir la ruta sin abrir un mapa embebido.
- La elección del proveedor, paradas visitadas y opción Galería persisten al recargar.
- El GPX se descarga con nombres y coordenadas correctos.
- La guía instalada puede abrir el shell y los recursos locales sin conexión después de una primera carga.

### Calidad técnica

- `npm run check` pasa sin errores.
- No se añaden dependencias innecesarias.
- No hay imágenes sin `alt`, dimensiones o crédito cuando corresponda.
- El diseño sigue siendo legible con 320 px de ancho.
- `prefers-reduced-motion` desactiva desplazamiento suave/parallax de forma suficiente.
- Los botones de mapas, glosario, progreso, GPX y paradas visitadas funcionan con teclado y táctil.

## Entregable de la implementación

Al terminar, el directorio debe contener una PWA estática autocontenida y esta documentación editorial, con una estructura equivalente a:

```text
index.html
styles.css
src/app.js
sw.js
manifest.webmanifest
icons/compass.svg
images/*.webp
IMAGE_CREDITS.md
tour.md
implementation-plan.md
package.json
```

La primera implementación debe priorizar una experiencia fiable para caminar y leer en el móvil. Las ampliaciones —mapa propio, audio, reservas, restaurantes externos o cuentas de usuario— quedan fuera de alcance hasta validar el recorrido base.
