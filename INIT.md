# INIT.md

This file is a quick `/init`-style technical context for this repository.

## Project
- Name: `ANYA_v1` (A.N.Y.A desktop AI assistant)
- Main entrypoint: `main.py`
- UI: `ui.py` (PyQt6)
- Core model stack: Gemini Live API + function/tool calling
- Platforms: Windows, macOS, Linux (with many OS-specific branches)

## High-Level Flow
1. `main.py` starts UI (`JarvisUI`) and live audio session (`JarvisLive`).
2. Gemini live session receives user audio/text and may issue tool calls.
3. `_execute_tool` in `main.py` routes calls to modules in `actions/`.
4. UI logs system/user/assistant events and tracks uploaded file path.
5. Optional autonomous multi-step execution goes through `agent/`.

## Key Directories
- `actions/`: Tool implementations (browser, files, reminders, desktop control, code tools, etc.)
- `agent/`: Planner/executor/recovery/task queue for autonomous tasks
- `memory/`: Long-term memory + API key config helpers
- `config/`: Runtime config (`api_keys.json`)
- `core/`: Prompt text (`prompt.txt`)

## Important Files
- `main.py`: Live session lifecycle, tool declarations, tool dispatch
- `ui.py`: Main desktop interface and setup overlay
- `actions/browser_control.py`: Playwright-based multi-browser automation
- `actions/file_processor.py`: Multi-format file processing (image/pdf/code/audio/video/etc.)
- `actions/computer_settings.py`: OS-level shortcut/actions
- `actions/computer_control.py`: Mouse/keyboard/clipboard/screenshot primitives
- `actions/code_helper.py`: Generate/edit/run/build code from prompts
- `actions/dev_agent.py`: Scaffold/fix whole projects iteratively
- `agent/executor.py`: Executes generated plans and replans on failure

## Tool Surface (from `main.py`)
Declared tools include:
- `open_app`, `web_search`, `weather_report`, `send_message`, `reminder`
- `youtube_video`, `screen_process`, `computer_settings`, `browser_control`
- `file_controller`, `desktop_control`, `code_helper`, `dev_agent`
- `agent_task`, `computer_control`, `game_updater`, `flight_finder`
- `shutdown_jarvis`, `file_processor`, `save_memory`

## Setup / Run
```bash
pip install -r requirements.txt
playwright install
python main.py
```

## Current Risks / Tech Debt
1. `config/api_keys.json` currently stores a plain-text API key; do not commit secrets.
2. No `.gitignore` currently in repo root; cache/config artifacts are easy to commit accidentally.
3. Some actions use `shell=True` with user-influenced input (command injection risk).
4. LLM-generated code execution paths exist (`desktop`, `agent/executor` fallback).
5. Dependency drift: some optional imports are used but not listed in `requirements.txt`.
6. `requirements.txt` contains duplicate `pillow`/`Pillow`.

## Suggested Immediate Hardening
1. Add `.gitignore` for `__pycache__/`, `.venv/`, `config/api_keys.json`.
2. Rotate exposed API key and move secrets to env vars or local untracked config.
3. Remove/limit `shell=True` paths and validate executable/input boundaries.
4. Gate generated-code execution behind stricter allowlists or confirmation.

## Notes for Future Edits
- Keep tool names and parameter schemas in `main.py` aligned with action modules.
- `ui.py` passes uploaded file path via `current_file`; `file_processor` uses this fallback.
- Many modules assume desktop automation libraries are present (`pyautogui`, etc.).
- Cross-platform behavior is feature-rich but fragile; test per-OS before refactors.
