# JARVIS-X — Voice Core

This is the **first working slice** of the JARVIS-X project: a voice-first
assistant loop that runs entirely in the browser, with no backend, no API
keys, and no build dependencies beyond Vite + TypeScript.

It is deliberately narrow. It does **not** include gesture control, desktop
automation, Supabase-backed cloud memory, or real LLM calls yet — those are
separate milestones (see [Roadmap](#roadmap)). What it does include is a
fully working, testable voice loop with a memory interface designed so the
mock pieces can be swapped for real ones without touching the rest of the app.

## What it does

1. On first launch, asks **"What should I call you?"** (spoken aloud, typed
   in a prompt for reliability) and remembers your name in `localStorage`.
2. On every future launch, greets you by name — no re-asking.
3. Listens continuously for the wake word **"Jarvis"**.
4. Once woken, captures your spoken command, sends it to a small rule-based
   "brain" (`MockConversationEngine`), and speaks the reply back via
   text-to-speech.
5. Logs every exchange in a visible transcript and in `localStorage`
   (`chat_history` equivalent).
6. Supports a few real commands out of the box: `"what time is it"`,
   `"what's the date"`, `"remember that I like X"`, `"what do you remember
   about me"`, `"forget everything"`.

## Why it's structured this way

The whole point of this slice is the **seam between voice/UI and
brain/memory**. Look at `src/types.ts`:

- `MemoryStore` is the contract `MockMemoryStore` implements today
  (backed by `localStorage`). The Supabase milestone implements the same
  contract against real tables (`users`, `user_preferences`, `memories`,
  `chat_history`) — the voice loop and UI don't change at all.
- `ConversationEngine` is the contract `MockConversationEngine` implements
  today (simple if/else rules). The AI Tool Integration Hub milestone
  implements the same contract by calling OpenRouter, OpenAI, Claude,
  Gemini, DeepSeek, or a local Ollama model — again, nothing upstream
  changes.

This is the pattern the rest of JARVIS-X should follow: build the real
interaction loop against a mock, define the interface precisely, then swap
implementations underneath it.

## Getting started

Requires Node.js 18+.

```bash
npm install
npm run dev
```

Open the printed local URL in **Chrome or Edge on desktop** — Safari and
Firefox don't reliably support the `SpeechRecognition` API this prototype
uses for wake-word detection. Grant microphone access when prompted, click
**"Start listening"**, and say "Jarvis" followed by a command.

```bash
npm run build     # type-checks and produces a static dist/ bundle
npm run preview   # serves the production build locally
```

## Project structure

```
jarvis-x-voice-core/
├── index.html                 Entry HTML shell
├── src/
│   ├── main.ts                 State machine: wires everything together
│   ├── types.ts                 MemoryStore / ConversationEngine contracts
│   ├── speech.d.ts              Ambient types for the Web Speech API
│   ├── styles.css               Arc-reactor-inspired UI, state-driven
│   ├── voice/
│   │   ├── speechRecognition.ts  Wrapper around SpeechRecognition
│   │   ├── wakeWord.ts           Continuous listening + wake-word matching
│   │   └── textToSpeech.ts       Wrapper around speechSynthesis
│   ├── memory/
│   │   └── mockMemoryStore.ts    localStorage-backed MemoryStore
│   └── conversation/
│       └── conversationEngine.ts Rule-based ConversationEngine (mock brain)
├── package.json
├── tsconfig.json
└── README.md
```

## Known limitations (by design, for this slice)

- **Browser-only, Chromium-based.** No Whisper, no offline STT yet.
- **Single user, single device.** Memory lives in `localStorage`; clearing
  browser data forgets everything. This is what Supabase replaces.
- **Mock brain.** Replies are rule-based, not an LLM. See the swap-point
  comment at the top of `conversationEngine.ts`.
- **No wake-word privacy guarantees.** Continuous `SpeechRecognition` sends
  audio to the browser vendor's speech service (e.g. Google, for Chrome).
  A production build should document this clearly or use a local wake-word
  model instead.
- **No gesture control, desktop control, or plugin hub yet.** Those are
  separate, larger milestones — see below.

## Roadmap

| Stage | Milestone | Depends on this slice? |
|---|---|---|
| ✅ Done | Voice-first loop (wake word, STT, TTS, mock memory) | — |
| Next | Supabase memory backend (real `MemoryStore` implementation) | Yes — same interface |
| Next | AI Tool Integration Hub (real `ConversationEngine` implementations) | Yes — same interface |
| Later | Iron-Man-inspired HUD (Next.js, Framer Motion, themes) | Wraps this loop in a fuller UI |
| Later | Hand gesture control (MediaPipe) | Additive, separate module |
| Later | Screen/desktop control (open apps, manage files, Windows APIs) | Additive, separate module |
| Later | Local AI support (Ollama, Llama, Mistral, Gemma) | Implements `ConversationEngine` |
| Later | Cross-platform (macOS, Linux, Android, Web packaging) | Wraps the whole app |

## License

MIT — see `LICENSE` (add one at the repo root when you publish; this slice
doesn't include one on its own).
