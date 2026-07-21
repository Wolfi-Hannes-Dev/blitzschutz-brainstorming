# Angebots-Automatisierung — Die drei Ausbaustufen im Überblick

**Kunde:** Blitzschutz Willibald Reichenhauser GmbH · **Stand:** Juli 2026 · **Stundensatz:** 69 €/h
**Detail-Schätzungen:** `aufwandsschaetzung-angebotstool.xlsx` (Vollausbau) · `aufwandsschaetzung-v1-kern.xlsx` (Kern) · `aufwandsschaetzung-mvp.xlsx` (MVP)

---

## Worum es geht

Eingehende Kunden-E-Mails (Angebotsanfragen, v. a. jährliche Wiederholungsprüfungen von Blitzschutzanlagen) sollen automatisch erkannt und zu fertigen Angebots-Entwürfen verarbeitet werden. Volumen: ~50 Mails/Monat.

**Prinzipien, die für alle drei Varianten gelten:**

1. **Der Mensch entscheidet, das System bereitet vor.** Kein Angebot verlässt das Haus ohne Prüfung und aktiven Versand durch das Büro. Nie Auto-Send.
2. **Preise kommen nie von der KI.** Sie werden deterministisch aus der Datenbank ermittelt: aus dem letzten Angebot des Kunden („gelebter Preis") oder aus dem Leistungskatalog (Listenpreis). Die KI klassifiziert und formuliert — Zahlen rechnet der Code.
3. **Datenschutz by Design.** Dediziertes Postfach `anfragen@` mit hartem technischem Zugriffs-Limit (Application Access Policy) — das System kann das allgemeine `office@`-Postfach physisch nicht lesen. KI-Verarbeitung über Azure OpenAI in EU-Region.
4. **Kein Agent-Framework, sondern eine deterministische State-Machine** auf dem bestehenden Stack (Node/TypeScript/Express, MySQL, Nuxt-Dashboard, Docker auf VPS). Voll auditierbar, jeder Schritt nachvollziehbar.
5. **Additiver Ausbau:** Das Datenmodell ist in allen Varianten (nahezu) identisch. Jede Stufe ist eine echte Teilmenge der nächsten — Upgrades sind Erweiterungen, keine Umbauten.

---

## Die drei Varianten auf einen Blick

| | **MVP** | **Kern** | **Vollausbau** |
|---|---|---|---|
| **Aufwand** | 108–175 h | 209–334 h | 315–516 h |
| **Kosten (69 €/h)** | 7.450–12.100 € | 14.400–23.000 € | 21.700–35.600 € |
| **Planwert (Mittelwert)** | ~142 h ≈ **9.800 €** | ~272 h ≈ **18.700 €** | ~416 h ≈ **28.700 €** |
| Automatisiert | nur eindeutige Wiederholungsprüfungen | Wiederholungsprüfung voll, Rest halbautomatisch | alle Szenarien inkl. Rückfragen |
| Cockpit im Dashboard | nein (nur Read-only-Monitor) | ja, mit Angebots-Editor | ja, mit KI-Diff-Editor |
| Freigabe-Ort | Outlook (Entwurf) | Dashboard → Outlook-Entwurf | Dashboard → Outlook-Entwurf |
| KI-Einsatz | 1 Call (Klassifikation) | Klassifikation + Anschreiben-Prosa | + strukturierte Änderungsvorschläge |
| Leistungskatalog | nicht nötig (nur Klonen) | ja, mit Pflege-UI | ja, + Textbausteine-Pflege |
| Historie-Import | letztes Angebot je Kunde | letztes Angebot je Kunde | vollständig, mit Review-UI |
| Template-Varianten | 1 | 2 | 4 |
| Outcome-Tracking (gewonnen/verloren) | nein | ja | ja |

Alle Schätzungen: Von = optimistisch, Bis = realistisch — **die Spanne ist der Puffer**, kein zusätzlicher Aufschlag. Basis: 1 erfahrener Full-Stack-Entwickler auf dem Bestands-Stack; das vorhandene `voice_requests`-Feature dient als erprobte Vorlage für Architektur und UI-Muster.

---

## Variante 1 — MVP „Der stille Kollege" (< 10 k€)

**Idee in einem Satz:** Ein System, das nichts falsch machen kann, weil es nur die todsicheren Fälle anfasst.

Verarbeitet werden ausschließlich Anfragen, die **alle vier** Bedingungen erfüllen: als Wiederholungsprüfung erkannt (hohe Confidence) · Kunde per Exakt-Match eindeutig · Vorjahres-Angebot in der Datenbank · Vorlage vorhanden. Dann: Vorjahres-Angebot klonen (Positionen und Preise 1:1, neues Datum), PDF rendern, **fertigen Entwurf mit PDF- und Word-Anhang direkt in Carinas Outlook-Entwürfe legen**. Sie prüft, korrigiert notfalls im Word-Dokument, sendet selbst. Alle anderen Mails bleiben unangetastet im Postfach — dort ändert sich nichts am heutigen Arbeiten.

Es gibt kein Cockpit. Die einzige neue Dashboard-Seite ist ein Read-only-Monitor: was kam rein, was wurde vorbereitet, was wurde (mit Grund) unangetastet gelassen, Fehler mit Retry. Die KI macht genau einen Call (Klassifikation); das Anschreiben ist fixer Vorlagentext.

**Schätzung: 108–175 h → 7.450–12.100 €, Planwert ~9.800 € (inkl. Einführung).**

**Begründung der Schätzung:** Der große Kostentreiber aller Varianten ist das Cockpit (Editor, Freigabe-Flows, Stammdaten-UIs) — er entfällt komplett (~90–130 h gespart gegenüber Vollausbau). Ebenso entfallen Leistungskatalog (der Klon braucht keinen), LLM-Prosa, Anhang-Textextraktion und Webhook-Infrastruktur (Delta-Polling reicht bei diesem Volumen). Was bleibt, ist bewusst schmal, aber vollständig: Zero-Loss-Maileingang, Klassifikation, Klon-Engine, PDF, Outlook-Übergabe, Monitor.

**Bewusste Trade-offs:** Editiert Carina das Word-Dokument im Entwurf, kennt die Datenbank nur die generierte Version (Preishistorie kann bei editierten Angeboten leicht abweichen — beim Cockpit-Ausbau strukturell behoben). Kein Outcome-Tracking, keine Kandidaten-Bestätigung bei unsicherem Kunden-Match.

**Wofür der MVP zusätzlich gut ist:** Er misst mit echten Daten die entscheidende Business-Case-Frage — *wie viele der ~50 Mails/Monat sind wirklich eindeutige Wiederholungsprüfungen?* Das Ergebnis rechtfertigt (oder widerlegt) den weiteren Ausbau mit Zahlen statt Bauchgefühl.

---

## Variante 2 — Kern „Das Cockpit" (~ 15 k€)

**Idee in einem Satz:** Volle Kontrolle und Übersicht im Dashboard — die KI ordnet ein, der Mensch baut mit Werkzeug.

Alles aus dem MVP, plus das **Cockpit im bestehenden Dashboard**: Anfragen-Inbox mit Status und Confidence, Anfrage-Detail mit Ein-Klick-Kundenbestätigung (bei unsicherem Match) und Notizfeld, **Angebots-Editor** mit Positionstabelle, Katalog-Picker und Live-Summen, PDF-Vorschau, Freigabe-Flow mit Angebotsnummern-Vergabe, Verlauf mit Outcome-Erfassung (gewonnen/verloren + Marge — die Datenbasis für spätere Margen-Vorhersage fällt ab Tag 1 an).

Klassifiziert wird in drei Kategorien (Wiederholungsprüfung / Sonstige Anfrage / Spam). Sonstige Anfragen und Neukunden landen kontrolliert im Editor, wo Carina aus dem Leistungskatalog baut — halbautomatisch, aber deutlich schneller als heute. Wichtige Ehrlichkeit dieser Stufe: **Die KI schlägt hier noch keine Angebots-Änderungen vor** — das Angebot ist ein deterministischer Klon bzw. Katalog-Aufbau, Carina passt selbst an (ihre Notizen als Seitenpanel). Vorlagen pflegt sie als Word-Dateien in einem SharePoint-Ordner — kein Deployment nötig.

**Schätzung: 209–334 h → 14.400–23.000 €, Planwert ~18.700 € inkl. Einführung; reine Entwicklung (ohne Workshop/PM/Go-Live/Schulung): 182–289 h, Planwert ~16.250 €.**

**Begründung der Schätzung:** Gegenüber dem Vollausbau eingespart (zusammen ~80–130 h): Webhook-Infrastruktur (→ Delta-Polling), KI-Änderungsvorschläge samt Diff-UI, Rückfragen-Automatik, Textbausteine-Verwaltung (→ Word-Vorlagen), Import-Review-UI (→ Excel-Kontrolldatei), zwei Template-Varianten, OCR, automatisches Versand-Tracking (→ manueller Button). Alle Schnitte sind Logik/UI — das Datenmodell bleibt vollständig, jeder gestrichene Punkt ist als „Phase 2" (81–132 h) nachrüstbar, ohne etwas umzubauen.

**Ehrliche Einordnung des 15k-Ziels:** 15 k€ entspricht etwa dem Planwert der reinen Entwicklung. Seriös darstellbar als Budget-Deckel für die Entwicklung mit priorisiertem Backlog plus Begleitung (27–45 h) transparent nach Aufwand. Als Festpreis für alles inklusive Begleitung wäre 15 k€ am optimistischen Rand kalkuliert — davon ist abzuraten.

---

## Variante 3 — Vollausbau „Der Assistent" (~ 22–36 k€)

**Idee in einem Satz:** Carina prüft drei markierte Änderungen, statt vierzehn Positionen zu tippen.

Alles aus dem Kern, plus die intelligenten Features: Die KI liest Mail und Notizen und macht **strukturierte Änderungsvorschläge** — „Position X aus dem Katalog dazu, weil der Kunde den neuen Anbau erwähnt", „Menge 10 → 12". Diese Vorschläge erscheinen im Editor als **Diff-Ansicht**: gold = neu (KI, mit Begründung), navy = geändert, rot = nicht zuordenbar (blockiert die Freigabe, bis aufgelöst). Ein Toggle „Nur Änderungen zeigen" reduziert die Prüfarbeit auf das Wesentliche. Entscheidend fürs Sicherheitsmodell: Die KI referenziert dabei **nur Katalogpositionen — die Preise kommen weiterhin deterministisch aus der Datenbank.**

Dazu: fünf Szenarien statt drei Kategorien, automatischer **Rückfragen-Flow** (KI formuliert Rückfragen ohne jede Zahl, Antwort des Kunden wird dem Vorgang zugeordnet), Webhook für Sekunden-Latenz, OCR für gescannte Anhänge, vollständiger Historie-Import mit komfortabler Review-UI, vier Template-Varianten mit Validierungs-Check, Textbausteine-Verwaltung, automatisches Versand-Tracking, E2E-Tests.

**Schätzung: 315–516 h → 21.700–35.600 €, Planwert ~28.700 € (davon nur MUST-Features: 299–488 h).**

**Begründung der Schätzung:** Die drei größten Blöcke sind die Angebots-Engine (51–82 h — hier steckt die KI-Vorschlagslogik samt Leitplanken), das Cockpit (50–80 h, davon allein der Diff-Editor 16–26 h) und Stammdaten & Import (48–78 h — der vollständige Historie-Import mit Review-UI ist aufwendig, liefert aber die beste Datenqualität für Preisauflösung und spätere Auswertungen).

---

## Empfehlung & Weg

**Empfohlener Einstieg: MVP oder Kern — nicht der Vollausbau.** Der Vollausbau ist als Zielbild richtig, aber seine teuersten Features (KI-Diff, Rückfragen-Automatik) lohnen sich erst, wenn die Basisdaten belastbar sind und das Mengengerüst bestätigt ist. Der natürliche Pfad:

1. **MVP** (~10 k€): beweist den Wert am häufigsten Fall, produziert echte Messdaten, minimales Risiko.
2. **→ Kern** (Delta ~6–9 k€): Cockpit, Editor, Katalog, Outcome-Tracking — sobald der MVP zeigt, dass sich mehr Automatisierung lohnt.
3. **→ Vollausbau** (Delta ~6–9 k€, entspricht der „Phase 2"-Liste): KI-Vorschläge, Rückfragen, Komfort — auf inzwischen sauberer Datenbasis.

Der Stufenweg kostet in Summe etwa dasselbe wie der direkte Vollausbau, verteilt aber Investition und Risiko und lässt jede Stufe mit echten Erfahrungen über die nächste entscheiden.

**Größtes Einzelrisiko (alle Varianten):** der Import der Alt-Angebote. Die Schätzungen gelten für PDFs mit Textlayer bzw. Excel-Altbestand; liegt die Historie nur als Scan vor, kommen ca. 10–20 h dazu. → Vor Beauftragung mit 2–3 echten Alt-PDFs verifizieren.

**Voraussetzungen beim Kunden (alle Varianten):** Zugriff auf den Azure/M365-Tenant (Admin-Consent), 2–3 anonymisierte Beispiel-Angebote, Kundenliste als Excel, Verfügbarkeit der Büro-Mitarbeiterin für einen Workshop und die Pilotphase. Nicht enthalten: M365-Lizenzen, KI-Betriebskosten (~1–3 €/Monat), laufende Wartung nach der Einführungsphase.
