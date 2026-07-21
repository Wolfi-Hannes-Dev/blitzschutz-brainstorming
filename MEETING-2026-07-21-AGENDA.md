# Meeting-Agenda — 2026-07-21
**Projekt:** Blitzschutz Reichenhauser — Dashboard Ausbau
**Basis:** Roadmap v2 + Angebote v2 (Stand 2026-07-03)

---

## Agenda (kurz)

| # | Topic | Zeit | Ziel |
|---|-------|------|------|
| 0 | Status Angebote v2 — raus? Rückmeldung? | 5 min | Go/No-Go Phase 0 |
| 1 | Zeichnerrolle & RBAC-Konsequenzen | 20 min | Rechte-Matrix fixieren |
| 2 | Toggl: Nachtracken vergangener Tage | 15 min | Umsetzungsweg bestätigen |
| 3 | Termine für Arbeiter erstellen & verwalten | 20 min | Scope Phase 1 vs. 2 |
| 4 | Flow Auftragserstellung: E-Mail → Angebot — **3 Varianten + Prototyp** | 35 min | **Variante entscheiden** |
| 5 | Reihenfolge & nächste Schritte | 10 min | Startdatum + Verantwortliche |

**Gesamt: ~105 min**

> Topic 4 ist der Schwerpunkt: Prototyp zeigen, drei Ausbaustufen gegenüberstellen, Entscheidung holen. Topics 1–3 sind fertig geschätzt und brauchen nur Bestätigung — bei Zeitdruck dort kürzen.

---

## Topic 1 — Zeichnerrolle integrieren (RBAC)

**Stand:** Angebot v2 liegt vor — 23h / 1.587€ (Phase 1: 17h Migration+Core, Phase 2: 6h Admin-UI)

**Kern:** Gruppenlogik ist heute **nur im Frontend hardcoded**. Umstellung auf DB-basierte Zuweisung:
`User → user_groups (JOIN) → Group → Permissions (bleiben im Code)`

**Gruppen:** `admin` (Vollzugriff) · `worker` (eigene Baustellen, Zeiten tracken) · `designer` (Zeichner — Baustellen sehen + **sich selbst zuweisen**)

**Konsequenzen der Rollenvergabe:**
- Neuer Mitarbeiter / E-Mail-Änderung / Rollenwechsel → Admin macht das im Dashboard, kein Deployment
- Zeichner-Selbstzuweisung ist eine eigene Permission-Regel: nur Zeichner + Admin dürfen Zeichner einer Baustelle zuweisen
- Route Guards + bedingte UI-Elemente hängen ab jetzt an der Gruppe
- **RBAC ist Fundament** — Dashboard-Umbau (rollenbasierte Menüs) und Terminplanung (nur BO darf Termine anlegen) bauen darauf auf

**Zu klären:**
- Kann ein User in **mehreren** Gruppen sein (z. B. Zeichner + Monteur)?
- Bekommt der Zeichner einen eigenen View oder einen eingeschränkten Admin-View?
- Was genau darf der Zeichner an der Baustelle — nur lesen + zuweisen, oder auch editieren?
- Wer ist Admin? Nur BO oder mehrere?
- Braucht es weitere Rollen (Büro/Carina eigene Gruppe)?

---

## Topic 2 — Toggl: vergangene Tage nachtracken

**Stand:** Angebot v2 liegt vor — 21h / 1.449€ (Phase 1 MVP: 16h, Phase 2 Robustheit: 5h)

**Kern:** Neuer Admin-Button — Arbeiter + Zeitraum wählen → System holt Toggl-Einträge, schreibt in DB, schickt **jeden Tag einzeln** an den bestehenden Power Automate Flow.

**Wichtigste Design-Entscheidung (v2):** Tage einzeln senden → **Flow + Excel-Script bleiben unverändert**. Die 3h Script-Umbau aus v1 entfallen.

**Bestehende Architektur:** nächtlicher Auto-Export Toggl → DB → Power Automate → Excel-Script; DB hält Mapping E-Mail → Toggl-API-Token; ~10 Arbeiter mit eigenen Free-Tier-Accounts.

**Risiken:**
- Toggl Free Tier: **1 Request/Sekunde** — große Zeiträume dauern entsprechend
- Token veraltet, wenn Arbeiter Account/Token erneuert → kein Auto-Sync möglich, Admin muss manuell nachziehen
- Doppelte Einträge bei Wiederholung → Upsert-Logik ist Phase 2

**Zu klären:**
- Maximaler Zeitraum pro Export? (1 Monat? Quartal?)
- Was passiert bei erneutem Export desselben Zeitraums — überschreiben oder skippen? → **Upsert (Phase 2) mit reinnehmen oder nicht?**
- Nur Admin, oder darf der Arbeiter selbst nachtracken?
- Wie oft kommt das real vor (Häufigkeit rechtfertigt Phase 2?)

---

## Topic 3 — Termine von Arbeitern erstellen & verwalten

**Stand:** Angebot v2 liegt vor — 27h / 1.863€ (Phase 1: 21h Kalender, Phase 2: 6h WhatsApp)

**Kern:** Terminerfassung **existiert bereits** in der DB → wird erweitert, nicht neu gebaut. Neu ist die **Monteur-Zuweisung** (`appointment_employees`) plus ein Dashboard-Kalender (vue-cal, kostenlos, keine Google/Outlook-Abhängigkeit).

**Kalender-Features:**
- **Gesamtansicht:** alle Monteure, Farbcodierung pro Monteur, per Checkbox filterbar
- **Einzelansicht:** ein Monteur → nur seine Termine → freie Tage sofort sichtbar
- Wochen-/Monatsansicht, Österreichische Feiertage automatisch
- Offene/fuzzy Termine („Juli 2026", TBD) als Hinweis-Block
- Desktop first — Tablet/Mobile später (Kundenaussage: „momentan immer im Büro")

**Aus den Kundenantworten (wichtig für den Scope):**
- **Keine Stundenangaben nötig** — es reicht „Monteur A ist Montag auf Baustelle XY". Zeitaufwand ist wetter-/vor-Ort-abhängig und ohnehin nicht planbar
- Baustellen laufen teils **2–3 Jahre**; Monteure fahren selbstständig hin. Fixtermine legt der BO nur an, wenn vom Bauherrn/AG vorgegeben
- Benachrichtigung: **WhatsApp oder SMS, kein E-Mail** — bei Termin-Zuweisung **und** bei neuer Baustellen-Notiz
- Klick auf Termin → Sprung zur Baustelle: für BO nicht nötig, erst relevant bei einem Monteur-Kalender

**Bewusst nicht im Scope:** Google/Outlook-Sync (Phase 3, eigenes Angebot) · Monteur-Login mit eigener Kalenderansicht · Konflikt-Warnung bei Doppelbelegung

**Zu klären:**
- WhatsApp (Phase 2) jetzt mitbeauftragen? → **Meta-Template-Genehmigung braucht Vorlauf**, laufend ~15–25€/Monat
- Handynummern der 10 Monteure vorhanden?
- Wer darf Termine anlegen — wirklich nur BO? (koppelt an Topic 1)
- Konflikt-Warnung bei Doppelbelegung wirklich nicht nötig?

---

## Topic 4 — Flow Auftragserstellung: E-Mail → Angebot

**Stand:** Drei durchgerechnete Ausbaustufen + klickbarer Prototyp. **Ziel des Blocks: Variante entscheiden.**

### Prinzipien (gelten für alle drei Varianten)

1. **Der Mensch entscheidet, das System bereitet vor** — nie Auto-Send, kein Angebot ohne aktiven Versand durchs Büro
2. **Preise kommen nie von der KI** — deterministisch aus der DB (letztes Angebot = „gelebter Preis", oder Leistungskatalog). Die KI klassifiziert und formuliert, Zahlen rechnet der Code
3. **Datenschutz by Design** — dediziertes Postfach `anfragen@` mit Application Access Policy: das System kann `office@` technisch nicht lesen. Azure OpenAI in EU-Region
4. **Kein Agent-Framework** — deterministische State-Machine auf dem Bestands-Stack (Node/TS/Express, MySQL, Nuxt, Docker/VPS), voll auditierbar
5. **Additiver Ausbau** — Datenmodell in allen Varianten nahezu identisch, jede Stufe echte Teilmenge der nächsten. Upgrades sind Erweiterungen, keine Umbauten

### Die drei Varianten

| | **MVP** | **Kern** | **Vollausbau** |
|---|---|---|---|
| **Planwert** | ~142h ≈ **9.800€** | ~272h ≈ **18.700€** | ~416h ≈ **28.700€** |
| Spanne | 108–175h · 7.450–12.100€ | 209–334h · 14.400–23.000€ | 315–516h · 21.700–35.600€ |
| Automatisiert | nur eindeutige Wiederholungsprüfungen | Wiederholungsprüfung voll, Rest halbautomatisch | alle Szenarien inkl. Rückfragen |
| Cockpit | nein (Read-only-Monitor) | ja, mit Angebots-Editor | ja, mit KI-Diff-Editor |
| Freigabe-Ort | Outlook (Entwurf) | Dashboard → Outlook-Entwurf | Dashboard → Outlook-Entwurf |
| KI-Einsatz | 1 Call (Klassifikation) | + Anschreiben-Prosa | + strukturierte Änderungsvorschläge |
| Leistungskatalog | nicht nötig (nur Klonen) | ja, mit Pflege-UI | ja, + Textbausteine |
| Outcome-Tracking | nein | ja | ja |
| Templates | 1 | 2 | 4 |

*Von = optimistisch, Bis = realistisch — **die Spanne ist der Puffer**, kein Aufschlag. Basis: 1 erfahrener Full-Stack-Dev auf dem Bestands-Stack; `voice_requests` dient als erprobte Vorlage.*

**MVP „Der stille Kollege"** — macht nur, was todsicher ist: Anfrage muss *alle vier* Bedingungen erfüllen (als Wiederholungsprüfung erkannt, Kunde per Exakt-Match, Vorjahres-Angebot in DB, Vorlage vorhanden) → Vorjahres-Angebot klonen, PDF+Word als Entwurf in Carinas Outlook. Alles andere bleibt unangetastet liegen. Kein Cockpit. **Nebeneffekt, der den Ausschlag geben kann:** misst mit echten Daten, wie viele der ~50 Mails/Monat wirklich eindeutige Wiederholungsprüfungen sind — der Business Case für den Ausbau wird damit rechenbar statt geschätzt.

**Kern „Das Cockpit"** — Anfragen-Inbox mit Confidence, Ein-Klick-Kundenbestätigung, Angebots-Editor mit Katalog-Picker und Live-Summen, Freigabe-Flow mit Angebotsnummer, Outcome-Erfassung ab Tag 1. Klassifikation in 3 Kategorien. **Ehrlich sagen:** Die KI schlägt hier noch *keine* Angebots-Änderungen vor — Carina baut selbst, nur schneller. Vorlagen pflegt sie als Word-Dateien in SharePoint, kein Deployment.

**Vollausbau „Der Assistent"** — KI macht strukturierte Änderungsvorschläge, Diff-Ansicht im Editor (gold = neu mit Begründung, navy = geändert, rot = blockiert die Freigabe), Toggle „nur Änderungen". Dazu 5 Szenarien, Rückfragen-Flow, Webhook, OCR, vollständiger Historie-Import, 4 Templates. Auch hier: KI referenziert nur Katalogpositionen, Preise bleiben deterministisch.

### Empfehlung

**Einstieg MVP oder Kern — nicht Vollausbau.** Der Vollausbau ist als Zielbild richtig, aber seine teuersten Features (KI-Diff, Rückfragen-Automatik) lohnen erst bei belastbarer Datenbasis und bestätigtem Mengengerüst.

```
MVP (~10k€) → Kern (Delta ~6–9k€) → Vollausbau (Delta ~6–9k€)
```

Der Stufenweg kostet in Summe etwa dasselbe wie der direkte Vollausbau, verteilt aber Investition und Risiko — und jede Stufe entscheidet mit echten Erfahrungen über die nächste.

**Zum 15k-Ziel beim Kern:** 15k€ entspricht etwa dem Planwert der *reinen Entwicklung* (182–289h, ~16.250€). Seriös darstellbar als **Budget-Deckel für die Entwicklung mit priorisiertem Backlog**, plus Begleitung (27–45h) transparent nach Aufwand. Als Festpreis inkl. Begleitung wäre 15k€ am optimistischen Rand — davon ist abzuraten.

### Risiken & Voraussetzungen

- **Größtes Einzelrisiko (alle Varianten): Import der Alt-Angebote.** Schätzung gilt für PDFs mit Textlayer bzw. Excel-Altbestand. Liegt die Historie nur als Scan vor: **+10–20h**. → vor Beauftragung mit 2–3 echten Alt-PDFs verifizieren
- **Kundenseitig nötig:** Azure/M365-Tenant-Zugriff (Admin-Consent) · 2–3 anonymisierte Beispiel-Angebote · Kundenliste als Excel · Carina verfügbar für Workshop + Pilotphase
- **Nicht enthalten:** M365-Lizenzen, KI-Betriebskosten (~1–3€/Monat), Wartung nach Einführung

### Zu entscheiden im Meeting

- Welche Variante — und wenn Kern: Budget-Deckel-Modell akzeptiert?
- Alt-Angebote: PDF mit Textlayer oder Scan? → **2–3 Beispiele anschauen, bevor irgendwas beauftragt wird**
- `anfragen@`-Postfach anlegen — wer macht das, bis wann?
- Reihenfolge zum Rest: Angebots-Tool ist mit ~10–29k€ größer als die gesamte übrige Roadmap (6.624€). Läuft das parallel oder danach?

---

## Topic 5 — Reihenfolge & nächste Schritte

**Empfohlene Reihenfolge (Roadmap v2):**

```
Phase 0 Fundament    RBAC (23h) → Dashboard-Umbau (25h)
Phase 1 Kernfeatures Terminplanung (27h) + Toggl (21h)
Phase 2 Automatis.   Angebots-Tool (offen)
```

| Feature | Aufwand | Netto |
|---------|---------|-------|
| RBAC Rechteverwaltung | 23h | 1.587€ |
| Dashboard-Umbau | 25h | 1.725€ |
| Terminplanung & Kalender | 27h | 1.863€ |
| Toggl Manueller Export | 21h | 1.449€ |
| Angebots-Automatisierung | offen | offen |
| **Gesamt (ohne Angebots-Tool)** | **96h** | **6.624€** |

*exkl. 20% MwSt. · laufende Kosten ~15–25€/Monat (WhatsApp)*

**Offene Punkte aus der Roadmap:**
- [ ] Analyse-Session mit Carina terminisieren
- [ ] Stitch-Design für Dashboard-Umbau
- [ ] Angebote v2 an Kunden (RBAC, Dashboard, Terminplanung, Toggl)
- [ ] Angebots-Tool nach Analyse konkret schätzen

---

## Interne Notiz (nicht für den Kunden)

Die **BACKLOG.md-Dateien sind noch auf v1-Stand** und widersprechen den v2-Angeboten:

| Thema | Backlog Phase 1 | Angebot v2 Phase 1 |
|-------|-----------------|--------------------|
| RBAC | ~31h | 17h |
| Terminplanung | ~31,5h | 21h |
| Toggl | ~22h | 16h |

Die Reduktionen sind durch die Architektur-Analyse begründet und dokumentiert — die Backlogs sollten trotzdem nachgezogen werden, bevor sie jemand als Umsetzungsgrundlage nimmt.

**Dringender:** `angebots-automatisierung/ANGEBOT-AUTOMATISIERUNG.md` nennt **76h / 5.244€** für den Vollausbau (Phase 1–3). Die neue Variantenübersicht rechnet mit **315–516h / 21.700–35.600€** — Faktor 4–6. Das alte Angebot ist damit nicht nur veraltet, sondern gefährlich: wenn es beim Kunden liegt oder im Meeting aufgeht, steht eine 5k-Zahl gegen eine 29k-Zahl fürs selbe Zielbild. **Vor dem Meeting klären, ob das je rausgegangen ist**, und die Datei entweder löschen oder klar als überholt markieren.

Quelle der neuen Zahlen: `~/Downloads/variantenuebersichtangebotstool.md` (+ die drei `aufwandsschaetzung-*.xlsx`). Liegt noch außerhalb des Repos — sollte rein, sonst ist die Agenda die einzige Kopie.
