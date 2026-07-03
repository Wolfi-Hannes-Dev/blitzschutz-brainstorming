# Backlog: Terminplanung & Kalender
**Projekt:** Blitzschutz Reichenhauser  
**Stand:** 2026-07-03 (finales Konzept nach Kundenantworten + Abstimmung)

---

## Finales Konzept

**Dashboard-eigener Kalender** (vue-cal, kostenlos) — keine Abhängigkeit von Google/Outlook.

Wenn ein Termin im Dashboard erstellt wird → landet sofort im Kalender. Keine manuelle Doppeleingabe. Die .ics bleibt als optionaler Download erhalten.

### Kalender-Features:
- **Gesamtansicht:** Alle Monteure gleichzeitig, jeder in seiner Farbe — "wer ist wo"
- **Filter:** Monteure per Checkbox abwählbar (Gesamtansicht bereinigen)
- **Einzelansicht:** Einen Monteur auswählen → nur seine Termine → freie Tage sofort erkennbar
- **Österreichische Feiertage** automatisch markiert (vue-cal AT-Plugin, kein Mehraufwand)
- **Offene Termine** (fuzzy: "Juli 2026", "TBD") als Hinweis-Block am Monatsanfang
- Navigation: Wochen-/Monatsansicht, vor/zurück blättern
- Desktop first (Tablet/Mobile: spätere Phase)

### Termin-Erfassung (unverändert):
- Pro Baustelle: fixer Termin (Datum) oder offener Termin (Dropdown)
- Mehrtages-Option (von/bis)
- Ein oder mehrere Monteure zuweisen
- Nur BO kann Termine erstellen/bearbeiten/löschen

### Benachrichtigungen:
- Monteur bekommt WhatsApp wenn er einem Termin zugewiesen wird
- Monteur bekommt WhatsApp wenn neue Notiz bei seiner Baustelle erscheint

### Kalender-Sync (Phase 3, optional):
- "In Google Calendar exportieren" Button
- Einmaliger OAuth-Consent → neue Termine landen automatisch extern
- Nicht im aktuellen Scope

---

## Epics & Stories

---

### 🔵 EPIC 1 — Datenmodell & Migration

| ID | Story | Aufwand |
|----|-------|---------|
| E1-1 | DB-Schema: `appointments` (constructionSiteId, date, endDate, notes, fuzzyLabel, isMultiDay) | 2h |
| E1-2 | DB-Schema: `appointment_employees` JOIN-Tabelle | 1h |
| E1-3 | Migration: bestehende Monteur-Zuweisung pro Baustelle prüfen + überführen | 2h |

**Summe Epic 1: ~5h**

---

### 🟢 EPIC 2 — Termin-Erfassung

| ID | Story | Aufwand |
|----|-------|---------|
| E2-1 | BE: CRUD-Endpoints für Termine + Monteur-Zuweisung | 4h |
| E2-2 | FE: "Neuer Termin" Dialog (Typ, Datum/Fuzzy, Mehrtages, Monteur-Auswahl) | 5h |
| E2-3 | FE: Termine auf Baustellendetailseite anzeigen, bearbeiten, löschen | 2h |

**Summe Epic 2: ~11h**

---

### 🟢 EPIC 3 — Dashboard-Kalender

| ID | Story | Aufwand |
|----|-------|---------|
| E3-1 | BE: Endpoint — alle Termine im Zeitraum, gruppiert nach Mitarbeiter | 3h |
| E3-2 | vue-cal Integration: Wochen-/Monatsansicht, Navigation | 3h |
| E3-3 | Farbcodierung: jeder Monteur = eigene Farbe, Legende | 2h |
| E3-4 | Gesamtansicht: alle Monteure + Filter (Checkboxen zum Abwählen) | 3h |
| E3-5 | Einzelansicht: Monteur auswählen → nur seine Termine sichtbar | 2h |
| E3-6 | Österreichische Feiertage einbinden (vue-cal AT-Plugin) | 0.5h |
| E3-7 | Offene Termine (fuzzy) als Hinweis-Block am Monatsanfang | 2h |

**Summe Epic 3: ~15.5h**

---

### 🔔 EPIC 4 — WhatsApp-Benachrichtigungen

| ID | Story | Aufwand |
|----|-------|---------|
| E4-1 | BE: Benachrichtigung bei Termin-Zuweisung (WhatsApp Business API) | 3h |
| E4-2 | BE: Benachrichtigung bei neuer Baustellen-Notiz (nur zugewiesene Monteure) | 2h |
| E4-3 | WhatsApp Templates erstellen + Meta-Genehmigung (2 Templates) | 1h |
| E4-4 | DB: Monteur-Profil um WhatsApp-Nummer erweitern | 1h |

**Summe Epic 4: ~7h**

---

## Phasen-Planung

### Phase 1 — MVP
Epic 1 + Epic 2 + Epic 3
→ **~31.5h** | Ergebnis: Termin-Erfassung + Kalender mit Filter- und Einzelansicht, Feiertage

### Phase 2 — Benachrichtigungen
Epic 4
→ **+7h** | Ergebnis: Monteure werden automatisch per WhatsApp informiert

### Phase 3 — Google Calendar Sync (optional, auf Anfrage)
→ Separates Angebot bei Bedarf

---

## Bewusst nicht im Scope
- Google/Outlook Kalender-Sync (Phase 3, optional)
- Monteur-Login / Monteur-eigene Kalenderansicht
- Konflikt-Warnung bei Doppelbelegung
- Kostenpflichtige Kalender-Bibliotheken

---

*Erstellt von Wolfi, 2026-07-03*
