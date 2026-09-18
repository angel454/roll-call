# 📣 Roll Call — decide el plan de hoy

App web para decidir en 2 minutos qué hacéis cuando quedáis. Uno crea la sala con una lista de planes candidatos, comparte un código de 5 letras, y todos entran desde su móvil. Cada persona veta los planes que no le apetecen y luego se vota entre los que sobreviven.

Es un único archivo (`index.html`), sin backend propio que mantener: usa **Firebase Realtime Database** (plan gratuito) solo para sincronizar el estado entre dispositivos.

## Cómo funciona el juego

1. **Sala** — Alguien crea una quedada: pone su nombre y (opcional) ajusta cuántos vetos tendrá cada persona (por defecto, 2). Obtiene un código de 5 letras.
2. **Se unen los demás** — Con el código y su nombre, sin cuentas.
3. **Fase de veto** — Todos ven la misma pool de planes predeterminados (cañas, cine, escape room, senderismo, etc.). Cada uno tacha los que no le apetecen, hasta agotar sus vetos. Se ve en vivo quién vetó cada uno. Puedes deshacer tus propios vetos.
4. **Fase de voto** — Entre los planes que sobrevivieron, cada uno vota su favorito. Marcador visible en directo.
5. **Ganador** — Cualquiera cierra la votación y se anuncia el plan más votado. Si hay empate, se elige al azar entre los empatados.

El anfitrión no tiene poder especial sobre la partida: cualquiera puede pulsar los botones de "Ir a votación" y "Cerrar votación" cuando la sala esté lista. La única diferencia es que quien crea la sala eligió el número de vetos.

## Personalizar el catálogo de planes

Los planes están definidos como una constante `CATALOG` cerca del inicio del `<script>` en `index.html`. Cada uno lleva `id`, `emoji` y `name`. Añade, quita o edita entradas ahí y haz push — la próxima sala que se cree usará la nueva lista.

## Paso 1 — Crear el proyecto Firebase (gratis, ~5 min)

1. Ve a **https://console.firebase.google.com** e inicia sesión con una cuenta Google.
2. "Añadir proyecto" → nombre a elección → puedes desactivar Google Analytics.
3. Dentro del proyecto, en el menú lateral: **Compilación → Realtime Database** → "Crear base de datos".
   - Elige cualquier ubicación.
   - Empieza en modo de prueba (luego aplicamos las reglas del paso 2).
4. En **Configuración del proyecto** (engranaje) → pestaña General → "Tus apps" → icono `</>` (Web) → registra una app (nombre libre, sin marcar Firebase Hosting).
5. Copia el objeto `firebaseConfig` que aparece — lo pegarás en la app en el paso 4.

## Paso 2 — Reglas de la base de datos

En **Realtime Database → Reglas**, pega el contenido de `rules.json` y publica.

Es una configuración abierta (sin login) pensada para uso casual entre amigos: cualquiera con el enlace de tu app podría, técnicamente, leer o crear salas. Suficiente para partidas privadas; si más adelante quieres cerrarlo, se puede añadir autenticación anónima de Firebase.

## Paso 3 — Publicar la app desde GitHub

Sube los tres archivos (`index.html`, `README.md`, `rules.json`) a la raíz de tu repo y conéctalo a **Vercel** o **Netlify** desde GitHub (ambos gratis). Cada `git push` a la rama principal despliega la nueva versión automáticamente.

## Paso 4 — Primer uso

Abre tu URL pública. La primera vez, la app te pedirá pegar el `firebaseConfig` como JSON (con **comillas dobles** en las claves — el bloque tal cual sale de Firebase es JavaScript, hay que convertirlo). Se guarda en el navegador. Comparte la misma URL con quien vaya a jugar.

## Estructura de datos (por si quieres tocar el código)

Cada sala vive en `rooms/{código}`:

```
status: 'lobby' | 'vetoing' | 'voting' | 'ended'
vetoesPerPlayer: number
plans: { plXX: { name, vetoedBy: playerId | null } }
players: { pXX: { name, isHost, vetoesLeft } }
votes: { playerId: planId }
winner: planId
```

El anfitrión es a la vez jugador (participa vetando y votando) y tiene controles extra para avanzar de fase. Todo el código está en `index.html`, sin build ni dependencias que instalar.
