# Roadmap: Blitzschutz Reichenhauser — Dashboard Ausbau
**Stand:** 2026-07-03 — v2 (nach Architektur-Analyse)

**Änderungen gegenüber v1:**
- RBAC: nur FE hardcoded → einfachere Migration
- Dashboard: Routing/Auth vorhanden, weniger Aufwand
- Toggl: Flow+Script bleiben, Tage einzeln senden
- Terminplanung: Basis existiert bereits, nur Erweiterung
- Angebots-Tool: Schätzung offen bis Analyse-Session mit Carina

---

## Empfohlene Reihenfolge

---

## 🏗️ Phase 0 — Fundament

### 1. RBAC — Rollen- & Rechteverwaltung (23h / 1.587€)

**Was wird gemacht:**
- Neue DB-Tabellen: Gruppen + User-Zuweisung
- Hardcoded Gruppen-Checks im FE auf DB-basierte Lösung umstellen
- Neue Gruppe "Zeichner" eingeführt
- Admin-UI: User per Klick Gruppen zuweisen

**Was das für den Kunden bedeutet:**
- Neuer Mitarbeiter → Admin weist Rolle zu, fertig — kein Entwickler nötig
- E-Mail ändert sich → Admin ändert selbst
- Zeichner können sich selbst Baustellen zuweisen

---

### 2. Dashboard-Umbau (25h / 1.725€)

**Was wird gemacht:**
- Nuxt UI + Firmenfarben (nach Stitch-Design)
- Neue Sidebar-Navigation mit allen Bereichen
- Bestehende Ansichten integriert
- Rollenbasierte Sichtbarkeit der Menüpunkte

**Was das für den Kunden bedeutet:**
- Klare Struktur statt alles auf einem Haufen
- Jede Rolle sieht genau was sie braucht
- Fundament für alle weiteren Features

---

## ⚙️ Phase 1 — Kernfeatures

### 3. Terminplanung & Kalender (27h / 1.863€)

**Was wird gemacht:**
- Bestehende Terminerfassung erweitern: Monteure zu Terminen zuweisen
- Dashboard-Kalender: Gesamtansicht (alle Monteure, filterbar) + Einzelansicht (wer hat wann Zeit)
- Farbcodierung pro Monteur, Österreichische Feiertage
- WhatsApp-Benachrichtigung bei Einplanung + neuer Notiz

**Was das für den Kunden bedeutet:**
- "Wer hat Donnerstag noch Zeit?" → ein Blick in den Kalender
- Monteure wissen sofort per WhatsApp wenn sie eingeplant werden
- Keine doppelte Dateneingabe

---

### 4. Toggl Manueller Export (21h / 1.449€)

**Was wird gemacht:**
- Neuer Admin-Button: Arbeiter + Zeitraum wählen → Export starten
- System holt Toggl-Daten, schickt jeden Tag einzeln an bestehenden Power Automate Flow
- Flow + Excel-Script bleiben vollständig unverändert

**Was das für den Kunden bedeutet:**
- Fehlende Tage jederzeit nachholbar
- Kein Warten auf den nächtlichen automatischen Export
- Volle Kontrolle über Zeiterfassungsdaten

---

## 🚀 Phase 2 — Automatisierung

### 5. Angebots-Automatisierung (Schätzung offen*)

**Was wird gemacht:**
- E-Mail kommt rein → KI erkennt Szenario
- Automatische Drafts in Carinas Outlook
- Carina prüft, klickt "Senden"

**Was das für den Kunden bedeutet:**
- ~60% der Anfragen laufen automatisch
- Carina gibt frei statt alles selbst zu tippen
- Schnellere Antwortzeiten, weniger Fehler

*Schätzung nach Analyse-Session mit Carina (Kundendaten, Angebotsstruktur, Textbausteine)

---

## 💰 Gesamtkosten (aktualisiert)

| Feature | Aufwand | Netto |
|---------|---------|-------|
| RBAC Rechteverwaltung | 23h | 1.587€ |
| Dashboard-Umbau | 25h | 1.725€ |
| Terminplanung & Kalender | 27h | 1.863€ |
| Toggl Manueller Export | 21h | 1.449€ |
| Angebots-Automatisierung | offen | offen |
| **Gesamt (ohne Angebots-Tool)** | **96h** | **6.624€** |

*Alle Preise exkl. 20% MwSt.*  
*Laufende Kosten: ~15–25€/Monat (WhatsApp)*

---

## Nächste Schritte

- [ ] Analyse-Session mit Carina: Kundendaten, Angebotsstruktur, Textbausteine klären
- [ ] Stitch-Design für Dashboard-Umbau erstellen
- [ ] Angebote v2 an Kunden schicken (RBAC, Dashboard, Terminplanung, Toggl)
- [ ] Nach Analyse: Angebots-Tool konkret schätzen + Angebot nachliefern

---

*Erstellt von Wolfi, 2026-07-03*
