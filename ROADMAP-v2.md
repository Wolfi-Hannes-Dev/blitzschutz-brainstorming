# Roadmap: Blitzschutz Reichenhauser — Dashboard Ausbau
**Stand:** 2026-07-21 — v2, Angebots-Tool aktualisiert (Varianten + Prototyp)

**Änderungen 2026-07-21:**
- Angebots-Tool: aus „Schätzung offen" werden 3 durchgerechnete Varianten (MVP / Kern / Vollausbau)
- Altes Angebot (76h / 5.244€) als überholt markiert — neue Spanne ist Faktor 4–6 höher
- Gesamtkosten-Tabelle um die drei Varianten erweitert

**Änderungen v1 → v2 (2026-07-03):**
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

### 5. Angebots-Automatisierung (3 Varianten — Entscheidung offen)

**Detailschätzung liegt vor:** → [`angebots-automatisierung/VARIANTEN-UEBERSICHT.md`](./angebots-automatisierung/VARIANTEN-UEBERSICHT.md)

| Variante | Planwert | Spanne |
|----------|----------|--------|
| **MVP** „Der stille Kollege" | ~142h ≈ **9.800€** | 108–175h · 7.450–12.100€ |
| **Kern** „Das Cockpit" | ~272h ≈ **18.700€** | 209–334h · 14.400–23.000€ |
| **Vollausbau** „Der Assistent" | ~416h ≈ **28.700€** | 315–516h · 21.700–35.600€ |

**Was wird gemacht:**
- E-Mail kommt rein → KI klassifiziert Szenario + erkennt Kunden
- Angebots-Entwurf landet fertig in Carinas Outlook — nie Auto-Send
- Preise kommen deterministisch aus der DB, nie von der KI

**Was das für den Kunden bedeutet:**
- Carina gibt frei statt alles selbst zu tippen
- MVP misst mit echten Daten, wie viele der ~50 Mails/Monat wirklich automatisierbar sind
- Schnellere Antwortzeiten, weniger Fehler

**Empfehlung:** Einstieg über MVP oder Kern, nicht Vollausbau. Stufenweg MVP → Kern (Delta ~6–9k€) → Vollausbau (Delta ~6–9k€) kostet in Summe etwa dasselbe, verteilt aber Investition und Risiko.

**Größtes Risiko:** Import der Alt-Angebote. Nur als Scan vorliegend (kein Textlayer) = +10–20h. Vor Beauftragung mit 2–3 echten Alt-PDFs verifizieren.

⚠️ Das frühere Angebot (`ANGEBOT-AUTOMATISIERUNG.md`, 76h / 5.244€) ist **überholt** und dort entsprechend markiert.

---

## 💰 Gesamtkosten (aktualisiert)

| Feature | Aufwand | Netto |
|---------|---------|-------|
| RBAC Rechteverwaltung | 23h | 1.587€ |
| Dashboard-Umbau | 25h | 1.725€ |
| Terminplanung & Kalender | 27h | 1.863€ |
| Toggl Manueller Export | 21h | 1.449€ |
| **Zwischensumme Dashboard-Features** | **96h** | **6.624€** |
| Angebots-Automatisierung — MVP | ~142h | ~9.800€ |
| Angebots-Automatisierung — Kern | ~272h | ~18.700€ |
| Angebots-Automatisierung — Vollausbau | ~416h | ~28.700€ |
| **Gesamt mit MVP** | **~238h** | **~16.400€** |
| **Gesamt mit Kern** | **~368h** | **~25.300€** |
| **Gesamt mit Vollausbau** | **~512h** | **~35.300€** |

*Alle Preise exkl. 20% MwSt. · Stundensatz 69€*  
*Laufende Kosten: ~15–25€/Monat (WhatsApp) · ~1–3€/Monat (KI-Betrieb)*

**Größenverhältnis beachten:** Das Angebots-Tool ist je nach Variante 1,5–4× so groß wie die gesamten übrigen Dashboard-Features zusammen.

Rechenbare Fassung mit Positionsdetails: [`aufwandsschaetzung-4-topics.xlsx`](./aufwandsschaetzung-4-topics.xlsx)

---

## Nächste Schritte

- [ ] **Variante für das Angebots-Tool entscheiden** (MVP / Kern / Vollausbau)
- [ ] Alt-Angebote prüfen: PDF mit Textlayer oder Scan? → 2–3 Beispiele vor Beauftragung ansehen
- [ ] `anfragen@`-Postfach anlegen (wer, bis wann?)
- [ ] Stitch-Design für Dashboard-Umbau erstellen
- [ ] Angebote v2 an Kunden schicken (RBAC, Dashboard, Terminplanung, Toggl)
- [ ] Angebot für die gewählte Angebots-Tool-Variante ausformulieren
- [ ] Backlogs auf v2-Stand nachziehen (RBAC, Terminplanung, Toggl — siehe Hinweis unten)

---

## ⚠️ Offene Inkonsistenz (intern)

Die `BACKLOG.md`-Dateien stehen noch auf v1-Schätzungen und widersprechen den v2-Angeboten:

| Thema | Backlog Phase 1 | Angebot v2 Phase 1 |
|-------|-----------------|--------------------|
| RBAC | ~31h | 17h |
| Terminplanung | ~31,5h | 21h |
| Toggl | ~22h | 16h |

Die Reduktionen sind durch die Architektur-Analyse begründet — die Backlogs sollten trotzdem nachgezogen werden, bevor sie jemand als Umsetzungsgrundlage nimmt.

---

*Erstellt von Wolfi, 2026-07-03*
