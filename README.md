# Midnight Gamepad 1.0

Ein kleines, **frameworkfreies virtuelles Gamepad** als Web Component für HTML5-Canvas-Spiele, Ionic (auch ohne React/Angular/Vue), Capacitor, p5.js und Three.js.

- Linker analoger Stick: **x/y von −1 bis +1**, diagonale Bewegung normalisiert
- Vier Aktionstasten: **A, B, X, Y** (gleichzeitig mit dem Stick nutzbar)
- Touch, Maus und optional Tastatur (WASD / Pfeile; J, K, U, I / Leertaste)
- Kein npm-Paket, kein Build-Schritt und keine Runtime-Abhängigkeit erforderlich
- Funktioniert als **Overlay über deinem Canvas**
- Pointer Events, Pointer Capture und `pointercancel` für zuverlässige Eingaben

## ZIP-Inhalt

```text
midnight-gamepad/
├── gamepad.js          # Komplette Bibliothek, ohne externe Abhängigkeiten
├── README.md           # Diese Anleitung
├── LICENSE             # MIT-Lizenz
└── demo/
    ├── index.html      # Demo-Seite
    └── game.js         # Ball mit Beschleunigung und Reibung
```

## 1. Demo starten

Das ZIP entpacken und im Ordner `midnight-gamepad` einen kleinen Webserver starten:

```bash
python3 -m http.server 8000
```

Unter Windows ist häufig auch `py -m http.server 8000` möglich.

Im Browser öffnen: **http://localhost:8000/demo/**

- Links den Stick mit Finger oder Maus bewegen.
- **A**: kurzer Geschwindigkeitsschub, **B**: bremsen, **X**: Farbe wechseln, **Y**: zurücksetzen.
- Am PC: WASD oder Pfeiltasten; **J=A**, **K=B**, **U=X**, **I=Y** (auch Leertaste=A).

Zum Testen auf einem Smartphone im gleichen WLAN kannst du die Server-IP statt `localhost` verwenden. Browser und Betriebssystem müssen dabei den Zugriff erlauben. Für echte Apps ist keine Netzwerkverbindung nötig, wenn die Dateien lokal mitgeliefert werden.

## 2. In eigene HTML/Canvas-Seiten einbauen

Kopiere `gamepad.js` in deinen Webordner, z. B.:

```text
www/
├── index.html
└── gamepad.js
```

Minimalbeispiel:

```html
<!doctype html>
<html lang="de">
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
  <style>
    html, body { margin: 0; height: 100%; }
    #spiel { position: relative; height: 100dvh; background: #101827; overflow: hidden; }
    #spiel canvas { width: 100%; height: 100%; display: block; }
  </style>
  <script src="gamepad.js" defer></script>
</head>
<body>
  <div id="spiel">
    <canvas id="canvas"></canvas>
    <midnight-gamepad keyboard></midnight-gamepad>
  </div>
  <script defer src="game.js"></script>
</body>
</html>
```

In `game.js`:

```javascript
const pad = document.querySelector('midnight-gamepad');

function update(dt) {
  // dt in Sekunden, z. B. aus requestAnimationFrame
  player.vx += pad.x * 900 * dt;
  player.vy += pad.y * 900 * dt;

  if (pad.A) console.log('A wird gehalten');
}

pad.addEventListener('gamepadbutton', event => {
  const { button, pressed } = event.detail;
  if (button === 'B' && pressed) console.log('B frisch gedrückt');
});
```

Die Bibliothek erledigt **nur die Eingaben**. Zeichen- und Physiklogik bleiben in deinem Spiel. Die fertige Demo zeigt beides.

## 3. In Ionic + Capacitor verwenden

Bei deinem bereits eingerichteten Projekt mit einer `www`-Web-App:

1. `gamepad.js` nach `www/gamepad.js` kopieren.
2. Das Element `<midnight-gamepad></midnight-gamepad>` in einen **positionierten Container** neben den Canvas setzen.
3. `<script src="gamepad.js" defer></script>` im HTML-Head einbinden.
4. In deiner Spiel-Logik mit `document.querySelector('midnight-gamepad')` darauf zugreifen.
5. Aus dem Projektverzeichnis aktualisieren:

```bash
npx cap sync android
npx cap open android
```

Du brauchst für dieses Gamepad **keine zusätzliche Ionic- oder Capacitor-Erweiterung**. In einer Ionic-Seite darf es beispielsweise innerhalb von `<ion-content>` liegen; für Vollbildspiele solltest du Scrollen und Abstände deaktivieren und eine definierte Höhe für den Spielbereich vergeben. Alternativ die komplette `demo`-Seite in `www` integrieren und die relativen Skriptpfade anpassen.

**Querformat:** Die Demo passt sich der Fenstergröße an, erzwingt aber nicht die Android-Bildschirmausrichtung. Die feste Landscape-Ausrichtung ist eine separate Einstellung deines Android-Projekts (z. B. die `screenOrientation` der Activity in `AndroidManifest.xml`). Prüfe die Ausrichtung nach einem Neustart auf dem Gerät.

## 4. API

Abfrage innerhalb deiner Spielschleife:

| Eigenschaft | Rückgabewert |
|---|---|
| `pad.x` | Stick horizontal, −1 links bis +1 rechts |
| `pad.y` | Stick vertikal, −1 oben bis +1 unten |
| `pad.A` / `.B` / `.X` / `.Y` | `true`, solange Taste gedrückt wird |
| `pad.axes` | Neues Objekt `{x, y}` |
| `pad.buttons` | Neues Objekt `{A, B, X, Y}` |
| `pad.state` | Aktueller, veränderbarer Zustandscontainer – besser die Eigenschaften oben lesen |
| `pad.reset()` | Achsen neutral und Tasten loslassen |

Events:

```javascript
pad.addEventListener('gamepadmove', e => {
  console.log(e.detail.x, e.detail.y);
});

pad.addEventListener('gamepadbutton', e => {
  console.log(e.detail.button, e.detail.pressed);
});
```

`gamepadmove` wird bei Änderungen der Achsen ausgelöst, `gamepadbutton` bei Drücken/Loslassen. Beide sind normale DOM-`CustomEvent`s und steigen im DOM auf (`bubbles: true`). **Es werden absichtlich keine künstlichen `keydown`-Events ausgelöst.** Dein Spiel liest direkt die Zustandswerte. Tastatureingaben können über das Attribut `keyboard` zusätzlich aktiviert werden.

Attribute am Element:

```html
<midnight-gamepad
  keyboard
  stick="left"
  buttons="4"
  size="120"
  opacity="0.85">
</midnight-gamepad>
```

- `keyboard`: Tastatursteuerung aktivieren (beim Erstellen setzen; dynamisches Umschalten wird derzeit nicht unterstützt).
- `stick="left"` oder `stick="right"`: Position von Stick und Tasten tauschen.
- `buttons="0"` bis `"4"`: sichtbare Aktionstasten (Reihenfolge A, B, X, Y).
- `size="120"`: Stick-Durchmesser in CSS-Pixeln (80–240), die Buttons skalieren mit.
- `opacity="0.85"`: Deckkraft (0.1–1).

Die Werte für `stick`, `buttons`, `size` und `opacity` kannst du zur Laufzeit über `setAttribute()` ändern.

## 5. Warum Pointer Events?

`pointerdown`, `pointermove`, `pointerup`, `pointercancel` und `setPointerCapture()` unterstützen Touch und Maus über eine gemeinsame API. Jeder Finger hat eine eigene `pointerId` – dadurch kannst du **gleichzeitig links lenken und rechts eine Taste halten**. Die Buttons verhindern über `touch-action: none` störendes Scrollen.

## 6. Physik wie SDL2

Die Demo verwendet Geschwindigkeiten in **Pixel pro Sekunde** und ein `dt` in **Sekunden**:

```javascript
vx += axisX * ACCELERATION * dt;
vx *= Math.exp(-FRICTION * dt);
x  += vx * dt;
```

`ACCELERATION = 900`, `FRICTION = 6`, `MAX_SPEED = 350`. Je höher die Reibung, desto schneller kommt der Ball nach dem Loslassen zum Stehen. Die Animation läuft über `requestAnimationFrame()`, dessen tatsächliche Frequenz vom Display und Gerät abhängt. Das ist **kein Versprechen fester 60 FPS**.

## 7. Hinweise

- JavaScript-Datei unbedingt **vor** der eigenen Spiellogik laden oder `defer` wie in den Beispielen verwenden.
- Der Elterncontainer braucht `position: relative` und eine sichtbare Größe. Das Gamepad selbst ist ein transparentes Overlay, das das Spielfeld in der Mitte nicht blockiert.
- Bei kleinen Hochformat-Handys können sich Tasten und Stick überlappen; die Bibliothek ist primär für **Querformat** gedacht.
- `keyboard` verarbeitet Tasten global, solange das Element eingebunden ist; bei Textfeldern wird die Tastatur ignoriert. Mehrere Gamepads mit aktivierter Tastatur gleichzeitig solltest du vermeiden.
- Auf Mobilgeräten sind Safe-Areas (Displayaussparungen) berücksichtigt. Für komplette Immersive-Fullscreen-Apps sind zusätzliche native Einstellungen nötig.
- Bei Unterbrechung/Fokusverlust setzt das Gamepad Eingaben zurück.
- Browser-Sicherheit: Die Bibliothek benötigt **kein CDN** und keine Berechtigungen. Für produktive Apps CSS/HTML lokal mitliefern.

## Lizenz

MIT-Lizenz, siehe `LICENSE`. Du darfst die Bibliothek in privaten und kommerziellen Projekten verwenden und verändern.
# gamepad
