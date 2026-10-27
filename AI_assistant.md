# Voice Cloning
Voice cloning is a form of speech synthesis that generates new speech resembling a specific person's voice.

```mermaid
flowchart TD
    A["🎙️ Reference Audio<br/>Speaker's Voice"]
    B["📝 New Text<br/>Words to Speak"]
    C["🧠 Voice-Cloning TTS Model<br/>Combines Content + Vocal Identity"]
    D["🔊 Generated Speech<br/>New Sentence in the Target Voice"]

    A --> C
    B --> C
    C --> D

    style A fill:#E3F2FD,stroke:#1976D2,color:#111
    style B fill:#E3F2FD,stroke:#1976D2,color:#111
    style C fill:#FFF3E0,stroke:#EF6C00,color:#111
    style D fill:#E8F5E9,stroke:#388E3C,color:#111
```
## Three types of voice cloning
| Technique | Input | What it does |
|---|---|---|
| Zero-shot / few-shot TTS | Text + short reference audio | Clones an unfamiliar voice without updating model weights |
| Fine-tuned voice cloning | Text + larger voice dataset | Adapts model parameters to a specific speaker |
| Voice conversion | Source speech + target voice | Changes who appears to be speaking while preserving the words |

For reference, ElevenLabs recommends 1–2 minutes of good audio for its instant cloning product and 30–180 minutes for professional cloning.

Zero-shot means the model can generalize to a new speaker without training specifically on that speaker.

```mermaid
flowchart TD
    A["🎙️ Reference Audio<br/>Speaker's Voice"]
    B["Audio preprocessing<br/>Resampling, trimming silence, etc"]
    C["Speaker Representation<br/>Speaker Encoder (speaker embedding) or Reference Audio Features"]
    D["🔑 Voice Identity<br/>Speaker Conditioning"]
    E["📝 Target Text<br/>Text Tokens or Phonemes"]
    F["Neural Speech Generator<br/>Predicts Acoustic Features or Audio Tokens"]
    G["Vocoder / Audio Decoder<br/>Produces Final Waveform"]
    H["🔊 Synthesized Speech<br/>New Sentence in Target Voice"]

    A --> B
    B --> C
    C --> D
    C --> E
    D --> F
    E --> F
    F --> G
    G --> H
```

### What is a speaker embedding?
An audio encoder can transform a speech recording into a fixed-dimensional vector representing speaker characteristics like timbre and accent.


