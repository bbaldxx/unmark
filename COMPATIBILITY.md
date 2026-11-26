# COMPATIBILITY.md — Audit Fase 0 (Safari iOS)

> Stato: **teorico**, basato sulla documentazione nota a metà 2026.
> Nessun test è stato eseguito su un dispositivo reale. I valori vanno confermati con `probe/index.html`
> sul tuo iPhone (vedi PROGRESS.md). Ogni riga marcata ⚠️ è critica per l'architettura.

| Tecnologia | Supporto atteso su iOS Safari | Impatto | Fallback |
|---|---|---|---|
| PWA standalone (manifest + "Aggiungi a Home") | Sì | Base del progetto | — |
| Service Worker / offline | Sì (lo storage può essere eliminato dopo ~7 giorni di inutilizzo se l'app non è installata) | Offline | Installare sempre su Home |
| `<input type=file accept="video/*">` | Sì (Galleria + File) | Import | — |
| ⚠️ WebCodecs `VideoDecoder`/`VideoEncoder` | Sì su iOS 17+ (H.264; HEVC dipende dal dispositivo) | Cuore della pipeline | FFmpeg WASM (lento) |
| ⚠️ WebCodecs `AudioDecoder`/`AudioEncoder` | Arrivati più tardi, **da verificare** | Audio originale | Copia diretta dei pacchetti AAC senza ricodifica (preferibile comunque) |
| Demux MP4/MOV | Nessuna API nativa | Serve una libreria JS | `mp4box.js` o `mediabunny` (~leggere) |
| Mux MP4 | Nessuna API nativa | Export | `mediabunny` / `mp4-muxer` |
| `MediaRecorder` MP4 | Sì | Non adatto (tempo reale, qualità scarsa) | Solo come fallback estremo |
| Web Workers + `OffscreenCanvas` | Sì (iOS 16.4+, WebGL in worker; 2D OK) | UI fluida | Main thread a blocchi |
| WebGL2 | Sì | Inpainting FAST su GPU | Canvas 2D |
| WebGPU | iOS 26+ | Accelerazione SMART/AI | WebGL2 |
| WebAssembly SIMD | Sì | OpenCV.js | — |
| ⚠️ WASM multithread (`SharedArrayBuffer`) | Solo con header COOP/COEP | OpenCV/FFmpeg più veloci | Single-thread |
| OPFS (+ `createSyncAccessHandle` nei worker) | Sì | File temporanei grandi | IndexedDB |
| IndexedDB | Sì | Progetti, maschere | — |
| `navigator.share` con file | Sì | Condivisione nativa | Download del link |
| Wake Lock | Sì (in standalone era instabile prima di iOS 18.4) | Evita lo standby | Avviso all'utente |
| ⚠️ Esecuzione in background | **No**: la PWA viene sospesa appena esce dal primo piano | Elaborazioni lunghe | Checkpoint su OPFS e ripresa |
| ⚠️ Memoria | Limite per scheda non documentato, nella pratica ~1–1,5 GB prima del crash | 4K e buffer | Elaborazione a blocchi, chiusura dei `VideoFrame` |
| ONNX Runtime Web | WASM sì, WebGPU su iOS 26+ | AI MODE | Da valutare in Fase 7 |

## Conclusioni di fattibilità

1. **Fattibile**: PWA offline, import, maschere, FAST MODE su GPU, export MP4 H.264 via WebCodecs + muxer JS.
2. **Fattibile con riserva**: SMART MODE (costo in calcolo e temperatura); 4K (memoria, encoder).
3. **Incerto**: AI MODE (dimensioni dei modelli, tempi per frame). Va deciso dopo i benchmark della Fase 6.
4. **Non realizzabile**: elaborazione in background a schermo bloccato. La UI dovrà dirlo chiaramente.

## Hosting e header

GitHub Pages **non** permette header personalizzati → niente COOP/COEP nativi.
Scelta consigliata: **Cloudflare Pages** (gratuito, file `_headers`, deploy da GitHub).
Alternativa: `coi-serviceworker` su GitHub Pages (funziona, ma è più fragile al primo avvio).
