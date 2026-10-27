# 09 — Voice Pipeline

> Talk to your agents: voice notes in → local transcription → agent works → synthesized voice reply. Hands-free operations.

## Inbound (voice → text)

1. Voice notes arrive on Telegram/Discord as audio attachments (`.ogg`/`.oga`).
2. Transcribe **locally** with [faster-whisper](https://github.com/SYSTRAN/faster-whisper) — no audio leaves the box:

```python
from faster_whisper import WhisperModel
model = WhisperModel("small", device="cpu", compute_type="int8")
segments, _ = model.transcribe(audio_path, language="en")
text = " ".join(s.text.strip() for s in segments)
```

3. Keep a plaintext backup of every transcription next to the audio — the transcript is a record, not a byproduct.

### The PyAV trap (and the workaround that always works)

faster-whisper decodes audio with PyAV, and version mismatches surface as cryptic `TypeError: open() got an unexpected keyword argument 'metadata_errors'`. Bypass decoding entirely — ffmpeg to WAV, then feed a numpy array:

```bash
ffmpeg -y -i note.ogg -ar 16000 -ac 1 -f wav /tmp/note.wav
```

```python
import wave, numpy as np
w = wave.open("/tmp/note.wav")
audio = np.frombuffer(w.readframes(w.getnframes()), dtype=np.int16).astype(np.float32) / 32768.0
segments, _ = model.transcribe(audio, language="en")  # array input skips decode_audio
```

Also note: a pip-installed `faster-whisper` may live in a *different* interpreter than your default `python3`. Check `pip show faster-whisper → Location` and call that interpreter's absolute path.

## Outbound (text → voice)

- Neural TTS per agent — each agent gets a **consistent named voice** so the founder knows who's speaking without looking.
- Deliver as a voice bubble, not a file (Telegram: send with voice-note flags), so it plays inline.
- Default to voice for status reports; keep a short text backup in the same message for scannability.

## Conventions that make voice usable

- **Transcribe everything, even failures.** A garbled transcript still backs up to text; map it to known context (personas, project names) before asking the sender to repeat.
- **Verdict-first speech:** lead with the decision/answer, then the why, then mechanics — rambling voice updates get skipped.
- **Times in the human's timezone**, always converted before speaking.

## Why local

Privacy (business audio never leaves), cost (zero per-minute API fees), and availability (works when the ISP is having a day — see the IPv6 notes in [03](03-chat-gateways.md)).
