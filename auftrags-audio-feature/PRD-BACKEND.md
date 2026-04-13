# PRD — Backend: Voice Request Feature

**Projekt:** Blitzschutz Reichenhauser Middleware  
**Feature:** Audio-Aufnahme → Auftragserfassung (Voice Requests)  
**Datum:** 2026-04-13  
**Status:** Ready for Development

---

## Ziel

Ein Vorarbeiter spricht per App einen Auftrag ein. Das Backend empfängt das Audio, transkribiert es via self-hosted Whisper, extrahiert via LLM die Auftragsdaten und erstellt einen **Voice Request** — analog zur bestehenden Phone-Request-Logik vom vapi.ai-Telefonagenten.

---

## Neue Dateien

```
src/
├── api/
│   ├── controllers/
│   │   └── voiceRequestController.ts      ← neu
│   ├── routes/
│   │   └── voiceRequestRoutes.ts          ← neu
│   └── models/
│       └── VoiceRequest.ts                ← neu
└── services/
    ├── voiceRequestService.ts             ← neu
    └── transcriptionService.ts            ← neu

migrations/
└── 002_create_voice_requests.sql          ← neu

uploads/
└── voice-requests/                        ← neu (Audio-Dateien)
```

**Geänderte Dateien:**
- `src/server.ts` — neue Route mounten
- `package.json` — `multer` dependency hinzufügen

---

## Dependencies

```bash
npm install multer
npm install --save-dev @types/multer
```

`multer` für `multipart/form-data` File-Uploads (Audio).

---

## 1. DB-Migration

**Datei:** `migrations/002_create_voice_requests.sql`

```sql
-- Migration: Create voice_requests table
-- Date: 2026-04-13
-- Description: Tabelle für Voice-Aufträge (Mikrofon-Feature im Frontend)

CREATE TABLE IF NOT EXISTS voice_requests (
  id                      INT AUTO_INCREMENT PRIMARY KEY,
  construction_site_id    INT NOT NULL            COMMENT 'Zugehörige Baustelle',
  employee_id             INT NULL                COMMENT 'Eingeloggter Mitarbeiter',
  employee_name           VARCHAR(255) NULL        COMMENT 'Name des Mitarbeiters',
  audio_path              VARCHAR(500) NULL        COMMENT 'Pfad zur gespeicherten WebM-Datei',
  transcript              TEXT NULL               COMMENT 'Whisper-Transkript (für Back-Office Debug)',
  extracted_data          JSON NULL               COMMENT 'Vollständiges LLM-Output als JSON',
  betreff                 VARCHAR(255) NULL        COMMENT 'Extrahierter Betreff',
  ansprechpartner_vor_ort VARCHAR(255) NULL        COMMENT 'Extrahierter Ansprechpartner',
  notizen                 TEXT NULL               COMMENT 'Extrahierte Notizen / Freitext',
  predefined_url          TEXT NOT NULL           COMMENT 'Pre-Filled MS Forms URL',
  done                    TINYINT(1) DEFAULT 0    COMMENT 'Erledigt ja/nein',
  created_at              TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at              TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  done_at                 TIMESTAMP NULL          COMMENT 'Zeitpunkt der Erledigung',

  INDEX idx_done (done),
  INDEX idx_construction_site_id (construction_site_id),
  INDEX idx_employee_id (employee_id),
  INDEX idx_created_at (created_at),

  FOREIGN KEY (construction_site_id) REFERENCES construction_sites(id),
  FOREIGN KEY (employee_id) REFERENCES employees(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 2. Model / Types

**Datei:** `src/api/models/VoiceRequest.ts`

```typescript
export interface VoiceRequest {
  id?: number;
  construction_site_id: number;
  employee_id?: number;
  employee_name?: string;
  audio_path?: string;
  transcript?: string;
  extracted_data?: Partial<ExtractedProjectData>;
  betreff?: string;
  ansprechpartner_vor_ort?: string;
  notizen?: string;
  predefined_url: string;
  done: boolean;
  created_at?: string;
  updated_at?: string;
  done_at?: string;
}

export interface ExtractedProjectData {
  betreff?: string;
  gebäudeteil?: string;
  terminNachVereinbarung?: boolean;
  isPlanVorhanden?: boolean;
  ansprechpartnerAuftraggeber?: string;
  telefonAnsprechpartner?: string;
  ansprechpartnerVorOrt?: string;
  notiz?: string;
  dringlichkeit?: boolean;
  leiterNotwendig?: boolean;
  zutritt?: string;
}

export interface CreateVoiceRequestParams {
  constructionSiteId: number;
  employeeId?: number;
  employeeName?: string;
  audioPath?: string;
  transcript?: string;
  extractedData?: Partial<ExtractedProjectData>;
}
```

---

## 3. Transcription Service

**Datei:** `src/services/transcriptionService.ts`

Ruft die self-hosted Whisper API (speaches, OpenAI-kompatibel) auf.

```typescript
import fs from 'fs';
import FormData from 'form-data';
import fetch from 'node-fetch';
import logger from '../utils/logger';

const TRANSCRIPTION_URL = process.env.TRANSCRIPTION_API_URL || 'http://localhost:9000';
const WHISPER_MODEL = process.env.WHISPER_MODEL || 'Systran/faster-whisper-large-v3-turbo';

export const transcribeAudio = async (audioFilePath: string): Promise<string> => {
  const form = new FormData();
  form.append('file', fs.createReadStream(audioFilePath), {
    filename: 'audio.webm',
    contentType: 'audio/webm',
  });
  form.append('model', WHISPER_MODEL);
  form.append('language', 'de');
  form.append('response_format', 'text');

  const response = await fetch(`${TRANSCRIPTION_URL}/v1/audio/transcriptions`, {
    method: 'POST',
    body: form,
    headers: form.getHeaders(),
  });

  if (!response.ok) {
    const body = await response.text();
    logger.error(`Transcription API error: ${response.status} ${body}`);
    throw new Error(`Transcription failed: ${response.status}`);
  }

  const transcript = await response.text();
  logger.info(`Transcription successful (${transcript.length} chars)`);
  return transcript.trim();
};
```

**Env-Variablen:**
```env
TRANSCRIPTION_API_URL=http://localhost:9000
WHISPER_MODEL=Systran/faster-whisper-large-v3-turbo
```

---

## 4. Voice Request Service

**Datei:** `src/services/voiceRequestService.ts`

Enthält gesamte Business-Logik: Extraktion, DB-Schreiben, URL-Generierung.

```typescript
import pool from '../config/database';
import { RowDataPacket, ResultSetHeader } from 'mysql2';
import { transcribeAudio } from './transcriptionService';
import { buildMSFormsUrl } from './msFormsService';
import { getConstructionSiteById } from './constructionSiteService';
import { getEmployeeById } from './employeeService';
import logger from '../utils/logger';
import OpenAI from 'openai';
import type { VoiceRequest, CreateVoiceRequestParams, ExtractedProjectData } from '../api/models/VoiceRequest';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// --- LLM Extraktion ---

export const extractProjectData = async (
  transcript: string,
  siteName: string,
  siteAddress: string
): Promise<Partial<ExtractedProjectData>> => {
  const prompt = `Du bist ein Assistent für das Elektrounternehmen Blitzschutz Reichenhauser.

Baustelle: ${siteName}
Adresse: ${siteAddress}

Ein Vorarbeiter hat folgenden Auftrag per Spracheingabe erfasst:
"${transcript}"

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
}`;

  const completion = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [{ role: 'user', content: prompt }],
    temperature: 0.1,
    max_tokens: 500,
  });

  const raw = completion.choices[0]?.message?.content || '{}';
  
  try {
    return JSON.parse(raw) as Partial<ExtractedProjectData>;
  } catch {
    logger.warn('LLM returned invalid JSON, using empty extraction', { raw });
    return {};
  }
};

// --- DB Operationen ---

export const createVoiceRequest = async (params: CreateVoiceRequestParams): Promise<VoiceRequest> => {
  const { constructionSiteId, employeeId, employeeName, audioPath, transcript, extractedData } = params;

  const site = await getConstructionSiteById(constructionSiteId);
  if (!site) throw new Error(`Construction site ${constructionSiteId} not found`);

  const predefinedUrl = buildMSFormsUrl({
    betreff: extractedData?.betreff || 'Voice-Auftrag',
    baustelle: site.folder_name,
    pruefer: employeeName || 'Voice-Auftrag',
    adresse: site.straße,
  });

  const [result] = await pool.execute<ResultSetHeader>(
    `INSERT INTO voice_requests
     (construction_site_id, employee_id, employee_name, audio_path, transcript, extracted_data,
      betreff, ansprechpartner_vor_ort, notizen, predefined_url, done)
     VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, FALSE)`,
    [
      constructionSiteId,
      employeeId || null,
      employeeName || null,
      audioPath || null,
      transcript || null,
      extractedData ? JSON.stringify(extractedData) : null,
      extractedData?.betreff || null,
      extractedData?.ansprechpartnerVorOrt || null,
      extractedData?.notiz || null,
      predefinedUrl,
    ]
  );

  return {
    id: result.insertId,
    construction_site_id: constructionSiteId,
    employee_id: employeeId,
    employee_name: employeeName,
    audio_path: audioPath,
    transcript,
    extracted_data: extractedData,
    predefined_url: predefinedUrl,
    done: false,
  };
};

export const getVoiceRequests = async (onlyOpen = true): Promise<VoiceRequest[]> => {
  const query = onlyOpen
    ? 'SELECT * FROM voice_requests WHERE done = FALSE ORDER BY created_at DESC'
    : 'SELECT * FROM voice_requests ORDER BY created_at DESC';
  const [rows] = await pool.execute<RowDataPacket[]>(query);
  return rows as VoiceRequest[];
};

export const getVoiceRequestById = async (id: number): Promise<VoiceRequest | null> => {
  const [rows] = await pool.execute<RowDataPacket[]>(
    'SELECT * FROM voice_requests WHERE id = ?', [id]
  );
  return rows.length > 0 ? rows[0] as VoiceRequest : null;
};

export const markVoiceRequestDone = async (id: number): Promise<boolean> => {
  const [result] = await pool.execute<ResultSetHeader>(
    'UPDATE voice_requests SET done = TRUE, done_at = NOW() WHERE id = ?', [id]
  );
  return result.affectedRows > 0;
};

export const deleteVoiceRequest = async (id: number): Promise<boolean> => {
  const [result] = await pool.execute<ResultSetHeader>(
    'DELETE FROM voice_requests WHERE id = ?', [id]
  );
  return result.affectedRows > 0;
};
```

---

## 5. Controller

**Datei:** `src/api/controllers/voiceRequestController.ts`

```typescript
import { Request, Response } from 'express';
import path from 'path';
import fs from 'fs';
import logger from '../../utils/logger';
import { transcribeAudio } from '../../services/transcriptionService';
import { extractProjectData, createVoiceRequest, getVoiceRequests,
         getVoiceRequestById, markVoiceRequestDone, deleteVoiceRequest } from '../../services/voiceRequestService';
import { getConstructionSiteById } from '../../services/constructionSiteService';
import { getEmployeeById } from '../../services/employeeService';

const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5 MB

/**
 * POST /api/v1/voice-requests
 * Empfängt Audio + constructionSiteId, verarbeitet und speichert Voice Request
 */
export const create = async (req: Request, res: Response): Promise<void> => {
  const audioFile = req.file;

  try {
    // 1. Validierung
    const constructionSiteId = parseInt(req.body.constructionSiteId, 10);
    const employeeId = req.body.employeeId ? parseInt(req.body.employeeId, 10) : undefined;

    if (isNaN(constructionSiteId)) {
      res.status(400).json({ success: false, error: 'constructionSiteId fehlt oder ungültig' });
      return;
    }
    if (!audioFile) {
      res.status(400).json({ success: false, error: 'Audio-Datei fehlt' });
      return;
    }
    if (audioFile.size > MAX_FILE_SIZE) {
      fs.unlinkSync(audioFile.path);
      res.status(400).json({ success: false, error: 'Audio zu groß (max 5 MB)' });
      return;
    }

    const site = await getConstructionSiteById(constructionSiteId);
    if (!site) {
      fs.unlinkSync(audioFile.path);
      res.status(404).json({ success: false, error: 'Baustelle nicht gefunden' });
      return;
    }

    // Employee-Infos auflösen
    let employeeName: string | undefined;
    if (employeeId) {
      const employee = await getEmployeeById(employeeId);
      employeeName = employee?.email?.split('@')[0] ?? undefined;
    }

    // 2. Sofort antworten (Verarbeitung läuft async weiter)
    res.status(202).json({ success: true, message: 'Audio empfangen, wird verarbeitet' });

    // 3. Transkription (async nach Response)
    const transcript = await transcribeAudio(audioFile.path);

    // 4. LLM-Extraktion
    const siteAddress = [site.straße, site.plz, site.ort].filter(Boolean).join(', ');
    const extractedData = await extractProjectData(transcript, site.folder_name, siteAddress);

    // 5. Voice Request speichern
    await createVoiceRequest({
      constructionSiteId,
      employeeId,
      employeeName,
      audioPath: audioFile.path,
      transcript,
      extractedData,
    });

    logger.info(`Voice request created: site=${constructionSiteId}, employee=${employeeName}`);

  } catch (error) {
    logger.error('Error processing voice request:', error);
    // Audio aufräumen falls vorhanden und Fehler aufgetreten
    if (audioFile?.path && fs.existsSync(audioFile.path)) {
      try { fs.unlinkSync(audioFile.path); } catch {}
    }
  }
};

/**
 * GET /api/v1/voice-requests
 */
export const getAll = async (req: Request, res: Response): Promise<void> => {
  try {
    const showAll = req.query.all === 'true';
    const requests = await getVoiceRequests(!showAll);
    res.json({ success: true, count: requests.length, data: requests });
  } catch (error) {
    logger.error('Error in voiceRequest.getAll:', error);
    res.status(500).json({ success: false, error: 'Fehler beim Abrufen der Voice Requests' });
  }
};

/**
 * GET /api/v1/voice-requests/:id
 */
export const getById = async (req: Request, res: Response): Promise<void> => {
  try {
    const id = parseInt(req.params.id, 10);
    if (isNaN(id)) { res.status(400).json({ success: false, error: 'Ungültige ID' }); return; }
    const request = await getVoiceRequestById(id);
    if (!request) { res.status(404).json({ success: false, error: 'Nicht gefunden' }); return; }
    res.json({ success: true, data: request });
  } catch (error) {
    logger.error('Error in voiceRequest.getById:', error);
    res.status(500).json({ success: false, error: 'Fehler beim Abrufen' });
  }
};

/**
 * GET /api/v1/voice-requests/:id/audio
 * Streamt die Audio-Datei (für den Back-Office Player)
 */
export const streamAudio = async (req: Request, res: Response): Promise<void> => {
  try {
    const id = parseInt(req.params.id, 10);
    if (isNaN(id)) { res.status(400).json({ success: false, error: 'Ungültige ID' }); return; }

    const request = await getVoiceRequestById(id);
    if (!request?.audio_path) {
      res.status(404).json({ success: false, error: 'Audio nicht gefunden' });
      return;
    }

    if (!fs.existsSync(request.audio_path)) {
      res.status(404).json({ success: false, error: 'Audio-Datei nicht gefunden' });
      return;
    }

    res.setHeader('Content-Type', 'audio/webm');
    res.setHeader('Accept-Ranges', 'bytes');
    fs.createReadStream(request.audio_path).pipe(res);
  } catch (error) {
    logger.error('Error in voiceRequest.streamAudio:', error);
    res.status(500).json({ success: false, error: 'Fehler beim Streamen' });
  }
};

/**
 * POST /api/v1/voice-requests/:id/done
 */
export const markDone = async (req: Request, res: Response): Promise<void> => {
  try {
    const id = parseInt(req.params.id, 10);
    if (isNaN(id)) { res.status(400).json({ success: false, error: 'Ungültige ID' }); return; }
    const success = await markVoiceRequestDone(id);
    if (!success) { res.status(404).json({ success: false, error: 'Nicht gefunden' }); return; }
    res.json({ success: true, message: 'Als erledigt markiert' });
  } catch (error) {
    logger.error('Error in voiceRequest.markDone:', error);
    res.status(500).json({ success: false, error: 'Fehler' });
  }
};

/**
 * DELETE /api/v1/voice-requests/:id
 */
export const deleteById = async (req: Request, res: Response): Promise<void> => {
  try {
    const id = parseInt(req.params.id, 10);
    if (isNaN(id)) { res.status(400).json({ success: false, error: 'Ungültige ID' }); return; }
    
    // Audio-File ebenfalls löschen
    const request = await getVoiceRequestById(id);
    if (request?.audio_path && fs.existsSync(request.audio_path)) {
      fs.unlinkSync(request.audio_path);
    }
    
    const success = await deleteVoiceRequest(id);
    if (!success) { res.status(404).json({ success: false, error: 'Nicht gefunden' }); return; }
    res.json({ success: true, message: 'Voice Request gelöscht' });
  } catch (error) {
    logger.error('Error in voiceRequest.deleteById:', error);
    res.status(500).json({ success: false, error: 'Fehler' });
  }
};
```

---

## 6. Routes

**Datei:** `src/api/routes/voiceRequestRoutes.ts`

```typescript
import { Router } from 'express';
import multer from 'multer';
import path from 'path';
import * as voiceRequestController from '../controllers/voiceRequestController';

const router = Router();

// Multer: Audio-Uploads nach uploads/voice-requests/
const storage = multer.diskStorage({
  destination: (_req, _file, cb) => {
    cb(null, path.join(process.cwd(), 'uploads', 'voice-requests'));
  },
  filename: (_req, _file, cb) => {
    const timestamp = Date.now();
    cb(null, `${timestamp}.webm`);
  },
});

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5 MB
  fileFilter: (_req, file, cb) => {
    if (file.mimetype.startsWith('audio/')) {
      cb(null, true);
    } else {
      cb(new Error('Nur Audio-Dateien erlaubt'));
    }
  },
});

// Sicherstellen, dass Upload-Ordner existiert
import fs from 'fs';
const uploadDir = path.join(process.cwd(), 'uploads', 'voice-requests');
if (!fs.existsSync(uploadDir)) fs.mkdirSync(uploadDir, { recursive: true });

router.post('/', upload.single('audio'), voiceRequestController.create);
router.get('/', voiceRequestController.getAll);
router.get('/:id', voiceRequestController.getById);
router.get('/:id/audio', voiceRequestController.streamAudio);
router.post('/:id/done', voiceRequestController.markDone);
router.delete('/:id', voiceRequestController.deleteById);

export default router;
```

---

## 7. Server Integration

**Datei:** `src/server.ts` — ergänzen:

```typescript
import voiceRequestRoutes from './api/routes/voiceRequestRoutes';

// Nach den bestehenden Routes:
app.use('/api/v1/voice-requests', authenticateApiToken, voiceRequestRoutes);
```

---

## API-Übersicht

| Method | Endpoint | Auth | Beschreibung |
|--------|----------|------|-------------|
| `POST` | `/api/v1/voice-requests` | ✅ | Audio + constructionSiteId empfangen |
| `GET` | `/api/v1/voice-requests` | ✅ | Alle Voice Requests (Back-Office) |
| `GET` | `/api/v1/voice-requests/:id` | ✅ | Einzelner Voice Request |
| `GET` | `/api/v1/voice-requests/:id/audio` | ✅ | Audio-Stream für Player |
| `POST` | `/api/v1/voice-requests/:id/done` | ✅ | Als erledigt markieren |
| `DELETE` | `/api/v1/voice-requests/:id` | ✅ | Löschen (inkl. Audio-File) |

### POST `/api/v1/voice-requests` Request

```
Content-Type: multipart/form-data
Authorization: Bearer <API_TOKEN>

Fields:
  audio              File    WebM/Opus, max 5 MB
  constructionSiteId number  Pflichtfeld
  employeeId         number  Optional (eingeloggter User)
```

### Response (202 Accepted)
```json
{ "success": true, "message": "Audio empfangen, wird verarbeitet" }
```
→ Verarbeitung (Transkription + Extraktion) läuft async. Das Frontend muss nicht auf das Ergebnis warten.

---

## Env-Variablen (neu)

```env
TRANSCRIPTION_API_URL=http://localhost:9000
WHISPER_MODEL=Systran/faster-whisper-large-v3-turbo
OPENAI_API_KEY=sk-...          # bereits vorhanden
```

---

## speaches Docker Compose

```yaml
# docker-compose.yml (ergänzen)
services:
  speaches:
    image: ghcr.io/speaches-ai/speaches:latest-cpu
    container_name: transcription
    restart: unless-stopped
    ports:
      - "127.0.0.1:9000:8000"
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

## Implementierungsreihenfolge

1. `npm install multer @types/multer`
2. Migration `002_create_voice_requests.sql` ausführen
3. `uploads/voice-requests/` Ordner anlegen (Route macht das automatisch)
4. `transcriptionService.ts` implementieren + lokal testen (curl gegen speaches)
5. `voiceRequestService.ts` implementieren
6. `voiceRequestController.ts` implementieren
7. `voiceRequestRoutes.ts` implementieren
8. `server.ts` Route mounten
9. speaches auf VPS via Docker Compose starten
10. End-to-End Test mit echtem Audio-File
