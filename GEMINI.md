# Project Summary

This repository is designed for converting ebooks to audiobooks using a Text-to-Speech (TTS) engine. The primary application logic resides in `gradio_gui.py`, which provides a Gradio-based web interface for users to upload ebooks and generate audiobooks.

## Key Technologies & Dependencies

*   **TTS Engine:** `f5-tts-mlx` (recently migrated from `piper-tts`).
*   **Web UI:** Gradio.
*   **Ebook Processing:** Calibre's `ebook-convert` command-line tool.
*   **Audio Manipulation:** `pydub`.
*   **Text Processing:** `nltk` (for sentence tokenization).
*   **Core Libraries:** `tqdm`, `requests`, `json`, `os`, `shutil`, `subprocess`, `re`, `tempfile`, `urllib.request`, `zipfile`, `time`, `ebooklib`, `epub`, `BeautifulSoup`, `csv`.

## Workflow & Best Practices

1.  **Dependency Management:**
    *   `requirements.txt` should be kept minimal, containing only direct dependencies.
    *   Always run `pip install -r requirements.txt` after modifying the file.

2.  **Application Execution:**
    *   Run the Gradio application in the background using `python3 gradio_gui.py > gradio.log 2>&1 &`. This redirects all output (stdout and stderr) to `gradio.log`.
    *   Note the Process Group ID (PGID) returned by the `run_shell_command` tool. This is crucial for terminating the process later.

3.  **Debugging:**
    *   Always check `gradio.log` for detailed error messages and stack traces.
    *   Before starting a new application instance or reading the log, ensure all previous background processes are terminated using `kill -- -<PGID>`.

4.  **External Tooling:**
    *   Calibre's `ebook-convert` is a critical external dependency. Its absolute path (`/Applications/calibre.app/Contents/MacOS/ebook-convert`) must be correctly specified in the Python code where it's called (e.g., `gradio_gui.py`).
    *   Ensure necessary NLTK data (e.g., `punkt`, `punkt_tab`) is downloaded programmatically or manually.

5.  **`f5-tts-mlx` Specifics:**
    *   The `generate` function is imported from `f5_tts_mlx.generate`.
    *   The text input parameter for `generate` is `generation_text`, not `text`.
    *   The `generate` function does not return audio data. Instead, it saves the audio to a file specified by the `output_path` parameter.
    *   The code should check for the existence of the output file to verify that the TTS generation was successful.

## Common Issues & Resolutions

*   **`FileNotFoundError: [Errno 2] No such file or directory: 'ebook-convert'`**:
    *   **Cause:** Calibre's `ebook-convert` executable not found in PATH or incorrect path specified.
    *   **Resolution:** Update the hardcoded path to `/Applications/calibre.app/Contents/MacOS/ebook-convert` in `gradio_gui.py` where `subprocess.run` is called, and in the `calibre_installed` function.

*   **`ValueError: too many values to unpack (expected 2)`**:
    *   **Cause:** Often a symptom of `calibre_installed()` returning a single error message string instead of the expected tuple, due to `ebook-convert` not being found.
    *   **Resolution:** Ensure `calibre_installed()` correctly returns a boolean and that `ebook-convert` is accessible.

*   **`AttributeError: module 'f5_tts_mlx' has no attribute 'TTS'`**:
    *   **Cause:** Incorrect import or usage of the `f5-tts-mlx` library.
    *   **Resolution:** Use `from f5_tts_mlx.generate import generate` and call `generate()` directly.

*   **`TypeError: generate() got an unexpected keyword argument 'text'`**:
    *   **Cause:** Incorrect parameter name used when calling `f5_tts_mlx.generate`.
    *   **Resolution:** Use `generation_text` instead of `text` as the parameter name.

*   **`AttributeError: 'NoneType' object has no attribute 'ndim'`**:
    *   **Cause:** The `generate` function from `f5-tts-mlx` does not return audio data, so the `audio` variable is `None`.
    *   **Resolution:** Pass an `output_path` to the `generate` function and check for the file's existence to confirm successful generation.

*   **`LookupError: Resource punkt_tab not found.` (from NLTK)**:
    *   **Cause:** Missing NLTK data for sentence tokenization.
    *   **Resolution:** Add `nltk.download('punkt_tab')` to the script.

This `GEMINI.md` file should provide a comprehensive overview for future reference.
