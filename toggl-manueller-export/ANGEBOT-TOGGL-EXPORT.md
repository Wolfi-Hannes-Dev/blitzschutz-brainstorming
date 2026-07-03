# Angebot: Toggl Manueller Export
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX

---

## Projektziel

Erweiterung des bestehenden Toggl-Export-Systems um einen manuellen Admin-Button: Der Admin wählt einen Arbeiter und einen Zeitraum aus — das System holt die Toggl-Zeiteinträge, schreibt sie in die DB und löst den bestehenden Power Automate Flow aus. Gleicher Ablauf wie der nächtliche automatische Export, aber manuell steuerbar.

---

## Leistungsumfang

### Phase 1 — MVP

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | BE-Endpoint: manueller Export (user, Zeitraum) | 2h | 69€ | 138€ |
| 1.2 | Toggl API-Aufruf: Token aus DB + Zeiteinträge holen (inkl. Pagination + Rate-Limit) | 5h | 69€ | 345€ |
| 1.3 | DB-Schreibung + Power Automate Trigger | 3h | 69€ | 207€ |
| 1.4 | Fehlerbehandlung (ungültiger Token, keine Einträge, API-Fehler) | 2h | 69€ | 138€ |
| 1.5 | Frontend: Arbeiter-Dropdown + Datepicker + Button + Status-Feedback | 7h | 69€ | 483€ |
| **Zwischensumme Phase 1** | | **19h** | | **1.311€** |

---

### Phase 2 — Robustheit (optional)

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 2.1 | Upsert-Logik: Doppelte Einträge bei Wiederholung vermeiden | 2h | 69€ | 138€ |
| 2.2 | Queue + Backoff für große Zeiträume (Rate-Limit) | 2h | 69€ | 138€ |
| 2.3 | Audit-Log: Wer hat wann welchen Export ausgelöst | 1h | 69€ | 69€ |
| **Zwischensumme Phase 2** | | **5h** | | **345€** |

---

## Gesamtübersicht

| Phase | Aufwand | Betrag netto |
|-------|---------|--------------|
| Phase 1 — MVP | 19h | **1.311€** |
| Phase 2 — Robustheit | 5h | **345€** |
| **Gesamt** | **24h** | **1.656€** |

*Alle Preise exkl. 20% MwSt.*

---

## Hinweise & Risiken

**⚠️ Excel-Script Kompatibilität (nicht im Scope):**  
Das bestehende Power Automate / Excel-Script ist auf einzelne Tageseinträge ausgelegt. Bei manuellen Exporten über mehrere Tage/Wochen können mehrere Einträge gleichzeitig übergeben werden. Es sollte geprüft werden ob das Script das korrekt verarbeitet — andernfalls ist eine Anpassung des Scripts separat zu beauftragen.

**ℹ️ Toggl Free Tier:**  
Kein zentraler Admin-Zugriff möglich. Das System verwendet weiterhin die individuellen API-Tokens der Arbeiter. Wenn ein Arbeiter seinen Token erneuert, muss er manuell in der DB aktualisiert werden.

**ℹ️ Laufende API-Kosten:**  
Toggl API ist kostenlos. Keine zusätzlichen Drittanbieter-Kosten.

---

## Voraussetzungen (kundenseitig)

- Bestehende DB mit E-Mail → API-Token Mapping (vorhanden ✓)
- Bestehender Power Automate Flow mit Webhook-Trigger (vorhanden ✓)
- Zugang zur bestehenden Codebase (Backend + Frontend)

---

*Angebot gültig bis: 2026-08-03*  
*Erstellt von Wolfi, 2026-07-03*
