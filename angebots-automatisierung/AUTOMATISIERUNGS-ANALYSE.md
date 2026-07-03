# Automatisierungs-Analyse: Angebotserstellung
**Erstellt:** 2026-07-03  
**Basis:** 6 Szenarien + Angebotserstellungsvorgang + Excel-Vorlage

---

## Übersicht: Was ist automatisierbar?

| Szenario | Automatisierungsgrad | Human in the Loop |
|----------|---------------------|-------------------|
| 1 – Wiederholungsprüfung bekannter Kunde | 🟢 Hoch | Nur Freigabe |
| 2 – Erstprüfung, unbekanntes Objekt | 🟡 Mittel | Google Maps Auswertung |
| 3 – Errichtung mit Einreichplan | 🔴 Niedrig | Kalkulation bleibt manuell |
| 4 – Anfrage ohne Unterlagen | 🟢 Hoch | Nur Freigabe der Rückfrage |
| 5 – Unklare Anfrage | 🟡 Mittel | Einschätzung durch Carina |
| 6 – Vergleichsangebot Fremdfirma | 🔴 Niedrig | Muss manuell ausgefüllt werden |

---

## Szenario-Analyse im Detail

### 🟢 Szenario 1 — Wiederholungsprüfung bekannter Kunde
**Was passiert heute:** Letztes Angebot/Rechnung suchen → Index drauf → zurückschicken.

**Automatisierbar:** Fast vollständig.
- E-Mail kommt rein → KI erkennt: bekannter Kunde + Prüfungsanfrage
- System sucht letztes Angebot in der DB
- Erstellt neues Angebot mit aktuellem Datum + Index-Nummer
- **Draft in Carinas Outlook** → sie klickt nur noch „Senden"

**Was bleibt manuell:** Freigabe (1 Klick)

---

### 🟡 Szenario 2 — Erstprüfung, unbekanntes Objekt
**Was passiert heute:** Adresse in Google Maps eingeben, Objekt einschätzen, dann Angebot.  
Falls unklar → Screenshot zurück an Kunden mit Bitte um Markierung.

**Automatisierbar:** Teilweise.
- E-Mail + Adresse erkennen → Google Maps Static API aufrufen → Screenshot ins Draft
- KI kann Objekttyp grob einschätzen (Einfamilienhaus, Gewerbe, Wohnblock)
- Bei eindeutigem Objekt: Angebotsdraft erstellen (Pauschale nach Objekttyp)
- Bei mehrdeutigem Objekt: automatische Rückfrage-Mail mit Screenshot generieren

**Was bleibt manuell:** Preisfindung bei Sonderlagen, finale Freigabe

---

### 🔴 Szenario 3 — Errichtung mit Einreichplan
**Was passiert heute:** PDF-Plan öffnen, Umfang berechnen, Referenzpreise aus Vorjahr heranziehen.

**Automatisierbar:** Nur Randschritte.
- E-Mail + PDF-Anhang erkennen → Carina informieren mit allen Infos aufbereitet
- Vielleicht: KI liest Sonderwünsche/Notizen aus der Mail raus
- Die eigentliche Kalkulation (Umfang messen, Referenzpreise) bleibt manuell

**Was bleibt manuell:** Kernkalkulation (komplexes Fachurteil)

**Zukunft:** Wenn Referenzpreise + Objektgrößen historisch geloggt werden → Basis für ML-Schätzung

---

### 🟢 Szenario 4 — Anfrage ohne Unterlagen
**Was passiert heute:** Rückfrage-Mail schreiben (immer ähnlicher Text).

**Automatisierbar:** Sehr hoch.
- KI erkennt: Anfrage ohne Anhang + keine Infos
- Rückfrage-Mail automatisch generieren (höflicher Standardtext nach Vorlage)
- Draft in Outlook → Freigabe mit 1 Klick
- Wenn Antwort „Nein" (keine Unterlagen) → automatisch zweite Mail mit Besichtigungsanfrage

**Was bleibt manuell:** Freigabe der Mails

---

### 🟡 Szenario 5 — Unklare Anfrage / Sonstiges
**Was passiert heute:** Carina schaut sich das an und entscheidet.

**Automatisierbar:** Routing.
- KI klassifiziert die Mail (passt zu Szenario 1–4? → weiterleiten)
- Falls unklar: Mail mit KI-Zusammenfassung + Kategorisierungsvorschlag an Carina
- Carina entscheidet mit 1 Klick welche Route

**Was bleibt manuell:** Die eigentliche Entscheidung

---

### 🔴 Szenario 6 — Vergleichsangebot Fremdfirma
**Was passiert heute:** Manuell Preise eintragen in fremde Vorlage.

**Automatisierbar:** Kaum.
- Erkennen dass es sich um ein Vergleichsangebot handelt → Carina benachrichtigen
- Eventuell: häufige Positionen aus eigener Preisliste vorschlagen

**Was bleibt manuell:** Ausfüllen (jede Firma hat andere Bezeichnungen)

---

## Was müsste ins Projekt dazu?

### Neu notwendige Komponenten

**1. E-Mail-Eingang (Trigger)**
- Power Automate Flow: neues Mail in info@-Postfach → Webhook an unser System
- Alternative: direkte IMAP/Graph-API Anbindung

**2. KI-Klassifizierung**
- LLM liest Betreff + Body + Anhänge und ordnet zu: Szenario 1–6
- Confidence-Score → unter Schwellwert = direkt zu Carina
- Kundenerkennung: Name/Adresse abgleichen mit bestehender Kundendatenbank

**3. Kundendatenbank**
- Heute: SVerweis in Excel (Kürzel → Kundendaten)
- Brauchen wir: echte DB mit Kunden + letzten Angeboten/Rechnungen
- Basis für: Wiederholungsangebote, Referenzpreise, Historien

**4. Angebots-Generator**
- Bestehende Textbausteine aus Excel-Vorlage → digitalisiert in DB
- Template-Engine (DOCX oder HTML→PDF)
- Logik: welche Textbausteine bei welchem Szenario?

**5. Draft-Erstellung in Outlook**
- Microsoft Graph API: Draft-Mail erstellen mit fertigem Angebot als Anhang
- Carina sieht den Draft, prüft, sendet → kein Kopieren/Einfügen mehr

**6. Feedback-Loop**
- Wenn Carina einen Draft abändert → Änderungen loggen (lernen was sie ändert)
- Basis für spätere Verbesserung der Automatisierung

---

## Empfohlene Ausbaustufen

### Phase 1 — Quick Wins (wenige Wochen)
- Szenarien 1 + 4 vollständig automatisieren (Draft-Erstellung)
- E-Mail-Klassifizierung für Routing
- Kundendaten aus Excel migrieren

### Phase 2 — Mittelfristig
- Szenario 2 mit Google Maps Integration
- Szenario 5 Routing + Zusammenfassung
- Angebots-Historien in DB

### Phase 3 — Langfristig
- Szenario 3 mit Plan-Analyse
- Preisschätzung auf Basis historischer Daten
- Vollautomatisches Angebot mit Freigabe-Workflow

---

## Zusammenfassung

**~60% der Anfragen** (Szenarien 1 + 4, teils 2 + 5) sind gut automatisierbar.  
**Carinas Rolle** verschiebt sich von "alles selbst machen" zu "Drafts freigeben + Ausnahmen bearbeiten".  
**Szenarien 3 + 6** bleiben manuell — da steckt komplexes Fachwissen drin das wir kurz- bis mittelfristig nicht ersetzen.

Der größte Hebel: **E-Mail-Klassifizierung + Kundenerkennung + Draft in Outlook** — das allein spart schon täglich Zeit.

---

*Analyse: Wolfi, 2026-07-03*
