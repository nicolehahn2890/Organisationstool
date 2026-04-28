---
name: organisationstool
description: >
  Use this skill for EVERY request related to the Organisationstool — a personal
  finance and weekly schedule tracker built in standalone HTML, deployed to GitHub
  Pages at nicolehahn2890.github.io/Organisationstool. Trigger on any mention of:
  "Organisationstool", "Tracker", "Ausgaben-Tracker", "Termin-Tracker",
  "Finanz-Tracker", "Kreditkarten-Tracker", "Amex/Visa/Girokonto Tracker",
  "Buchungsdatum", "Abbuchung", "Drei Wochen-Übersicht",
  "nicolehahn2890.github.io/Organisationstool", or any request to add/fix/style
  features in this app. Also trigger when the user uploads an HTML file related
  to the app or mentions the GitHub repo nicolehahn2890/Organisationstool.
  Never skip this skill for Organisationstool work.
---

# Organisationstool — Claude Code Skill

## Projekt-Überblick

- **Live-URL:** https://nicolehahn2890.github.io/Organisationstool
- **GitHub Repo:** https://github.com/nicolehahn2890/Organisationstool
- **Dateistruktur:** Einzelne `index.html` — alles in einer Datei, kein Build-Step
- **Deployment:** Direkt via GitHub Pages (push → live)
- **Zweck:** Persönlicher Tracker für Nicole mit zwei Hauptbereichen:
  1. **Ausgaben** — Tracking von Kreditkarten-Ausgaben mit automatischer Buchungsdatum-Berechnung
  2. **Termine** — 3-Wochen-Übersicht (diese + nächste + übernächste Woche)

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

### Statistik-Box (4 Stats nebeneinander)
1. **Ausgegeben** — Summe nach Ausgabedatum
2. **Abbuchung gesamt** — Summe nach Buchungsdatum (was geht diesen Monat tatsächlich aufs Konto)
3. **vs. Vormonat** — Prozent-Vergleich (rot wenn höher, grün wenn niedriger)
4. **Buchungen** — Anzahl Einträge

Plus: horizontales Balkendiagramm pro Kategorie (sortiert nach Höhe).

### Buchungsliste
- Sortiert nach Ausgabedatum (neueste zuerst)
- Pro Eintrag: Emoji, Beschreibung, Karte (mit Farb-Pill), Kategorie, Datum
- Bei Amex/Visa: italic "→ abgebucht TT.MM." als Zusatzhinweis
- Hover zeigt Lösch-X

### Monatsnavigation
- Pfeile ‹ › neben Monatslabel im Karten-Section-Header
- Funktionen `prevMonth(yyyymm)` / `nextMonth(yyyymm)` arbeiten **rein mit Strings** — KEIN `Date`-Objekt verwenden (Timezone-Probleme)
- Button-IDs heißen `btnPrevMonth` / `btnNextMonth` — NICHT `prevMonth`/`nextMonth`, sonst überschreibt der Browser die globalen Funktionen

---

## Tab 2: Termine

### Quick-Tags (9 feste Buttons)
```
💪 Training · 👥 Meeting · 💇 Haare waschen · 🧹 Putztag ·
🛒 Einkaufen · 📚 Lernen · 💉 Botox · ❤️ Date · 📞 Call
```

Klick auf Tag füllt das Beschreibungs-Feld vor. Erneuter Klick deaktiviert.
Eigener Freitext bleibt natürlich auch möglich.

### 3-Wochen-Übersicht
Zeigt immer:
1. **Diese Woche** (aktueller Wochenstart Montag bis Sonntag)
2. **Nächste Woche**
3. **Übernächste Woche**

Heute-Markierung: Tag wird mit minze-tint Hintergrund + minze Border hervorgehoben.

### Termin-Daten
- Beschreibung (Pflicht)
- Datum (Pflicht)
- Uhrzeit (optional)
- Farbe ist immer minze (für Termine fest)

### Termin-Detail-Modal
- Klick auf Termin-Chip öffnet Modal mit Datum + Uhrzeit
- Lösch-Button in rot

---

## Storage

- **localStorage-Key:** `nicole_tracker_v2`
- **NIEMALS** den Key ohne triftigen Grund ändern — würde Nutzerdaten löschen
- Falls schemata-breaking: Key bewusst inkrementieren (`_v3` etc.) und Migrations-Logik in `loadState()` ergänzen
- Persistiert wird: theme, selectedCardId, expenses, events, viewMonth
- **NICHT** persistiert: cards-Array (ist hartkodiert), Buchungsdaten (werden berechnet)

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
- Bei Änderungen: **Python-Script** zum Lesen → Modifizieren → Einmal Schreiben
- **Nicht** mehrere `str_replace` hintereinander auf dieselbe Datei wenn nicht nötig
- Bei deutschen Umlauten: Unicode-Escape verwenden falls Probleme
  - ä = `\u00e4`, ö = `\u00f6`, ü = `\u00fc`, Ä = `\u00c4` usw.

### Element-IDs vs. globale Funktionsnamen
- **Browser legt für jede Element-ID automatisch eine globale Variable an**
- Daher: NIEMALS Funktionsnamen als Element-ID verwenden
- Konkret: Buttons heißen `btnPrevMonth`/`btnNextMonth`, Funktionen heißen `prevMonth()`/`nextMonth()`

### KRITISCHE VERBOTE
- `°` Zeichen nicht verwenden → JavaScript-Fehler möglich, "Grad" schreiben
- localStorage-Key (`nicole_tracker_v2`) nicht ändern → Datenverlust
- Keine Karten hinzufügen/entfernen — die 3 sind fix
- Keine globale Akzent-Auswahl (Toggle) hinzufügen — Bereiche haben FESTE Farben

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

---

## Aktuelle Feature-Liste (Stand: April 2026)

✅ 3 feste Karten: Amex, Visa, Girokonto
✅ Automatische Buchungsdatum-Berechnung pro Karten-Typ
✅ Pending-Hinweis "+ X € folgt im nächsten Monat" auf Karten
✅ Light + Dark Mode (durchgängig, nicht nur Kacheln)
✅ 11 Kategorien inkl. 🎁 Geschenke
✅ Wiederkehrende Ausgaben (Abos) mit automatischer Projektion
✅ Statistik mit "Ausgegeben" + "Abbuchung gesamt" parallel
✅ Kategorien-Balkendiagramm
✅ Monatsnavigation (timezone-safe)
✅ 3-Wochen-Termin-Übersicht
✅ 9 Quick-Tags inkl. 💉 Botox und ❤️ Date
✅ localStorage-Persistenz unter `nicole_tracker_v2`
