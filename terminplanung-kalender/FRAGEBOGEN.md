# Fragebogen & Konzeption: Terminplanung & Kalender-Feature
**Projekt:** Blitzschutz Reichenhauser  
**Erstellt:** 2026-06-25  
**Zuletzt aktualisiert:** 2026-06-26 (Feedback Round 2 eingearbeitet)  
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

- `Appointment` hat: `constructionSiteId`, `startTime`, `endTime` (berechnet), `notes`, `fuzzyLabel` (optional, für offene Termine)
- `appointment_employee` (JOIN): `appointmentId`, `employeeId`
- Pro Termin können **ein oder mehrere Monteure** zugewiesen werden

### Termin-Eingabe (Frontend)

**Button „Neuer Termin"** (direkt auf der Baustellendetailseite):
1. Mitarbeiter auswählen (ein oder mehrere, aus Liste der 8 Monteure)
2. **Termintyp wählen:**
   - **Fixer Termin:** Datum + Uhrzeit → Endzeit wird automatisch berechnet
   - **Offener Termin:** Dropdown mit vordefinierten Werten (z.B. "Juli 2026", "August 2026", "Q3 2026", "TBD") → kein Datum, nur Label
3. Endzeitberechnung bei fixen Terminen (automatisch):
   - **Startzeit vor 18:00 Uhr** → Dauer bis 18:00 Uhr
   - **Startzeit ab 18:00 Uhr** → Dauer fix 1 Stunde

**Begründung für vordefinierte Werte bei offenen Terminen:** Freitext ("August" vs "im August" vs "Aug.") ist nicht parsbar. Mit einem Dropdown-Set bleibt die Datenbasis sauber und offene Termine können im Kalender sinnvoll dargestellt werden (z.B. als schwebender Block am Monatsanfang).

### Kalenderansicht (Admin / Back-Office)

**Bibliothek:** FullCalendar.io — bewährt, TypeScript-kompatibel, gute Ressourcenansicht, freie Tier reicht für diesen Use Case

**Features:**
- Alle Termine aller Mitarbeiter in einer Gesamtansicht (Wochen-/Tagesansicht)
- **Farbcodierung** der Einträge nach Mitarbeiter (jeder Monteur = eigene Farbe)
- **Filterleiste** (Sidebar oder Checkboxen oben): Mitarbeiter ein-/ausblenden
- Termin anklicken → springt zur zugehörigen Baustelle
- Offene Termine (fuzzy): erscheinen als Hinweis-Block am jeweiligen Monatsanfang (z.B. grau/gestrichelt)
- Responsive: primär Desktop, Tablet soll funktionieren

### Verfügbarkeits-View (Prio 1 Use Case)

> **Kernanforderung (Prio 1):** „Wer hat in X Tagen Zeit für einen neuen Termin?"

Der klassische Kalender beantwortet diese Frage nur indirekt. Wir empfehlen daher zusätzlich eine dedizierte **Verfügbarkeitsansicht**:

- Zeigt alle 8 Monteure nebeneinander (Spalten) für einen wählbaren Zeitraum (z.B. nächste 7 Tage)
- Belegte Zeitblöcke sind eingefärbt → freie Slots sofort erkennbar
- Ähnlich einer Ressourcen-/Raumplanung (FullCalendar unterstützt das als „Resource Timeline View")
- Diese View ist der eigentliche Mehrwert für das Back-Office: Tatjana sieht auf einen Blick, wer wann verfügbar ist

**Sekundäre View:** "Nächste Termine fürs Team" — einfache Liste der nächsten N Termine sortiert nach Datum mit Monteur-Zuweisung.

### Kalender-Sync (Google Cal / Outlook)

- Reichenhauser nutzt beides (Outlook + Google Calendar)
- **Entscheidung:** Sync als optionales Zusatz-Feature, **nicht im aktuellen Scope**
- Prio liegt auf der eigenen Dashboard-Kalenderansicht
- Kann in einem späteren Schritt als Add-on angeboten werden (z.B. via iCal-Export oder Google/Microsoft OAuth)

---

## Beantwortete Fragen ✅

| # | Frage | Antwort |
|---|-------|---------|
| 1 | Wie viele Monteure? | **8 Mitarbeiteraccounts** |
| 2 | Mehrere Monteure pro Termin? | **Ja, mehrere möglich** |
| 4 | Offene Termine im Kalender? | **Vordefinierte Werte (Dropdown), Darstellung als Fuzzy-Block** |
| 5 | Externer Kalender? | **Outlook + Google Cal vorhanden, Sync optional/Zukunft** |
| 7 | Sortierung Baustellenliste? | **Prio-Use-Case: Verfügbarkeits-Check, nicht bloße Sortierung** |

---

## Noch offene Fragen (blockierend für Schätzung)

### Bestandssystem
1. **Wie ist die aktuelle DB-Struktur für Monteur-Zuweisung?** Läuft das aktuell als einfaches `employeeId`-Feld auf der Baustelle, oder schon als separate Tabelle? → Bestimmt den Migrationsaufwand.
2. **Soll die bestehende Monteur-Zuweisung pro Baustelle** (ohne Termin) **erhalten bleiben**, oder wird sie durch die neue Termin-basierte Zuweisung komplett ersetzt? → Architekturentscheidung mit Auswirkung auf Backend + UI.

### Verfügbarkeits-View
3. **Soll die Verfügbarkeits-View Teil des MVPs sein**, oder reicht im ersten Schritt der klassische Kalender (und die Verfügbarkeitsansicht kommt danach)? → Größter Aufwandstreiber.

### Termineingabe
4. **Gibt es eine maximale Termindauer?** Oder können Monteure theoretisch auch Mehrtages-Einsätze haben? → Relevant für die Endzeit-Logik.

---

## Konzept-Challenge: Was könnte fehlen?

Ein paar Punkte die wir noch durchdenken sollten bevor wir schätzen:

- **Terminkonflikte:** Soll das System warnen, wenn ein Monteur zu einer Zeit bereits verplant ist? Oder ist das bewusst kein Constraint (weil z.B. Halbtagseinsätze möglich sind)?
- **Editieren/Löschen von Terminen:** Wer darf das? Nur BO, oder auch Monteure selbst?
- **Benachrichtigungen:** Soll ein Monteur eine Benachrichtigung bekommen, wenn er einem Termin zugewiesen wird? (E-Mail, Push, WhatsApp?)
- **Mobile App für Monteure:** Gibt es aktuell eine Monteur-App, oder sehen Monteure ihre Termine gar nicht selbst?

---

## Nächste Schritte

1. Offene Fragen (oben) klären
2. Aufwandsschätzung: BASIC (Kalender-View + Termin-Erfassung) vs. PROFESSIONAL (+ Verfügbarkeits-View + Benachrichtigungen)
3. Empfehlung finale Kalender-Bibliothek (FullCalendar Resource View benötigt kostenpflichtige Lizenz ~400 USD/Jahr — muss in Angebot rein)

---

*Erstellt von Wolfi, 2026-06-25 — aktualisiert 2026-06-26*
