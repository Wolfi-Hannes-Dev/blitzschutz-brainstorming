# Angebot: Angebots-Automatisierung
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX

---

## Projektziel

Automatisierung des Angebots-Prozesses: Kundenanfragen per E-Mail werden automatisch erkannt, klassifiziert und — wo möglich — als fertiger Draft direkt in Carinas Outlook-Postfach bereitgestellt. Carina prüft und sendet mit einem Klick.

**Kein Power Automate.** Technisch umgesetzt über Microsoft Graph API direkt — sauber, wartbar, unabhängig von Microsoft-Lizenzstufen.

---

## Leistungsumfang

### Phase 1 — MVP
**Inhalt:**
- Microsoft Graph API Integration (OAuth2, Azure App Registration)
- Automatischer E-Mail-Eingang-Trigger (Webhook auf Carinas Inbox)
- KI-Klassifizierung eingehender Anfragen (Szenario-Erkennung)
- Automatische Rückfrage-Drafts bei Anfragen ohne Unterlagen (Szenario 4)
- Angebots-Engine Grundgerüst (Textbausteine digitalisiert, DOCX + PDF)

**Ergebnis:** Häufigste Routine-Rückfragen laufen vollautomatisch als Draft in Outlook.

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | Azure App Registration + OAuth2 Setup | 2h | 69€ | 138€ |
| 1.2 | Graph Webhook + Auto-Renew | 3h | 69€ | 207€ |
| 1.3 | Webhook-Endpoint + Mail-Parser | 6h | 69€ | 414€ |
| 1.4 | LLM-Klassifizierung (Szenario 1–6) | 4h | 69€ | 276€ |
| 1.5 | Kundenerkennung (Datenbankabgleich) | 3h | 69€ | 207€ |
| 1.6 | Rückfrage-Drafts Szenario 4 (inkl. Follow-up) | 8h | 69€ | 552€ |
| 1.7 | Angebots-Engine: Textbausteine + DOCX/PDF | 12h | 69€ | 828€ |
| **Zwischensumme Phase 1** | | **38h** | | **2.622€** |

---

### Phase 2 — Wiederholungsprüfungen
**Inhalt:**
- Kundendatenbank aufbauen (Import aus Excel)
- Vollautomatische Angebotserstellung für bekannte Kunden (Szenario 1)
- Draft mit fertigem Angebot als PDF-Anhang in Outlook

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 2.1 | Kundendatenbank + Excel-Import | 4h | 69€ | 276€ |
| 2.2 | Letztes Angebot abrufen + neu generieren | 6h | 69€ | 414€ |
| 2.3 | Draft-Erstellung mit PDF-Anhang | 4h | 69€ | 276€ |
| **Zwischensumme Phase 2** | | **14h** | | **966€** |

---

### Phase 3 — Erstprüfungen + Routing-Cockpit
**Inhalt:**
- Google Maps Integration für unbekannte Objekte (Szenario 2)
- Einfaches Web-Cockpit für Carina: offene Mails sehen, Szenario bestätigen
- Feedback-Loop: Änderungen werden geloggt

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 3.1 | Google Maps Static API + Objekterkennung | 8h | 69€ | 552€ |
| 3.2 | Rückfrage-Draft mit Screenshot (Szenario 2) | 5h | 69€ | 345€ |
| 3.3 | Web-Cockpit (Routing + Freigabe) | 8h | 69€ | 552€ |
| 3.4 | Feedback-Loop + Logging | 3h | 69€ | 207€ |
| **Zwischensumme Phase 3** | | **24h** | | **1.656€** |

---

## Gesamtübersicht

| Phase | Aufwand | Betrag netto |
|-------|---------|--------------|
| Phase 1 — MVP | 38h | **2.622€** |
| Phase 2 — Wiederholungsprüfungen | 14h | **966€** |
| Phase 3 — Erstprüfungen + Cockpit | 24h | **1.656€** |
| **Gesamt alle Phasen** | **76h** | **5.244€** |

*Alle Preise exkl. 20% MwSt.*

---

## Voraussetzungen (kundenseitig)

- Microsoft 365 Account mit Berechtigung für Azure App Registration ✓ (vorhanden)
- Zugang zu Carinas E-Mail-Adresse für OAuth-Consent
- Excel-Kundendaten für Import bereitstellen
- 2–3 anonymisierte Beispiel-Angebote als PDF (für Template-Erstellung)

---

## Nicht inkludiert

- Szenario 3 (Einreichplan-Kalkulation) — Fachkalkulation bleibt manuell
- Szenario 6 (Vergleichsangebote) — nicht automatisierbar
- Laufende KI-Kosten (OpenAI/Azure) — verbrauchsabhängig, ca. 0,01–0,05€/Mail
- Laufende Wartung/Hosting nach Übergabe

---

## Empfehlung

**Start mit Phase 1** — liefert sofort Mehrwert bei überschaubarem Aufwand.  
Phase 2 + 3 können direkt im Anschluss oder nach Feedback folgen.

---

*Angebot gültig bis: 2026-08-03*  
*Erstellt von Wolfi, 2026-07-03*
