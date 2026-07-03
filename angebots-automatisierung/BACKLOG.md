# Backlog: Angebots-Automatisierung
**Projekt:** Blitzschutz Reichenhauser  
**Stand:** 2026-07-03

---

## Architektur-Entscheidung: Ohne Power Automate ✅

**Microsoft Graph API** direkt — kein Power Automate nötig.

- **E-Mail-Trigger:** Graph API Webhook-Subscription auf Inbox (`/me/mailFolders('Inbox')/messages`)
  - Subscription läuft max. ~3 Tage → unser Backend erneuert sie automatisch
  - Kein Drittanbieter, kein Microsoft-Lock-in über Power Automate
- **Draft erstellen:** `POST /users/{userId}/messages` → Draft landet direkt in Carinas Outlook
- **Anhänge lesen:** `GET /messages/{id}/attachments`
- **Auth:** OAuth 2.0 (Client Credentials oder Delegated) über Azure App Registration

**Stack (passt zum bestehenden Projekt):**
- Node.js + TypeScript (Backend)
- Bestehende Blitzschutz-API als Basis

---

## Epics & Stories

---

### 🔵 EPIC 1 — E-Mail-Eingang & Klassifizierung
*Kern des Systems: Jede neue Anfrage wird erkannt und kategorisiert.*

| ID | Story | Aufwand |
|----|-------|---------|
| E1-1 | Azure App Registration einrichten (OAuth2, Graph-Permissions: Mail.Read, Mail.ReadWrite) | 2h |
| E1-2 | Graph Webhook Subscription auf Carinas Inbox (create + auto-renew) | 3h |
| E1-3 | Webhook-Endpoint im Backend: neue Mail empfangen + rohen Inhalt extrahieren | 3h |
| E1-4 | Mail-Parser: Body + Betreff + Anhänge (PDF/Bild) extrahieren | 3h |
| E1-5 | LLM-Klassifizierung: Szenario 1–6 erkennen + Confidence-Score | 4h |
| E1-6 | Kundenerkennung: Name/Adresse abgleichen mit Kundendatenbank | 3h |

**Summe Epic 1: ~18h**

---

### 🟢 EPIC 2 — Szenario 1: Wiederholungsprüfung
*Bekannter Kunde, letztes Angebot suchen + neues Draft erstellen.*

| ID | Story | Aufwand |
|----|-------|---------|
| E2-1 | Datenbank: Kunden + letzte Angebote/Rechnungen importieren (aus Excel) | 4h |
| E2-2 | Letzte Angebot/Rechnung für Kunden abrufen | 2h |
| E2-3 | Neues Angebot generieren (neues Datum, neue Angebotsnummer, gleiche Positionen) | 4h |
| E2-4 | Draft-Mail in Carinas Outlook erstellen (Graph API) mit PDF-Anhang | 3h |
| E2-5 | Carina per Benachrichtigung informieren (Mail/Teams-Nachricht) | 1h |

**Summe Epic 2: ~14h**

---

### 🟢 EPIC 3 — Szenario 4: Anfrage ohne Unterlagen
*Automatische Rückfrage-Mail als Draft generieren.*

| ID | Story | Aufwand |
|----|-------|---------|
| E3-1 | Textbaustein-DB: Rückfrage-Texte (Stufe 1: Unterlagen?, Stufe 2: Besichtigung) | 2h |
| E3-2 | Draft für Rückfrage-Mail erstellen (inkl. Anrede aus Kundennamen) | 3h |
| E3-3 | Follow-up: Wenn Antwort "Nein/keine Unterlagen" → zweiten Draft (Besichtigungsanfrage) | 3h |

**Summe Epic 3: ~8h**

---

### 🟡 EPIC 4 — Szenario 2: Erstprüfung unbekanntes Objekt
*Adresse erkennen + Google Maps Screenshot + Angebotsdraft bei klarem Objekt.*

| ID | Story | Aufwand |
|----|-------|---------|
| E4-1 | Adresse aus Mail-Text extrahieren (LLM oder Regex) | 2h |
| E4-2 | Google Maps Static API: Screenshot des Objekts generieren | 3h |
| E4-3 | Objekttyp einschätzen (Einfamilienhaus / Gewerbe / Wohnblock) via LLM + Bild | 3h |
| E4-4 | Bei klarem Objekt: Pauschalen-Angebot als Draft erstellen | 3h |
| E4-5 | Bei unklarem Objekt: Rückfrage-Mail mit Screenshot als Draft | 2h |

**Summe Epic 4: ~13h**

---

### 🟡 EPIC 5 — Szenario 5: Routing & Carina-Cockpit
*Unklare Mails aufbereiten + Carina entscheiden lassen.*

| ID | Story | Aufwand |
|----|-------|---------|
| E5-1 | Bei niedrigem Confidence-Score: Mail-Zusammenfassung + Kategorisierungsvorschlag | 3h |
| E5-2 | Einfaches Cockpit (Web-UI): Carina sieht alle offenen Mails + kann Szenario bestätigen | 6h |
| E5-3 | Feedback-Loop: Was Carina ändert wird geloggt (für spätere Verbesserung) | 2h |

**Summe Epic 5: ~11h**

---

### 🔵 EPIC 6 — Angebots-Engine (Querschnitt)
*Grundlage für alle Szenarien die ein Angebot generieren.*

| ID | Story | Aufwand |
|----|-------|---------|
| E6-1 | Textbausteine aus Excel digitalisieren (DB-Struktur) | 4h |
| E6-2 | DOCX-Template-Engine: Angebot aus Bausteinen zusammensetzen | 5h |
| E6-3 | PDF-Generierung aus DOCX | 2h |
| E6-4 | Angebotsnummern-Vergabe (fortlaufend, Jahresindex) | 1h |

**Summe Epic 6: ~12h**

---

## Phasen-Planung

### Phase 1 — MVP (Empfehlung zum Start)
Epic 1 + Epic 3 + Epic 6 (Basis)
→ **~38h** | Ergebnis: E-Mails kommen rein, werden klassifiziert, Rückfrage-Drafts landen automatisch in Outlook

### Phase 2
Epic 2 (Wiederholungsprüfung)
→ **+14h** | Ergebnis: Häufigster Fall vollautomatisch

### Phase 3
Epic 4 + Epic 5
→ **+24h** | Ergebnis: Auch Erstprüfungen + Routing abgedeckt

---

## Nicht im Scope (bewusst ausgelassen)
- Szenario 3 (Einreichplan-Kalkulation) — zu komplex, manuell bleibt besser
- Szenario 6 (Vergleichsangebot) — technisch nicht sinnvoll automatisierbar
- Power Automate — bewusst ausgeschlossen
- Mobile App — nicht nötig, Cockpit läuft im Browser

---

*Erstellt von Wolfi, 2026-07-03*
