# Backlog: Rollen- & Rechteverwaltung (RBAC)
**Projekt:** Blitzschutz Reichenhauser  
**Stand:** 2026-07-03

---

## Konzept

**Role-Based Access Control (RBAC)** — Gruppen werden im Code definiert (Rechte/Ansichten), User-Zuweisungen laufen über die DB. Admin pflegt alles im Dashboard.

```
User  →  user_groups (JOIN)  →  Group  →  Permissions (hardcoded im Code)
```

**Gruppen (aktuell bekannt):**
- `admin` — Vollzugriff
- `worker` (Arbeiter/Monteur) — eigene Baustellen sehen, Zeiten tracken
- `designer` (Zeichner) — Baustellen sehen, sich selbst zuweisen

**Neues Verhalten:**
- Rechte/Ansichten bleiben im Code definiert (kein DB-Overhead für Permissions)
- **Wer welcher Gruppe angehört** → DB, pflegbar durch Admin im Dashboard
- E-Mail-Änderungen, neue Mitarbeiter, Rollenwechsel: alles über UI, kein Code-Deployment nötig

---

## Epics & Stories

---

### 🔵 EPIC 1 — Datenmodell & Migration

| ID | Story | Aufwand |
|----|-------|---------|
| E1-1 | DB-Schema: `groups` Tabelle (id, name, description) | 1h |
| E1-2 | DB-Schema: `user_groups` JOIN-Tabelle (userId, groupId) | 1h |
| E1-3 | Migration: bestehende hardcoded Gruppen/User in DB überführen | 2h |
| E1-4 | Bestehende hardcoded Gruppen-Checks im Code auf DB-basierte Lösung umstellen | 3h |

**Summe Epic 1: ~7h**

---

### 🟢 EPIC 2 — Backend: Auth & Middleware

| ID | Story | Aufwand |
|----|-------|---------|
| E2-1 | BE: Gruppen beim Login aus DB laden + in JWT/Session schreiben | 2h |
| E2-2 | BE: Auth-Middleware refactoren — prüft Gruppe aus Token statt hardcoded Liste | 3h |
| E2-3 | BE: Alle bestehenden Endpoints auf neue Middleware umstellen (Audit + Test) | 4h |
| E2-4 | BE: Neue Endpoints für Gruppen-Management (CRUD user_groups, nur Admin) | 3h |

**Summe Epic 2: ~12h**

---

### 🟢 EPIC 3 — Frontend: Ansichten & Rechte

| ID | Story | Aufwand |
|----|-------|---------|
| E3-1 | FE: Route Guards / Navigation Guards auf Gruppen-basierte Prüfung umstellen | 3h |
| E3-2 | FE: Bedingte UI-Elemente (Buttons, Menüs, Tabs) nach Gruppe ein-/ausblenden | 3h |
| E3-3 | FE: Zeichner-Ansicht — Baustellen sehen + sich selbst zuweisen (eigener View oder eingeschränkter Admin-View) | 4h |
| E3-4 | FE: Bestehende Worker-Ansicht prüfen + ggf. anpassen | 2h |

**Summe Epic 3: ~12h**

---

### 🟡 EPIC 4 — Admin-UI: Gruppenverwaltung

| ID | Story | Aufwand |
|----|-------|---------|
| E4-1 | FE: User-Liste im Admin-Dashboard mit Gruppen-Anzeige | 2h |
| E4-2 | FE: User einer Gruppe zuweisen / entfernen (Dropdown oder Checkboxen) | 3h |
| E4-3 | FE: Übersicht "Wer ist in welcher Gruppe" | 1h |

**Summe Epic 4: ~6h**

---

## Phasen-Planung

### Phase 1 — Migration + Core (Pflicht)
Epic 1 + Epic 2 + Epic 3
→ **~31h** | Ergebnis: Hardcoded Groups weg, alles DB-basiert, Zeichner-Rolle funktioniert

### Phase 2 — Admin-UI
Epic 4
→ **+6h** | Ergebnis: Admin kann Gruppen selbst pflegen ohne Entwickler

---

## Risiken / Hinweise

- **Bestehende Endpoints:** Jeder Endpoint der aktuell Gruppen prüft muss migriert werden — Aufwand hängt von der Anzahl der betroffenen Stellen ab. Schätzung basiert auf typischem Umfang; Abweichung möglich nach Code-Analyse.
- **Reihenfolge:** Epic 1 + 2 müssen vor Epic 3 + 4 fertig sein (Abhängigkeit).
- **Zeichner-Selbstzuweisung:** Nur Zeichner und Admin können Zeichner einer Baustelle zuweisen — das wird als eigene Permission-Regel im Code definiert.

---

## ⚠️ Technical Debt — Entscheidung 2026-07-27: Variante A

Nach Code-Analyse: Das Backend hat **keine Nutzer-Identität** — die gesamte Auth ist ein Shared Secret (`VITE_API_TOKEN`), für alle Aufrufer gleich, aus dem Browser-Bundle auslesbar. RBAC läuft daher **nur im Frontend** (Anzeige-Logik), nicht serverseitig erzwungen.

**Bewusst gewählt (Variante A). Als Technical Debt aufgeschoben ist Variante B:**
- BE: MSAL-Token serverseitig validieren (JWKS, Tenant, Cache)
- BE: Identität in die Request-Pipeline (`req.user`)
- BE: ~30 Bestandsrouten auf identitätsbasierte Auth umstellen
- FE: MSAL-Token mitsenden statt Shared Secret
- Azure: API-Scope registrieren
- **Aufwand bei Nachrüstung: +19–30 h** (ggf. mehr, je mehr Endpunkte bis dahin existieren)

**Konsequenz bis dahin:** „Nur eigene Daten" und Admin-/Rollen-Sperren sind Anzeige-Logik, keine echte Sperre. Wer das Bundle-Token kennt, kann jeden Endpunkt aufrufen und die Daten jedes Mitarbeiters abfragen.

**Auslöser zum Abbau:** heikle Daten · nicht vertrauenswürdiger Nutzer · eine Rolle muss etwas wirklich *verhindern* statt nur *verbergen*.

---

*Erstellt von Wolfi, 2026-07-03 · Technical-Debt-Vermerk ergänzt 2026-07-27*
