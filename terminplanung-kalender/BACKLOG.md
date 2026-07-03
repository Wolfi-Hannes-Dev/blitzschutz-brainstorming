# Backlog: Terminplanung & Kalender
**Projekt:** Blitzschutz Reichenhauser  
**Stand:** 2026-07-03  
**Basis:** Fragebogen + Kundenantworten

---

## Architektur-Entscheidungen (aus Kundenantworten)

- **Keine Stundengenauigkeit nötig:** Tatjana plant auf Tages-Ebene ("Monteur A ist am Montag auf Baustelle XY") — keine Start/End-Zeit zwingend erforderlich für Verfügbarkeitsview
- **Desktop first**, Tablet/Mobile später (kein Responsive-Fokus im MVP)
- **Kein Klick zur Baustelle** aus dem Kalender im MVP (nur relevant wenn Monteure selbst den Kalender sehen — aktuell kein Monteur-Login geplant)
- **Benachrichtigungen via WhatsApp** (WhatsApp Business API bereits im Projekt vorhanden)
- **Mehrtages-Einsätze:** Tatjana trägt nur Fixtermine ein wenn sie Bescheid bekommt — kein automatisches Spanning nötig, jeder Termin ist ein einzelner Tag oder ein explizit eingetragener Zeitraum

---

## Epics & Stories

---

### 🔵 EPIC 1 — Datenmodell & Migration

| ID | Story | Aufwand |
|----|-------|---------|
| E1-1 | DB-Schema: `appointments` Tabelle (constructionSiteId, date, notes, fuzzyLabel, isMultiDay, endDate) | 2h |
| E1-2 | DB-Schema: `appointment_employees` JOIN-Tabelle (appointmentId, employeeId) | 1h |
| E1-3 | Migration: bestehende Monteur-Zuweisung pro Baustelle prüfen + ggf. in neues Schema überführen | 2h |

**Summe Epic 1: ~5h**

---

### 🟢 EPIC 2 — Termin-Erfassung (Backend + Frontend)

| ID | Story | Aufwand |
|----|-------|---------|
| E2-1 | BE: CRUD-Endpoints für Termine (create, update, delete, list by constructionSite) | 4h |
| E2-2 | BE: Monteur-Zuweisung zu Termin (ein oder mehrere) | 2h |
| E2-3 | FE: "Neuer Termin" Button auf Baustellendetailseite | 1h |
| E2-4 | FE: Termintyp wählen — Fixer Termin (Datum) oder Offener Termin (Dropdown: "Juli 2026", "August 2026", "Q3 2026", "TBD", ...) | 3h |
| E2-5 | FE: Monteur(e) auswählen (Multi-Select aus 8 Mitarbeitern) | 2h |
| E2-6 | FE: Mehrtages-Option (von/bis Datum) | 2h |
| E2-7 | FE: Termin bearbeiten + löschen (nur BO) | 2h |

**Summe Epic 2: ~16h**

---

### 🟢 EPIC 3 — Verfügbarkeitsansicht (Prio-1-Use-Case)

> „Wer hat nächste Woche Zeit?" — Alle 8 Monteure nebeneinander, belegte Tage sofort sichtbar.

| ID | Story | Aufwand |
|----|-------|---------|
| E3-1 | BE: Endpoint — alle Termine im Zeitraum, gruppiert nach Mitarbeiter | 3h |
| E3-2 | FE: Ressourcen-Grid (Monteure = Spalten, Tage = Zeilen) für wählbaren Zeitraum (Standard: nächste 7 Tage) | 5h |
| E3-3 | FE: Belegte Tage farblich markiert (je Baustelle ein Block mit Baustellenname), freie Tage leer | 3h |
| E3-4 | FE: Zeitraum-Navigation (vor/zurück Woche, Monatssprung) | 2h |

**Summe Epic 3: ~13h**

---

### 🟡 EPIC 4 — Kalenderansicht (Sekundär)

> Klassischer Kalender mit allen Terminen aller Monteure.

| ID | Story | Aufwand |
|----|-------|---------|
| E4-1 | vue-cal Integration (Wochen-/Monatsansicht, Open Source, kostenlos) | 3h |
| E4-2 | Termine darstellen: Farbcodierung nach Monteur | 2h |
| E4-3 | Filter: Monteure ein-/ausblenden (Checkboxen) | 2h |
| E4-4 | Offene Termine (fuzzy) als Hinweis-Block am Monatsanfang darstellen | 2h |

**Summe Epic 4: ~9h**  

---

### 🔔 EPIC 5 — WhatsApp-Benachrichtigungen

| ID | Story | Aufwand |
|----|-------|---------|
| E5-1 | BE: Benachrichtigung senden wenn Monteur einem Termin zugewiesen wird (WhatsApp via bestehende Business API) | 3h |
| E5-2 | BE: Benachrichtigung senden wenn neue Info/Notiz bei einer Baustelle hinzugefügt wird (nur für zugewiesene Monteure) | 2h |
| E5-3 | WhatsApp Template erstellen + genehmigen lassen (Meta): "Du wurdest für [Baustelle] am [Datum] eingeplant." | 1h |
| E5-4 | WhatsApp Template erstellen + genehmigen lassen: "Neue Info bei Baustelle [X]: [Notiz]" | 1h |
| E5-5 | DB: Monteur-Profil um WhatsApp-Nummer erweitern | 1h |

**Summe Epic 5: ~8h**

---

## Phasen-Planung

### Phase 1 — MVP
Epic 1 + Epic 2 + Epic 3
→ **~34h** | Ergebnis: Termin-Erfassung funktioniert + Tatjana sieht auf einen Blick wer wann verfügbar ist

### Phase 2 — Vollständig
Epic 4 + Epic 5
→ **+17h** | Ergebnis: Klassischer Kalender + WhatsApp-Benachrichtigungen für Monteure

---

## Bewusst nicht im Scope

- Kalender-Sync mit Google Calendar / Outlook (Zukunft)
- Monteur-Login / Monteur-Kalenderansicht (Zukunft)
- Konflikt-Warnung bei Doppelbelegung (Tatjana entscheidet selbst)

---

*Erstellt von Wolfi, 2026-07-03*
