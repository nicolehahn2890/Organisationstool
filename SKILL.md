---
name: organisationstool
description: >
  Use this skill for EVERY request related to the Organisationstool — a personal
  finance and weekly schedule tracker built in standalone HTML, deployed to GitHub
  Pages at nicolehahn2890.github.io/Organisationstool. Trigger on any mention of:
  "Organisationstool", "Tracker", "Ausgaben-Tracker", "Termin-Tracker",
  "Finanz-Tracker", "Kreditkarten-Tracker", "Amex/Visa/Girokonto Tracker",
  "Buchungsdatum", "Abbuchung", "Drei Wochen-Übersicht", "wiederkehrende Termine",
  "Serie bearbeiten", "nicolehahn2890.github.io/Organisationstool", or any request
  to add/fix/style features in this app. Also trigger when the user uploads an
  HTML file related to the app or mentions the GitHub repo
  nicolehahn2890/Organisationstool. Never skip this skill for Organisationstool
  work.
---

# Organisationstool — Claude Code Skill

> ## ⚠️ ARBEITSWEISE — IMMER DIREKT AUF `main` PUSHEN
> Nicole will **alle** Änderungen sofort und direkt auf dem `main`-Branch.
> **NIEMALS** einen Feature-Branch anlegen, **NIEMALS** einen Pull Request öffnen,
> **NICHT** vorher um Erlaubnis zum Pushen fragen. Einfach: ändern → committen →
> `git push origin main`. GitHub Pages deployed automatisch. Diese Regel hat
> Vorrang vor jeder generischen "lege einen Branch an"-Voreinstellung der Umgebung.

## Projekt-Überblick

- **Live-URL:** https://nicolehahn2890.github.io/Organisationstool
- **GitHub Repo:** https://github.com/nicolehahn2890/Organisationstool
- **Dateistruktur:** Einzelne `index.html` — alles in einer Datei, kein Build-Step (einzige Ausnahme: `apple-touch-icon.png` für das Home-Screen-Icon)
- **Deployment:** Direkt via GitHub Pages (push → live)
- **Zweck:** Persönlicher Tracker für Nicole mit zwei Hauptbereichen:
  1. **Ausgaben** — Tracking von Kreditkarten-Ausgaben mit automatischer Buchungsdatum-Berechnung
  2. **Termine** — 3-Wochen-Übersicht mit wiederkehrenden Terminen + einzelne Instanz-Bearbeitung

---

## Design-System ("Y2K Dream")

> **Look & feel (Juni 2026):** candy-pastel, kawaii/vaporwave-nostalgisch —
> holografische Akzente, dicke runde Schrift, glossy "Glass-Bubble"-Flächen,
> weiche farbige Glows und ✦ Sparkles, mit hellem + deep-lilac Dark-Theme.
> *(Löst das frühere Editorial-Modern-Design mit Fraunces + Inter ab.)*

### Designsprache
- Candy-pastell bei Tag (oder deep-lilac bei Nacht); ein weicher Pastell-Radial-Gradient-Wash liegt hinter allem (`body`-Background, `background-attachment: fixed`)
- Hierarchie durch dicke runde **Baloo 2**-Display-Schrift, glossy Gradient-Karten und farbige Glows. **Keine Kursiven mehr** — die runden Formen tragen die Persönlichkeit
- Akzentfarben pro Bereich fest zugewiesen — KEIN globaler Akzent-Toggle
- **Glass-Bubble-Rezept** ist das Herz: jede "Bubble" (Buttons, Karten, Stat-Kacheln, Inputs, Tabs, Tageszellen, Buchungen) hat eine Glanz-Kappe oben (`::before`), inneren Top-Highlight + Bottom-Schatten, einen inneren Ring und einen äußeren Glow. Farbige Bubbles tragen zusätzlich einen Holo-Schimmer (`::after` mit `--holo`, `mix-blend-mode: screen`, radial maskiert)
  - **Farbige Bubble:** Fill = `--zone-grad`, Ink = `--on-grad` (+ `text-shadow: 0 1px 0 var(--on-grad-sh)`)
  - **Weiße/neutrale Bubble:** Fill = `linear-gradient(180deg, var(--surface), var(--surface-2))`, 1–2px `--line`, innerer Glass-Ring
  - Alle Gloss-Werte über `--glass-cap / --glass-hi / --glass-ring` → dimmen im Dark Mode automatisch zu bläulichem "Moonlight"
- Wordmark **`✦tracker`**: das ✦ mit `--holo` gefüllt (background-clip:text), das Wort "tracker" solide in `--text` (NICHT holo — war unleserlich), lowercase
- **Buttons (exakt nach `references/glass-look-reference.html`):** ALLE Buttons sind Glass-Pills mit Glanz-Kappe (`.btn::before`). **Primary** (`.btn-primary`) = `--zone-grad`-Fill (Fallback `--holo`), `--on-grad`-Ink, Holo-Lichtbrechung (`::after`), farbiger Glow `color-mix(--zone 65%)` + Oberkanten-Highlight `inset 0 2px 0`, weight 700, Padding 17px 34px, ✦-Sparkle voran. **Ghost** (`.btn-ghost`) = neutrale weiße Glass-Pill (`--surface-2` + Gloss), NICHT transparent. **Danger** (`.btn-danger`) = danger-getönte Glass-Pill, `--danger-strong`-Text. NICHT auf flachen Text reduzieren.
- Tabs = Pill-Container (weiße Bubble); aktiver Tab = farbige Bubble (Ausgaben=peach-grad, Termine=minze-grad)
- Stat-Reihe ("Überblick") ist ein Candy-Rainbow aus farbigen Bubbles: Ausgegeben=lilac, Abbuchung gesamt=butter (+✦), Mein Anteil=mint, vs.Vormonat=sky, Buchungen=pink
- Buchungsliste: jede Buchung ist eine eigene **weiße** Glass-Karte (keine Zeilen in einer Box)
- ✦ Sparkle ist das Signatur-Ornament — Wordmark, Primär-Buttons, Karten, Toasts, Empty-States
- "Heute"-Tag in der Wochenansicht: glossy minze-Bubble, `--on-grad`-Ink + Suffix " — heute" am Tagesnamen (CSS ::after)
- Motion: bouncy Hover-Lift/Scale (`--ease-bounce`), Press = `scale(0.97)`; Entrances animieren **nur transform** (Basis `opacity:1`), gegated mit `prefers-reduced-motion: no-preference`

### Schriften (Google Fonts CDN)
- **Baloo 2** (rund, dick, bubbly) — Headlines, Beträge, Tagesnummern, Wordmark, Buttons (`--font-display`)
- **Quicksand** (weich, geometrisch) — Body, Labels, alle UI (`--font-body`)
- Field-Labels sind UPPERCASE, 11px, `letter-spacing: 0.12em`, weight 700

### Farbpalette (5 Candy-Akzente)
```
peach (bubblegum) — #FF7FC4 (light) / #FF9AD6 (dark)   → AMEX-Bereich, Karten-Section
rosa  (lilac)     — #B488FF / #C4A2FF                   → "Neue Ausgabe"-Section, Visa-Karte
minze (mint)      — #6FE9C0 / #7FF0CE                   → Termine-Bereich, Girokonto, "Heute"
buttermilk(butter)— #FFD15C / #FFDD7A                   → Statistik / Überblick
violet (sky)      — #6FD2FF / #8FDBFF                   → Buchungsliste, Abo-Tags
```
- Jeder Akzent hat zusätzlich: `--*-tint`, `--*-grad` (glossy Verlauf) und `--glow-*` (farbiger Glow).
- **`--holo`** (pink→lilac→sky→mint→butter) ist die Signatur: Wordmark, Primär-Buttons, aktiver Tab, Toast-Rand, Empty-✦.
- Neutrals sind pink-getönt, nicht grau. Ink ist **deep-grape `#4A2D63`**, niemals Schwarz.
- Status-Farben (theme-unabhängig): trend-up/over = `#FF5FA0`, trend-down/under = `#2FD39E`, danger = `#FF5C8A`.

### Bereich-zu-Farbe-Zuordnung (FEST, nicht ändern)
| Bereich | Farbe | CSS-Klasse | Ink auf Fill (`--on-grad`) |
|---|---|---|---|
| Karten-Übersicht | peach | `zone-peach` | `#8E1556` |
| Neue Ausgabe (Form) | rosa | `zone-rosa` | `#4C2299` |
| Überblick / Stats | buttermilk | `zone-buttermilk` | `#8A5E00` |
| Buchungsliste | violet | `zone-violet` | `#115F86` |
| Termine (kompletter Tab) | minze | `zone-minze` | `#0A7A58` |

Die Stat-Kacheln bekommen ihre Zone-Klasse zur Laufzeit in `renderStats()` gesetzt (Candy-Rainbow), die Karten in `renderCards()` (`zone-${card.color}`).

### Theme: Light / Dark Mode
- **Light:** candy-pastel Hintergrund `#FFF2FB`, Surface `#FFFFFF`, deep-grape Ink `#4A2D63`
- **Dark:** deep-lilac Nacht `#1A1030` (NICHT pures Schwarz!), Surface deutlich abgesetzt `#271A45`, Ink `#F3E9FF`
- Theme-Attribut wird auf `<html>` UND `<body>` gesetzt — beide Selektoren werden gebraucht
- `:root` enthält nur Akzent-Basiswerte + theme-unabhängige Tokens (Fonts, Radii, Motion, Trend) — die theme-abhängigen Tokens stehen ausschließlich in den `[data-theme]`-Blöcken (kein `:root`-Light-Fallback, sonst gewinnt Light gegen Dark)
- Native Date/Time-Picker-Icons im Dark Mode: `filter: invert(1) brightness(1.9)` auf `::-webkit-calendar-picker-indicator`
- Toggle-Button rechts oben, Icon ist SVG (Mond/Sonne)

---

## Tab 1: Ausgaben

### Karten-System (HARTCODIERT, nicht erweiterbar)
Genau **3 feste Karten** — kein "+ Neue Karte"-Button, kein Modal:

```javascript
const CARDS = [
  { id: 'amex', name: 'Amex',      color: 'peach', logo: 'amex' },
  { id: 'visa', name: 'Visa',      color: 'rosa',  logo: 'visa' },
  { id: 'giro', name: 'Girokonto', color: 'minze', logo: 'giro' }
];
```

Logos sind als Inline-SVG in `CARD_LOGOS` definiert — alle drei einzeilig, fett, in `currentColor`. KEINE Boxen/Rahmen — alle drei haben gleichen Stil (AMEX, VISA, GIRO als reiner Text).

### Buchungsdatum-Logik (automatisch berechnet, NICHT manuell)

Funktion: `computeBookingDate(cardId, expenseDateISO)`

| Karte | Regel |
|---|---|
| **Girokonto** | Buchung = Ausgabedatum (sofort) |
| **Amex** | Bis 21. des Monats → Abbuchung am **22. desselben Monats**. Ab 22. → Abbuchung am **22. des Folgemonats** |
| **Visa** | Buchung = Ausgabedatum + 30 Tage |

**WICHTIG:** Buchungsdatum wird NIE persistiert — immer on-the-fly berechnet, damit Logik-Änderungen ohne Migration möglich sind.

### Karten-Kacheln zeigen
- **Hauptzahl:** Was im angezeigten Monat **abgebucht** wird (nach Buchungsdatum gefiltert)
- **Label:** "ABBUCHUNG [Monat] [Jahr]"
- **Pending-Hinweis:** "+ X € folgt im nächsten Monat" — wenn es Käufe aus diesem Monat gibt deren Buchungsdatum in einem späteren Monat liegt (in `--on-grad`-Ink auf der Karte)

### Kategorien (12 Stück, mit Emojis)
```
🍽️ Essen · 🚆 Transport · 🏠 Wohnen · 💪 Sport · 🎬 Freizeit ·
👗 Shopping · 💊 Gesundheit · 💄 Kosmetik · 🎁 Geschenke · 📱 Abos · ✈️ Reisen · 📦 Sonstiges
```

### Wiederkehrende Ausgaben (Abos)
- Checkbox bei "Neue Ausgabe": "Wiederkehrende Ausgabe (Abo)"
- Werden automatisch in Folgemonate projiziert (`isProjected: true`)
- Werden mit Tag "ABO" in violet markiert
- Beim Löschen einer projizierten Instanz wird die Original-Quelle gelöscht (`recurringSource`)

**WICHTIG — Projektion in `getMonthExpenses` ist mode-abhängig:**
- `mode === 'expense'`: Projektion **nur in den aktuellen viewMonth** (Kandidat = `[viewMonth]`).
- `mode === 'booking'`: Projektion **in Vormonat UND aktuellen viewMonth** (Kandidaten = `[prevMonth(viewMonth), viewMonth]`).

Grund: Bei Amex (Käufe ab dem 22.) und Visa (+30 Tage) liegt das Buchungsdatum eines Abos aus dem Vormonat im aktuellen Monat. Ohne Vormonats-Projektion fehlen diese Abo-Buchungen in der Abbuchungs-Summe. Dedup pro `recurringSource`, damit nicht doppelt gezählt wird.

### Eigener Anteil pro Buchung (`eigenAmount`)
- Optionales Feld "Mein Anteil (€) — optional" im Formular bei JEDER Kategorie
- Anwendungsfall: Nicole geht für sich + ihren Freund einkaufen → trackt was davon ihr eigener Anteil ist
- Gespeichert als `expense.eigenAmount: number | null`
- **Hat NULL Einfluss** auf Karten-Summen, Buchungen, Abbuchungen — reine Info-Spalte
- Wird in Summe als 3. Stat-Box "Mein Anteil" angezeigt
- In der Liste pro Eintrag dezent als Hinweis: "davon mein: X €" in minze-Farbe, weight 700 (`.eigen-hint`)
- Leeres Feld = `null` → kein Hinweis, zählt nicht zur Summe

### Statistik-Box (5 Stats)
1. **Ausgegeben** — Summe nach Ausgabedatum
2. **Abbuchung gesamt** — Summe nach Buchungsdatum (was geht diesen Monat tatsächlich aufs Konto)
3. **Mein Anteil** — Summe der `eigenAmount`-Werte
4. **vs. Vormonat** — Prozent-Vergleich (rot wenn höher, grün wenn niedriger)
5. **Buchungen** — Anzahl Einträge

Plus: horizontales Balkendiagramm pro Kategorie (sortiert nach Höhe).

**Layout:** Auf Desktop 5 Spalten, auf Mobile 2 Spalten. Bei 2 Spalten + 5 Boxen entstehen 2+2+1 → letzte Box würde halb-leer aussehen. Fix:
```css
.stats-row .stat:last-child { grid-column: span 2; }
```
Letzte Box bekommt im Mobile-Layout volle Breite — kein "Loch" mehr. Bei Erweiterung auf andere Box-Anzahlen unbedingt prüfen!

### Buchungsliste
- Sortiert nach Ausgabedatum (neueste zuerst)
- Pro Eintrag: Emoji, Beschreibung, Karte (mit Farb-Pill), Kategorie, Datum
- Bei Amex/Visa: "→ abgebucht TT.MM." als Zusatzhinweis (weight 600)
- Bei eigenAmount > 0: "davon mein: X €" in minze (weight 700)
- Jede Buchung ist eine eigene weiße Glass-Karte; Hover lupft sie leicht
- Hover zeigt Lösch-X; Karten-Farb-Pill (`.cc-pill`) hat einen glühenden Punkt
- **Beschreibung MUSS umbrechen können** — `.exp-desc` darf NICHT `white-space: nowrap` + `text-overflow: ellipsis` haben (kappt sonst lange Texte). Stattdessen: `word-break: break-word; overflow-wrap: anywhere; flex-wrap: wrap`

### Monatsnavigation
- Pfeile ‹ › neben Monatslabel im Karten-Section-Header
- Funktionen `prevMonth(yyyymm)` / `nextMonth(yyyymm)` arbeiten **rein mit Strings** — KEIN `Date`-Objekt verwenden (Timezone-Probleme)
- Button-IDs heißen `btnPrevMonth` / `btnNextMonth` — NICHT `prevMonth`/`nextMonth`, sonst überschreibt der Browser die globalen Funktionen
- **"Heute"-Button** (`btnMonthToday`): erscheint nur, wenn `viewMonth` ≠ aktueller Monat; springt zurück zum aktuellen Monat. Sichtbarkeit wird in `updateMonthLabel()` gesteuert.

### Abo-Projektion: Tag wird auf Monatsende geklemmt
In `getMonthExpenses` wird der Tag der Projektion via `Math.min(origTag, letzterTagDesZielmonats)` begrenzt — sonst entstehen ungültige Daten wie "2026-02-31" bei Abos vom 29.–31. (führte zu "Invalid Date" in der Anzeige und falschen Visa-Buchungsdaten). Letzter Tag via `new Date(y, m, 0).getDate()` (m = 1-basiert).

---

## Tab 2: Termine

### Quick-Tags (10 feste Buttons mit Default-Farben)
Jeder Tag hat eine fest zugeordnete `data-color` als Vorschlag. Die Buttons sind
in ihrer Default-Farbe GETÖNT (CSS `[data-color=...]` setzt `--qt`, Hintergrund
via `color-mix`; Dark Mode kräftiger); im aktiven Zustand glossy Akzent-Gradient
mit `--on-grad`-Ink — so sieht man vor dem Speichern, wie der Termin aussehen wird:

| Tag | Default-Farbe |
|---|---|
| 💪 Training | peach |
| 👥 Meeting | violet |
| 💇 Haare waschen | rosa |
| 🧹 Putztag | minze |
| 🛒 Einkaufen | buttermilk |
| 📚 Lernen | violet |
| 💉 Botox | rosa |
| ❤️ Date | rosa |
| 📞 Call | violet |
| 🎉 Feiertag | buttermilk |

Klick auf Tag füllt das Beschreibungs-Feld vor + setzt Default-Farbe. Erneuter Klick deaktiviert. Eigener Freitext bleibt natürlich auch möglich.

### Farb-Picker (im Termin-Formular)
6 Buttons im Formular:
- "A" (Auto) — nimmt die Default-Farbe vom Quick-Tag (oder minze als Fallback)
- 5 Farb-Punkte (peach/rosa/minze/buttermilk/violet) zum manuellen Überschreiben

Logik: `selectedManualColor || selectedQuickTagColor || 'minze'`

### Wiederholungen
Dropdown im Formular:
- Keine
- Täglich
- Alle 2 Tage / Alle 3 Tage / Alle 4 Tage / Alle 5 Tage / Alle 6 Tage / Alle 10 Tage
- Wöchentlich
- Alle 2 Wochen / Alle 4 Wochen
- Monatlich

Gespeichert als `event.recurrence: string | null`. Erlaubte Werte:
- `'daily'` (= alle 1 Tag)
- `'weekly'` (= alle 7 Tage)
- `'monthly'` (kalender-basiert, gleicher Tag im Folgemonat)
- `'every-Nd'` (alle N Tage, z.B. `every-2d`, `every-5d`)
- `'every-Nw'` (alle N Wochen, z.B. `every-2w`, `every-4w`)

**Helper-Funktionen** (in `index.html` definiert, vor `eventOccursOn`):
- `recurrenceIntervalDays(rec)` → liefert Intervall in Tagen für tagebasierte Wiederholungen (`daily`, `weekly`, `every-Nd`, `every-Nw`), `null` für `monthly` oder unbekannt.
- `recurrenceLabel(rec)` → liefert lesbares Label (`"täglich"`, `"alle 5 Tage"`, `"alle 2 Wochen"`, ...).

Wiederkehrende Instanzen werden im Chip mit `↻ ` + Label markiert (z.B. "↻ alle 2 Wochen"). Im Modal wird "Wiederholt: [Label] ab TT.MM." angezeigt.

**Abwärtskompatibilität:** Alte gespeicherte Termine mit `recurrence: 'weekly'` oder `'monthly'` funktionieren unverändert. Beim Erweitern um neue Intervalle nur das Format `every-Nd` / `every-Nw` nutzen, dann ist kein Migration nötig.

### Override-System für einzelne Instanzen ⭐ KRITISCH
Wiederkehrende Termine können pro Instanz überschrieben werden, ohne die Serie zu zerstören.

Datenstruktur:
```javascript
{
  id: 'v123',
  text: 'Putztag',
  date: '2026-04-28',
  recurrence: 'weekly',
  overrides: {
    '2026-05-05': null,                                  // diese Instanz GELÖSCHT
    '2026-05-12': { date: '2026-05-13', text: 'Großputz' }  // verschoben + umbenannt
  }
}
```

**Schlüssel-Funktionen:**
- `eventOccursOn(ev, isoDate)` — prüft ob ein Termin an diesem Datum erscheinen soll. Berücksichtigt:
  1. Lösch-Override (`overrides[isoDate] === null`) → false
  2. Verschiebungs-Ziel (eine andere Instanz wurde auf dieses Datum verschoben) → true
  3. Original-Datum (außer wenn auf anderes Datum verschoben)
  4. Reguläre Recurrence-Logik:
     - Tagebasiert (`daily`, `weekly`, `every-Nd`, `every-Nw`) → `diffDays % intervalDays === 0` via `recurrenceIntervalDays()`
     - `monthly` → gleicher Tag (`cd === sd`) in einem späteren Monat
- `resolveEventForDate(ev, isoDate)` — liefert die effektiven Werte (Original mit angewandten Overrides) und setzt `occurrenceDate` + `sourceDate`

**Im Modal: 4 Buttons bei wiederkehrenden Terminen:**
- "Diesen löschen" → `overrides[sourceDate] = null`
- "Serie löschen" → ganzes Event aus `state.events` entfernen
- "Diesen bearbeiten" → öffnet Edit-Form, schreibt in `overrides[sourceDate] = { ... }`
- "Serie bearbeiten" → ändert das Original-Event direkt

**Bei Einzelterminen (recurrence: null):** nur "Löschen" + "Bearbeiten", beide wirken aufs Original.

### 3-Wochen-Übersicht
Zeigt immer:
1. **Diese Woche** (aktueller Wochenstart Montag bis Sonntag)
2. **Nächste Woche**
3. **Übernächste Woche**

Heute-Markierung: Tag wird mit minze-tint Hintergrund + minze Border hervorgehoben.

### Termin-Daten (Schema)
- Beschreibung (Pflicht)
- Datum (Pflicht)
- Uhrzeit (optional)
- Farbe (peach/rosa/minze/buttermilk/violet — Default vom Quick-Tag)
- Recurrence (null / `daily` / `weekly` / `monthly` / `every-Nd` / `every-Nw`)
- Overrides (optional, nur wenn einzelne Instanzen geändert wurden)

### Termin-Chip Layout
- Hintergrund getönt via `color-mix(in srgb, var(--evt-color) 18%, var(--surface))` (Dark: 30% mit `--surface-2`) — bleibt in beiden Themes farbig. `--evt-color` wird in JS gesetzt
- Border in derselben gemischten Farbe; linke Border (4px) in der vollen Termin-Farbe
- Hover: `translateX(3px) scale(1.02)` (bouncy)
- Optional Uhrzeit (klein)
- Beschreibung — **MUSS** umbrechen können (`white-space: normal`, `overflow-wrap: anywhere`) — KEIN `text-overflow: ellipsis` mit `nowrap` (kappt sonst lange Texte ab)
- Optional Recurrence-Label

---

## Storage

- **localStorage-Key:** `nicole_tracker_v2`
- **NIEMALS** den Key ohne triftigen Grund ändern — würde Nutzerdaten löschen
- **Hintergrund (Juni 2026):** Nicole hat einmal alle Daten verloren, weil sie das
  Home-Screen-Icon gelöscht hat (eigener Storage-Container, kein Backup vorhanden).
  Die App startete danach bei Null. Deshalb: Backups (Export ⬇) bei jeder Gelegenheit
  freundlich in Erinnerung rufen; eine automatische Backup-Erinnerung in der App wurde
  angeboten, ist aber noch NICHT eingebaut — guter Kandidat für eine nächste Iteration.
- Zusätzliche Hilfs-Keys (KEINE Nutzerdaten, dürfen gelöscht werden):
  - `nicole_tracker_version_hash` — Hash der zuletzt gesehenen App-Version (für Auto-Update)
  - sessionStorage `nicole_tracker_just_updated` — Flag für den "App aktualisiert"-Toast
- Falls schemata-breaking: Key bewusst inkrementieren (`_v3` etc.) und Migrations-Logik in `loadState()` ergänzen
- Persistiert wird: theme, selectedCardId, expenses, events (inkl. overrides), viewMonth
- **NICHT** persistiert: cards-Array (ist hartkodiert), Buchungsdaten (werden berechnet)

### Schema-Erweiterungen müssen abwärtskompatibel sein
Beim Hinzufügen neuer Felder (wie damals `recurrence` und `overrides`):
- Altes Schema muss weiter funktionieren
- Defaults via `??` oder Existenz-Checks (`if (ev.overrides) { ... }`)
- KEIN Zwang auf neues Feld

### Auto-Update beim Öffnen
iOS Home-Screen-Webapps frieren den alten Stand ein und laden nie von selbst nach.
Darum prüft `checkForUpdate()` beim Start, bei `visibilitychange` (App kommt in
den Vordergrund) und bei `pageshow` mit `persisted` (iOS Back-Forward-Cache):

1. `fetch(<base-URL>?_cb=<timestamp>, { cache: 'no-store' })` — holt die aktuell deployte Version. **Wichtig:** eindeutiger Cache-Buster-Query, weil iOS-Home-Screen-Webapps (WKWebView) `no-store` teilweise ignorieren und sonst die alte Datei aus dem Cache liefern → Hash bliebe gleich → kein Update.
2. djb2-Hash des Textes vs. gespeicherter Hash (`nicole_tracker_version_hash`)
3. Bei Unterschied: einmal `location.reload()` + Toast "App aktualisiert ✓"
   (Flag via sessionStorage). Hash wird VOR dem Reload aktualisiert → kein Loop.
4. Schutzregeln: max. 1 Check/Minute; KEIN Reload, wenn gerade in einem
   Eingabefeld Text steht oder das Modal offen ist (dann nur Hinweis-Toast);
   bei `file://` komplett deaktiviert.

GitHub-Pages-CDN cached ~10 Minuten — so lange kann es nach einem Push maximal
dauern, bis der Check die neue Version sieht. Nutzerdaten sind von Reloads
nicht betroffen. **Nicht entfernen** — sonst sieht Nicole auf dem Handy
Updates erst nach manuellem Cache-Leeren.

### Datenexport/-import (JSON)
Header oben rechts (neben Theme-Toggle) hat zwei Icon-Buttons:
- **Export (⬇):** `exportData()` schreibt den kompletten state als JSON mit `_meta`-Block in eine Download-Datei `tracker-backup-YYYY-MM-DD.json`.
- **Import (⬆):** öffnet versteckten `<input type="file">`, ruft `importData(file)` auf. Validiert `expenses`/`events` als Arrays, zeigt `confirm()` mit Anzahl-Zusammenfassung, ersetzt dann den state und ruft `saveState()` + `renderAll()`.
- Karten-IDs werden beim Import wie in `loadState()` validiert (unbekannte → `'amex'`).

Anwendungsfall: Migration zwischen verschiedenen Origins (lokale Datei `file://` ↔ GitHub Pages), regelmäßiges Backup. **Niemals entfernen** — ist die einzige Möglichkeit für Nicole, ihre Daten zu sichern.

---

## Technische Regeln (KRITISCH)

### Datum & Zeit
- **Niemals** `new Date().toISOString()` für Datums-Strings verwenden — UTC-Versatz
- Stattdessen: `dateToLocalISO(d)` Helper nutzen, der mit `getFullYear()/getMonth()/getDate()` arbeitet
- `todayISO()` und `viewMonth`-Initialisierung müssen lokal arbeiten
- Beim Parsen von ISO-Strings für Anzeige: `new Date(iso + 'T12:00:00')` — die 12-Uhr-Verschiebung verhindert Timezone-Drift

### Allgemein
- **Immer standalone HTML** — eine einzige `index.html`, keine externen JS/CSS Dateien
- Kein Node.js, kein npm, kein Build-Step
- Fonts via Google Fonts CDN
- Kein `<form>`-Tag — immer Button mit `onclick` oder `addEventListener`
- **KEIN PWA-Manifest / kein `apple-mobile-web-app-capable`** — eine installierte Home-Screen-App bekäme auf iOS einen EIGENEN, leeren localStorage; Nicoles Daten wären dort scheinbar weg. Erlaubt sind nur: `theme-color`-Meta (wird in `applyTheme()` mit dem Theme synchron gehalten, ID `metaThemeColor`), das SVG-Favicon als data-URI und das Home-Screen-Icon (nächster Punkt).
- **`apple-touch-icon.png`** (180×180, im Repo-Root) — EINZIGE erlaubte Zusatzdatei neben `index.html`. Motiv: Mini-Kalender mit bunten Termin-Punkten auf Off-White, von Nicole im Juni 2026 ausgewählt. Ist nur ein Bild, ändert NICHTS am Storage-Verhalten. Erscheint erst bei NEU hinzugefügten Home-Screen-Icons (bestehende behalten ihren Screenshot).

### Feedback-System (Toast + Validierung)
- `showToast(msg)` — kurze Bestätigung unten mittig (Element `#appToast`, Klasse `.toast.show`, 2,2 s). Wird genutzt bei: Ausgabe/Abo gespeichert, gelöscht, Termin gespeichert/aktualisiert, Export, Import.
- `flagInvalid(el)` — markiert ein Pflichtfeld rot mit Shake-Animation (Klasse `.input-error`), fokussiert es; Markierung verschwindet beim Tippen. Alle Pflichtfeld-Checks nutzen das statt stillem `focus()`.
- Lösch-Bestätigungen bleiben `confirm()`. Bei Abos warnt `deleteExpense` explizit, dass das KOMPLETTE Abo aus allen Monaten entfernt wird (auch wenn nur eine projizierte Instanz angeklickt wurde).

### Tastatur-Bedienung
- Enter in einem Input der Formular-Sections (`#expenseFormSection`, `#eventFormSection`) löst den jeweiligen Speichern-Button aus (Helper `enterSubmits`).
- Enter im Modal-Edit speichert (`#actSave`), Escape schließt das Modal.
- Section-IDs heißen bewusst `...Section` — keine Kollision mit Funktionsnamen.

### Touch / Mobile
- `@media (hover: none)`: Lösch-X (`.exp-del`) ist IMMER sichtbar (Hover existiert auf Touch nicht), größere Tippflächen für Monats-Pfeile, Tag-Plus und Farb-Punkte.
- Mobile (≤880px): letzter Tag der Woche (`.day-grid .day:last-child`) spannt 2 Spalten — sonst "Loch" neben Sonntag (gleiche Logik wie bei den Stat-Boxen).
- `.tabs` ist ein Pill-Container (weiße Glass-Bubble, `display: inline-flex`) — NICHT mehr sticky. Aktiver Tab = farbige Bubble.
- `prefers-reduced-motion: reduce` wird respektiert (Animationen/Transitions quasi aus); Entrances animieren ohnehin nur `transform` (Inhalt bleibt sichtbar, falls Animation nicht läuft).

### String-Replacement in Claude Code
- Bei Änderungen: gezielt mit `str_replace` für kleine Edits — oder Python read/modify/write für große Umbauten
- Bei deutschen Umlauten: Unicode-Escape verwenden falls Probleme
  - ä = `\u00e4`, ö = `\u00f6`, ü = `\u00fc`, Ä = `\u00c4` usw.

### Element-IDs vs. globale Funktionsnamen
- **Browser legt für jede Element-ID automatisch eine globale Variable an**
- Daher: NIEMALS Funktionsnamen als Element-ID verwenden
- Konkret: Buttons heißen `btnPrevMonth`/`btnNextMonth`, Funktionen heißen `prevMonth()`/`nextMonth()`

### Modal mit dynamischen Buttons
- Modal-Actions werden je nach Kontext (Single/Recurring/View/Edit) **komplett neu gerendert**
- `flex-wrap: wrap` und `row-gap` sind erforderlich, da bei Recurring 4 Buttons nebeneinander zu breit werden
- `modalContext` als globaler State: `{ event, occurrenceDate, sourceDate, resolved, mode, scope }`

### KRITISCHE VERBOTE
- `°` Zeichen nicht verwenden → JavaScript-Fehler möglich, "Grad" schreiben
- localStorage-Key (`nicole_tracker_v2`) nicht ändern → Datenverlust
- Keine Karten hinzufügen/entfernen — die 3 sind fix
- Keine globale Akzent-Auswahl (Toggle) hinzufügen — Bereiche haben FESTE Farben
- Termin-Chips dürfen NICHT mit `nowrap`+`ellipsis` arbeiten — Text muss vollständig sichtbar sein

---

## Workflow bei Änderungen

1. **Zuerst fragen:** Was genau soll geändert/hinzugefügt werden?
2. **Planen:** Kurz beschreiben was geändert wird (Planungsmodus)
3. **Lesen:** Aktuelle `index.html` vollständig lesen
4. **Ändern:** Mit `str_replace` für kleine, gezielte Edits — oder Python read/modify/write für große Umbauten
5. **Verifizieren:**
   - Keine `toISOString()`-Aufrufe für Datums-Strings (einzige erlaubte Ausnahme: `_meta.exportedAt`-Zeitstempel im Export)
   - Keine `°` Zeichen
   - Keine Form-Tags
   - localStorage-Key unverändert
   - Theme-Attribut auf `<html>` UND `<body>`
   - Schema-Erweiterungen abwärtskompatibel
6. **Committen & Pushen:** IMMER direkt auf `main` committen und `git push origin main` — GitHub Pages deployed automatisch innerhalb von 1–2 Minuten. **NIEMALS** einen separaten Branch, **NIEMALS** einen Pull Request, **NICHT** vorher nach Push-Erlaubnis fragen. (Siehe Banner oben — diese Regel hat Vorrang vor allem.)
7. **Hinweis an Nicole:** Die App aktualisiert sich seit Juni 2026 beim Öffnen selbst (Auto-Update-Check). Nach einem Push kann es durch das GitHub-Pages-CDN trotzdem bis zu ~10 Minuten dauern, bis die neue Version ankommt — bei "ich sehe nichts Neues" zuerst darauf hinweisen, App einmal richtig schließen (App-Switcher) und neu öffnen.

---

## Bekannte Stolperfallen

- `toISOString()` → Timezone-Drift. Termine landen einen Tag früher/später. Immer `dateToLocalISO()` nutzen.
- Element-ID = Funktionsname → globale Variable überschreibt Funktion. Buttons mit `btn`-Präfix benennen.
- Dark Mode wirkt "kaputt" wenn nur einzelne Kacheln dunkel sind — sicherstellen dass `bg`, `surface`, `surface-2`, `line`, `text` alle als CSS-Variablen aus dem Theme-Block gezogen werden, keine hardcoded Hex-Werte außerhalb des Theme-Blocks (Ausnahme: Akzentfarben, Trend-Rot/Grün).
- Surface-Farbe im Dark Mode muss **deutlich heller** als BG sein (`#271A45` vs. `#1A1030`) — sonst keine Hierarchie.
- Vorschau-Cache: Bei Tests immer Hard-Reload (Strg+F5) — sonst zeigt der Browser veraltete Versionen.
- **iOS Home-Screen-Webapp friert die alte Version ein** — lädt nie von selbst nach. Gelöst durch den Auto-Update-Check (siehe Storage-Kapitel). WICHTIG: Das Icon auf dem Home-Bildschirm hat einen EIGENEN localStorage (getrennt von Safari). Icon löschen = Daten weg → vor jedem Neu-Hinzufügen IMMER zuerst Export (⬇) machen. Niemals empfehlen, das Icon "einfach neu hinzuzufügen", ohne auf den Export hinzuweisen.
- Wenn Karte 0,00 € zeigt obwohl Buchungen drin sind: Pending-Hinweis prüfen — Käufe könnten erst im Folgemonat abgebucht werden (richtige Logik, missverständliche Anzeige verhindert das).
- Termin-Texte werden abgeschnitten → IMMER `white-space: normal` + `overflow-wrap: anywhere` für `.evt-text`
- **Buchungslisten-Texte werden abgeschnitten** → IMMER `word-break: break-word` + `overflow-wrap: anywhere` + `flex-wrap: wrap` für `.exp-desc`. Ellipsis ist hier nie erwünscht (anders als z.B. in Mail-Listen).
- **Stats-Grid mit ungerader Box-Anzahl in Mobile-2-Spalten-Layout** → letzte Box wirkt halb-leer (zeigt grauen Border-Bereich). Fix: `.stats-row .stat:last-child { grid-column: span 2 }` im Mobile-Media-Query. Bei Box-Anzahl-Änderung neu prüfen.
- Bei Override-Logik: `overrides[sourceDate] === null` ist NICHT dasselbe wie `!overrides[sourceDate]` — explizit auf null prüfen, sonst werden gelöschte und nicht-existente Instanzen verwechselt

---

## Aktuelle Feature-Liste (Stand: Juni 2026)

### Ausgaben
✅ 3 feste Karten: Amex, Visa, Girokonto
✅ Automatische Buchungsdatum-Berechnung pro Karten-Typ (Giro=sofort, Amex=22./Folgemonat, Visa=+30 Tage)
✅ Pending-Hinweis "+ X € folgt im nächsten Monat" auf Karten
✅ 12 Kategorien inkl. 💄 Kosmetik und 🎁 Geschenke
✅ Wiederkehrende Ausgaben (Abos) mit automatischer Projektion in Folge- UND Vormonat (für Buchungsansicht von Amex/Visa)
✅ Optionales `eigenAmount`-Feld pro Buchung (z.B. wenn für sich + Partner gekauft)
✅ Statistik mit 5 Boxen: Ausgegeben + Abbuchung gesamt + Mein Anteil + vs. Vormonat + Buchungen
✅ Kategorien-Balkendiagramm
✅ Monatsnavigation (timezone-safe)
✅ Buchungslisten-Texte brechen um (kein Abschneiden)

### Termine
✅ 10 Quick-Tags mit Default-Farben (inkl. 💉 Botox, ❤️ Date, 🎉 Feiertag)
✅ Farb-Picker im Formular (Auto + 5 Farben, manuell überschreibbar)
✅ Wiederholungen: keine / täglich / alle 2/3/4/5/6/10 Tage / wöchentlich / alle 2/4 Wochen / monatlich (über `every-Nd` / `every-Nw`-Format, abwärtskompatibel)
✅ Override-System: einzelne Serien-Instanzen löschen/verschieben/umbenennen ohne Serie zu zerstören
✅ Modal mit 4 Aktionen bei Recurring (Diesen/Serie × Löschen/Bearbeiten)
✅ 3-Wochen-Übersicht (timezone-safe)
✅ Termin-Texte brechen um (kein Abschneiden)

### Allgemein
✅ Light + Dark Mode (durchgängig, nicht nur Kacheln)
✅ localStorage-Persistenz unter `nicole_tracker_v2`
✅ JSON-Export/-Import im Header (Backup + Migration zwischen Origins)
✅ **"Y2K Dream"-Design** (Juni 2026): Baloo 2 + Quicksand, Candy-Pastell-Palette, Holo-Akzente, glossy Glass-Bubbles, ✦ Sparkles, deep-lilac Dark Mode
✅ "Heute"-Button in der Monatsnavigation (nur sichtbar wenn nicht im aktuellen Monat)
✅ Toast-Bestätigungen (Holo-Rand) + rote Pflichtfeld-Markierung mit Shake
✅ Enter speichert Formulare, Escape schließt das Modal
✅ Touch-optimiert: Lösch-X auf Handy immer sichtbar, größere Tippflächen, Sonntag ohne Layout-Loch
✅ Pill-Tabs, Favicon (candy), theme-color synchron zum Theme (`#FFF2FB`/`#1A1030`), prefers-reduced-motion
✅ Abo-Projektion klemmt Tag auf Monatsende (kein "31. Februar" mehr)
✅ Auto-Update beim Öffnen (löst das iOS-Home-Screen-Cache-Problem; Toast "App aktualisiert ✓")
✅ Glass-Bubble-System: farbige Bubbles (Karten/Buttons/Stats/aktiver Tab) + weiße Bubbles (Inputs/Buchungen/Tageszellen), color-mix-getönte Termin-Chips & Quick-Tags, Pastell-Radial-Wash-Hintergrund
✅ Home-Screen-Icon `apple-touch-icon.png` (Mini-Kalender mit bunten Termin-Punkten — candy-Refresh ist ein guter nächster Schritt)
