# AGENTS.md — ANYA_v1 (A.N.Y.A Desktop AI Assistant)

## Quick Start
```bash
pip install -r requirements.txt
playwright install
python main.py
```
- Requires Python 3.11+ and a **Gemini API key** (entered in UI on first run; stored in `config/api_keys.json`).
- Windows/macOS/Linux supported; some actions are OS-specific.

## Architecture Overview
| Component | Location | Purpose |
|-----------|----------|---------|
| Entry point | `main.py` | Starts UI + Gemini Live session, declares tools, dispatches calls |
| UI | `ui.py` (PyQt6) | Desktop window, log panel, file-drop zone, API key setup overlay |
| Tools | `actions/*.py` | 20+ tool implementations (browser, files, system control, code, etc.) |
| Autonomous tasks | `agent/` | Planner/executor/task queue for multi-step goals |
| Memory | `memory/` | Long-term memory (`long_term.json`) + config helpers |
| Prompt | `core/prompt.txt` | System instruction injected into every session |

## Key Conventions
- **Tool declarations** live in `main.py:73-480` as `TOOL_DECLARATIONS`; keep schemas in sync with `actions/` implementations.
- **Tool dispatch** is in `main.py:_execute_tool` (lines 567-700) — routes by `fc.name` to the matching action module.
- **Uploaded files**: UI stores path in `JarvisUI.current_file`; `file_processor` falls back to this if `file_path` arg omitted.
- **Memory writes**: `save_memory` tool updates `memory/long_term.json` silently; values must be English.
- **Async pattern**: Most actions run via `loop.run_in_executor(None, ...)` to avoid blocking the audio loop.
- **Audio constants** (sample rates, chunk size) defined at `main.py:45-49`.

## Common Commands
```bash
# Install deps (includes Playwright browsers)
pip install -r requirements.txt && playwright install

# Run
python main.py

# Clean cache
rm -rf __pycache__ actions/__pycache__ agent/__pycache__ memory/__pycache__ config/__pycache__
```

## Testing / Verification
- No formal test suite exists.
- Manual smoke test: run `python main.py`, grant mic permission, speak/type a command (e.g., "open notepad"), verify tool fires and UI logs show `A.N.Y.A: ...`.
- For action changes: test the specific tool via voice/text in the running UI.

## Gotchas & Risks
- **Secrets**: `config/api_keys.json` stores the Gemini key in plain text — **never commit**; add to `.gitignore`.
- **Duplicate dep**: `requirements.txt` lists both `pillow` and `Pillow` (line 4 & 15); keep only `Pillow`.
- **Shell injection**: Several actions use `shell=True` with user-influenced input — audit before accepting PRs.
- **LLM code execution**: `code_helper` and `dev_agent` can run generated code; sandboxing is minimal.
- **Cross-platform fragility**: Many actions have OS-specific branches; test on target OS before merging changes.
- **Missing `.gitignore`**: Repo lacks one — create it for `__pycache__/`, `.venv/`, `config/api_keys.json`.

## File Paths to Know
- `main.py` — tool registry + dispatch + audio loop
- `ui.py` — PyQt6 UI, file drop, API key entry
- `actions/browser_control.py` — Playwright multi-browser automation
- `actions/file_processor.py` — Multi-format file handling (images, PDFs, code, audio, video, archives)
- `actions/computer_settings.py` / `computer_control.py` — OS-level shortcuts & input simulation
- `agent/executor.py` — Autonomous task execution with replanning
- `memory/memory_manager.py` — Load/save/format long-term memory
- `config/api_keys.json` — Runtime API key (git-ignored)
- `core/prompt.txt` — Base system prompt

## Adding a New Tool
1. Implement in `actions/<name>.py` with signature `func(parameters, player, speak=None)`.
2. Add declaration to `TOOL_DECLARATIONS` in `main.py` (name, description, JSON schema).
3. Add dispatch branch in `_execute_tool` (follow existing pattern).
4. Test via UI: trigger tool by voice/text and verify logs.