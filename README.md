# TARS

TARS is a local Windows AI assistant using a GGUF model through `llama-cpp-python`,
with a PySide6 overlay interface and a user-controlled Internet Search toggle.

## Features

- Local Qwen GGUF model for response generation.
- Optional web research: the globe button turns searching ON or OFF.
- Compact dock reports live activity: THINKING, SEARCHING WEB, GENERATING, READY, or ERROR.
- Google-first search with DDGS as a backup provider.
- Reads short excerpts from top search results before composing a response.
- Persistent chat history with continue and delete controls.
- Local Second Brain profile, explicit saved memories, project-context capture, and in-app reminders.
- Windows application launching tools.
- Compact overlay UI with `Ctrl+Alt`, Enter to send, and Esc to close.

## Runtime requirements

- Windows 10/11 x64.
- Internet connection for web research.
- Enough RAM for the selected model and context length. 8 GB may be tight; 16 GB is a more practical starting point.
- The model weights are separate from the source repository.

## Run from source

Recommended build environment: Python 3.11 x64.

1. Put the licensed model at `models/Qwen3-1.7B-Q4_K_M.gguf`.
2. Create and activate a virtual environment.
3. Install dependencies:

   ```powershell
   python -m pip install -r requirements-build.txt
   ```

4. Start TARS:

   ```powershell
   python -m app.ui.main_window
   ```

## Build the Windows app folder

On a Windows x64 computer with Python 3.11 x64 installed:

1. Put the model file in `models/Qwen3-1.7B-Q4_K_M.gguf`.
2. Double-click `build_windows.bat`.
3. When it completes, run `dist\TARS\TARS.exe`.
4. To distribute it, ZIP the **entire** `dist\TARS` folder, including `_internal` and `models`.

This intentionally uses PyInstaller's **onedir** mode. A standalone EXE would have to unpack a very large model and native libraries on startup. The portable folder is usually easier to troubleshoot and update. The recipient does not need Python installed, but the app bundle is large.

## GitHub publishing

- Commit the source code, build script, spec, and README.
- Do **not** commit `.venv`, `dist`, or the `.gguf` model into normal Git history.
- Upload the complete ZIP of `dist\TARS` as a GitHub Release asset, after confirming you are allowed to redistribute the model weights.
- Include the upstream model license and notices in the release.

## Web-search behavior

When Internet Search is ON, TARS attempts web research. Google is preferred, with DDGS as a backup. When it is OFF, the request is handled by the local model and no search tool is called. If search is enabled but providers fail or the computer is offline, TARS reports that it could not obtain results rather than pretending it searched. Web results can be incomplete or incorrect, so verify important claims against the source websites.

## Troubleshooting

- **Model not found:** check `dist\TARS\models\Qwen3-1.7B-Q4_K_M.gguf`.
- **DLL or import errors:** rebuild on Windows x64 with Python 3.11 and use the dependencies in `requirements-build.txt`.
- **Slow generation:** reduce `N_CTX` or `N_BATCH` in `app/ui/main_window.py`; performance depends on CPU, RAM, and the llama.cpp build.
- **No search results:** confirm internet access; Google or DDGS may temporarily block automated requests.

## Model and third-party licensing

Review `MODEL-LICENSE-NOTICE.md` and the licenses of all third-party packages before publishing a release.

## Second Brain and privacy

TARS stores chat history, profile, memories, and reminders in `%LOCALAPPDATA%\TARS` on Windows. These files are not written into the GitHub source folder. Use the history icon to reopen or delete chats and the star icon to edit your profile, review memories, and inspect reminders. Say `Remember: ...` or `Remember that ...` to save a memory; clear statements such as `I'm working on ...` are saved as project context. Reminders currently support phrases like `Remind me to review BER in 30 minutes` (the app must be running to show the notification). Back up this local folder if you want to keep the memory data when changing computers.

TARS cannot literally know everything about its user. The profile is editable, saved memories can be deleted, and it should not treat inferred details as certain. Do not put passwords, API keys, or other secrets in the profile.


## Second Brain commands

- `remember: I prefer worked examples` saves a durable local memory.
- `project named Antenna Fabrication` creates/selects a project.
- `log measured 13 cm copper element` appends a numbered log to the current project.
- `log in Antenna Fabrication: adjusted the feed point` appends to a named project.
- `remove log 1` removes log 1 from the current project; `remove log 1 from Antenna Fabrication` targets a named project.
- `remind me I have food in 10 min`, `remind me to revise in 2 hours`, or `remind me to submit in 3 days` creates a local reminder.

Project logs are stored in `%LOCALAPPDATA%\TARS\projects.json`. Reminders are checked against the PC's local clock while TARS is running. Windows toast notifications use `winotify`; if unavailable, TARS falls back to an in-app notification dialog. Reminders do not fire while TARS is closed.


## Second Brain commands

- `remember <fact>` saves an explicit personal memory.
- `create project <name>` creates/selects a project.
- `load project <name>` loads that project's logs into the active model context.
- `unload project <name>` removes that project's logs from active context.
- `log <update>` appends a numbered log to the loaded project.
- `remove log <number>` removes a log from the loaded project.
- `remind me <task> in 30 min`, `remind me <task> at 9 pm`, or
  `remind me <task> tomorrow 9 am` schedules a local reminder.

Reminders trigger while TARS is running. The model is configured for an 8,192-token context, four CPU threads, a 512-token batch size, and a 768-token default response cap. The larger batch is intended to improve prompt processing, while the larger context preserves more conversation history. Actual speed depends on prompt length, RAM, and the CPU build; longer context can increase prompt-processing time.


## Response-generation reliability fix

- The streaming answer budget is now 2048 tokens, while the model context remains 32768 tokens. This is a maximum output allowance, not a requirement to generate 2048 tokens every time.
- The stream parser no longer discards a whole chunk merely because a backend includes a `reasoning_content` field alongside visible `content`.
- The system prompt explicitly tells TARS to honor requested quantities and complete deliverables (for example, all 10 requested programs) instead of returning only an introduction.
- These source changes pass syntax and mocked-stream tests; the real model/UI still needs testing on the target Windows computer.


## Final Second Brain UI update

The Second Brain tabs are ordered as:
1. How to use AI
2. Memories
3. Projects
4. Reminders
5. Creation of TARS

The dialog uses a refined dark theme, hover states, styled scrollbars, and a subtle fade when switching tabs. The Creation of TARS page records the Harish AI origins, the transition through Rudra to TARS, major design goals, setbacks, fixes, and the intended local-first second-brain direction. The timeline distinguishes intended features from behavior that still requires end-to-end testing.


## Conversation archive and UI responsiveness

- Removed the tab-wide window-opacity animation because it could stall repainting on some Windows graphics drivers. Tab changes now use native immediate transitions with styled hover/selection feedback.
- The Projects tab uses a horizontal split between projects and their numbered logbook to reduce unused vertical space.
- Added a searchable Chat Archive after the five requested Second Brain tabs. It searches persisted TARS chats, previews full messages, and lets the user continue an old chat.
- Query-aware retrieval adds relevant excerpts from older TARS chats to the model's system context. All saved messages remain stored locally; only relevant excerpts are injected into any one prompt because model context is finite.
- TARS cannot automatically access conversations from other ChatGPT sessions or chats that were never saved inside TARS. Importing/exporting an archive would be required for those.


## Switch between local AI models

TARS now discovers `.gguf` files in the `models` directory. Expand the dock and use the model selector beside the TARS title to switch models. Only one model is loaded at a time; changing models loads the selected model in a background thread and preserves the current agent conversation. The build script copies every `.gguf` file into the packaged application's `models` folder.

Supported: GGUF models compatible with the installed `llama-cpp-python` runtime. Models using unusual chat templates or architectures unsupported by the installed llama.cpp build may not work. Model files are large and are not included in the source ZIP.


## Final speed update: cached memory and fewer model passes

- `Agent` now keeps one `SecondBrain` instance for the app lifetime rather than constructing and loading a new one for every response.
- The profile, saved memories, open reminders and loaded project logbook are assembled into a cached context string. TARS checks lightweight file metadata on each request and re-reads those JSON files only if a relevant file changes.
- The system prompt is cached and rebuilt only when the persisted personal context changes.
- Past-chat archive retrieval is activated when the user refers to prior conversations (for example, "remember", "last time", "previous chat", or "what did I tell you"). Ordinary requests do not scan/reload the full chat archive.
- Internet mode no longer asks the local model to generate a JSON research plan before searching. It uses a focused query based on the current request and recent user context for short follow-ups, then generates the final answer. This removes one full local model inference from every web-enabled request.
- The configured 32,768-token context window and 2,048-token response budget are unchanged.

Performance depends on the selected model, CPU, prompt length and network/search provider. The source changes have been syntax-checked and cache behavior tested, but end-to-end latency should be measured on the target PC.


## Restored tab animation

The Second Brain tabs now use a short 155 ms OutCubic fade on the newly selected page. The effect animates the page's `QGraphicsOpacityEffect`, not `windowOpacity` on the non-top-level tab widget, so the transition is a real widget animation and avoids fading the entire window.


## Final UI preferences update

- The model selector is no longer shown in the main dock header. It appears in the Chat History dialog under "AI model".
- Internet Search starts OFF on application launch; users can turn it on per their preference.
- Added a final "Profile Memory" tab after Chat Archive. It provides readable editable fields for the profile name and one profile fact per line, saves to the existing local profile JSON, and invalidates the cached context so future answers use the update.


## UI preference: no model selector

The model-selection dropdown has been removed from the Chat History dialog as requested. Existing model-loading code and saved model preference remain unchanged, but there is no model-selection control in the UI.


## Fix: remove the floating model-name window

Removed the orphaned model dropdown widget and its show/hide calls. It had no parent layout placement but was shown when the TARS overlay expanded, allowing Qt to treat it as a separate top-level window with its own taskbar button. TARS continues loading the saved/default model at startup; only the stray model-name UI and in-app model switching controls are removed.
