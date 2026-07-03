# Angebot: Terminplanung & Kalender — v2
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX  
**Version:** 2 (Schätzung nach Analyse der bestehenden Architektur)

**Änderungen gegenüber v1:**
- Terminerfassung existiert bereits in DB (Projekt hat Termin oder offen) → kein Neu-Aufbau, Erweiterung
- Monteur-Zuweisung zum Termin ist neu, Rest bereits vorhanden
- Schätzung reduziert von 38.5h / 2.657€ auf 27h / 1.863€

---

## Projektziel

Erweiterung der bestehenden Terminerfassung: Monteure werden Terminen zugewiesen. Neuer Dashboard-Kalender zeigt auf einen Blick wer wann wo eingeplant ist — mit Filter- und Einzelansicht. Monteure werden per WhatsApp benachrichtigt.

---

## Leistungsumfang

### Phase 1 — MVP: Erweiterung + Kalender

**Datenmodell (Erweiterung)**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | DB: `appointment_employees` JOIN-Tabelle (Monteur-Zuweisung zu Termin) | 1h | 69€ | 69€ |
| 1.2 | BE: Monteur-Zuweisung zu Termin (CRUD-Erweiterung bestehender Endpoints) | 3h | 69€ | 207€ |

**Termin-Erfassung (Erweiterung bestehender UI)**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.3 | FE: Termin-Dialog erweitern (Monteur-Auswahl, Mehrtages-Option) | 3h | 69€ | 207€ |
| 1.4 | FE: Termine auf Baustellendetailseite: Monteure anzeigen + bearbeiten | 2h | 69€ | 138€ |

**Dashboard-Kalender**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.5 | BE: Endpoint — Termine im Zeitraum, gruppiert nach Mitarbeiter | 2h | 69€ | 138€ |
| 1.6 | vue-cal Integration: Wochen-/Monatsansicht, Navigation, Farbcodierung pro Monteur | 4h | 69€ | 276€ |
| 1.7 | Gesamtansicht: alle Monteure + Filter (abwählbar per Checkbox) | 2h | 69€ | 138€ |
| 1.8 | Einzelansicht: Monteur auswählen → nur seine Termine + freie Tage sichtbar | 2h | 69€ | 138€ |
| 1.9 | Österreichische Feiertage (vue-cal AT-Plugin) | 0.5h | 69€ | 35€ |
| 1.10 | Offene Termine (fuzzy) als Hinweis-Block im Kalender | 1.5h | 69€ | 104€ |

| **Zwischensumme Phase 1** | | **21h** | | **1.449€** |

---

### Phase 2 — WhatsApp-Benachrichtigungen

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 2.1 | BE: Benachrichtigung bei Termin-Zuweisung | 2h | 69€ | 138€ |
| 2.2 | BE: Benachrichtigung bei neuer Baustellen-Notiz | 2h | 69€ | 138€ |
| 2.3 | WhatsApp Templates + Meta-Genehmigung (2 Templates) | 1h | 69€ | 69€ |
| 2.4 | DB: Monteur-Profile um WhatsApp-Nummer erweitern | 1h | 69€ | 69€ |

| **Zwischensumme Phase 2** | | **6h** | | **414€** |

---

## Gesamtübersicht

| Phase | Aufwand | Betrag netto |
|-------|---------|--------------|
| Phase 1 — Erweiterung + Kalender | 21h | **1.449€** |
| Phase 2 — WhatsApp-Benachrichtigungen | 6h | **414€** |
| **Gesamt** | **27h** | **1.863€** |

*Alle Preise exkl. 20% MwSt.*

**Laufende Kosten:** WhatsApp ~15–25€/Monat (bei 10 Monteuren, Ø 2–3 Nachrichten/Tag)

---

*Angebot gültig bis: 2026-08-03 | Erstellt von Wolfi, 2026-07-03*
