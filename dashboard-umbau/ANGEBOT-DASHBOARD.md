# Angebot: Dashboard-Umbau
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX

---

## Projektziel

Struktureller Umbau des bestehenden Dashboards: Eine neue Navigationsebene fasst alle Features (Kalender, Auftragsgenerierung, Mitarbeiter/Gruppen) als eigene Seiten zusammen. Die bestehende Baustellenansicht bleibt erhalten und wird integriert. Jede Benutzergruppe (Admin, Worker, Zeichner) sieht nur die für sie relevanten Bereiche.

Technische Basis: **Nuxt UI** (kostenlos, Open Source) mit den Firmenfarben. Design wird kundenseitig via Stitch geliefert.

---

## Leistungsumfang

### Phase 1 — Struktur + Admin-Seiten

**Basis & Navigation**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | Nuxt UI Setup + Firmenfarben-Theme | 2h | 69€ | 138€ |
| 1.2 | Sidebar-Navigation + Header Layout (Desktop first) | 4h | 69€ | 276€ |
| 1.3 | Routing-Struktur: alle Seiten + Subpages | 2h | 69€ | 138€ |
| 1.4 | Rollenbasierte Navigation (Menüpunkte nach Gruppe) | 2h | 69€ | 138€ |

**Admin-Seiten (nach Stitch-Design)**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.5 | Home/Übersicht: Kennzahlen-Cards (Baustellen, Termine, offene Drafts) | 3h | 69€ | 207€ |
| 1.6 | Baustellen-Seite: bestehende Ansicht in neues Layout integrieren | 3h | 69€ | 207€ |
| 1.7 | Kalender-Seite: vue-cal einbetten | 2h | 69€ | 138€ |
| 1.8 | Auftragsgenerierung — Posteingang (Mails + KI-Klassifizierung + Szenario-Badge) | 4h | 69€ | 276€ |
| 1.9 | Auftragsgenerierung — Drafts (Freigabe-Liste + Human-in-the-Loop Button) | 4h | 69€ | 276€ |
| 1.10 | Auftragsgenerierung — Verlauf (gesendete Angebote, filterbar) | 3h | 69€ | 207€ |
| 1.11 | Mitarbeiter-Übersicht + Gruppenverwaltung | 4h | 69€ | 276€ |

| **Zwischensumme Phase 1** | | **33h** | | **2.277€** |

---

### Phase 2 — Worker/Zeichner Ansichten + Polish

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 2.1 | Worker-Ansicht: eingeschränkte Navigation + eigene Baustellen | 3h | 69€ | 207€ |
| 2.2 | Zeichner-Ansicht: Baustellen + Selbst-Zuweisung | 3h | 69€ | 207€ |
| 2.3 | Profil-Seite: eigene Daten + WhatsApp-Nummer | 2h | 69€ | 138€ |
| 2.4 | Loading/Error/Leer-Zustände (alle Seiten) | 3h | 69€ | 207€ |
| 2.5 | Mobile/Tablet: Navigation kollabierbar | 3h | 69€ | 207€ |
| 2.6 | Konsistenz-Check: bestehende Seiten an Nuxt UI angleichen | 4h | 69€ | 276€ |

| **Zwischensumme Phase 2** | | **18h** | | **1.242€** |

---

## Gesamtübersicht

| Phase | Aufwand | Betrag netto |
|-------|---------|--------------|
| Phase 1 — Struktur + Admin-Seiten | 33h | **2.277€** |
| Phase 2 — Worker/Zeichner + Polish | 18h | **1.242€** |
| **Gesamt** | **51h** | **3.519€** |

*Alle Preise exkl. 20% MwSt.*

---

## Hinweise

**Design via Stitch (kundenseitig):**
Das visuelle Design (Layouts, Farben, Komponenten-Auswahl) wird vom Kunden via Stitch geliefert. Die Implementierung erfolgt nach dieser Vorgabe. Abweichungen oder Änderungen nach Start der Implementierung werden nach Aufwand (69€/h) berechnet.

**Abhängigkeiten:**
Dieser Umbau setzt folgende Features voraus (parallel oder vorher):
- RBAC Rechteverwaltung (rollenbasierte Navigation)
- Terminplanung & Kalender (Kalender-Seite)
- Angebots-Automatisierung (Auftragsgenerierungs-Seiten)

**Nuxt UI:**
Open Source, kostenlos, keine Lizenzkosten.

---

## Voraussetzungen (kundenseitig)

- Stitch-Design vor Implementierungsstart liefern
- Firmenfarben (HEX-Werte) bereitstellen ✓ (bereits vorhanden)
- Zugang zur bestehenden Codebase ✓

---

*Angebot gültig bis: 2026-08-03*  
*Erstellt von Wolfi, 2026-07-03*
