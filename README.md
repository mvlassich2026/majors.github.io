# Galaxia Major — Panel de Torneo

Sitio estático (HTML/CSS/JS puro, sin build) listo para publicar en **GitHub Pages**, con:

- 👁 **Modo Espectador**: solo ve tablas, resultados y el bracket. No puede editar nada.
- ⚡ **Modo Administrador**: protegido por contraseña, puede cargar resultados, sortear grupos, generar playoffs, etc.
- 🖼 Logo personalizable y 🎵 música de fondo al entrar.
- Los datos "publicados" viven en `data.json`, versionado en el repo con git.

## Archivos del proyecto

```
/
├── index.html      → la app entera (no requiere backend)
├── data.json        → los datos del torneo (jugadores, grupos, resultados, playoffs)
├── logo.png         → tu logo (opcional, lo agregás vos)
└── song.mp3         → música de fondo (opcional, la agregás vos)
```

Si `logo.png` o `song.mp3` no existen, la app funciona igual (el logo simplemente no se muestra y no suena música).

## 1. Poner tu logo y tu canción

Simplemente copiá tus archivos a la carpeta del proyecto con esos nombres exactos:

- `logo.png` (recomendado: fondo transparente, ancho ~400-600px)
- `song.mp3`

Si querés otros nombres/formatos (por ejemplo `logo.svg` o `song.ogg`), avisame y te ajusto las referencias en el código (son 2 líneas).

> **Sobre el autoplay:** los navegadores bloquean que un audio arranque solo, sin que el usuario haga clic en algo. Por eso la música arranca recién cuando la persona toca "Entrar como Espectador" o "Entrar como Administrador" en la pantalla inicial — eso cuenta como interacción y el navegador lo permite. Además hay un botón 🔊/🔇 en el header para pausar.

## 2. Configurar la contraseña de administrador

Por seguridad, la contraseña **no se guarda en texto plano** en el código: se guarda su hash SHA-256. La contraseña de fábrica en este archivo es `galaxia2026` — **cambiala** antes de publicar el sitio:

1. Abrí `index.html` en el navegador (o cualquier página) y abrí la consola (F12 → pestaña "Console").
2. Ejecutá, reemplazando `TU_PASSWORD` por la que quieras usar:

```js
crypto.subtle.digest('SHA-256', new TextEncoder().encode('TU_PASSWORD'))
  .then(b => console.log(Array.from(new Uint8Array(b)).map(x=>x.toString(16).padStart(2,'0')).join('')));
```

3. Copiá el hash que imprime en la consola.
4. En `index.html`, buscá esta línea (cerca del bloque `AUTENTICACIÓN`):

```js
const ADMIN_PASSWORD_HASH = "c17cd382617f46f8eb154c477b714f724a8431b71dfa96e5dedac82efda28e6c"; // sha256("galaxia2026") — CAMBIAR
```

5. Reemplazá el valor entre comillas por tu hash nuevo.

> ⚠️ **Importante sobre seguridad:** esto es un sitio 100% estático (GitHub Pages no tiene servidor propio), así que cualquiera puede ver el código fuente. El hash no puede "descifrarse" directamente para obtener la contraseña, pero esto **no es seguridad de nivel bancario** — es un candado razonable para que espectadores casuales no toquen los datos. No la reutilices en otras cuentas importantes.

## 3. Subir el proyecto a GitHub Pages

Si todavía no tenés el repo creado:

```bash
cd galaxia-major          # la carpeta con index.html, data.json, logo.png, song.mp3
git init
git add .
git commit -m "Primera versión del panel de torneo"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
git push -u origin main
```

Luego, en GitHub:

1. Andá a **Settings → Pages** del repositorio.
2. En "Source" elegí la rama `main` y la carpeta `/ (root)`.
3. Guardá. En 1-2 minutos tu sitio va a estar en:
   `https://TU_USUARIO.github.io/TU_REPO/`

## 4. Flujo de trabajo para actualizar resultados

Esto es lo que vos (el admin) hacés cada vez que hay resultados nuevos:

1. Entrá al sitio publicado y tocá **"Entrar como Administrador"**, poné la contraseña.
2. Cargá los resultados, sorteos, playoffs, etc. normalmente (se van guardando en tu navegador mientras trabajás).
3. Cuando termines, andá a la pestaña **Configuración → Publicar Resultados** (o tocá el botón que aparece en el banner azul "Tenés cambios sin publicar") y tocá **"⬇ Descargar data.json"**.
4. Eso descarga un archivo `data.json` actualizado a tu carpeta de Descargas.
5. Reemplazá el `data.json` de tu repo local por ese archivo descargado.
6. En la terminal, dentro de la carpeta del repo:

```bash
git add data.json
git commit -m "Actualizar resultados"
git push
```

7. En 1-2 minutos GitHub Pages actualiza el sitio y todos los espectadores ven los resultados nuevos.

> Nota: como es un sitio estático, **no hay una base de datos compartida en tiempo real**. Los espectadores ven la última versión de `data.json` que vos publicaste con `git push`. Si necesitás actualizaciones en vivo (por ejemplo, mostrar el marcador segundo a segundo durante un partido en stream), fijate la sección "Recomendaciones" más abajo.

## 5. Cómo se comportan los dos modos

| | Espectador | Administrador |
|---|---|---|
| Ver tablas de posiciones, bracket, stream | ✅ | ✅ |
| Ver resultados cargados | ✅ | ✅ |
| Cargar/editar resultados de partidos | ❌ | ✅ |
| Sortear grupos / generar playoffs | ❌ | ✅ |
| Editar nombres de jugadores | ❌ | ✅ |
| Exportar/importar `data.json` | ❌ | ✅ |

El modo administrador se recuerda mientras la pestaña del navegador esté abierta (usa `sessionStorage`); al cerrar la pestaña hay que volver a poner la contraseña.

## Recomendaciones adicionales

- **Backups**: además de "Descargar data.json", hay un botón "Copia de seguridad con fecha" que descarga un `.json` con fecha en el nombre — usalo antes de un sorteo o de generar playoffs, así podés volver atrás si algo sale mal (con "Importar torneo").
- **Marcador en vivo real**: si en algún momento querés que el marcador se actualice solo en la pantalla de stream sin tener que hacer `git push` cada vez (por ejemplo mapa por mapa durante la transmisión), la única forma de lograr eso de verdad es agregar una base de datos simple en la nube (por ejemplo Firebase Firestore, o Supabase, ambos con planes gratuitos) en lugar del archivo `data.json` estático. Es un cambio de arquitectura más grande — avisame si te interesa y te ayudo a armarlo.
- **Overlay para OBS**: la pestaña "Vista Stream/OBS" ya está pensada para capturarla como fuente de navegador en OBS. Si querés, puedo agregar una versión con fondo transparente (chroma) pensada específicamente para overlay sobre el juego.
- **Historial de resultados**: podría agregar una sección que muestre un log de cambios (quién cargó qué resultado y cuándo) dentro del propio `data.json`, útil para auditar si hay discusiones sobre un resultado.
- **Múltiples administradores con contraseñas distintas**: si van a cargar resultados varias personas, puedo armar varios hashes (uno por persona) para poder identificar quién hizo qué cambio.
- **Favicon**: si me pasás el logo también te dejo armado el favicon del sitio.
- **Responsive/mobile**: el panel ya es razonablemente responsive, pero si lo van a operar mucho desde el celular durante el torneo puedo afinar esa vista.

Cualquiera de estos lo puedo sumar — decime cuáles te interesan y seguimos.
