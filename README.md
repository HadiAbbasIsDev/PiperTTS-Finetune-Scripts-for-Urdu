# PiperTTS — Urdu voice

Fine-tune a [Piper](https://github.com/rhasspy/piper) (VITS) text-to-speech voice for Urdu.
Training runs on a Colab GPU; data prep and inference run locally.

## What's in here

- `UrduTTS_Colab_Training.ipynb` — the training notebook, run on a GPU runtime
- `resample.py` — audio resampler utility
- `README.md`

The Piper training code is not committed — the notebook clones it (`rmcpantoja/piper`) on the runtime, so a fresh clone of this repo is all you need.

## What you need

- A Colab GPU runtime (a free T4 is enough), or any CUDA GPU.
- The dataset (see below).
- Nothing else — the notebook installs all dependencies.

One important detail: Colab's Python is 3.12, which has no prebuilt `piper-phonemize` wheel. The notebook installs `piper-phonemize-fix` instead — it provides the same `piper_phonemize` module along with the Urdu (`ur`) phonemizer. This is the key fix; without the correct `ur` phonemizer the model trains on garbage.

## The data

The dataset uses the single-speaker / LJSpeech layout:

```
urdu_dataset/
  metadata.csv
  wavs/
    common_voice_ur_31771683.wav
    common_voice_ur_31771684.wav
    ...
```

- `metadata.csv` — one line per clip, pipe-delimited, no header:
  ```
  common_voice_ur_31771683.wav|کبھی کبھار ہی خیالی پلاو بناتا ہوں
  ```
- `wavs/` — mono WAV at 16 kHz (the notebook upsamples to 22.05 kHz during preprocessing).
- Current set: ~4,129 Mozilla Common Voice Urdu clips with human transcripts.

The data is packaged as `urdu_dataset.zip` and is **not in this repo** (too large for git). It is available on my Google Drive:

**Dataset (urdu_dataset.zip):** `<add your Google Drive link here>`

## Workflow

1. Open `UrduTTS_Colab_Training.ipynb` against a Colab GPU runtime (the Colab website, or VS Code with a Colab GPU connected). Set the runtime to GPU.
2. **Cells 1–2** — GPU check and install (clones Piper, installs deps including `piper-phonemize-fix`).
3. **Cell 3** — get `urdu_dataset.zip` onto the runtime's `/content` (download from Drive with `gdown`, or drag the single zip in), then run the cell to unzip. It auto-detects the folder and sets `DATASET_DIR`.
4. **Cell 4** — download the base checkpoint (Arabic `kareem-medium`; Arabic shares Urdu's script and much of its phonology, so it's a strong base to fine-tune from).
5. **Cell 5** — preprocess with `--language ur`. Confirm it prints `espeak voice: ur`.
6. **Cell 6** — train. A checkpoint is saved every epoch (`last.ckpt`). `/content` is wiped when the runtime disconnects, so copy `last.ckpt` to Drive periodically.
7. **Cell 7** — export to `urdu.onnx` + `urdu.onnx.json` and download both.

To continue training another time, point `--resume_from_checkpoint` at your saved `last.ckpt` instead of the base checkpoint.

## Using the trained voice (local)

```sh
pip install piper-tts
echo 'یہ اردو میں بولنے والی آواز ہے' | piper --model urdu.onnx --output_file test.wav
```

## Notes

- espeak-ng's Urdu (`ur`) is marked "testing" — good overall, with occasional edge-case mispronunciations.
- Common Voice is multi-speaker, so the result is a consistent "average" Urdu voice rather than one specific person.
