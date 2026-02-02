# [TRP1] AI Content Generation Challenge - Submission Reporta
## 1. Environment Setup & API Configuration
*   **APIs Configured:** Google Gemini API (via Google AI Studio).
*   **Provider Usage:** Primarily utilized the **Google** provider suite (`lyria` for music and `veo` for video).
*   **Setup Process:**
    1.  Cloned repository and initialized environment using `uv sync`.
    2.  Configured `.env` with the `GEMINI_API_KEY`.
    3.  Verified installation via `uv run ai-content --help`.
*   **Initial Issues:** Encountered CLI parsing restrictions where the verbose flag `-v` must precede the subcommand. Resolved by adjusting command patterns.

## 2. Codebase Exploration
*   **Architecture Description:** 
    *   The system utilizes a **Registry Pattern** located in `src/ai_content/core/registry.py` to dynamically load providers.
    *   **Provider Organization:** Providers are grouped by vendor (Google, AIMLAPI) in `src/ai_content/providers/`. 
    *   **Pipelines:** Orchestration logic is separated into `music.py` and `video.py` pipelines, which handle the flow from prompt input to file export.
*   **Key Insights:** 
    *   The `lyria` provider implements a real-time WebSocket connection (`aio.live.music.connect`), allowing for streamed music generation rather than simple static file generation.
    *   The system uses a preset system in `src/ai_content/presets/` to standardize output formats (BPM, aspect ratios).

## 3. Technical Challenges & Diagnostic Log
During the generation phase, I identified three critical bugs in the codebase that demonstrate technical regressions:

### Challenge A: Video Provider (Veo) Code Bug
*   **Symptom:** `AttributeError: module 'google.genai.types' has no attribute 'GenerateVideoConfig'`
*   **Diagnosis:** The codebase in `veo.py` is out of sync with the installed version of the `google-genai` SDK. It attempts to reference a type that does not exist in the current library version.

### Challenge B: Music Provider (Lyria) Encoding Failure
*   **Symptom:** Successfully streamed 384KB of data, but the resulting `.wav` file was unreadable/corrupt.
*   **Diagnosis:** Forensic analysis of the `lyria.py` provider revealed that it writes raw binary chunks directly to the file but **fails to write the RIFF/WAV header**. 
*   **Workaround/Solution:** I manually salvaged the audio by using FFmpeg to decode the raw PCM stream:
    `ffmpeg -f s16le -ar 24000 -ac 1 -i <corrupt_file> final_audio.mp3`

### Challenge C: CLI Validation Logic
*   **Symptom:** Commands failed when using the `--style` flag alone.
*   **Diagnosis:** The CLI requires the `--prompt` flag even when a style preset is selected, despite presets containing pre-defined prompt logic.

## 4. Generation Log
*   **Music Generation:** Successfully triggered `lyria` with prompt "Simple drums".
*   **Audio Recovery:** Successfully recovered audio from the malformed WAV file using raw PCM decoding.
*   **Video Generation:** Created a technical surrogate video using FFmpeg to merge the recovered audio with a static visualization to meet YouTube submission requirements.

## 5. Insights & Learnings
*   **Codebase Observations:** The system is highly modular but lacks a "Safety Layer" in the file-handling utilities to ensure valid media headers are written.
*   **Improvements:** If deployed to production, I would implement a decorator for the file-saver to automatically wrap raw streams in the appropriate container (WAV/MP4) before closing the file handle.
*   **Comparison:** This system is more powerful than standard wrappers because of its WebSocket implementation for live streaming music, but it is currently fragile due to tight SDK version coupling.

## 6. Links
*   **GitHub Repository:** https://github.com/Macmilan24/trp1-ai-artist-submission.git
*   **YouTube Video:** https://youtu.be/_kJi-q7i7xI