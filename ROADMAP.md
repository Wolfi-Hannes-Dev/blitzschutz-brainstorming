# Roadmap: Blitzschutz Reichenhauser — Dashboard Ausbau
**Stand:** 2026-07-03

---

## Empfohlene Reihenfolge

Die Reihenfolge ist nicht zufällig — RBAC und Dashboard-Umbau sind Fundament für alles andere.

---

## 🏗️ Phase 0 — Fundament (Voraussetzung für alles)

### 1. RBAC — Rollen- & Rechteverwaltung (37h / 2.553€)

**Was wird gemacht:**
- Hardcoded Gruppen (Admin, Worker) werden aus dem Code entfernt
- Neue Gruppe "Zeichner" wird eingeführt
- Wer welche Rolle hat, wird in der Datenbank gepflegt
- Admin kann im Dashboard selbst User zuweisen — kein Entwickler nötig
- Frontend + Backend werden komplett auf das neue System umgestellt

**Was das für den Kunden bedeutet:**
- Neuer Mitarbeiter kommt → Admin weist Rolle zu, fertig
- E-Mail-Adresse ändert sich → Admin ändert es selbst im Dashboard
- Zeichner können sich selbst Baustellen zuweisen
- Kein Anruf beim Entwickler mehr für Benutzerverwaltung

---

### 2. Dashboard-Umbau (51h / 3.519€)

**Was wird gemacht:**
- Neue Sidebar-Navigation mit allen Bereichen
- Bestehende Ansicht bleibt erhalten, wird als Seite integriert
- Alle neuen Features bekommen eigene Seiten/Subpages
- Jede Rolle sieht nur was sie sehen soll
- Nuxt UI als Komponenten-Basis, Firmenfarben übernommen
- Design nach Stitch-Vorgabe

**Was das für den Kunden bedeutet:**
- Klare, übersichtliche Struktur statt alles auf einem Haufen
- Admin sieht alles, Worker nur seine Baustellen, Zeichner seine Ansicht
- Vorbereitung für alle nachfolgenden Features — ohne diesen Umbau würde alles irgendwo "dranhängen"

---

## ⚙️ Phase 1 — Kernfeatures

### 3. Terminplanung & Kalender (38.5h / 2.657€)

**Was wird gemacht:**
- Termine pro Baustelle erfassen (fix, offen "Juli 2026", mehrtägig)
- Einen oder mehrere Monteure pro Termin zuweisen
- Dashboard-Kalender: alle Monteure mit Farbcodierung
- Gesamtansicht (wer ist wo) + Einzelansicht (wer hat wann Zeit)
- Österreichische Feiertage automatisch markiert
- WhatsApp-Benachrichtigung wenn Monteur eingeplant wird oder neue Notiz erscheint

**Was das für den Kunden bedeutet:**
- Kein manuelles Durchsuchen mehr: "Wer hat Donnerstag noch Zeit?" → ein Blick in den Kalender
- Monteure wissen sofort per WhatsApp wenn sie eingeplant werden
- Offene Termine ("irgendwann im August") sind auch sichtbar, nicht verloren
- Österreichische Feiertage sichtbar beim Einplanen

---

### 4. Toggl Manueller Export (27h / 1.863€)

**Was wird gemacht:**
- Neuer Button im Admin-Dashboard: Arbeiter auswählen + Zeitraum wählen
- System holt Toggl-Zeiteinträge für den gewählten Zeitraum
- Schreibt sie in die Datenbank
- Löst den bestehenden Power Automate Flow + Excel-Script aus
- Excel-Script wird angepasst: verarbeitet mehrere Einträge auf einmal

**Was das für den Kunden bedeutet:**
- Fehlende oder falsche Tage können jederzeit nachgeholt/korrigiert werden
- Kein Warten auf den nächsten automatischen Nacht-Export
- Admin hat volle Kontrolle über Zeiterfassungsdaten

---

## 🚀 Phase 2 — Automatisierung

### 5. Angebots-Automatisierung (76h / 5.244€)

**Was wird gemacht:**

**Schritt 1 — MVP (38h / 2.622€):**
- E-Mails kommen rein → KI erkennt automatisch um welches Szenario es sich handelt
- Bei Anfragen ohne Unterlagen: Rückfrage-Mail wird automatisch als Draft in Carinas Outlook erstellt
- Carina prüft, klickt "Senden" — fertig
- Angebots-Engine: Textbausteine aus Excel digitalisiert, DOCX + PDF Generierung

**Schritt 2 — Wiederholungsprüfungen (14h / 966€):**
- Bekannter Kunde schreibt → System findet letztes Angebot → neues Draft fertig in Outlook
- Carina klickt nur noch "Senden"

**Schritt 3 — Erstprüfungen + Cockpit (24h / 1.656€):**
- Google Maps Screenshot für unbekannte Objekte
- Web-Cockpit: Carina sieht alle offenen Mails, bestätigt Szenario mit einem Klick

**Was das für den Kunden bedeutet:**
- ~60% der Anfragen laufen automatisch
- Carinas Aufgabe: Drafts freigeben statt alles selbst tippen
- Weniger Fehler, schnellere Antwortzeiten
- Laufende KI-Kosten: ~2–5€/Monat

---

## 💰 Gesamtkosten

| Feature | Aufwand | Netto |
|---------|---------|-------|
| RBAC Rechteverwaltung | 37h | 2.553€ |
| Dashboard-Umbau | 51h | 3.519€ |
| Terminplanung & Kalender | 38.5h | 2.657€ |
| Toggl Manueller Export | 27h | 1.863€ |
| Angebots-Automatisierung | 76h | 5.244€ |
| **Gesamt** | **229.5h** | **15.836€** |

*Alle Preise exkl. 20% MwSt.*  
*Laufende Kosten: ~15–30€/Monat (WhatsApp + KI-API)*

---

## Was nicht im Scope ist (bewusst)

- Google/Outlook Kalender-Sync → optionales Add-on auf Anfrage
- Szenario 3 (Einreichplan-Kalkulation) → zu komplex, bleibt manuell
- Szenario 6 (Vergleichsangebot Fremdfirma) → nicht automatisierbar
- Mobile App für Monteure → separates Projekt

---

*Erstellt von Wolfi, 2026-07-03*
