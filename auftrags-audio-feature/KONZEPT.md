# 🎙️ Auftrags-Audio-Feature — Konzept

**Projekt:** Blitzschutz Reichenhauser  
**Feature:** Audio-Aufnahme pro Baustelle → automatische Auftragserfassung  
**Datum:** 2026-04-13  
**Status:** Konzeptphase 🚧

---

## Idee (Zusammenfassung)

Zu jeder Baustelle im Frontend gibt es einen 🎙️-Button.  
Ein Vorarbeiter drückt drauf, spricht einen Auftrag ein → das System transkribiert die Aufnahme und extrahiert automatisch die relevanten Auftragsdaten.

Am Ende entsteht eine **Pre-Filled URL** fürs Back-Office — analog zu den bestehenden **Phone Requests**.

---

## Architektur-Überblick

```
[Frontend – Vorarbeiter-App]
  └── 🎙️ Mikrofon-Button pro Baustelle
      └── Web Audio API → MediaRecorder (WebM/Opus)
          └── POST /api/voice-requests
                ├── audio: File (WebM/Opus, max 2 min)
                └── constructionSiteId: string (UUID)

[Backend – Middleware]
  └── POST /api/voice-requests
      ├── 1. Validierung (Größe, Format)
      ├── 2. Audio → Transcription Service (self-hosted Whisper)
      ├── 3. Transkript + Baustellen-Kontext → LLM → MicrosoftProject-Felder
      └── 4. Generic Request erstellen → Pre-Filled MS Forms URL
          └── Response: { requestId, status: "ok" }
              ← Frontend zeigt nur ✅ (Backend verarbeitet weiter)

[Back-Office]
  └── Sieht neue "Voice Requests" in der Liste
      └── Öffnet Pre-Filled URL → MS Forms vorausgefüllt
```

---

## Frontend

### Änderungen an der Baustellen-Ansicht

- Mikrofon-Icon-Button zu jeder Baustellen-Karte
- **Aufnahme-Flow:**
  1. Button drücken → Aufnahme startet (visuelles Feedback: pulsierendes Icon + Timer)
  2. Max-Länge erreicht (120s) oder Button erneut drücken → Aufnahme stoppt
  3. Audio-Blob wird als `multipart/form-data` + `constructionSiteId` ans Backend geschickt
  4. Frontend zeigt **✅ grünes Häkchen** sobald Backend `200 OK` zurückgibt
  5. Fertig — die Auswertung passiert im Back-Office

### Kein Preview nötig
Der Vorarbeiter sieht nur das Häkchen. Die inhaltliche Auswertung des Transkripts findet im Back-Office statt (andere Accounts).

### Audio-Format
- **WebM/Opus** (Browser-Default von `MediaRecorder`) — kein Konvertierungsschritt nötig
- Max-Länge: **2 Minuten** (120s), wird per Timer im Frontend durchgesetzt

---

## Backend

### Neuer Endpoint

```
POST /api/voice-requests
Content-Type: multipart/form-data

Fields:
  - audio:              binary  (WebM/Opus, max ~5MB)
  - constructionSiteId: string  (UUID der Baustelle)
```

**Response (sofort):**
```json
{ "requestId": 42, "status": "ok" }
```

### Verarbeitungs-Pipeline

```
1. Validierung
   ├── constructionSiteId vorhanden & existiert
   ├── audio-File vorhanden, max 5MB
   └── Format-Check (Content-Type)

2. Transkription
   └── POST http://localhost:9000/v1/audio/transcriptions
       ├── file: audio.webm
       ├── model: whisper-large-v3
       └── language: de

3. Extraktion (LLM)
   ├── Prompt: Transkript + Baustellen-Info → JSON mit MicrosoftProject-Feldern
   └── Output: Partial<MicrosoftProject>

4. Voice Request erstellen (analog zu Phone Requests)
   ├── DB-Eintrag: voice_requests-Tabelle
   └── predefined_url via buildMSFormsUrl()
```

### Refactoring: GenericRequest

Die bestehende Phone-Request-Logik wird auf **Generic Requests** verallgemeinert:

```typescript
type RequestSource = 'phone' | 'voice';  // erweiterbar

interface GenericRequest {
  source: RequestSource;
  construction_site_id?: number;  // neu: bei Voice immer vorhanden
  // ... bestehende PhoneRequest-Felder
}
```

`phoneRequestService.ts` → `requestService.ts` (oder `createRequest(source, params)`)

---

## Extraktions-Felder (aus `MicrosoftProject`)

Alle Felder werden per LLM aus dem Transkript extrahiert — soweit erkennbar:

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `betreff` | string | Worum geht's (z.B. "Neue Erdung") |
| `gebäudeteil` | string | Welcher Teil des Gebäudes |
| `terminNachVereinbarung` | boolean | Termin noch offen? |
| `isPlanVorhanden` | boolean | Ist ein Plan vorhanden? |
| `ansprechpartnerAuftraggeber` | string | Kontaktperson Auftraggeber |
| `telefonAnsprechpartner` | string | Telefon Ansprechpartner |
| `ansprechpartnerVorOrt` | string | Wer ist vor Ort |
| `notiz` | string | Sonstiges / Freitext |
| `dringlichkeit` | boolean | Dringend? |
| `leiterNotwendig` | boolean | Braucht man eine Leiter? |
| `zutritt` | string | Wie kommt man rein? |

**Nicht per Voice erfassbar** (werden aus Kontext/System befüllt):
- `datum` → automatisch (heute)
- `name`, `tel` → aus Baustellen-Daten
- `prüfer` → aus eingeloggtem User
- `auftragsdatum` → automatisch
- `straßeKoordinaten` → aus Baustellen-Adresse
- `wetter` → optional, kann weggelassen werden

### LLM-Prompt (Entwurf)

```
Du bist ein Assistent für das Elektrounternehmen Blitzschutz Reichenhauser.

Baustelle: {folder_name}
Adresse: {straße}, {plz} {ort}

Ein Vorarbeiter hat folgenden Auftrag eingesprochen:
"{transkript}"

Extrahiere alle erkennbaren Informationen als JSON. Felder die nicht erkennbar sind, weglassen.

Antworte NUR mit validem JSON (kein Markdown, keine Erklärung):
{
  "betreff": "...",
  "gebäudeteil": "...",
  "terminNachVereinbarung": true/false,
  "isPlanVorhanden": true/false,
  "ansprechpartnerAuftraggeber": "...",
  "telefonAnsprechpartner": "...",
  "ansprechpartnerVorOrt": "...",
  "notiz": "...",
  "dringlichkeit": true/false,
  "leiterNotwendig": true/false,
  "zutritt": "..."
}
```

---

## Transkriptions-Service (Self-Hosted)

### Empfehlung: `speaches`

- **Repo:** https://github.com/speaches-ai/speaches
- OpenAI-kompatible API (`POST /v1/audio/transcriptions`)
- Powered by `faster-whisper` (CTranslate2 — optimiert für CPU)
- Drop-in Ersatz für OpenAI Whisper API → **kein Code-Umbau** nötig
- Docker-deploybar, interner Port (nicht nach außen exponiert)

### Server-Hardware (VPS)

```
CPU:  AMD EPYC™ 9645 — 8 dedizierte Kerne
RAM:  16 GB DDR5 ECC
SSD:  1024 GB
GPU:  keine → CPU-Inference
```

### Modell-Empfehlung

Mit 8 EPYC-Kernen ist **`large-v3`** oder **`turbo` (large-v3-turbo)** machbar:

| Modell | Größe | Qualität Deutsch | CPU-Zeit (60s Audio) |
|--------|-------|-----------------|----------------------|
| `base` | 145 MB | ausreichend | ~3-5s |
| `small` | 461 MB | gut | ~8-15s |
| `medium` | 1.5 GB | sehr gut | ~20-40s |
| `large-v3` | 3 GB | exzellent | ~60-90s |
| `large-v3-turbo` | 1.6 GB | exzellent | ~30-50s |

**Empfehlung: `large-v3-turbo`** — beste Balance aus Qualität (Fachbegriffe, Dialekt) und Geschwindigkeit auf CPU.

### Docker Compose (Entwurf)

```yaml
services:
  speaches:
    image: ghcr.io/speaches-ai/speaches:latest-cpu
    container_name: transcription
    restart: unless-stopped
    ports:
      - "127.0.0.1:9000:8000"  # nur lokal erreichbar
    environment:
      - DEFAULT_MODEL=Systran/faster-whisper-large-v3-turbo
      - DEFAULT_LANGUAGE=de
    volumes:
      - speaches_models:/home/user/.cache/huggingface
    deploy:
      resources:
        limits:
          memory: 8G

volumes:
  speaches_models:
```

---

## Maximale Aufnahmelänge

**Entscheidung: 2 Minuten (120s)**

| Länge | Dateigröße (WebM/Opus ~32kbps) | Transkription (large-v3-turbo, CPU) |
|-------|-------------------------------|--------------------------------------|
| 30s   | ~120 KB                       | ~15-25s                              |
| 60s   | ~240 KB                       | ~25-40s                              |
| 120s  | ~480 KB                       | ~50-80s                              |

- Frontend: Auto-stop + visueller Timer nach 120s
- Backend: Reject bei > 5 MB Dateigröße

---

## Neue DB-Tabelle: `voice_requests`

```sql
CREATE TABLE IF NOT EXISTS voice_requests (
  id                    INT AUTO_INCREMENT PRIMARY KEY,
  construction_site_id  INT NOT NULL,
  employee_id           INT NULL,
  employee_name         VARCHAR(255) NULL,
  transcript            TEXT NULL,
  extracted_data        JSON NULL,
  betreff               VARCHAR(255) NULL,
  ansprechpartner_vor_ort VARCHAR(255) NULL,
  notizen               TEXT NULL,
  predefined_url        TEXT NOT NULL,
  done                  TINYINT(1) DEFAULT 0,
  created_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at            TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  done_at               TIMESTAMP NULL,

  INDEX idx_done (done),
  INDEX idx_construction_site_id (construction_site_id),
  INDEX idx_created_at (created_at),

  FOREIGN KEY (construction_site_id) REFERENCES construction_sites(id),
  FOREIGN KEY (employee_id) REFERENCES employees(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## Offene Fragen

- [ ] Welche Tabelle heißt `construction_sites` in der DB? (Ggf. Schema checken)
- [ ] Soll der eingeloggte User als `employee` gemapped werden (wie bei Phone Requests)?
- [ ] Welches LLM soll für die Extraktion verwendet werden — GPT-4o-mini oder ein lokales Modell?
- [ ] Soll das Transkript auch im Back-Office sichtbar sein (für Debugging)?

---

## Nächste Schritte

- [ ] speaches Docker-Setup auf VPS testen
- [ ] DB-Migration schreiben (`voice_requests`-Tabelle)
- [ ] `voiceRequestService.ts` implementieren (analog zu `phoneRequestService.ts`)
- [ ] Backend-Endpoint `POST /api/voice-requests`
- [ ] Frontend Mikrofon-UI + Häkchen-Feedback
- [ ] End-to-End Test mit echtem Audio
