# Angebot: Terminplanung & Kalender
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX

---

## Projektziel

Strukturierte Terminplanung im Dashboard: Tatjana kann pro Baustelle Termine erfassen und Monteure zuweisen. Die neue **Verfügbarkeitsansicht** zeigt auf einen Blick, wer wann eingeplant ist — damit entfällt das manuelle Durchsuchen. Monteure werden automatisch per **WhatsApp** benachrichtigt wenn sie einem Termin zugewiesen oder eine neue Info bei ihrer Baustelle hinterlegt wird.

---

## Leistungsumfang

### Phase 1 — MVP

**Datenmodell & Migration**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | DB-Schema: Termine + Monteur-Zuweisung (inkl. Migration bestehender Daten) | 5h | 69€ | 345€ |

**Termin-Erfassung**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.2 | BE: CRUD-Endpoints für Termine + Monteur-Zuweisung | 6h | 69€ | 414€ |
| 1.3 | FE: Termin-Dialog (fixer Termin / offener Termin / Mehrtages, Monteur-Auswahl) | 8h | 69€ | 552€ |
| 1.4 | FE: Termine auf Baustellendetailseite anzeigen, bearbeiten, löschen | 2h | 69€ | 138€ |

**Verfügbarkeitsansicht**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.5 | BE: Endpoint — Termine im Zeitraum, gruppiert nach Mitarbeiter | 3h | 69€ | 207€ |
| 1.6 | FE: Verfügbarkeits-Grid (Monteure = Spalten, Tage = Zeilen, belegte Tage farblich markiert) | 8h | 69€ | 552€ |
| 1.7 | FE: Zeitraum-Navigation (Wochen- / Monatssprung) | 2h | 69€ | 138€ |

| **Zwischensumme Phase 1** | | **34h** | | **2.346€** |

---

### Phase 2 — Kalender + Benachrichtigungen

**Klassische Kalenderansicht**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 2.1 | Kalender-Integration (vue-cal, kostenlos): Wochen-/Monatsansicht, Farbcodierung nach Monteur | 5h | 69€ | 345€ |
| 2.2 | Filter: Monteure ein-/ausblenden | 2h | 69€ | 138€ |
| 2.3 | Offene Termine (fuzzy) als Hinweis-Block im Kalender | 2h | 69€ | 138€ |

**WhatsApp-Benachrichtigungen**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 2.4 | BE: Benachrichtigung bei Termin-Zuweisung (WhatsApp Business API) | 3h | 69€ | 207€ |
| 2.5 | BE: Benachrichtigung bei neuer Baustellen-Notiz | 2h | 69€ | 138€ |
| 2.6 | WhatsApp Templates erstellen + Meta-Genehmigung (2 Templates) | 2h | 69€ | 138€ |
| 2.7 | DB: Monteur-Profile um WhatsApp-Nummer erweitern | 1h | 69€ | 69€ |

| **Zwischensumme Phase 2** | | **17h** | | **1.173€** |

---

## Gesamtübersicht

| Phase | Aufwand | Betrag netto |
|-------|---------|--------------|
| Phase 1 — MVP (Termin-Erfassung + Verfügbarkeitsansicht) | 34h | **2.346€** |
| Phase 2 — Kalender + WhatsApp-Benachrichtigungen | 17h | **1.173€** |
| **Gesamt** | **51h** | **3.519€** |

*Alle Preise exkl. 20% MwSt.*

---

## Hinweise

**Kalender-Bibliothek:**  
Die Verfügbarkeitsansicht (Phase 1) wird als eigenes React-Grid umgesetzt — keine externe Bibliothek nötig. Für den klassischen Kalender in Phase 2 verwenden wir **vue-cal** (Open Source, kostenlos). Keine Lizenzkosten.

**WhatsApp Business API:**  
Die API ist bereits im Projekt eingerichtet. Laufende Kosten: WhatsApp-Benachrichtigungen (Utility Templates) kosten ca. 0,05–0,08€ pro Nachricht. Bei 10 Monteuren und durchschnittlich 2–3 Benachrichtigungen/Tag: ~15–25€/Monat.

**Meta Template-Genehmigung:**  
WhatsApp Templates müssen von Meta genehmigt werden (üblicherweise 24–48h). Timing ist nicht von uns steuerbar.

---

## Voraussetzungen (kundenseitig)

- Bestehende Mitarbeiterdaten im System (vorhanden ✓)
- WhatsApp-Nummern der 8 Monteure bereitstellen
- Bestehende Termineingabe-Logik im System (vorhanden ✓)

---

*Angebot gültig bis: 2026-08-03*  
*Erstellt von Wolfi, 2026-07-03*
