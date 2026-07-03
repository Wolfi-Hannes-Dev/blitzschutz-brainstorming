# Backlog: Dashboard-Umbau (Admin + Rollen-Ansichten)
**Projekt:** Blitzschutz Reichenhauser  
**Stand:** 2026-07-03

---

## Konzept

Bestehende Ansicht bleibt erhalten und wird als Seite ins neue Dashboard integriert. Neue Navigationsebene davor mit allen neuen Features als eigene Seiten/Subpages. Nuxt UI als Komponenten-Basis, Firmenfarben übernommen.

**Design:** Wird kundenseitig via Stitch geliefert. Implementierung erfolgt nach Stitch-Vorgabe.

---

## Neue Dashboard-Struktur

```
Admin-Dashboard
├── 🏠 Übersicht (Home)          ← neu: Kurzübersicht / Kennzahlen
├── 🏗️ Baustellen                ← bestehende Ansicht, integriert
├── 📅 Kalender                  ← neu (Terminplanung-Feature)
├── 📬 Auftragsgenerierung       ← neu
│   ├── Posteingang              (eingehende Mails + KI-Klassifizierung)
│   ├── Drafts                   (wartende Freigaben — Human in the Loop)
│   └── Verlauf                  (gesendete Angebote)
├── 👷 Mitarbeiter               ← neu/erweitert
│   ├── Übersicht                (alle User + Gruppen)
│   └── Gruppenverwaltung        (RBAC-Zuweisung)
└── ⚙️ Einstellungen             ← bestehend + erweitert

Worker-Ansicht (eingeschränkt):
├── 🏗️ Baustellen (nur eigene)
└── ⚙️ Profil

Zeichner-Ansicht (eingeschränkt):
├── 🏗️ Baustellen (alle sehen, selbst zuweisen)
└── ⚙️ Profil
```

---

## Epics & Stories

---

### 🔵 EPIC 1 — Nuxt UI Setup & Basis

| ID | Story | Aufwand |
|----|-------|---------|
| E1-1 | Nuxt UI installieren + konfigurieren (Firmenfarben, Basis-Theme) | 2h |
| E1-2 | Layout-Komponente: Sidebar-Navigation + Header (responsive, Desktop first) | 4h |
| E1-3 | Routing-Struktur aufbauen (alle neuen Seiten + Subpages anlegen) | 2h |
| E1-4 | Rollenbasierte Navigation: Menüpunkte nach Gruppe ein-/ausblenden | 2h |

**Summe Epic 1: ~10h**

---

### 🟢 EPIC 2 — Admin-Seiten implementieren (nach Stitch-Design)

| ID | Story | Aufwand |
|----|-------|---------|
| E2-1 | Home/Übersicht: Kennzahlen-Cards (offene Baustellen, nächste Termine, offene Drafts) | 3h |
| E2-2 | Baustellen-Seite: bestehende Ansicht in neues Layout integrieren | 3h |
| E2-3 | Kalender-Seite: vue-cal einbetten (aus Terminplanung-Feature) | 2h |
| E2-4 | Auftragsgenerierung — Posteingang: Mail-Liste + KI-Klassifizierung + Szenario-Badge | 4h |
| E2-5 | Auftragsgenerierung — Drafts: Liste wartender Freigaben + Freigabe-Button (Human in the Loop) | 4h |
| E2-6 | Auftragsgenerierung — Verlauf: gesendete Angebote, filterbar nach Datum/Kunde | 3h |
| E2-7 | Mitarbeiter-Übersicht: User-Liste mit Gruppen-Badges | 2h |
| E2-8 | Gruppenverwaltung: User-Gruppen-Zuweisung (aus RBAC-Feature) | 2h |

**Summe Epic 2: ~23h**

---

### 🟡 EPIC 3 — Worker & Zeichner Ansichten

| ID | Story | Aufwand |
|----|-------|---------|
| E3-1 | Worker-Ansicht: eingeschränkte Navigation + nur eigene Baustellen | 3h |
| E3-2 | Zeichner-Ansicht: Baustellen sehen + Selbst-Zuweisung (aus RBAC-Feature) | 3h |
| E3-3 | Profil-Seite: eigene Daten + WhatsApp-Nummer pflegen | 2h |

**Summe Epic 3: ~8h**

---

### 🟡 EPIC 4 — Polish & Qualität

| ID | Story | Aufwand |
|----|-------|---------|
| E4-1 | Loading States, Error States, leere Zustände (alle neuen Seiten) | 3h |
| E4-2 | Mobile/Tablet: Navigation kollabierbar (Hamburger-Menü) | 3h |
| E4-3 | Konsistenz-Check: bestehende Seiten an Nuxt UI angleichen | 4h |

**Summe Epic 4: ~10h**

---

## Phasen-Planung

### Phase 1 — Struktur + Admin-Core
Epic 1 + Epic 2
→ **~33h** | Ergebnis: Neue Dashboard-Struktur steht, alle Admin-Seiten vorhanden

### Phase 2 — Worker/Zeichner + Polish
Epic 3 + Epic 4
→ **+18h** | Ergebnis: Alle Rollen haben ihre Ansicht, mobile-ready, konsistent

---

## Abhängigkeiten

- Design (Stitch) muss vor Implementierung vorliegen
- RBAC-Feature (Rechteverwaltung) ist Voraussetzung für rollenbasierte Navigation
- Terminplanung-Feature für Kalender-Seite
- Angebots-Automatisierung für Auftragsgenerierungs-Seiten

---

## Hinweis: Kein eigenständiges Design im Scope

Design (Farben, Layouts, Komponenten-Auswahl) wird kundenseitig via Stitch geliefert.
Nuxt UI wird als Komponenten-Basis verwendet. Abweichungen vom Stitch-Design werden nach Aufwand berechnet.

---

*Erstellt von Wolfi, 2026-07-03*
