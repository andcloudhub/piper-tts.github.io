# Piper TTS — fully offline in the browser

Text-to-speech that runs entirely on the client: phonemization with espeak-ng (WebAssembly),
synthesis with onnxruntime-web (WASM). No server calls at runtime — verified in headless Chromium
with all non-local requests blocked. Ships with Russian (Ruslan) and English (Joe) medium voices;
any voice from [rhasspy/piper-voices](https://huggingface.co/rhasspy/piper-voices) can be added.

## Running locally

    python3 serve.py 8080      # then open http://localhost:8080/

An HTTP server is required (`file://` won't work: WASM and `fetch` of the model are blocked).
`serve.py` sends COOP/COEP headers so onnxruntime-web can use multithreaded WASM; without them
it silently falls back to a single thread. On nginx or any other server add:

    Cross-Origin-Opener-Policy: same-origin
    Cross-Origin-Embedder-Policy: require-corp

## Contents

| Path | What | Source |
|---|---|---|
| `ru_RU-ruslan-medium.onnx` + `.json` | Russian voice (63 MB) | rhasspy/piper-voices |
| `en_US-joe-medium.onnx` + `.json` | English voice (63 MB) | rhasspy/piper-voices |
| `voices.json` | voice list for the selector (id, label, paths, sample text) | |
| `vendor/ort/` | onnxruntime-web 1.17.3 (`ort.wasm.min.js` + 4 `.wasm` variants) | npm `onnxruntime-web@1.17.3` |
| `vendor/phonemize/` | piper_phonemize (espeak-ng) WASM with full `espeak-ng-data`, incl. `ru_dict` | npm `@diffusionstudio/piper-wasm@1.0.0` |
| `piper-browser.js` | pipeline: text → phonemes → ids → ONNX → PCM/WAV | extracted from piper.ttstool.com |
| `index.html` | demo page with voice selector | |
| `serve.py` | dev server with COOP/COEP | |
| `out_ru.wav`, `out_en.wav` | test run output | |

## API

```js
const eng = await Piper.createEngine("./ru_RU-ruslan-medium.onnx", "./ru_RU-ruslan-medium.onnx.json");
const pcm = await eng.speak("Привет, мир!", { lengthScale: 1.0 });   // Float32Array, 22050 Hz mono
await Piper.playPcm(pcm, eng.sampleRate);
const wav = Piper.toWav(pcm, eng.sampleRate);                         // Blob audio/wav
```

Lower level: `Piper.getPhonemizer()` → `.phonemize([text], "en-us")` returns
`[{ text, phonemes, phoneme_ids }]`; `eng.infer(ids, speakerId, { noiseScale, lengthScale, noiseW })`.

## Adding voices

Drop a `*.onnx` + `*.onnx.json` pair from rhasspy/piper-voices next to the others and add an
entry to `voices.json` — the selector picks it up. Models load lazily on first selection and are
cached for the session. The espeak language comes from `cfg.espeak.voice`; pass `speakerId`
for multi-speaker models.

To shrink the bundle, keep only `ort-wasm-simd-threaded.wasm` in `vendor/ort/`
(or `ort-wasm-simd.wasm` if you can't set COOP/COEP) — ORT picks one file by browser
capabilities and never downloads the rest.
EOF
cd /home/claude && rm -rf piper_offline_multi && cp -r offline piper_offline_multi && rm -f piper_offline_multi/nohup.out /mnt/user-data/outputs/piper_offline_multi.zip && zip -qr /mnt/user-data/outputs/piper_offline_multi.zip piper_offline_multi && cp offline/README.md /mnt/user-data/outputs/README.md && echo ok