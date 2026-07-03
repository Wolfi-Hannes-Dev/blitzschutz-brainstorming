# Angebot: Toggl Manueller Export — v2
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX  
**Version:** 2 (Schätzung nach Analyse der bestehenden Architektur)

**Änderungen gegenüber v1:**
- Umsetzung: Tage werden einzeln an Power Automate geschickt → Flow + Excel-Script bleiben unverändert, kein Script-Umbau nötig
- Excel-Script Anpassung (3h) entfällt
- Schätzung reduziert von 27h / 1.863€ auf 16h / 1.104€

---

## Projektziel

Neuer Admin-Button: Arbeiter auswählen + Zeitraum wählen → System holt Toggl-Zeiteinträge, schreibt sie in die DB und schickt jeden Tag einzeln an den bestehenden Power Automate Flow. Bestehender Flow + Excel-Script bleiben vollständig erhalten.

---

## Leistungsumfang

### Phase 1 — MVP

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | BE-Endpoint: manueller Export (user, Zeitraum) | 2h | 69€ | 138€ |
| 1.2 | Toggl API-Aufruf: Token aus DB + Zeiteinträge holen (inkl. Pagination + Rate-Limit) | 4h | 69€ | 276€ |
| 1.3 | DB-Schreibung + Tage einzeln an Power Automate schicken (Loop, bestehender Webhook) | 3h | 69€ | 207€ |
| 1.4 | Fehlerbehandlung (ungültiger Token, keine Einträge, API-Fehler) | 2h | 69€ | 138€ |
| 1.5 | Frontend: Arbeiter-Dropdown + Datepicker + Button + Status-Feedback | 5h | 69€ | 345€ |

| **Zwischensumme Phase 1** | | **16h** | | **1.104€** |

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
| Phase 1 — MVP | 16h | **1.104€** |
| Phase 2 — Robustheit | 5h | **345€** |
| **Gesamt** | **21h** | **1.449€** |

*Alle Preise exkl. 20% MwSt.*

---

*Angebot gültig bis: 2026-08-03 | Erstellt von Wolfi, 2026-07-03*
