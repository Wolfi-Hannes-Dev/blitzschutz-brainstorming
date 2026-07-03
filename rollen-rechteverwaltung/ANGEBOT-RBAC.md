# Angebot: Rollen- & Rechteverwaltung (RBAC)
**Für:** Blitzschutz Reichenhauser  
**Von:** [Firmenname]  
**Datum:** 2026-07-03  
**Angebotsnummer:** 2026-XXX

---

## Projektziel

Ablösung der hardcoded Gruppen-Logik durch eine saubere, datenbankgestützte Rollen- und Rechteverwaltung. Der Admin kann im Dashboard selbst steuern, welcher Mitarbeiter welche Rolle hat — ohne Code-Änderungen oder Deployments. Neue Rollen (z.B. "Zeichner") werden sauber integriert. Rechte und Ansichten bleiben im Code definiert; nur die Zuweisung "wer gehört welcher Gruppe" wandert in die DB.

---

## Leistungsumfang

### Phase 1 — Migration & Core (Pflicht)

**Datenmodell & Migration**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.1 | DB-Schema: `groups` + `user_groups` Tabellen | 2h | 69€ | 138€ |
| 1.2 | Migration: bestehende hardcoded User/Gruppen in DB überführen | 2h | 69€ | 138€ |
| 1.3 | Hardcoded Gruppen-Checks im Code auf DB-basierte Lösung umstellen | 3h | 69€ | 207€ |

**Backend**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.4 | BE: Gruppen beim Login aus DB laden + in Token schreiben | 2h | 69€ | 138€ |
| 1.5 | BE: Auth-Middleware refactoren (Gruppen-Prüfung aus Token statt hardcoded) | 3h | 69€ | 207€ |
| 1.6 | BE: Alle bestehenden Endpoints auf neue Middleware umstellen + testen | 4h | 69€ | 276€ |
| 1.7 | BE: Endpoints für Gruppen-Management (Admin: User zuweisen/entfernen) | 3h | 69€ | 207€ |

**Frontend**

| Pos. | Leistung | Stunden | Satz | Betrag |
|------|----------|---------|------|--------|
| 1.8 | FE: Route Guards auf Gruppen-basierte Prüfung umstellen | 3h | 69€ | 207€ |
| 1.9 | FE: Bedingte UI-Elemente (Buttons, Menüs, Tabs) nach Gruppe steuern | 3h | 69€ | 207€ |
| 1.10 | FE: Zeichner-Ansicht — Baustellen sehen + Selbst-Zuweisung | 4h | 69€ | 276€ |
| 1.11 | FE: Bestehende Worker-Ansicht prüfen + anpassen | 2h | 69€ | 138€ |

| **Zwischensumme Phase 1** | | **31h** | | **2.139€** |

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
| Phase 1 — Migration + Core | 31h | **2.139€** |
| Phase 2 — Admin-UI Gruppenverwaltung | 6h | **414€** |
| **Gesamt** | **37h** | **2.553€** |

*Alle Preise exkl. 20% MwSt.*

---

## Hinweise

**Scope der Endpoint-Migration:**
Der Aufwand für die Umstellung bestehender Endpoints (Pos. 1.6) basiert auf einer typischen Schätzung. Nach einer initialen Code-Analyse kann dieser Posten präzisiert werden — Abweichung nach oben oder unten möglich.

**Keine neuen Permissions in der DB:**
Rechte und Ansichten bleiben bewusst im Code definiert — das verhindert inkonsistente Zustände und hält die Logik wartbar. Nur die User-Gruppen-Zuweisung läuft über die DB.

**Reihenfolge:**
Phase 1 ist Voraussetzung für Phase 2. Innerhalb von Phase 1 müssen Backend-Änderungen (1.1–1.7) vor den Frontend-Änderungen (1.8–1.11) abgeschlossen sein.

---

## Voraussetzungen (kundenseitig)

- Zugang zur bestehenden Codebase (Backend + Frontend) ✓
- Liste aller aktuellen User + ihre Rollen für Migration bereitstellen

---

*Angebot gültig bis: 2026-08-03*  
*Erstellt von Wolfi, 2026-07-03*
