# Frontend UI — Análisis Completo

El AC usa Wails v2 que embebe un frontend WebView2 (Chromium) en el proceso Go.
La comunicación entre Go backend y frontend usa WebSocket IPC (paquete `hwvnZR_pbO`).

---

## HTML Principal — Página de Consola

Embebido en el binario en el filesystem virtual de Wails.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <meta content="width=device-width, initial-scale=1.0" name="viewport"/>
    <title>L4D2Center Anticheat</title>
    <style>
        html, body {
            height: 100%;
            margin: 0;
            padding: 0;
            overflow: hidden;
        }
        body {
            font-family: 'Courier New', monospace;
            background-color: #1e1e1e;
            color: #ffffff;
            display: flex;
            flex-direction: column;
        }
        #console {
            flex-grow: 1;
            overflow-y: auto;
            white-space: pre-wrap;
            word-wrap: break-word;
            padding: 20px;
            box-sizing: border-box;
        }
    </style>
</head>
<body>
    <div id="console"></div>
    <script type="module" src="script.js"></script>
</body>
</html>
```

---

## JavaScript Principal — script.js

Extraído directamente del binario.

```javascript
import { StartL4D2, ConnectL4D2Server, FrontendReady } from "./wailsjs/go/main/App.js";

const consoleElement = document.getElementById('console');

// Recibe eventos del backend Go y los renderiza en el console
window.runtime.EventsOn("console-update", (data) => {

    const stime = document.createElement('timeprint');
    stime.textContent = data.time + " ";
    stime.style.color = "#A9A9A9";  // Timestamp en gris
    consoleElement.appendChild(stime);

    const line = document.createElement('textprint');
    line.innerHTML = makeLinksClickable(data.text) + "\n";
    line.style.color = data.color;  // Color controlado por el backend
    consoleElement.appendChild(line);
    consoleElement.scrollTop = consoleElement.scrollHeight;
});

// Convierte URLs en el texto de consola en links clickables
function makeLinksClickable(text) {
    const urlRegex = /(https?:\/\/[^\s]+)/g;
    return text.replace(urlRegex, function(url) {
        return `<clink style="text-decoration: underline;cursor: pointer;"
                        onclick="openLink('${url}')">${url}</clink>`;
    });
}

// Abre link en el browser del sistema
window.openLink = openLink;
function openLink(url) {
    window.runtime.BrowserOpenURL(url);
}

// Botón de iniciar juego — expuesto globalmente (lo llama la UI externa)
window.buttonStartGame = buttonStartGame;
function buttonStartGame() {
    StartL4D2();
}

// Botón de conectar al servidor — recibe IP:Puerto como argumento
window.buttonConnectServer = buttonConnectServer;
function buttonConnectServer(ip) {
    ConnectL4D2Server(ip);
}

// Señal al backend que el frontend está listo — siempre al final
FrontendReady();
```

---

## Estructura del Evento `console-update`

El backend Go emite eventos al frontend con este objeto JSON:

```typescript
{
    time:  string,   // Timestamp formateado para mostrar (color #A9A9A9)
    text:  string,   // Mensaje de texto (puede contener URLs clickables)
    color: string    // Color CSS para el mensaje (ej: "#00FF00", "#FF0000")
}
```

El backend emite estos eventos via `window.runtime.EventsEmit("console-update", data)`.

**Colores probables de los mensajes:**
| Evento | Color probable |
|--------|----------------|
| Éxito / OK | `#00FF00` (verde) |
| Error / Ban | `#FF0000` (rojo) |
| Advertencia | `#FFFF00` (amarillo) |
| Info normal | `#FFFFFF` (blanco) |
| Timestamp | `#A9A9A9` (gris, confirmado) |

---

## TypeScript Declarations — App.d.ts

Interfaces del backend Go expuestas al frontend via ObfuscatedCall:

```typescript
// Auto-generated. DO NOT EDIT.
export function ConnectL4D2Server(arg1:string):Promise<void>;
export function FrontendReady():Promise<void>;
export function StartL4D2():Promise<void>;
```

---

## JavaScript Implementation — App.js

```javascript
// Auto-generated. DO NOT EDIT.
export function ConnectL4D2Server(arg1) {
    return ObfuscatedCall(0, [arg1]);  // ID 0 → ConnectL4D2Server
}
export function FrontendReady() {
    return ObfuscatedCall(1, []);      // ID 1 → FrontendReady
}
export function StartL4D2() {
    return ObfuscatedCall(2, []);      // ID 2 → StartL4D2
}
```

`ObfuscatedCall` es un mecanismo personalizado que usa IDs numéricos en vez de nombres
de función. Evita que el análisis estático del JS revele qué métodos del backend se llaman.

El mapa de IDs → funciones se actualiza en runtime via `UpdateObfuscatedCallMap`
(implementado en el paquete `aarD4xK0csQ`).

---

## Imagen Embebida (Logo)

El binario contiene una imagen PNG embebida como base64 data URI en el HTML.

- **Tamaño**: 150,356 bytes (PNG decodificado)
- **Dimensiones**: 1024 × ~768 (estimado por el header)
- **Formato**: PNG con metadatos XMP (Adobe)
- **Extraída a**: `analysis/ui_logo.png`

El logo aparece en una segunda página HTML embebida (pantalla de carga/bienvenida)
con CSS `.logo { width: 50%; height: 30%; }`.

---

## WebSocket IPC (Wails Backend ↔ Frontend)

La comunicación entre Go y WebView2 se hace via WebSocket local.
El paquete `hwvnZR_pbO` maneja el IPC.

```
hwvnZR_pbO.(*LFkHvs5).WebsocketIPC   — servidor WebSocket IPC
hwvnZR_pbO.(*LFkHvs5).DesktopIPC     — IPC de escritorio (nativo)
hwvnZR_pbO.(*LFkHvs5).RuntimeDesktopJS — JS inyectado en WebView2
```

El runtime de Wails inyecta JavaScript en el WebView2 al cargarse, exponiendo:
- `window.runtime.EventsOn` — suscribirse a eventos del backend
- `window.runtime.EventsEmit` — emitir eventos al backend
- `window.runtime.BrowserOpenURL` — abrir URL en el navegador del sistema
- `window.go` — namespace con las funciones Go expuestas

---

## Paquete Wails Runtime (Moh1QXpPW)

El paquete principal del runtime de Wails con el sistema de eventos:

```
Moh1QXpPW.(*LFkHvs5).WebsocketIPC
Moh1QXpPW.(*LFkHvs5).DesktopIPC
Moh1QXpPW.(*LFkHvs5).RuntimeDesktopJS
Moh1QXpPW.(*Ozow48G3Ot_).AddFrontend
Moh1QXpPW.(*Ozow48G3Ot_).Emit       — emitir evento al frontend
Moh1QXpPW.(*Ozow48G3Ot_).Notify     — notificar al frontend
Moh1QXpPW.(*Ozow48G3Ot_).On         — suscribirse a evento
Moh1QXpPW.(*Ozow48G3Ot_).Once       — suscribirse una vez
Moh1QXpPW.(*Ozow48G3Ot_).OnMultiple — suscribirse N veces
Moh1QXpPW.(*Ozow48G3Ot_).Off        — desuscribirse
Moh1QXpPW.(*Ozow48G3Ot_).OffAll     — desuscribirse de todo
```

---

## Listado de Países

El binario contiene la lista completa de códigos ISO 3166-1 alpha-2 (AA, AC, AD, AE, AF...),
probablemente usada para verificar el país de origen del jugador en el sistema anti-smurf.

---

## Implicaciones para el Bypass

### Interceptar eventos de consola
Hookear `window.runtime.EventsOn` en el contexto de WebView2 permite:
- Ver todos los mensajes del AC en tiempo real
- Suprimir o modificar mensajes de ban antes de que se muestren

### Suplantar respuestas de la UI
`buttonConnectServer(ip)` y `buttonStartGame()` son funciones globales que pueden
inyectarse en el contexto de WebView2.

### Bloquear el evento de ban
Si el backend emite `console-update` con `{ color: "#FF0000", text: "banned..." }`,
un hook en el frontend podría capturarlo y tomar acción antes de que el jugador vea el ban.
