# Angebot: Terminplanung & Kalender
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX

---

## Projektziel

Strukturierte Terminplanung direkt im Dashboard: Tatjana erfasst Termine pro Baustelle, weist Monteure zu — und sieht im eingebauten Kalender sofort, wer wann wo eingeplant ist und wer noch frei hat. Keine externe App, kein manueller Import, kein Hin- und Herwechseln zwischen Tools.

Monteure werden automatisch per **WhatsApp** benachrichtigt wenn sie eingeplant werden oder eine neue Info bei ihrer Baustelle erscheint.

---

## Leistungsumfang

### Phase 1 — MVP: Termin-Erfassung + Kalender

**Datenmodell & Migration**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | DB-Schema: Termine + Monteur-Zuweisung (inkl. Migration bestehender Daten) | 5h | 69€ | 345€ |

**Termin-Erfassung**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.2 | BE: CRUD-Endpoints für Termine + Monteur-Zuweisung | 4h | 69€ | 276€ |
| 1.3 | FE: Termin-Dialog (fix / offen / mehrtägig + Monteur-Auswahl) | 5h | 69€ | 345€ |
| 1.4 | FE: Termine auf Baustellendetailseite anzeigen, bearbeiten, löschen | 2h | 69€ | 138€ |

**Dashboard-Kalender**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.5 | BE: Termin-Endpoint für Kalender (nach Zeitraum + Mitarbeiter) | 3h | 69€ | 207€ |
| 1.6 | vue-cal Integration: Wochen-/Monatsansicht, Navigation, Farbcodierung pro Monteur | 5h | 69€ | 345€ |
| 1.7 | Gesamtansicht: alle Monteure + Filter (Monteure abwählbar per Checkbox) | 3h | 69€ | 207€ |
| 1.8 | Einzelansicht: einen Monteur auswählen → nur seine Termine + freie Tage sichtbar | 2h | 69€ | 138€ |
| 1.9 | Österreichische Feiertage automatisch im Kalender markiert | 0.5h | 69€ | 35€ |
| 1.10 | Offene Termine ("Juli 2026", "TBD") als Hinweis-Block im Kalender | 2h | 69€ | 138€ |

| **Zwischensumme Phase 1** | | **31.5h** | | **2.174€** |

---

### Phase 2 — WhatsApp-Benachrichtigungen

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 2.1 | BE: Benachrichtigung bei Termin-Zuweisung (WhatsApp Business API) | 3h | 69€ | 207€ |
| 2.2 | BE: Benachrichtigung bei neuer Baustellen-Notiz (nur zugewiesene Monteure) | 2h | 69€ | 138€ |
| 2.3 | WhatsApp Templates erstellen + Meta-Genehmigung (2 Templates) | 1h | 69€ | 69€ |
| 2.4 | DB: Monteur-Profile um WhatsApp-Nummer erweitern | 1h | 69€ | 69€ |

| **Zwischensumme Phase 2** | | **7h** | | **483€** |

---

## Gesamtübersicht

| Phase | Aufwand | Betrag netto |
|-------|---------|--------------|
| Phase 1 — Termin-Erfassung + Kalender | 31.5h | **2.174€** |
| Phase 2 — WhatsApp-Benachrichtigungen | 7h | **483€** |
| **Gesamt** | **38.5h** | **2.657€** |

*Alle Preise exkl. 20% MwSt.*

---

## Hinweise

**Kalender-Bibliothek:**
vue-cal (Open Source, kostenlos, Vue 3 native). Keine Lizenzkosten.

**WhatsApp Business API:**
Bereits im Projekt eingerichtet. Laufende Kosten: ca. 0,05–0,08€ pro Nachricht.
Bei 10 Monteuren und Ø 2–3 Benachrichtigungen/Tag: **~15–25€/Monat**.

**Meta Template-Genehmigung:**
WhatsApp Templates müssen von Meta genehmigt werden (üblicherweise 24–48h, nicht von uns steuerbar).

**Google/Outlook Sync:**
Nicht im Scope. Kann als separates Add-on in einer späteren Phase angeboten werden (einmaliger OAuth-Consent → Termine landen automatisch extern). Auf Anfrage.

---

## Voraussetzungen (kundenseitig)

- Bestehende Mitarbeiterdaten im System (vorhanden ✓)
- WhatsApp Business API eingerichtet (vorhanden ✓)
- WhatsApp-Nummern der 8 Monteure bereitstellen
- Bestehende .ics-Logik bleibt erhalten (kein Mehraufwand)

---

*Angebot gültig bis: 2026-08-03*  
*Erstellt von Wolfi, 2026-07-03*
