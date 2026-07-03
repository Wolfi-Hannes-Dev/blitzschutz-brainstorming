# Angebot: Rollen- & Rechteverwaltung (RBAC) — v2
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX  
**Version:** 2 (Schätzung nach Analyse der bestehenden Architektur)

**Änderungen gegenüber v1:**
- Gruppenlogik ist nur im FE hardcoded (nicht im BE) → BE-Migration entfällt großteils
- MS Auth bereits vorhanden → kein Auth-Aufwand
- Schätzung reduziert von 37h / 2.553€ auf 23h / 1.587€

---

## Projektziel

Ablösung der hardcoded Gruppen-Logik im Frontend durch eine DB-basierte Lösung. Admin kann User-Rollen selbst im Dashboard pflegen. Neue Gruppe "Zeichner" wird sauber integriert.

---

## Leistungsumfang

### Phase 1 — Migration & Core (Pflicht)

**Datenmodell & Migration**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | DB-Schema: `groups` + `user_groups` Tabellen | 2h | 69€ | 138€ |
| 1.2 | Migration: bestehende hardcoded User/Gruppen in DB überführen | 1h | 69€ | 69€ |

**Backend**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.3 | BE: Gruppen beim Login aus DB laden + in MS Auth Token/Session schreiben | 2h | 69€ | 138€ |
| 1.4 | BE: Endpoints für Gruppen-Management (Admin: User zuweisen/entfernen) | 3h | 69€ | 207€ |

**Frontend**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.5 | FE: Hardcoded Gruppen-Checks auf DB-basierte Lösung umstellen | 3h | 69€ | 207€ |
| 1.6 | FE: Route Guards + bedingte UI-Elemente nach Gruppe steuern | 3h | 69€ | 207€ |
| 1.7 | FE: Zeichner-Ansicht — Baustellen sehen + Selbst-Zuweisung | 3h | 69€ | 207€ |

| **Zwischensumme Phase 1** | | **17h** | | **1.173€** |

---

### Phase 2 — Admin-UI: Gruppenverwaltung

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 2.1 | FE: User-Liste mit Gruppen-Anzeige im Admin-Dashboard | 2h | 69€ | 138€ |
| 2.2 | FE: User einer Gruppe zuweisen / entfernen | 3h | 69€ | 207€ |
| 2.3 | FE: Übersicht "Wer ist in welcher Gruppe" | 1h | 69€ | 69€ |

| **Zwischensumme Phase 2** | | **6h** | | **414€** |

---

## Gesamtübersicht

| Phase | Aufwand | Betrag netto |
|-------|---------|--------------|
| Phase 1 — Migration + Core | 17h | **1.173€** |
| Phase 2 — Admin-UI Gruppenverwaltung | 6h | **414€** |
| **Gesamt** | **23h** | **1.587€** |

*Alle Preise exkl. 20% MwSt.*

---

*Angebot gültig bis: 2026-08-03 | Erstellt von Wolfi, 2026-07-03*
