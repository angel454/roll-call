# Sala Rápida — trivia en vivo con código de sala

Una app tipo Kahoot: alguien crea una sala, comparte un código de 5 letras, y cualquiera entra desde su móvil con un enlace, escribe su nombre, y juega en tiempo real. Nadie necesita cuenta.

Es un único archivo (`index.html`), sin backend propio que mantener: usa **Firebase Realtime Database** (plan gratuito) solo para sincronizar el estado de la sala entre dispositivos.

## Paso 1 — Crear el proyecto Firebase (gratis, ~5 min)

1. Ve a **https://console.firebase.google.com** e inicia sesión con una cuenta Google.
2. "Añadir proyecto" → ponle el nombre que quieras → puedes desactivar Google Analytics (no hace falta).
3. Dentro del proyecto, en el menú lateral: **Compilación → Realtime Database** → "Crear base de datos".
   - Elige cualquier ubicación.
   - Empieza en **modo de prueba** (luego aplicamos las reglas del paso 3).
4. En el menú lateral, ve a **Configuración del proyecto** (el engranaje) → pestaña **General** → sección "Tus apps" → icono `</>` (Web) → registra una app (el nombre no importa, no hace falta hosting de Firebase).
5. Verás un bloque de código con `const firebaseConfig = { ... }`. Copia solo el objeto `{ ... }` — lo pegarás dentro de la app en el Paso 4.

## Paso 2 — Configurar las reglas de la base de datos

En **Realtime Database → Reglas**, pega el contenido de `rules.json` (incluido en esta carpeta) y publica.

Esto permite lectura/escritura únicamente dentro del nodo `rooms`, que es donde vive el estado de las partidas. Es una configuración abierta (sin login) pensada para partidas casuales entre amigos — cualquiera con el enlace de tu app técnicamente podría leer/escribir salas. Para uso privado con amigos es razonable; si más adelante quieres cerrarlo más, se puede añadir autenticación anónima de Firebase.

## Paso 3 — Publicar la app desde tu repo de GitHub (despliegue automático)

En vez de subir el archivo a mano cada vez, conecta el repo una sola vez: a partir de ahí, cada `git push` publica la nueva versión sola, con historial y sin arrastrar nada.

1. Sube estos tres archivos (`index.html`, `README.md`, `rules.json`) a la raíz de tu repositorio de GitHub — con `git add`, `commit` y `push`, o subiéndolos desde la web de GitHub ("Add file → Upload files").
2. Elige un hosting y conéctalo al repo (cualquiera de los dos, ambos gratis):

   **Vercel:**
   - Ve a **vercel.com** → inicia sesión con tu cuenta de GitHub → "Add New… → Project".
   - Selecciona tu repositorio. No hace falta tocar ninguna configuración (es un sitio estático puro) — pulsa "Deploy".
   - Te da una URL pública (`tu-repo.vercel.app`). Cada `git push` a la rama principal la actualiza sola.

   **Netlify (vía GitHub, no Drop):**
   - Ve a **app.netlify.com** → "Add new site → Import an existing project" → conecta GitHub → elige el repo.
   - Build command: déjalo vacío. Publish directory: `.` (la raíz).
   - "Deploy site". Igual que con Vercel, cada push lo actualiza automáticamente.

3. (Opcional) Añade un dominio propio desde el panel de Vercel/Netlify si tienes uno — ambos lo soportan gratis, solo aportas el dominio.

Con esto, para cambiar las preguntas por defecto, el diseño o cualquier detalle, solo editas `index.html` en el repo y haces push: el sitio en producción se actualiza solo en menos de un minuto.

## Paso 4 — Primer uso

1. Abre la URL pública que te dieron.
2. La primera vez te pedirá pegar tu `firebaseConfig` (el objeto que copiaste en el Paso 1.5). Se guarda en el navegador de cada persona, no hace falta repetirlo.
   - **Nota:** solo quien vaya a crear salas (el anfitrión) necesita pegar la config real. Los jugadores que solo se unen con un código también verán esa pantalla la primera vez — deben pegar la misma config para conectarse a la misma base de datos. Compárteles el mismo `firebaseConfig` (no es información sensible, es una clave pública de cliente).
3. Ya puedes crear una sala, compartir el código de 5 letras con tus amigos, y jugar.

## Personalizar las preguntas

Al crear una sala, verás un cuadro de texto con el quiz en formato JSON. Puedes editarlo directamente ahí antes de crear la sala:

```json
[
  {"q": "¿Pregunta?", "options": ["A", "B", "C", "D"], "correct": 0}
]
```

`correct` es el índice (empezando en 0) de la opción correcta.

## Cómo funciona (por si quieres tocar el código)

- Cada sala vive en `rooms/{código}` dentro de la base de datos: preguntas, jugadores, respuestas y estado (`lobby` → `question` → `reveal` → ... → `ended`).
- El anfitrión controla el avance; los jugadores solo escuchan cambios de estado y envían sus respuestas.
- La puntuación premia acertar rápido: hasta 1000 puntos por pregunta, menos según el tiempo que tardes en responder dentro de los 20 segundos.
- Todo el código está en `index.html`, sin build ni dependencias que instalar — es HTML + JavaScript plano más el SDK de Firebase cargado desde su CDN.
