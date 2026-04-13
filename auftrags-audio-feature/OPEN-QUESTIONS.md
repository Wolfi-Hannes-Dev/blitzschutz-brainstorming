# Offene Fragen — Auftrags-Audio-Feature

Stand: 2026-04-13

## Alle Fragen geklärt ✅

| Frage | Entscheidung |
|-------|-------------|
| Max-Aufnahmelänge | **2 Minuten** |
| Audio-Format | **WebM/Opus** |
| UX nach Aufnahme | **✅ Häkchen** — Auswertung im Back-Office |
| Self-Hosted Transkription | **speaches** + `large-v3-turbo` |
| Server-Hardware | AMD EPYC 8 Kerne, 16 GB RAM, kein GPU |
| LLM für Extraktion | **GPT-4o-mini** oder Anthropic claude-haiku |
| Employee-Mapping | **Ja** — eingeloggter User als `employee_id` |
| Transkript im Back-Office | **Ja** — sichtbar für Debugging |
| DB-Tabelle Construction Sites | `construction_sites` ✓ (aus bestehendem Code bestätigt) |
