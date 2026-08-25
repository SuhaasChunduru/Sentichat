# SentiChat

A chat interface that shows the emotional arc of a conversation while you have it.

Every message is scored for sentiment and hung off a coloured rail down the left of
the thread — **amber** when the tone is warm, **indigo** when it turns cool, **stone**
when it's even. Stacked together the rails form one continuous ribbon, so you can
scroll back and see exactly where the mood changed. A gauge in the sidebar tracks
the running temperature of the whole conversation.

Everything lives in one file, [`Sentichat.html`](Sentichat.html). No build step, no
server, no account.

## Run it

```bash
open Sentichat.html
```

It works immediately in **demo mode**: replies are generated locally and respond to
your tone, so the sentiment features are fully usable with no API key.

To get real model replies, open **Settings** and paste an
[OpenRouter](https://openrouter.ai/keys) key. The key is stored in your browser's
`localStorage` and never leaves the page except in the request to OpenRouter.

> **On keys in a browser page:** anyone with access to this file or this device can
> read the key. Use a spend-limited one, and never commit a key to a repository.
> For anything public, put a small server between the page and the API instead.

Some browsers block `localStorage` on `file://` URLs. If your characters and
conversations don't survive a reload, serve the folder over HTTP:

```bash
python3 .claude/serve.py
```

Then open <http://127.0.0.1:8412/Sentichat.html>.

## What it does

**Reading the tone**

- **Sentiment ribbon** — per-message tone, rendered as the spine of the thread.
- **Live preview** — the composer tells you how your message reads *before* you send it.
- **Why this tone?** — the `?` under any message opens the scoring breakdown: every
  word that counted, what it was worth, and which negator or intensifier changed it.
  "not happy" shows `happy · flipped by "not" · −1.6`.
- **Tone gauge** — running conversation temperature, weighted toward recent messages.
- **Conversation arc** — the chart button summarises the whole exchange: overall tone,
  the warm/even/cool mix, the warmest and coolest lines, and where the mood turned.

**Talking**

- **Streaming replies** — text appears as the model produces it (with a key set).
- **Dictation** — the mic uses the browser's built-in speech recognition. No
  dependency, no upload; the button hides itself where the browser lacks support.
- **Per-message actions** — copy, delete, and regenerate the latest reply.
- **Characters** — three are included; add, edit (double-click), and delete your own.
  Each one's description becomes its system prompt. `Cmd`/`Ctrl` + `1`–`9` switches.
- **Separate conversations** — each character keeps their own history, saved locally.
- **Export** — download the conversation as Markdown, tone annotations included.

**Everything else**

- **Light and dark** — follows your system setting, with a manual override.
- **Markdown replies**, sanitised before they touch the page.
- Keyboard navigable, `prefers-reduced-motion` respected, AA contrast in both themes.

## How the sentiment scoring works

`scoreText()` walks the message and adds up hits against a positive/negative word
list, with two corrections that a plain word count gets wrong:

- **Negation** — "not great" scores negative, not positive. The three words before
  each hit are checked for negators.
- **Intensifiers** — "really good" counts for more than "good".

Emoji are matched as substrings, because `\b` word boundaries never match emoji.

Scores are clamped per message so one emphatic sentence can't peg the gauge.
`GAUGE_GAIN` in the script controls how hard the needle swings — raise it if the
gauge feels sluggish, lower it if a single message dominates.

The lexicon is deliberately small and readable. It is not a trained model, and it
will miss sarcasm, idiom, and context.

## Layout of the file

| Section | What's in it |
|---|---|
| `<style>` | Design tokens, then components top-down. Both themes are defined as token sets. |
| `1. store` | Everything persists to `localStorage`; nothing goes to a server. |
| `2. sentiment` | The lexicon and `scoreDetail()` / `toneOf()` / `conversationTone()`. |
| `3. rendering` | Message nodes, action bars, the ribbon, the gauge, the character list. |
| `4. reply` | OpenRouter request (streaming and not), and the local demo responder. |

`scoreDetail()` returns both the score and the list of hits behind it, which is what
the "why this tone?" panel renders — the explanation is the scorer's own output, not
a second implementation that could drift from it.

## Notes

- `Sentichat.original.html` is the previous version, kept for reference. The API key
  it contained has been redacted — **if you haven't already, revoke it** at
  [openrouter.ai/keys](https://openrouter.ai/keys), since it was committed in plain text.
- [`Project Report.pdf`](Project%20Report.pdf) is the accompanying project report and
  describes the earlier version.
