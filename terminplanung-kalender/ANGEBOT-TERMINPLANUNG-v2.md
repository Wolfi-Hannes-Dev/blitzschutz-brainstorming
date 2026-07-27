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

### Phase 3 — Mitarbeiter-Kalenderansicht

*Ergänzung 2026-07-27. Jeder Mitarbeiter erhält eine eigene Kalenderansicht mit seinen Terminen. Wird er einem Projekt zugewiesen, erscheint das Projekt automatisch in seinem Kalender. Umsetzung wie beim Admin-Kalender mit calendar.js — die Kalender-Komponente aus Phase 1 wird im Einzel-Modus wiederverwendet.*

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 3.1 | BE: Endpoint — Termine eines Mitarbeiters im Zeitraum | 2h | 69€ | 138€ |
| 3.2 | FE: Route + Ansicht „Mein Kalender" (nur für angemeldeten Monteur) | 3h | 69€ | 207€ |
| 3.3 | FE: Kalender-Komponente im Einzel-Modus wiederverwenden | 3h | 69€ | 207€ |
| 3.4 | FE: Zugewiesenes Projekt am Termin + Sprung zur Baustelle | 2h | 69€ | 138€ |
| 3.5 | FE: Einstieg aus „Meine Baustellen" + Leerzustand | 2h | 69€ | 138€ |

| **Zwischensumme Phase 3** | | **12h** | | **828€** |

*Setzt Phase 1 voraus (Datenmodell + Kalender-Komponente). Stunden aus der Code-Analyse vom 2026-07-27.*

---

## Gesamtübersicht

| Phase | Aufwand | Betrag netto |
|-------|---------|--------------|
| Phase 1 — Erweiterung + Kalender | 21h | **1.449€** |
| Phase 2 — WhatsApp-Benachrichtigungen | 6h | **414€** |
| Phase 3 — Mitarbeiter-Kalenderansicht | 12h | **828€** |
| **Gesamt** | **39h** | **2.691€** |

*Alle Preise exkl. 20% MwSt.*

**Laufende Kosten:** WhatsApp ~15–25€/Monat (bei 10 Monteuren, Ø 2–3 Nachrichten/Tag)

> **Hinweis (intern, 2026-07-27):** Die Stunden für Phase 1 + 2 stammen aus Angebot v2 (Wolfi).
> Die interne Code-Analyse veranschlagt allein Phase 1 deutlich höher (Datenmodell + Kalender);
> siehe `aufwandsschaetzung-4-topics.xlsx`. Vor Kundenkontakt abgleichen.

---

*Angebot gültig bis: 2026-08-03 | Erstellt von Wolfi, 2026-07-03 · Phase 3 ergänzt 2026-07-27*
