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

## Projekt-Überblick

- **Live-URL:** https://nicolehahn2890.github.io/Organisationstool
- **GitHub Repo:** https://github.com/nicolehahn2890/Organisationstool
- **Dateistruktur:** Einzelne `index.html` — alles in einer Datei, kein Build-Step
- **Deployment:** Direkt via GitHub Pages (push → live)
- **Zweck:** Persönlicher Tracker für Nicole mit zwei Hauptbereichen:
  1. **Ausgaben** — Tracking von Kreditkarten-Ausgaben mit automatischer Buchungsdatum-Berechnung
  2. **Termine** — 3-Wochen-Übersicht mit wiederkehrenden Terminen + einzelne Instanz-Bearbeitung

---

## Design-System (Editorial Modern)

### Designsprache
- Editorial / Magazin-Stil — warmes Off-White, viel Weißraum, Typografie als Hauptelement
- Klare Hierarchie durch große italic Section-Titles
- Akzentfarben pro Bereich fest zugewiesen — KEIN globaler Akzent-Toggle

### Schriften (Google Fonts CDN)
- **Fraunces** (Italic-Serife) — nur für Headlines, Beträge, Tagesnummern
- **Inter** — Body-Text, alles andere

### Farbpalette (5 Akzente)
```
peach       — #E89878 (light) / #F0AE92 (dark)   → AMEX-Bereich, Karten-Section
rosa        — #E29BB5 / #E5A8BC                   → "Neue Ausgabe"-Section, Visa-Karte
minze       — #7CC4A7 / #9DD4BC                   → Termine-Bereich, Girokonto, "Heute"
buttermilk  — #D9B85A / #E5C77A                   → Statistik / Überblick
violet      — #A98AC9 / #BFA5DA                   → Buchungsliste, Abo-Tags
```

### Bereich-zu-Farbe-Zuordnung (FEST, nicht ändern)
| Bereich | Farbe | CSS-Klasse |
|---|---|---|
| Karten-Übersicht | peach | `zone-peach` |
| Neue Ausgabe (Form) | rosa | `zone-rosa` |
| Überblick / Stats | buttermilk | `zone-buttermilk` |
| Buchungsliste | violet | `zone-violet` |
| Termine (kompletter Tab) | minze | `zone-minze` |

### Theme: Light / Dark Mode
- **Light:** Off-White Hintergrund `#F6F2EC`, warme Brauntöne
- **Dark:** Anthrazit `#1A1816` (NICHT pures Schwarz!), Surfaces deutlich abgesetzt `#252220`
- Theme-Attribut wird auf `<html>` UND `<body>` gesetzt — beide Selektoren werden gebraucht
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
- **Pending-Hinweis (italic):** "+ X € folgt im nächsten Monat" — wenn es Käufe aus diesem Monat gibt deren Buchungsdatum in einem späteren Monat liegt

### Kategorien (11 Stück, mit Emojis)
```
🍽️ Essen · 🚆 Transport · 🏠 Wohnen · 💪 Sport · 🎬 Freizeit ·
👗 Shopping · 💊 Gesundheit · 🎁 Geschenke · 📱 Abos · ✈️ Reisen · 📦 Sonstiges
```

### Wiederkehrende Ausgaben (Abos)
- Checkbox bei "Neue Ausgabe": "Wiederkehrende Ausgabe (Abo)"
- Werden automatisch in Folgemonate projiziert (`isProjected: true`)
- Werden mit Tag "ABO" in violet markiert
- Beim Löschen einer projizierten Instanz wird die Original-Quelle gelöscht (`recurringSource`)

### Eigener Anteil pro Buchung (`eigenAmount`)
- Optionales Feld "Mein Anteil (€) — optional" im Formular bei JEDER Kategorie
- Anwendungsfall: Nicole geht für sich + ihren Freund einkaufen → trackt was davon ihr eigener Anteil ist
- Gespeichert als `expense.eigenAmount: number | null`
- **Hat NULL Einfluss** auf Karten-Summen, Buchungen, Abbuchungen — reine Info-Spalte
- Wird in Summe als 3. Stat-Box "Mein Anteil" angezeigt
- In der Liste pro Eintrag dezent als italic Hinweis: "davon mein: X €" in minze-Farbe (`.eigen-hint`)
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
- Bei Amex/Visa: italic "→ abgebucht TT.MM." als Zusatzhinweis
- Bei eigenAmount > 0: italic "davon mein: X €" in minze
- Hover zeigt Lösch-X
- **Beschreibung MUSS umbrechen können** — `.exp-desc` darf NICHT `white-space: nowrap` + `text-overflow: ellipsis` haben (kappt sonst lange Texte). Stattdessen: `word-break: break-word; overflow-wrap: anywhere; flex-wrap: wrap`

### Monatsnavigation
- Pfeile ‹ › neben Monatslabel im Karten-Section-Header
- Funktionen `prevMonth(yyyymm)` / `nextMonth(yyyymm)` arbeiten **rein mit Strings** — KEIN `Date`-Objekt verwenden (Timezone-Probleme)
- Button-IDs heißen `btnPrevMonth` / `btnNextMonth` — NICHT `prevMonth`/`nextMonth`, sonst überschreibt der Browser die globalen Funktionen

---

## Tab 2: Termine

### Quick-Tags (10 feste Buttons mit Default-Farben)
Jeder Tag hat eine fest zugeordnete `data-color` als Vorschlag:

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
Dropdown im Formular: **Keine / Wöchentlich / Monatlich**

Gespeichert als `event.recurrence: 'weekly' | 'monthly' | null`.

Wiederkehrende Instanzen werden im Chip mit "↻ wöchentlich" / "↻ monatlich" markiert.

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
  4. Reguläre Recurrence-Logik (weekly/monthly)
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
- Recurrence (null/weekly/monthly)
- Overrides (optional, nur wenn einzelne Instanzen geändert wurden)

### Termin-Chip Layout
- Linke Border in der Termin-Farbe
- Optional Uhrzeit (klein)
- Beschreibung — **MUSS** umbrechen können (`white-space: normal`, `overflow-wrap: anywhere`) — KEIN `text-overflow: ellipsis` mit `nowrap` (kappt sonst lange Texte ab)
- Optional Recurrence-Label

---

## Storage

- **localStorage-Key:** `nicole_tracker_v2`
- **NIEMALS** den Key ohne triftigen Grund ändern — würde Nutzerdaten löschen
- Falls schemata-breaking: Key bewusst inkrementieren (`_v3` etc.) und Migrations-Logik in `loadState()` ergänzen
- Persistiert wird: theme, selectedCardId, expenses, events (inkl. overrides), viewMonth
- **NICHT** persistiert: cards-Array (ist hartkodiert), Buchungsdaten (werden berechnet)

### Schema-Erweiterungen müssen abwärtskompatibel sein
Beim Hinzufügen neuer Felder (wie damals `recurrence` und `overrides`):
- Altes Schema muss weiter funktionieren
- Defaults via `??` oder Existenz-Checks (`if (ev.overrides) { ... }`)
- KEIN Zwang auf neues Feld

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
   - Keine `toISOString()`-Aufrufe
   - Keine `°` Zeichen
   - Keine Form-Tags
   - localStorage-Key unverändert
   - Theme-Attribut auf `<html>` UND `<body>`
   - Schema-Erweiterungen abwärtskompatibel
6. **Bereitstellen:** Datei nach `/mnt/user-data/outputs/index.html` kopieren und via `present_files` an Nicole geben
7. **Deployment:** Nicole lädt die Datei selbst via GitHub Web-Interface ins Repo (Drag & Drop in "Add file → Upload files")

---

## Bekannte Stolperfallen

- `toISOString()` → Timezone-Drift. Termine landen einen Tag früher/später. Immer `dateToLocalISO()` nutzen.
- Element-ID = Funktionsname → globale Variable überschreibt Funktion. Buttons mit `btn`-Präfix benennen.
- Dark Mode wirkt "kaputt" wenn nur einzelne Kacheln dunkel sind — sicherstellen dass `bg`, `surface`, `surface-2`, `line`, `text` alle als CSS-Variablen aus dem Theme-Block gezogen werden, keine hardcoded Hex-Werte außerhalb des Theme-Blocks (Ausnahme: Akzentfarben, Trend-Rot/Grün).
- Surface-Farbe im Dark Mode muss **deutlich heller** als BG sein (`#252220` vs. `#1A1816`) — sonst keine Hierarchie.
- Vorschau-Cache: Bei Tests immer Hard-Reload (Strg+F5) — sonst zeigt der Browser veraltete Versionen.
- Wenn Karte 0,00 € zeigt obwohl Buchungen drin sind: Pending-Hinweis prüfen — Käufe könnten erst im Folgemonat abgebucht werden (richtige Logik, missverständliche Anzeige verhindert das).
- Termin-Texte werden abgeschnitten → IMMER `white-space: normal` + `overflow-wrap: anywhere` für `.evt-text`
- **Buchungslisten-Texte werden abgeschnitten** → IMMER `word-break: break-word` + `overflow-wrap: anywhere` + `flex-wrap: wrap` für `.exp-desc`. Ellipsis ist hier nie erwünscht (anders als z.B. in Mail-Listen).
- **Stats-Grid mit ungerader Box-Anzahl in Mobile-2-Spalten-Layout** → letzte Box wirkt halb-leer (zeigt grauen Border-Bereich). Fix: `.stats-row .stat:last-child { grid-column: span 2 }` im Mobile-Media-Query. Bei Box-Anzahl-Änderung neu prüfen.
- Bei Override-Logik: `overrides[sourceDate] === null` ist NICHT dasselbe wie `!overrides[sourceDate]` — explizit auf null prüfen, sonst werden gelöschte und nicht-existente Instanzen verwechselt

---

## Aktuelle Feature-Liste (Stand: April 2026)

### Ausgaben
✅ 3 feste Karten: Amex, Visa, Girokonto
✅ Automatische Buchungsdatum-Berechnung pro Karten-Typ (Giro=sofort, Amex=22./Folgemonat, Visa=+30 Tage)
✅ Pending-Hinweis "+ X € folgt im nächsten Monat" auf Karten
✅ 11 Kategorien inkl. 🎁 Geschenke
✅ Wiederkehrende Ausgaben (Abos) mit automatischer Projektion
✅ Optionales `eigenAmount`-Feld pro Buchung (z.B. wenn für sich + Partner gekauft)
✅ Statistik mit 5 Boxen: Ausgegeben + Abbuchung gesamt + Mein Anteil + vs. Vormonat + Buchungen
✅ Kategorien-Balkendiagramm
✅ Monatsnavigation (timezone-safe)
✅ Buchungslisten-Texte brechen um (kein Abschneiden)

### Termine
✅ 10 Quick-Tags mit Default-Farben (inkl. 💉 Botox, ❤️ Date, 🎉 Feiertag)
✅ Farb-Picker im Formular (Auto + 5 Farben, manuell überschreibbar)
✅ Wiederholungen: keine / wöchentlich / monatlich
✅ Override-System: einzelne Serien-Instanzen löschen/verschieben/umbenennen ohne Serie zu zerstören
✅ Modal mit 4 Aktionen bei Recurring (Diesen/Serie × Löschen/Bearbeiten)
✅ 3-Wochen-Übersicht (timezone-safe)
✅ Termin-Texte brechen um (kein Abschneiden)

### Allgemein
✅ Light + Dark Mode (durchgängig, nicht nur Kacheln)
✅ localStorage-Persistenz unter `nicole_tracker_v2`
✅ Editorial-Modern Design mit Fraunces + Inter
