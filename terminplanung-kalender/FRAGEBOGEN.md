# Fragebogen & Konzeption: Terminplanung & Kalender-Feature
**Projekt:** Blitzschutz Reichenhauser  
**Erstellt:** 2026-06-25  
**Zuletzt aktualisiert:** 2026-06-26 (Feedback von Hannes eingearbeitet)  
**Zweck:** Konzept, offene Fragen & Aufwandsschätzung

---

## Ausgangssituation (Kundenwunsch)

Der Kunde hat folgende Anforderungen beschrieben:

1. Möglichkeit, Baustellen nach Terminen zu sortieren (jüngster bis spätester bzw. noch nicht festgesetzter Termin)
2. Kalenderansicht mit allen bereits zugeteilten Terminen + zugehörigem Monteur
3. Allgemein: besserer Terminüberblick

**Aktueller Ablauf (Back-Office):**
Tatjana bekommt einen Auftrag, vereinbart einen Durchführungstermin mit dem Kunden und muss dafür manuell prüfen, welche Baustellen welche Termine haben und welcher Monteur verfügbar ist. Ein Gesamtüberblick über die Auslastung fehlt.

---

## Bereits bekannte Rahmenbedingungen

- 100+ aktive Baustellen gleichzeitig
- Eine Baustelle kann mehrere Termine haben (inkl. offene Termine ohne fixes Datum)
- Termine werden nur vom Back-Office (BO) eingetragen
- Termineingabe existiert bereits (Datum + Uhrzeit, oder Datum + Text, oder nur Text z.B. „Termin August")
- Monteur soll beim Termin sichtbar sein
- **8 Mitarbeiteraccounts** sind aktuell im System
- Derzeit: Monteur wird einer Baustelle zugewiesen; Termin hat (noch) keinen Monteur

---

## Konzept (Stand 2026-06-26)

### Datenmodell: Termin-Zuweisung

**Entscheidung:** Termine werden auf **Ebene der Baustelle** (`ConstructionSite`) an Mitarbeiter geknüpft.

**Begründung:** Ein Termin gehört zu einer konkreten Baustelle, nicht zu einem abstrakten Projekt. Das ist die natürlichere Ebene für die operative Planung.

**Datenbankstruktur:**
```
ConstructionSite  1---n  Appointment  n---m  Employee
                              |
                         (JOIN-Tabelle:
                      appointment_employee)
```

- `Appointment` hat: `constructionSiteId`, `startTime`, `endTime` (berechnet), `notes`
- `appointment_employee` (JOIN): `appointmentId`, `employeeId`
- Pro Termin können **ein oder mehrere Monteure** zugewiesen werden

### Termin-Eingabe (Frontend)

**Button „Neuer Termin"** (direkt auf der Baustellendetailseite):
1. Mitarbeiter auswählen (ein oder mehrere, aus Liste der 8 Monteure)
2. Startzeit setzen (Datum + Uhrzeit)
3. Endzeitberechnung (automatisch, keine Eingabe nötig):
   - **Startzeit vor 18:00 Uhr** → Dauer bis 18:00 Uhr
   - **Startzeit ab 18:00 Uhr** → Dauer fix 1 Stunde

### Kalenderansicht (Admin / Back-Office)

**Bibliothek:** FullCalendar.io (oder vergleichbar, z.B. DHTMLX Scheduler) — bewährt, TypeScript-kompatibel, gute Ressourcenansicht

**Features:**
- Alle Termine aller Mitarbeiter in einer Gesamtansicht (Wochen-/Tagesansicht)
- **Farbcodierung** der Einträge nach Mitarbeiter (jeder Monteur = eigene Farbe)
- **Filterleiste** (Sidebar oder Checkboxen oben): Mitarbeiter ein-/ausblenden
- Termin anklicken → springt zur zugehörigen Baustelle
- Responsive: primär Desktop, Tablet sollte funktionieren

---

## Noch offene Fragen

### Monteure
1. ~~Wie viele Monteure gibt es aktuell?~~ ✅ **8 Mitarbeiteraccounts**
2. ~~Soll man pro Termin einen oder mehrere Monteure zuweisen können?~~ ✅ **Mehrere möglich**
3. Soll die bestehende Monteur-Zuweisung **pro Baustelle** (ohne Termin) noch erhalten bleiben, oder nur noch über Termine laufen?

### Offene Termine
4. Sollen Termine ohne fixes Datum (z.B. „August" oder „TBD") im Kalender irgendwie angezeigt werden (z.B. am Monatsrand) — oder nur in der Listenansicht der Baustellen?

### Externe Kalender
5. Nutzt ihr aktuell **Google Kalender, Outlook** oder einen anderen Kalender im Büro?
6. Falls ja – wäre eine **Synchronisation** mit dem bestehenden Kalender interessant, oder soll alles im eigenen System bleiben?

### Sortierung in der Baustellenliste
7. Soll die Sortierung nach Termin als **feste Standardsortierung** gelten, oder als **zusätzliche Option** neben der bestehenden Suche/Filterung?

---

## Nächste Schritte

Nach Beantwortung der restlichen offenen Fragen:
- Aufwandsschätzung in Stunden + Kosten (BASIC / PROFESSIONAL)
- Empfehlung für konkrete Kalender-Bibliothek
- Technisches Konzept Backend (API-Endpunkte)

---

*Erstellt von Wolfi, 2026-06-25 — aktualisiert 2026-06-26*
