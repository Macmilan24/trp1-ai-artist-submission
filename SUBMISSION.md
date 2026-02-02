# TRP1 - AI Content Generation Challenge: Character-First Screening

## 1. Environment Setup & Configuration
*   **APIs Configured:** Google Gemini API (Lyria & Veo).
*   **System:** Windows 11 / PowerShell / VS Code.
*   **Installation:** Successfully used `uv sync` to manage the Python environment.
*   **Issues Encountered:** 
    *   The global `-v` flag must be placed before the subcommand (`ai-content -v music`) due to rigid CLI parsing.
    *   Network timeouts on longer (30s) generations; resolved by generating shorter (5s) clips to verify the pipeline.

## 2. Codebase Understanding
*   **Architecture:** The system follows a **Provider Pattern**.
    *   `src/ai_content/core/provider.py`: Defines the base interface.
    *   `src/ai_content/providers/google/`: Contains specific implementations for Lyria (Music) and Veo (Video).
*   **Orchestration:** The `pipelines/` directory manages the flow from prompt processing to final file saving.
*   **Insights:** The system is designed for extensibility, making it easy to add new vendors (like AIMLAPI) by implementing the abstract provider methods.

## 3. Generation Log & Technical Challenges
I successfully established connections to the Google GenAI models, but identified two critical defects in the codebase that prevent final artifact usage:

### Artifact A: Music (Lyria)
*   **Command:** `uv run ai-content music --prompt "Simple drums" --provider lyria --duration 5`
*   **Status:** Connection Established / Data Streamed.
*   **Result:** Generated `exports/lyria_20260202_120804.wav` (384KB).
*   **Technical Bug Found:** The file is unreadable by FFmpeg and media players. **Diagnosis:** The `lyria.py` provider writes raw binary chunks to the file but fails to prepend a valid RIFF/WAV header. This is a logic error in the file-handling utility of the repo.

### Artifact B: Video (Veo)
*   **Command:** `uv run ai-content video --prompt "Futuristic city" --style urban --provider veo`
*   **Technical Bug Found:** `AttributeError: module 'google.genai.types' has no attribute 'GenerateVideoConfig'`.
*   **Diagnosis:** This is a code regression. The implementation in `veo.py` is using an attribute name that does not exist in the current version of the `google-genai` Python SDK.

## 4. Troubleshooting & Persistence
1.  **CLI Debugging:** Used the `--verbose` flag to monitor WebSocket handshakes and confirm that API keys and connections were working perfectly.
2.  **SDK Forensics:** Identified that the Video failure was a software bug, not a configuration error.
3.  **WAV Reconstruction:** Attempted to use FFmpeg to "force-read" the generated audio, confirming that the data exists but the header is missing.

## 5. Insights & Learnings
*   **Surprise:** I was surprised that the music provider established a real-time WebSocket connection rather than a standard REST polling job.
*   **Improvements:** I would implement a `wave` library wrapper in `lyria.py` to ensure files are saved with proper headers and update the dependency requirements to match the `google-genai` SDK types.
*   **Comparison:** Compared to other tools, this codebase is highly modular but requires better unit testing on the provider output layer.

## 6. Links
*   **GitHub Repo:** [PASTE YOUR NEW REPO LINK HERE]
*   **Video Link:** N/A (Generation blocked by identified codebase bugs; evidence of 384KB data stream provided in logs).