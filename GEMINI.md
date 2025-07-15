# Project Summary

This repository is designed for converting ebooks to audiobooks using a Text-to-Speech (TTS) engine. The primary application logic resides in `gradio_gui.py`, which provides a Gradio-based web interface for users to upload ebooks and generate audiobooks.

## Key Technologies & Dependencies

*   **TTS Engine:** `f5-tts-mlx` (recently migrated from `piper-tts`).
*   **Web UI:** Gradio.
*   **Ebook Processing:** Calibre's `ebook-convert` command-line tool.
*   **Audio Manipulation:** `pydub`, `soundfile`.
*   **Text Processing:** `nltk` (for sentence tokenization).
*   **Core Libraries:** `numpy`, `tqdm`, `requests`, `json`, `os`, `shutil`, `subprocess`, `re`, `tempfile`, `urllib.request`, `zipfile`, `time`, `ebooklib`, `epub`, `BeautifulSoup`, `csv`.

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
    *   The output of `generate` is a NumPy array. For `soundfile.write`, it needs to be explicitly reshaped to include a channel dimension (e.g., `np.expand_dims(audio, axis=1)` for mono audio).

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

*   **`IndexError: tuple index out of range` (from `soundfile.write`)**:
    *   **Cause:** `f5-tts-mlx` returns a 1D audio array, but `soundfile.write` expects a 2D array (even for mono).
    *   **Resolution:** Reshape the audio array using `np.expand_dims(audio, axis=1)` before passing it to `sf.write`.

*   **`LookupError: Resource punkt_tab not found.` (from NLTK)**:
    *   **Cause:** Missing NLTK data for sentence tokenization.
    *   **Resolution:** Add `nltk.download('punkt_tab')` to the script.

This `GEMINI.md` file should provide a comprehensive overview for future reference.
