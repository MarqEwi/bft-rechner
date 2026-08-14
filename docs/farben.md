# Farben – BFT Tool

Diese Datei hält fest, welche Farben die App verwendet, wo sie im Code
stehen und wie sie ermittelt wurden. Sie ist die Quelle der Wahrheit; wenn
Logo und Code auseinanderlaufen, wird nach dieser Datei korrigiert.

Stand: geprüft und gemessen am 14. August 2026 (App-Version 1.1.3).

---

## Nicht mit der Pipette messen

**Die Logofarben dürfen nicht per Pipette aus dem PNG gegriffen werden.**

Die Logos der MERCwerk-Apps stammen aus KI-Erzeugungen. Solche Bilder
rauschen: Was wie eine einfarbige Fläche aussieht, schwankt in Wahrheit um
mehrere Stufen. Beim BFT-Logo liegen 99 % der Flächenpixel zwischen
`#545F33` und `#7B835B` – 5957 verschiedene Farbwerte auf einem Bild, das
drei Farben haben sollte. Wer die Pipette ansetzt, erwischt einen
Zufallswert aus diesem Rauschen.

Steht so ein Zufallswert dann als Hintergrundfarbe im Adaptive Icon
(`ic_launcher_background.xml`), sieht man auf dem Startbildschirm eine
sichtbare Kante zwischen Icon-Vordergrund und Icon-Hintergrund.

**Richtig ist der Modus** – der häufigste Farbwert der großen Fläche. Beim
BFT-Logo ist er eindeutig: `#5C673C` kommt auf 50 % aller Pixel des
512er-Icons und ist gleichzeitig der Median. Auf dem Vordergrund des
Launcher-Icons sind es sogar 86 %.

So wird gemessen (getrennt nach Helligkeit, weil ein Logo dieser Familie
genau drei Farben hat – Fläche, Piktogramm, Schriftzug):

```python
from PIL import Image
from collections import Counter
import numpy as np

im = Image.open('icons/icon-512.png').convert('RGBA')
a  = np.asarray(im).reshape(-1, 4).astype(int)
op = a[a[:, 3] > 250][:, :3]              # nur voll deckende Pixel
lum = op @ np.array([0.2126, 0.7152, 0.0722])

for name, mask in [('Fläche',      (lum >= 40) & (lum < 160)),
                   ('Piktogramm',   lum < 40),
                   ('Schrift',      lum >= 160)]:
    sel = op[mask]
    print(name, '#%02X%02X%02X' % Counter(map(tuple, sel)).most_common(1)[0][0])
```

Der Alphakanal muss dabei berücksichtigt werden: In den `ic_launcher*.png`
tragen durchsichtige Pixel beliebige RGB-Werte. Wer sie mitzählt, misst
zum Beispiel ein leuchtendes `#00FF00` als angebliche Schriftfarbe.

---

## 1. Die drei Logofarben

| Rolle | Hex | Anteil im 512er-Icon | Fundstelle im Code |
|---|---|---|---|
| **Fläche** (olivgrün) | `#5C673C` | 50 % (Modus = Median) | `android/app/src/main/res/values/ic_launcher_background.xml` |
| **Piktogramm** (schwarz) | `#000000` | 8 % | nur im Bild – keine Code-Stelle |
| **Schrift** (hell) | `#F0EEDF` | 4 % | nur im Bild – keine Code-Stelle |

Die Fläche ist die einzige Logofarbe, die auch im Code steht: Der
Adaptive-Icon-Hintergrund muss exakt sie sein, sonst entsteht die Kante.

Gemessen in allen Größen – die Werte sind über sämtliche Ableitungen
stabil:

| Datei | Fläche | Piktogramm | Schrift |
|---|---|---|---|
| `icons/icon-512.png` | `#5C673C` | `#000000` | `#F0EEDF` |
| `icons/icon-192.png` | `#5C673C` | `#000000` | `#F0EEDF` |
| `icons/icon-maskable-512.png` | `#5C673C` | `#000000` | `#F0EEDF` |
| `mipmap-*/ic_launcher.png` (5 Größen) | `#5C673C` | `#000000` | `#F0EEDF`\* |
| `mipmap-*/ic_launcher_foreground.png` | `#5C673C` | `#000000` | `#F0EEDF`\* |
| `drawable*/splash.png` | `#5C673C` | `#000000` | `#F0EEDF` |
| Favicon (Daten-URI in `index.html`) | `#5C673C` | `#000000` | `#F0EEDF`\* |
| Kopfzeilen-Logo (Daten-URI in `index.html`) | `#5C673C` | `#000000` | `#F0EEDF` |
| `https://www.mercwerk.de/assets/apps/bft.png` | `#5C673C` | `#000000` | `#F0EEDF` |

\* In den kleinen Größen weicht der Schriftmodus um wenige Stufen ab
(z. B. `#F8F5E8` bei 72 px). Das ist reine Skalierung – der Schriftzug ist
dort nur noch wenige Pixel hoch, jeder Pixel eine Mischfarbe. Kein Fehler.

Die Kopie auf mercwerk.de ist **pixelgleich** mit `icons/icon-192.png`
(maximale Abweichung 0). Alle Stellen zeigen dasselbe Logo.

---

## 2. Marken der App

Die App hat einen hellen und einen dunklen Modus. Die Werte stehen als
CSS-Variablen am Anfang von `index.html`
(`:root` bzw. `html[data-theme="dark"]` und der Block unter
`@media (prefers-color-scheme: dark)` – die beiden dunklen Blöcke müssen
identisch bleiben).

| Rolle | Variable | Hell | Dunkel |
|---|---|---|---|
| Akzent | `--accent` | `#4A5D3A` | `#8BA36B` |
| Akzent hell | `--accent2` | `#6D8250` | `#A5BD85` |
| Schrift auf Akzent | `--accent-text` | `#FFFFFF` | `#141810` |
| Grund (Seite) | `--bg` | `#F5F6F2` | `#171A14` |
| Fläche (Karten) | `--bg2` | `#FFFFFF` | `#20241C` |
| Fläche gedämpft | `--bg3` | `#ECEEE6` | `#282D22` |
| Schrift | `--text` | `#20241C` | `#E8ECDF` |
| Schrift gedämpft | `--text2` | `#5B6152` | `#A2A893` |
| Linien | `--line` | `#D9DCCF` | `#3A4032` |

Bewertungs- und Signalfarben:

| Rolle | Variable | Hell | Dunkel |
|---|---|---|---|
| Premium / Diamant | `--gold` | `#B8952F` | `#D4B04A` |
| sehr gut | `--ok` | `#2E7D32` | `#7CC47F` |
| gut | `--good` | `#558B2F` | `#A5C968` |
| zufriedenstellend | `--mid` | `#B58900` | `#D9B544` |
| ausreichend | `--low` | `#C96D00` | `#E09550` |
| nicht bestanden | `--bad` | `#C62828` | `#E57373` |

### Familienwerte – für alle Apps mit Flecktarn-Hintergrund gleich

Diese vier Werte sind **nicht** app-eigen. Sie verbinden die MERCwerk-Apps
und dürfen nicht abgewandelt werden:

| Rolle | Wert | Fundstelle |
|---|---|---|
| Grundton hell | `#3F4A33` | `body::before` und `fill` im hellen `--camo`-SVG |
| Grundton dunkel | `#232A1C` | `fill` im dunklen `--camo`-SVG |
| Kartenfläche hell | `rgba(255,255,255,.93)` | `--panel` |
| Kartenfläche dunkel | `rgba(24,28,19,.93)` | `--panel` |

Zusätzlich: `manifest.webmanifest` → `"theme_color": "#3F4A33"` (derselbe
Grundton) und `"background_color": "#F5F6F2"` (derselbe Wert wie `--bg`
hell).

Die `<meta name="theme-color">` in `index.html` folgen bewusst `--bg`
(`#F5F6F2` hell, `#171A14` dunkel), nicht dem Grundton – sie färben die
Systemleiste des Browsers, die an die Seite anschließen soll.

---

## 3. Was sonst noch Farbe trägt

**Kachel-Piktogramme** (`img.heroimg`, als Daten-URI in `index.html`):
eigene Zeichnungen mit Olivtönen um `#505C36`, `#515E32`, `#505B30`. Sie
sind Illustrationen, keine Markenflächen – die Abweichung vom Logo-Oliv
ist hier ohne Wirkung, weil sie nie an eine einfarbige Fläche stoßen.

**Android-Theme** (`android/app/src/main/res/values/styles.xml`):
`AppTheme` verweist auf `@color/colorPrimary`, `colorPrimaryDark` und
`colorAccent`. Diese Farben sind im Projekt **nicht** definiert, es greifen
die Vorgaben aus der Capacitor-Bibliothek: Indigo `#3F51B5` / `#303F9F` und
Pink `#FF4081`. Sichtbar wird das höchstens an System-Bedienelementen
(z. B. Griffe bei der Textauswahl), weil die WebView den Bildschirm füllt.
Wer es angleichen will, legt `android/app/src/main/res/values/colors.xml`
mit den Markenwerten an.

---

## 4. Dateiformat der Icons

Google Play lehnt beim Hochladen 24-Bit-PNG ab und verlangt **32 Bit**
(also mit Alphakanal). Betroffen sind das Store-Symbol und das
Produktsymbol eines In-App-Kaufs, falls eines hinterlegt wird.

| Datei | Format |
|---|---|
| `icons/icon-512.png` | 32 Bit (RGBA) |
| `icons/icon-192.png` | 32 Bit (RGBA) |
| `icons/icon-maskable-512.png` | 32 Bit (RGBA) |

Die Android-Ressourcen (`mipmap-*`, `drawable*/splash.png`) sind davon
nicht betroffen – dort baut Gradle die Bilder selbst ein.

Prüfen:

```bash
python3 -c "from PIL import Image; im=Image.open('icons/icon-512.png'); print(im.mode, len(im.getbands())*8, 'Bit')"
# erwartet: RGBA 32 Bit
```

---

## 5. Nicht vereinheitlichen

Jede App der Familie behält ihre **eigene** Markenfarbe. Das Oliv `#5C673C`
gehört zum BFT Tool und darf nicht aus einer Schwester-App übernommen oder
in eine solche übertragen werden.

Was die Familie verbindet, ist der **Aufbau** des Logos, nicht der Farbton:
eine große ruhige Fläche, ein schwarzes Piktogramm, ein heller Schriftzug
davor. Dazu die vier Flecktarn-Werte aus Abschnitt 2.
