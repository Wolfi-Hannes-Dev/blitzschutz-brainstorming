# 🎙️ Auftrags-Audio-Feature — Konzept

**Projekt:** Blitzschutz Reichenhauser  
**Feature:** Audio-Aufnahme pro Baustelle → automatische Auftragserfassung  
**Datum:** 2026-04-13  
**Status:** Konzept finalisiert ✅ — bereit zur Implementierung

---

## Idee (Zusammenfassung)

Zu jeder Baustelle im Frontend gibt es einen 🎙️-Button.  
Ein Vorarbeiter drückt drauf, spricht einen Auftrag ein → das System transkribiert die Aufnahme, extrahiert via LLM die relevanten Auftragsdaten und erstellt einen **Voice Request** mit Pre-Filled MS Forms URL — analog zu den bestehenden **Phone Requests** vom vapi.ai-Telefonagenten.

---

## Architektur-Überblick

```
[Frontend – Vorarbeiter-App]
  └── 🎙️ Mikrofon-Button pro Baustelle
      └── MediaRecorder → WebM/Opus Blob (max 2 min)
          └── POST /api/v1/voice-requests
                ├── audio:              File (WebM/Opus)
                └── constructionSiteId: number

[Backend – Middleware]
  └── POST /api/v1/voice-requests  (Bearer Token Auth, wie alle anderen Endpoints)
      ├── 1. Validierung (constructionSiteId existiert, Dateigröße ≤ 5 MB)
      ├── 2. Audio → self-hosted Whisper API (speaches, intern: localhost:9000)
      ├── 3. Transkript + Baustellen-Kontext → LLM → MicrosoftProject-Felder
      ├── 4. voice_requests-Eintrag in DB speichern (inkl. transcript, extracted_data)
      └── 5. buildMSFormsUrl() → predefined_url
          └── Response: { requestId, status: "ok" }

[Back-Office]
  └── Sieht neue Voice Requests in der Liste (neben Phone Requests)
      ├── Transkript sichtbar (für Debugging)
      └── Öffnet Pre-Filled URL → MS Forms vorausgefüllt
```

---

## Frontend

### Änderungen an der Baustellen-Ansicht

- Mikrofon-Icon-Button zu jeder Baustellen-Karte
- **Aufnahme-Flow:**
  1. Button drücken → Aufnahme startet (`MediaRecorder` API, WebM/Opus)
  2. Visuelles Feedback: pulsierendes Icon + Timer (MM:SS)
  3. Stop: Button erneut drücken **oder** automatisch nach 120s
  4. `multipart/form-data` POST ans Backend:
     - `audio`: Blob (WebM/Opus)
     - `constructionSiteId`: ID der Baustelle
  5. **✅ Grünes Häkchen** bei `200 OK` — fertig, keine Vorschau nötig

### Permissions
- Browser fragt automatisch nach Mikrofon-Erlaubnis (`getUserMedia`)
- Fehlerfall (Permission denied, kein Mikrofon) → klare Fehlermeldung anzeigen

### Audio-Format
- **WebM/Opus** — Browser-Default, kein Konvertierungsschritt nötig
- Max-Länge: **120 Sekunden** (Frontend-Timer + auto-stop)

---

## Backend

### Neuer Endpoint

```
POST /api/v1/voice-requests
Authorization: Bearer <API_TOKEN>
Content-Type: multipart/form-data

Fields:
  - audio:              binary  (WebM/Opus, max 5 MB)
  - constructionSiteId: number
```

**Response (sofort nach DB-Eintrag):**
```json
{ "success": true, "data": { "requestId": 42, "status": "ok" } }
```

### Verarbeitungs-Pipeline

```typescript
// 1. Validierung
const site = await getConstructionSiteById(constructionSiteId);
if (!site) throw new Error('Baustelle nicht gefunden');
if (audioFile.size > 5_000_000) throw new Error('Audio zu groß (max 5 MB)');

// 2. Transkription (self-hosted Whisper, OpenAI-kompatibler Endpoint)
const transcript = await transcribeAudio(audioFile);  // POST http://localhost:9000/v1/audio/transcriptions

// 3. LLM-Extraktion
const extracted = await extractProjectData(transcript, site);  // GPT-4o-mini oder claude-haiku

// 4. MS Forms URL
const predefinedUrl = buildMSFormsUrl({
  betreff: extracted.betreff || 'Voice-Auftrag',
  baustelle: site.folder_name,
  pruefer: employeeName,  // aus eingeloggtem User
});

// 5. DB speichern
await createVoiceRequest({ constructionSiteId, transcript, extracted, predefinedUrl, employeeId });
```

### Authentifizierung / User-Mapping
- Wie alle anderen Endpoints: `Bearer <API_TOKEN>` via `authenticateApiToken` Middleware
- **Employee-Mapping:** Der eingeloggte User soll als `employee_id` gespeichert werden  
  → Das Frontend muss beim Request den `employeeId` mitschicken (oder der Token wird dem Employee zugeordnet — abhängig vom bestehenden Auth-System)

### Refactoring: PhoneRequest → GenericRequest
Die `phoneRequestService.ts`-Logik wird auf eine gemeinsame Basis gehoben:

```typescript
// src/services/requestService.ts (neu)
type RequestSource = 'phone' | 'voice';

interface CreateRequestParams {
  source: RequestSource;
  constructionSiteId?: number;  // pflicht bei voice
  callerPhone?: string;         // bei phone
  employeeId?: number;          // bei voice
  betreff?: string;
  adresse?: string;
  ort?: string;
  ansprechpartnerVorOrt?: string;
  notizen?: string;
  transcript?: string;          // bei voice
  extractedData?: object;       // bei voice
}
```

---

## LLM-Extraktion

### Modell
- **GPT-4o-mini** (OpenAI) oder **claude-haiku** (Anthropic) — beide ausreichend für strukturierte Extraktion
- Empfehlung: `gpt-4o-mini` — günstiger, schneller, gut für DE-Text

### Extrahierbare Felder (aus `MicrosoftProject` Interface)

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `betreff` | string | Was ist zu tun? (z.B. "Neue Erdung") |
| `gebäudeteil` | string | Welcher Teil des Gebäudes |
| `terminNachVereinbarung` | boolean | Termin noch offen? |
| `isPlanVorhanden` | boolean | Ist ein Plan vorhanden? |
| `ansprechpartnerAuftraggeber` | string | Kontaktperson Auftraggeber |
| `telefonAnsprechpartner` | string | Telefonnummer Ansprechpartner |
| `ansprechpartnerVorOrt` | string | Wer ist vor Ort |
| `notiz` | string | Sonstiges / Freitext |
| `dringlichkeit` | boolean | Dringend? |
| `leiterNotwendig` | boolean | Braucht man eine Leiter? |
| `zutritt` | string | Wie kommt man rein? |

**Automatisch befüllt (nicht vom LLM):**
- `datum` → heute
- `auftragsdatum` → heute  
- `name`, `straßeKoordinaten` → aus `ConstructionSite`
- `prüfer` → aus eingeloggtem Employee

### LLM-Prompt

```
Du bist ein Assistent für das Elektrounternehmen Blitzschutz Reichenhauser.

Baustelle: {{folder_name}}
Adresse: {{straße}}, {{plz}} {{ort}}

Ein Vorarbeiter hat folgenden Auftrag per Spracheingabe erfasst:
"{{transkript}}"

Extrahiere alle erkennbaren Informationen als JSON.
Felder die nicht erkennbar sind, WEGLASSEN (nicht null setzen).
Antworte NUR mit validem JSON, ohne Markdown-Codeblock:

{
  "betreff": "...",
  "gebäudeteil": "...",
  "terminNachVereinbarung": true,
  "isPlanVorhanden": false,
  "ansprechpartnerAuftraggeber": "...",
  "telefonAnsprechpartner": "...",
  "ansprechpartnerVorOrt": "...",
  "notiz": "...",
  "dringlichkeit": false,
  "leiterNotwendig": true,
  "zutritt": "..."
}
```

---

## Transkriptions-Service (Self-Hosted)

### Empfehlung: `speaches`

| | Details |
|--|---------|
| Repo | https://github.com/speaches-ai/speaches |
| API | OpenAI-kompatibel: `POST /v1/audio/transcriptions` |
| Backend | `faster-whisper` (CTranslate2) |
| Deployment | Docker Compose, interner Port |
| Drop-in | ✅ Kein Code-Umbau wenn später wieder OpenAI API genutzt wird |

### Server (VPS)

```
CPU:  AMD EPYC™ 9645 — 8 dedizierte Kerne
RAM:  16 GB DDR5 ECC
GPU:  keine → CPU-Inference
```

### Modell-Empfehlung: `large-v3-turbo`

| Modell | Größe | Qualität DE | CPU-Zeit (60s Audio) |
|--------|-------|-------------|----------------------|
| `small` | 461 MB | gut | ~8-15s |
| `medium` | 1.5 GB | sehr gut | ~20-40s |
| `large-v3` | 3 GB | exzellent | ~60-90s |
| **`large-v3-turbo`** | **1.6 GB** | **exzellent** | **~30-50s** ← empfohlen |

→ Beste Balance aus Qualität (Fachbegriffe, österreichischer Dialekt) und Geschwindigkeit auf CPU.

### Docker Compose

```yaml
services:
  speaches:
    image: ghcr.io/speaches-ai/speaches:latest-cpu
    container_name: transcription
    restart: unless-stopped
    ports:
      - "127.0.0.1:9000:8000"   # nur lokal erreichbar, kein Public-Exposure
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

Erster Start lädt das Modell (~1.6 GB) herunter und cached es im Volume.

---

## Back-Office: Audio-Player

**Anforderung:** Das gespeicherte Audio soll im Back-Office abspielbar sein — ähnlich dem vapi.ai Recording-Player (Waveform-Visualisierung + Play-Button).

### Was gespeichert wird
- Das Audio-File (WebM/Opus) wird **serverseitig gespeichert** (z.B. im Dateisystem oder S3/Object Storage)
- In der DB: `audio_path` oder `audio_url` als Referenz

### Back-Office UI (Referenz: vapi.ai Screenshot)
```
┌─────────────────────────────────────────────────┐
│  🎙️ Voice Request #42                           │
│  Baustelle: Troststraße 50, 1100 Wien           │
│  Vorarbeiter: Max Mustermann  │  13.04.2026     │
├─────────────────────────────────────────────────┤
│  ▶  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  2:00  │  ← Waveform-Player
├─────────────────────────────────────────────────┤
│  📝 Transkript:                                  │
│  "Neue Erdung im Keller, Plan ist vorhanden..." │
├─────────────────────────────────────────────────┤
│  [🔗 MS Forms öffnen]           [✅ Erledigt]   │
└─────────────────────────────────────────────────┘
```

### Technische Umsetzung
- **Waveform:** [wavesurfer.js](https://wavesurfer.xyz/) — leichtgewichtig, kein Backend nötig, rendert WebM/Opus direkt im Browser
- **Audio-Storage:** Audio-File serverseitig unter `/uploads/voice-requests/{id}.webm` speichern
- **Audio-Endpoint:** `GET /api/v1/voice-requests/:id/audio` → streamt das File (mit Auth)
- Das Back-Office lädt die Audio-URL und wavesurfer.js rendert die Waveform + Player on-the-fly

### Backend-Ergänzung
```typescript
// Zusätzlicher Endpoint
GET /api/v1/voice-requests/:id/audio
→ res.sendFile(audioPath)  // mit Auth-Check
```

---

## Neue DB-Tabelle: `voice_requests`

```sql
CREATE TABLE IF NOT EXISTS voice_requests (
  id                      INT AUTO_INCREMENT PRIMARY KEY,
  construction_site_id    INT NOT NULL,
  employee_id             INT NULL,
  employee_name           VARCHAR(255) NULL,
  audio_path              VARCHAR(500) NULL,   -- Pfad zur gespeicherten Audio-Datei
  transcript              TEXT NULL,           -- Whisper-Output, sichtbar im Back-Office
  extracted_data          JSON NULL,           -- vollständiges LLM-Output
  betreff                 VARCHAR(255) NULL,
  ansprechpartner_vor_ort VARCHAR(255) NULL,
  notizen                 TEXT NULL,
  predefined_url          TEXT NOT NULL,
  done                    TINYINT(1) DEFAULT 0,
  created_at              TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at              TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  done_at                 TIMESTAMP NULL,

  INDEX idx_done (done),
  INDEX idx_construction_site_id (construction_site_id),
  INDEX idx_created_at (created_at),

  FOREIGN KEY (construction_site_id) REFERENCES construction_sites(id),
  FOREIGN KEY (employee_id) REFERENCES employees(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## Maximale Aufnahmelänge

**Entscheidung: 2 Minuten (120s)**

| Länge | Dateigröße (WebM/Opus ~32 kbps) | Transkription (large-v3-turbo, CPU) |
|-------|---------------------------------|--------------------------------------|
| 30s | ~120 KB | ~15-25s |
| 60s | ~240 KB | ~25-40s |
| **120s** | **~480 KB** | **~50-80s** |

- Frontend: Timer + auto-stop nach 120s
- Backend: Reject bei `audio.size > 5_000_000`

---

## Neue Dateien (Implementierung)

```
src/
├── api/
│   ├── controllers/
│   │   └── voiceRequestController.ts    (neu)
│   ├── routes/
│   │   └── voiceRequestRoutes.ts        (neu)
│   └── models/
│       └── VoiceRequest.ts              (neu, Interface)
└── services/
    ├── voiceRequestService.ts           (neu, analog phoneRequestService)
    └── transcriptionService.ts          (neu, Whisper API Client)
```

**Änderungen bestehender Dateien:**
- `src/server.ts` → neue Route einbinden
- `src/services/msFormsService.ts` → keine Änderung nötig (buildMSFormsUrl bleibt)
- `src/config/database.ts` → Migration für `voice_requests`-Tabelle

---

## Entscheidungslog

| Frage | Entscheidung |
|-------|-------------|
| Max-Aufnahmelänge | **2 Minuten** |
| Audio-Format | **WebM/Opus** (Browser-Default) |
| UX nach Aufnahme | **✅ Häkchen** — keine Vorschau, Auswertung im Back-Office |
| Transkription | **self-hosted speaches** (`large-v3-turbo`) |
| LLM Extraktion | **GPT-4o-mini** (oder claude-haiku) |
| Employee-Mapping | **Ja** — eingeloggter User wird als `employee_id` gespeichert |
| Transkript im Back-Office | **Ja** — sichtbar für Debugging |
| Audio im Back-Office | **Ja** — Waveform-Player (wavesurfer.js), Audio serverseitig gespeichert |
| Server | VPS: AMD EPYC 8 Kerne, 16 GB RAM, kein GPU |

---

## Nächste Schritte

- [ ] speaches auf VPS deployen & testen (Docker Compose)
- [ ] DB-Migration (`voice_requests`-Tabelle, inkl. `audio_path`)
- [ ] `transcriptionService.ts` implementieren
- [ ] `voiceRequestService.ts` implementieren (inkl. Audio-File speichern)
- [ ] Backend-Endpoint `POST /api/v1/voice-requests`
- [ ] Backend-Endpoint `GET /api/v1/voice-requests/:id/audio`
- [ ] Frontend: Mikrofon-Button + Recording-UI + Häkchen-Feedback
- [ ] Back-Office: wavesurfer.js Audio-Player + Transkript-Anzeige
- [ ] End-to-End Test mit echtem Audio
