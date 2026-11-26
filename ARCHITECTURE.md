# ARCHITECTURE.md

```
UI (main thread) ──postMessage──► Pipeline Worker
  player, timeline, maschere         demux (mediabunny) → VideoDecoder
                                     → VideoFrame → OffscreenCanvas WebGL2 (inpaint ROI)
                                     → VideoEncoder → mux MP4 → OPFS (chunk)
                                     audio: pacchetti copiati senza ricodifica
```

- **Streaming**: coda decoder limitata (≈8 frame), `frame.close()` immediato; il video non sta mai tutto in RAM.
- **ROI**: si elabora solo il riquadro delle maschere (più un margine), poi si ricompone il frame.
- **Checkpoint**: ogni N secondi si salvano su IndexedDB l'ultimo keyframe elaborato e l'offset del file OPFS → ripresa dopo la sospensione.
- **Annullamento**: `AbortController` propagato al worker, poi chiusura di decoder/encoder e pulizia OPFS.
- **Modalità**: FAST = inpainting per diffusione/Telea su GPU; SMART = riempimento dai frame vicini allineati (flusso ottico semplificato) + blend temporale; AI = modulo caricato a parte.
- **Diagnostica**: logger ad anello (IndexedDB) + pagina con compatibilità, tempi, errori ed export `.txt`.
