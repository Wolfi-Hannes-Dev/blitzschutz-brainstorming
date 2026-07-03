# Angebot: Dashboard-Umbau — v2
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX  
**Version:** 2 (Schätzung nach Analyse der bestehenden Architektur)

**Änderungen gegenüber v1:**
- Routing/Auth bereits vorhanden (MS Auth), wird übernommen
- Projektzuweisung zu MA existiert schon → weniger Neu-Aufbau
- Auftragsgenerierungs-Seiten aus Dashboard rausgezogen (gehören zum Angebots-Tool)
- Schätzung reduziert von 51h / 3.519€ auf 25h / 1.725€

---

## Projektziel

Neue Navigationsebene über dem bestehenden Dashboard. Bestehende Ansichten bleiben erhalten und werden integriert. Alle neuen Features (Kalender, Mitarbeiter/Gruppen) bekommen eigene Seiten. Jede Rolle sieht nur was sie soll. Design nach Stitch-Vorgabe, Nuxt UI als Komponenten-Basis.

---

## Leistungsumfang

### Phase 1 — Struktur + Admin-Seiten

**Basis & Navigation**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | Nuxt UI Setup + Firmenfarben-Theme | 2h | 69€ | 138€ |
| 1.2 | Sidebar-Navigation + Header Layout (Desktop first, nach Stitch-Design) | 3h | 69€ | 207€ |
| 1.3 | Rollenbasierte Navigation (Menüpunkte nach Gruppe ein-/ausblenden) | 2h | 69€ | 138€ |

**Seiten-Integration**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.4 | Home/Übersicht: Kennzahlen-Cards (offene Baustellen, nächste Termine, offene Drafts) | 3h | 69€ | 207€ |
| 1.5 | Baustellen-Seite: bestehende Ansicht in neues Layout integrieren | 2h | 69€ | 138€ |
| 1.6 | Kalender-Seite: vue-cal einbetten (aus Terminplanung-Feature) | 1h | 69€ | 69€ |
| 1.7 | Mitarbeiter-Übersicht + Gruppenverwaltung (aus RBAC-Feature) | 2h | 69€ | 138€ |

| **Zwischensumme Phase 1** | | **15h** | | **1.035€** |

---

### Phase 2 — Worker/Zeichner Ansichten + Polish

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 2.1 | Worker-Ansicht: eingeschränkte Navigation + eigene Baustellen | 2h | 69€ | 138€ |
| 2.2 | Zeichner-Ansicht: Baustellen + Selbst-Zuweisung | 2h | 69€ | 138€ |
| 2.3 | Profil-Seite: eigene Daten + WhatsApp-Nummer | 2h | 69€ | 138€ |
| 2.4 | Loading/Error/Leer-Zustände (alle Seiten) | 2h | 69€ | 138€ |
| 2.5 | Mobile/Tablet: Navigation kollabierbar | 2h | 69€ | 138€ |

| **Zwischensumme Phase 2** | | **10h** | | **690€** |

---

## Gesamtübersicht

| Phase | Aufwand | Betrag netto |
|-------|---------|--------------|
| Phase 1 — Struktur + Admin-Seiten | 15h | **1.035€** |
| Phase 2 — Worker/Zeichner + Polish | 10h | **690€** |
| **Gesamt** | **25h** | **1.725€** |

*Alle Preise exkl. 20% MwSt.*

**Hinweis:** Design (Layouts, Komponenten) wird kundenseitig via Stitch geliefert. Abweichungen nach Start werden nach Aufwand (69€/h) berechnet.

---

*Angebot gültig bis: 2026-08-03 | Erstellt von Wolfi, 2026-07-03*
