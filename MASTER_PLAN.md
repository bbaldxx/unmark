# MASTER_PLAN.md — UNMARK

Uso consentito: solo video propri o per cui si hanno i diritti.

## Principi
- Zero backend, zero account, offline dopo il primo caricamento.
- Funzionalità reali o assenti: niente simulazioni.
- Il dispositivo decide la qualità (feature detection + benchmark).

## Roadmap
| Fase | Obiettivo | Criterio di uscita (verificato su iPhone) |
|---|---|---|
| 0 | Audit di fattibilità | Report del probe esportato e analizzato |
| 1 | PWA installabile + shell UI | Si installa, si apre offline, safe area corrette |
| 2 | Import / play / export | Un MP4 da 1080p×30s viene re-esportato con audio |
| 3 | Maschere + FAST | Logo statico rimosso, export OK |
| 4 | Tracking | Il rettangolo segue un logo in movimento, con correzione manuale |
| 5 | SMART | Meno flickering rispetto a FAST (confronto visivo) |
| 6 | Benchmark | Tabella fps/temperatura/RAM per risoluzione |
| 7 | AI opzionale | Solo se < ~2 s/frame a 720p, altrimenti si abbandona |
| 8 | Rifinitura | Nessun crash su 10 export consecutivi |

## Stack scelto (provvisorio)
TypeScript + Vite (nessun framework UI), WebCodecs, `mediabunny` (demux/mux), WebGL2 (inpainting),
Workers + OffscreenCanvas, OPFS + IndexedDB, Cloudflare Pages, GitHub Actions.
OpenCV.js: solo se serve per il tracking (build minimale, caricata su richiesta).
FFmpeg WASM: escluso dal pacchetto iniziale.
