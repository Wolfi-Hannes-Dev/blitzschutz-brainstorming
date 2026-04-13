# PRD — Frontend: Voice Request Feature

**Projekt:** Blitzschutz Reichenhauser Frontend (Vue 3 + Vite + Tailwind)  
**Feature:** Audio-Aufnahme + Back-Office Audio-Player  
**Datum:** 2026-04-13  
**Status:** Ready for Development

---

## Ziel

Zwei UI-Bereiche:

1. **Vorarbeiter-App:** Mikrofon-Button bei jeder Baustelle → Aufnahme → ✅ Häkchen
2. **Back-Office:** Liste der Voice Requests mit Waveform-Player, Transkript und Pre-Fill URL

---

## Neue Dateien

```
src/
├── components/
│   ├── VoiceRecorder.vue          ← neu (Mikrofon-Button + Aufnahme-Flow)
│   └── VoiceRequestPlayer.vue     ← neu (Back-Office: Waveform + Transkript)
├── views/
│   └── VoiceRequestsView.vue      ← neu (Back-Office Liste)
├── services/
│   └── api.ts                     ← erweitern (neue API-Methoden)
└── types/
    └── VoiceRequest.ts            ← neu
```

**Geänderte Dateien:**
- `src/components/ConstructionSiteList.vue` — `VoiceRecorder` einbinden
- `src/router/index.ts` — neue Route `/voice-requests`
- `package.json` — `wavesurfer.js` dependency

---

## Dependencies

```bash
npm install wavesurfer.js
```

---

## 1. Types

**Datei:** `src/types/VoiceRequest.ts`

```typescript
export interface VoiceRequest {
  id: number;
  construction_site_id: number;
  employee_id?: number;
  employee_name?: string;
  audio_path?: string;
  transcript?: string;
  extracted_data?: Record<string, unknown>;
  betreff?: string;
  ansprechpartner_vor_ort?: string;
  notizen?: string;
  predefined_url: string;
  done: boolean;
  created_at: string;
  updated_at: string;
  done_at?: string;
}
```

---

## 2. API-Erweiterung

**Datei:** `src/services/api.ts` — neue Methoden anhängen:

```typescript
// Voice Requests
async submitVoiceRequest(audioBlob: Blob, constructionSiteId: number, employeeId?: number): Promise<void> {
  const formData = new FormData();
  formData.append('audio', audioBlob, 'recording.webm');
  formData.append('constructionSiteId', String(constructionSiteId));
  if (employeeId) formData.append('employeeId', String(employeeId));

  const token = import.meta.env.VITE_API_TOKEN;
  const baseUrl = import.meta.env.VITE_API_CONSTRUCTION_SITES || 'http://localhost:3000/api/v1/';
  const normalizedUrl = baseUrl.endsWith('/') ? baseUrl : `${baseUrl}/`;

  const response = await fetch(`${normalizedUrl}voice-requests`, {
    method: 'POST',
    headers: { Authorization: `Bearer ${token}` },
    body: formData,
    // Kein Content-Type Header setzen! Browser setzt boundary automatisch
  });

  if (!response.ok) throw new Error(`Voice request failed: ${response.status}`);
},

async getVoiceRequests(all = false): Promise<VoiceRequest[]> {
  const response = await apiClient.get<{ data: VoiceRequest[] }>(
    `voice-requests${all ? '?all=true' : ''}`
  );
  return response.data.data || [];
},

async markVoiceRequestDone(id: number): Promise<void> {
  await apiClient.post(`voice-requests/${id}/done`);
},

getVoiceRequestAudioUrl(id: number): string {
  const baseUrl = import.meta.env.VITE_API_CONSTRUCTION_SITES || 'http://localhost:3000/api/v1/';
  const normalizedUrl = baseUrl.endsWith('/') ? baseUrl : `${baseUrl}/`;
  return `${normalizedUrl}voice-requests/${id}/audio`;
},
```

---

## 3. VoiceRecorder Komponente

**Datei:** `src/components/VoiceRecorder.vue`

### Verhalten
- Button mit Mikrofon-Icon
- Drücken → Aufnahme startet (rotes Pulsieren + Timer)
- Nochmal drücken oder nach 120s → Aufnahme stoppt, Upload startet
- ✅ grünes Häkchen nach erfolgreichem Upload (bleibt 3s, dann Reset)
- Fehlerzustand: rotes ❌ + kurze Meldung

### Zustände
```
idle → recording → uploading → success
                             → error → idle
```

```vue
<script setup lang="ts">
import { ref, computed, onUnmounted } from 'vue';
import { api } from '../services/api';

const props = defineProps<{
  constructionSiteId: number;
  employeeId?: number;
}>();

type RecorderState = 'idle' | 'recording' | 'uploading' | 'success' | 'error';

const state = ref<RecorderState>('idle');
const errorMsg = ref('');
const elapsedSeconds = ref(0);

let mediaRecorder: MediaRecorder | null = null;
let chunks: BlobPart[] = [];
let timerInterval: ReturnType<typeof setInterval> | null = null;
let autoStopTimeout: ReturnType<typeof setTimeout> | null = null;

const MAX_DURATION_S = 120;

const formattedTime = computed(() => {
  const m = Math.floor(elapsedSeconds.value / 60).toString().padStart(2, '0');
  const s = (elapsedSeconds.value % 60).toString().padStart(2, '0');
  return `${m}:${s}`;
});

const startRecording = async () => {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    chunks = [];
    mediaRecorder = new MediaRecorder(stream, { mimeType: 'audio/webm;codecs=opus' });

    mediaRecorder.ondataavailable = (e) => { if (e.data.size > 0) chunks.push(e.data); };
    mediaRecorder.onstop = handleRecordingStop;

    mediaRecorder.start(1000); // Chunk every 1s
    state.value = 'recording';
    elapsedSeconds.value = 0;

    timerInterval = setInterval(() => { elapsedSeconds.value++; }, 1000);
    autoStopTimeout = setTimeout(stopRecording, MAX_DURATION_S * 1000);
  } catch (err: any) {
    errorMsg.value = err.name === 'NotAllowedError'
      ? 'Mikrofon-Zugriff verweigert'
      : 'Mikrofon nicht verfügbar';
    state.value = 'error';
    setTimeout(() => { state.value = 'idle'; }, 3000);
  }
};

const stopRecording = () => {
  if (timerInterval) { clearInterval(timerInterval); timerInterval = null; }
  if (autoStopTimeout) { clearTimeout(autoStopTimeout); autoStopTimeout = null; }
  mediaRecorder?.stop();
  mediaRecorder?.stream.getTracks().forEach(t => t.stop());
};

const handleRecordingStop = async () => {
  const blob = new Blob(chunks, { type: 'audio/webm' });
  state.value = 'uploading';
  try {
    await api.submitVoiceRequest(blob, props.constructionSiteId, props.employeeId);
    state.value = 'success';
    setTimeout(() => { state.value = 'idle'; }, 3000);
  } catch {
    errorMsg.value = 'Upload fehlgeschlagen';
    state.value = 'error';
    setTimeout(() => { state.value = 'idle'; }, 3000);
  }
};

const handleClick = () => {
  if (state.value === 'idle') startRecording();
  else if (state.value === 'recording') stopRecording();
};

onUnmounted(() => {
  if (state.value === 'recording') stopRecording();
});
</script>

<template>
  <div class="flex items-center gap-2">
    <!-- Haupt-Button -->
    <button
      @click="handleClick"
      :disabled="state === 'uploading'"
      :title="state === 'recording' ? 'Aufnahme stoppen' : 'Auftrag einsprechen'"
      class="relative flex items-center justify-center w-10 h-10 rounded-full transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-1"
      :class="{
        'bg-gray-100 hover:bg-gray-200 text-gray-600 focus:ring-gray-400':        state === 'idle',
        'bg-red-500 hover:bg-red-600 text-white animate-pulse focus:ring-red-400': state === 'recording',
        'bg-gray-100 text-gray-400 cursor-wait':                                   state === 'uploading',
        'bg-green-500 text-white':                                                 state === 'success',
        'bg-red-100 text-red-500':                                                 state === 'error',
      }"
    >
      <!-- Idle: Mikrofon -->
      <svg v-if="state === 'idle'" xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
          d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4M12 3a4 4 0 014 4v4a4 4 0 01-8 0V7a4 4 0 014-4z" />
      </svg>
      <!-- Recording: Stop -->
      <svg v-else-if="state === 'recording'" xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="currentColor" viewBox="0 0 24 24">
        <rect x="6" y="6" width="12" height="12" rx="1" />
      </svg>
      <!-- Uploading: Spinner -->
      <svg v-else-if="state === 'uploading'" class="h-4 w-4 animate-spin" fill="none" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"/>
      </svg>
      <!-- Success: Checkmark -->
      <svg v-else-if="state === 'success'" xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
      </svg>
      <!-- Error: X -->
      <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
      </svg>
    </button>

    <!-- Timer (nur während Aufnahme) -->
    <span v-if="state === 'recording'" class="text-sm font-mono text-red-600 font-medium">
      {{ formattedTime }}
    </span>

    <!-- Fehlermeldung -->
    <span v-if="state === 'error'" class="text-xs text-red-500">{{ errorMsg }}</span>

    <!-- Fortschritt -->
    <span v-if="state === 'uploading'" class="text-xs text-gray-400">Wird gesendet…</span>
  </div>
</template>
```

---

## 4. Integration in ConstructionSiteList.vue

In `ConstructionSiteList.vue` den `VoiceRecorder` im aufgeklappten Bereich der Baustelle einbinden:

```vue
<!-- Import hinzufügen -->
<script setup lang="ts">
import VoiceRecorder from './VoiceRecorder.vue';
// auth.employeeId muss aus dem Auth-Store kommen
</script>

<!-- Im Dropdown-Bereich, vor den bestehenden Buttons -->
<template>
  <!-- ... bestehende Buttons ... -->
  <div class="mt-3 flex items-center gap-2">
    <span class="text-sm text-gray-500">Auftrag einsprechen:</span>
    <VoiceRecorder
      :construction-site-id="site.id"
      :employee-id="auth.employeeId"
    />
  </div>
</template>
```

**Auth Store:** `auth.employeeId` muss verfügbar sein. Falls noch nicht vorhanden, in `src/stores/auth.ts` ergänzen.

---

## 5. Back-Office: VoiceRequestPlayer Komponente

**Datei:** `src/components/VoiceRequestPlayer.vue`

Zeigt einen einzelnen Voice Request mit Waveform-Player (wavesurfer.js), Transkript und Aktions-Buttons.

```vue
<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue';
import WaveSurfer from 'wavesurfer.js';
import type { VoiceRequest } from '../types/VoiceRequest';
import { api } from '../services/api';

const props = defineProps<{ request: VoiceRequest }>();
const emit = defineEmits<{ done: [id: number] }>();

const waveformRef = ref<HTMLElement | null>(null);
let wavesurfer: WaveSurfer | null = null;
const isPlaying = ref(false);
const isLoading = ref(true);

onMounted(() => {
  if (!waveformRef.value) return;

  wavesurfer = WaveSurfer.create({
    container: waveformRef.value,
    waveColor: '#4ade80',       // grün
    progressColor: '#a00c17',   // Blitzschutz-Rot
    cursorColor: '#a00c17',
    barWidth: 2,
    barGap: 1,
    barRadius: 2,
    height: 48,
    normalize: true,
  });

  wavesurfer.on('ready', () => { isLoading.value = false; });
  wavesurfer.on('play', () => { isPlaying.value = true; });
  wavesurfer.on('pause', () => { isPlaying.value = false; });
  wavesurfer.on('finish', () => { isPlaying.value = false; });

  // Audio-URL mit Auth-Token laden
  const audioUrl = api.getVoiceRequestAudioUrl(props.request.id);
  const token = import.meta.env.VITE_API_TOKEN;
  
  wavesurfer.load(audioUrl, undefined, undefined);
  // Da WaveSurfer intern fetch nutzt, benötigen wir einen Workaround für Auth:
  // Audio als Blob voraufladen und als Object-URL übergeben
  fetch(audioUrl, { headers: { Authorization: `Bearer ${token}` } })
    .then(r => r.blob())
    .then(blob => {
      const objectUrl = URL.createObjectURL(blob);
      wavesurfer?.load(objectUrl);
    })
    .catch(() => { isLoading.value = false; });
});

onUnmounted(() => {
  wavesurfer?.destroy();
});

const togglePlay = () => {
  wavesurfer?.playPause();
};

const handleMarkDone = async () => {
  await api.markVoiceRequestDone(props.request.id);
  emit('done', props.request.id);
};

const formatDate = (iso: string) =>
  new Date(iso).toLocaleString('de-AT', { day: '2-digit', month: '2-digit', year: 'numeric', hour: '2-digit', minute: '2-digit' });
</script>

<template>
  <div class="bg-white rounded-lg border border-gray-200 overflow-hidden"
       :class="{ 'opacity-60': request.done }">
    <!-- Header -->
    <div class="px-4 pt-4 pb-2 flex items-start justify-between gap-2">
      <div>
        <p class="text-xs text-gray-400">{{ formatDate(request.created_at) }}</p>
        <p class="text-sm font-medium text-gray-900 mt-0.5">
          {{ request.betreff || 'Voice-Auftrag' }}
        </p>
        <p v-if="request.employee_name" class="text-xs text-gray-500">
          von {{ request.employee_name }}
        </p>
      </div>
      <span v-if="request.done"
            class="inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium bg-green-100 text-green-700">
        Erledigt
      </span>
    </div>

    <!-- Waveform Player -->
    <div class="px-4 py-2">
      <div class="bg-gray-900 rounded-lg p-3 flex items-center gap-3">
        <!-- Play/Pause Button -->
        <button @click="togglePlay"
                :disabled="isLoading"
                class="flex-shrink-0 w-9 h-9 rounded-full bg-[#a00c17] hover:bg-[#8a0a14] disabled:bg-gray-600
                       flex items-center justify-center text-white transition-colors">
          <svg v-if="isLoading" class="h-4 w-4 animate-spin" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"/>
          </svg>
          <svg v-else-if="!isPlaying" xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 ml-0.5" fill="currentColor" viewBox="0 0 24 24">
            <path d="M8 5v14l11-7z"/>
          </svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="currentColor" viewBox="0 0 24 24">
            <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z"/>
          </svg>
        </button>
        <!-- Waveform -->
        <div ref="waveformRef" class="flex-1 min-w-0" />
      </div>
    </div>

    <!-- Transkript -->
    <div v-if="request.transcript" class="px-4 pb-3">
      <details class="group">
        <summary class="text-xs text-gray-400 cursor-pointer hover:text-gray-600 select-none">
          Transkript anzeigen
        </summary>
        <p class="mt-2 text-sm text-gray-700 bg-gray-50 rounded p-2 leading-relaxed">
          {{ request.transcript }}
        </p>
      </details>
    </div>

    <!-- Aktionen -->
    <div v-if="!request.done" class="px-4 pb-4 flex gap-2">
      <a :href="request.predefined_url"
         target="_blank" rel="noopener noreferrer"
         class="flex-1 inline-flex items-center justify-center px-3 py-2 bg-[#a00c17] hover:bg-[#8a0a14]
                text-white text-sm font-medium rounded-lg transition-colors">
        MS Forms öffnen
      </a>
      <button @click="handleMarkDone"
              class="flex-1 inline-flex items-center justify-center px-3 py-2 bg-green-600 hover:bg-green-700
                     text-white text-sm font-medium rounded-lg transition-colors">
        ✓ Erledigt
      </button>
    </div>
  </div>
</template>
```

---

## 6. Back-Office View

**Datei:** `src/views/VoiceRequestsView.vue`

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue';
import type { VoiceRequest } from '../types/VoiceRequest';
import { api } from '../services/api';
import VoiceRequestPlayer from '../components/VoiceRequestPlayer.vue';

const requests = ref<VoiceRequest[]>([]);
const showAll = ref(false);
const isLoading = ref(true);

const load = async () => {
  isLoading.value = true;
  requests.value = await api.getVoiceRequests(showAll.value);
  isLoading.value = false;
};

const handleDone = (id: number) => {
  if (!showAll.value) {
    requests.value = requests.value.filter(r => r.id !== id);
  } else {
    const r = requests.value.find(r => r.id === id);
    if (r) r.done = true;
  }
};

onMounted(load);
</script>

<template>
  <div class="max-w-2xl mx-auto px-4 py-6">
    <div class="flex items-center justify-between mb-4">
      <h1 class="text-xl font-semibold text-gray-900">Voice Requests</h1>
      <label class="flex items-center gap-2 text-sm text-gray-500 cursor-pointer">
        <input type="checkbox" v-model="showAll" @change="load" class="rounded" />
        Erledigte anzeigen
      </label>
    </div>

    <div v-if="isLoading" class="text-center py-12 text-gray-400">Lädt…</div>

    <div v-else-if="requests.length === 0" class="text-center py-12 text-gray-400">
      Keine offenen Voice Requests
    </div>

    <div v-else class="flex flex-col gap-4">
      <VoiceRequestPlayer
        v-for="r in requests"
        :key="r.id"
        :request="r"
        @done="handleDone"
      />
    </div>
  </div>
</template>
```

---

## 7. Router

**Datei:** `src/router/index.ts` — ergänzen:

```typescript
import VoiceRequestsView from '../views/VoiceRequestsView.vue';

// In routes Array:
{
  path: '/voice-requests',
  name: 'VoiceRequests',
  component: VoiceRequestsView,
  meta: { requiresAuth: true }
}
```

---

## Schnittstellenvertrag Frontend ↔ Backend

| Was | Frontend | Backend |
|-----|----------|---------|
| Upload | `multipart/form-data`, Felder: `audio` (Blob), `constructionSiteId`, `employeeId?` | `POST /api/v1/voice-requests` → `202 Accepted` |
| Liste | `GET /api/v1/voice-requests[?all=true]` | `{ success, count, data: VoiceRequest[] }` |
| Audio-Stream | `GET /api/v1/voice-requests/:id/audio` + Bearer Token | `audio/webm` stream |
| Erledigt | `POST /api/v1/voice-requests/:id/done` | `{ success: true }` |

**Wichtig:** Der Upload gibt `202 Accepted` zurück (nicht `201`), da Transkription + Extraktion async laufen. Das Frontend zeigt sofort ✅ nach `202`.

---

## Implementierungsreihenfolge

1. `npm install wavesurfer.js`
2. `src/types/VoiceRequest.ts` anlegen
3. `src/services/api.ts` um Voice-Methoden erweitern
4. `src/components/VoiceRecorder.vue` implementieren + in `ConstructionSiteList.vue` einbinden
5. `src/stores/auth.ts` prüfen: `employeeId` verfügbar? Sonst ergänzen
6. `src/components/VoiceRequestPlayer.vue` implementieren
7. `src/views/VoiceRequestsView.vue` implementieren
8. `src/router/index.ts` Route ergänzen
9. Back-Office Navigation-Link zu `/voice-requests` hinzufügen (in `App.vue` oder Navigation-Komponente)
10. End-to-End Test
