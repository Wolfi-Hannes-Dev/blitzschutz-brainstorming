# Backlog: Toggl Manueller Export (Admin Button)
**Projekt:** Blitzschutz Reichenhauser  
**Stand:** 2026-07-03

---

## Kontext & Architektur

**Bestehend:**
- Nächtlicher automatischer Export: Toggl → DB → Power Automate → Excel-Script
- DB enthält Mapping: E-Mail-Adresse → Toggl API-Token (pro Arbeiter, Free Tier)
- Ca. 10 Arbeiter mit eigenen privaten Toggl-Accounts

**Neu (dieser Backlog):**
- Admin-UI: Arbeiter auswählen + Zeitraum wählen → Button → gleicher Flow wie nächtlicher Export, aber manuell + für beliebigen Zeitraum

---

## Epics & Stories

---

### 🔵 EPIC 1 — Backend: Manueller Export-Endpoint

| ID | Story | Aufwand |
|----|-------|---------|
| E1-1 | Neuer BE-Endpoint: `POST /toggl/manual-export` (user_email, date_from, date_to) | 2h |
| E1-2 | API-Token aus DB holen anhand user_email | 1h |
| E1-3 | Toggl Reports API aufrufen: Zeiteinträge für Zeitraum holen (mit Pagination + Rate-Limit-Handling) | 4h |
| E1-4 | Daten in DB schreiben (gleiche Struktur wie nächtlicher Export) | 2h |
| E1-5 | Power Automate Flow triggern (gleicher Webhook wie nächtlicher Export) | 1h |
| E1-6 | Fehlerbehandlung: ungültiger Token, keine Einträge, Toggl-Fehler → sinnvolle Fehlermeldung ans Frontend | 2h |
| E1-7 | Excel-Script anpassen: Mehrere Einträge pro Übergabe verarbeiten (Loop statt Single-Entry-Logik) | 3h |

**Summe Epic 1: ~15h**

---

### 🟢 EPIC 2 — Frontend: Admin-UI

| ID | Story | Aufwand |
|----|-------|---------|
| E2-1 | Dropdown: Arbeiter auswählen (Liste aus DB, Name + E-Mail) | 2h |
| E2-2 | Datumsauswahl: Von/Bis (Datepicker, Validierung: Von ≤ Bis, kein Zukunftsdatum) | 2h |
| E2-3 | Button "Export starten" → BE-Request abschicken | 1h |
| E2-4 | Status-Feedback: Loading, Erfolg (X Einträge importiert), Fehler | 2h |

**Summe Epic 2: ~7h**

---

### 🟡 EPIC 3 — Robustheit & Edge Cases

| ID | Story | Aufwand |
|----|-------|---------|
| E3-1 | Doppelte Einträge vermeiden: wenn Zeitraum bereits in DB → überschreiben oder skippen (Upsert-Logik) | 2h |
| E3-2 | Rate-Limit-Handling: Toggl erlaubt 1 req/sec → bei großen Zeiträumen Queue + Backoff | 2h |
| E3-3 | Logging: welcher Admin hat wann welchen Export für wen ausgelöst | 1h |

**Summe Epic 3: ~5h**

---

## Phasen-Planung

### Phase 1 — MVP (Empfehlung)
Epic 1 + Epic 2
→ **~22h** | Ergebnis: Funktionierender Admin-Button, End-to-End

### Phase 2 — Qualität
Epic 3
→ **+5h** | Ergebnis: Robuster gegen Doppelexporte + große Zeiträume

---

## Bekannte Schwachstellen / Risiken

- **Free Tier Toggl:** API-Zugriff auf eigene Einträge ist möglich, aber Rate-Limit (1 req/sec) muss beachtet werden. Bei Zeiträumen > 1 Woche mit vielen Einträgen kann der Export mehrere Sekunden dauern.
- **Tokens in DB:** Wenn ein Arbeiter seinen Toggl-Account löscht oder Token erneuert, veraltet der Token. → Kein automatischer Sync möglich (Free Tier). Admin muss Token manuell aktualisieren.
- **Power Automate / Excel-Script:** Das bestehende Script ist auf 1 Eintrag ausgelegt. Anpassung für mehrere Einträge pro Export ist in Epic 1 (E1-7) inkludiert.

---

*Erstellt von Wolfi, 2026-07-03*
