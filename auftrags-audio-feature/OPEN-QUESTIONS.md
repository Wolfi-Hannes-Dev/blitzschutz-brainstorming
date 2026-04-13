# Offene Fragen — Auftrags-Audio-Feature

Stand: 2026-04-13

## Entschieden ✅

| Frage | Entscheidung |
|-------|-------------|
| Max-Aufnahmelänge | **2 Minuten** |
| Audio-Format | **WebM/Opus** (Browser-Default, kein Konvertierungsschritt) |
| UX nach Aufnahme | **Nur ✅ Häkchen** — keine Vorschau, Auswertung im Back-Office |
| Transkription | **Self-hosted**, OpenAI-kompatible API |
| Server | VPS: AMD EPYC 8 Kerne, 16GB RAM, kein GPU |
| Empfohlener Service | **speaches** + `large-v3-turbo` Modell |

## Noch offen ❓

1. **LLM für Extraktion** — GPT-4o-mini (Cloud) oder lokales Modell?
2. **Eingeloggter User** — soll er als `employee_id` gemappt werden, wie bei Phone Requests?
3. **Transkript im Back-Office sichtbar?** — hilfreich fürs Debugging
4. **DB-Schema** — exakter Name der `construction_sites`-Tabelle bestätigen
