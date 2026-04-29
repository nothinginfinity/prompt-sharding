# Prompt Sharding — Roadmap

> Last updated: 2026-04-28

---

## Current State (v0.1 Live)

- `index.html` — landing page live on GitHub Pages
- `docs/` — spec, quickstart, threat model, enterprise guide (stubs)
- GitHub as the durable ledger (no servers, no database)
- Signed envelope protocol defined
- `shards.json` routing schema defined

---

## Next Build: Interactive Demo Section (`#demo`)

### Overview

Add a `#demo` section to `index.html` that lets a visitor experience Prompt Sharding
through 5 Perplexity Spaces without writing any code. The demo is purely copy-paste
instructions + GitHub. No backend. No API keys. Perplexity account required.

---

### Part 1 — Animated Walkthrough (zero account needed)

- CSS/JS animated flow diagram embedded in `index.html`
- Shows a message moving through 5 roles: Architect → Designer → Writer → Reviewer → Assembler
- GitHub commit log appears alongside each step to illustrate the ledger
- Purely illustrative — no live data, works for everyone including visitors who haven't set anything up yet
- Goal: make the protocol *visual* before asking anyone to do anything

---

### Part 2 — Live Setup (Perplexity account required)

#### The Scenario: "Build a tiny website across 5 Spaces"

A visitor creates 5 Perplexity Spaces, each with a dedicated role in building
a simple landing page. The visitor plays "postman" — copying the outbox of one
Space and pasting it into the inbox of the next. GitHub records every move.

#### The 5 Roles

| Space Name | Role | Responsibility |
|---|---|---|
| `ps-architect` | Architect | HTML structure only — no CSS, no copy |
| `ps-designer` | Designer | CSS and visual design only |
| `ps-writer` | Copywriter | Headlines, body copy, CTAs |
| `ps-reviewer` | Reviewer | Reviews combined output, flags issues |
| `ps-assembler` | Assembler | Merges all outputs into final `index.html` |

#### UI on the Landing Page

- 5 expandable cards, one per Space
- Each card shows:
  - Space name to use (exact string, ready to copy)
  - **Copy button** for the full Space Instructions block
  - One-line instruction: "Paste into Perplexity → New Space → Instructions"
- Step-by-step numbered guide below the cards:
  1. Create all 5 Spaces using the instructions above
  2. Fork the demo repo (one click)
  3. Come back here and click **Start Demo** — we send the first task to `ps-architect`'s inbox
  4. Open `ps-architect` in Perplexity, ask it to check its inbox
  5. Copy its outbox into `ps-designer`'s inbox → continue through the chain
  6. When `ps-assembler` is done, your final `index.html` is in the ledger

---

### Space Instructions Format

Each Space gets a copy-paste block in this exact format (mirrors Studio-OS-Chat convention):

```
My Space name is: ps-architect
My repo: [YOUR_GITHUB_USERNAME]/prompt-sharding-demo
My role: Architect — I design HTML structure only. I do not write CSS or copy.
My outbox: spaces/ps-architect/outbox.md — I write here when I complete my task
My inbox: spaces/ps-architect/inbox.md — I read here to receive tasks
I send to designer at: spaces/ps-designer/inbox.md

When I receive a task:
1. Read my inbox
2. Design the HTML skeleton only (no styles, no copy — just structure and semantic elements)
3. Append my output to my outbox.md
4. Tell the user: "Done — now copy my outbox into ps-designer's inbox"

Rules:
- Only output HTML structure. No <style> tags. No inline styles. No placeholder copy.
- Every output must start with a one-line summary: "# Architect output — [task name]"
- Sign your output: append "— ps-architect" at the end
```

All 5 instruction blocks will live in `demo/space-instructions/` as individual `.md` files
so they can also be linked directly without opening the landing page.

---

### Demo Repo Structure (new: `prompt-sharding-demo`)

A separate lightweight repo visitors fork to run the demo:

```
prompt-sharding-demo/
├── README.md                        ← Fork me to run the demo
├── spaces/
│   ├── ps-architect/
│   │   ├── inbox.md
│   │   └── outbox.md
│   ├── ps-designer/
│   │   ├── inbox.md
│   │   └── outbox.md
│   ├── ps-writer/
│   │   ├── inbox.md
│   │   └── outbox.md
│   ├── ps-reviewer/
│   │   ├── inbox.md
│   │   └── outbox.md
│   └── ps-assembler/
│       ├── inbox.md
│       └── outbox.md
├── demo/
│   ├── space-instructions/
│   │   ├── ps-architect.md
│   │   ├── ps-designer.md
│   │   ├── ps-writer.md
│   │   ├── ps-reviewer.md
│   │   └── ps-assembler.md
│   └── task-01-website.md           ← The first task (sent to architect's inbox)
└── output/
    └── .gitkeep                     ← Final assembled files land here
```

---

### Multi-Provider Extension (v2 Demo)

Once the Perplexity-only demo is working and tested:

- Add Claude as `ps-reviewer` (separate provider, reviews the combined output)
- Add ChatGPT as `ps-assembler` (assembles without seeing individual shards)
- The landing page gets a "Provider" badge on each Space card showing which AI it runs on
- This is the moment the demo visually proves the core claim: *no single provider sees the whole*

---

## v1 Milestone Checklist

- [ ] `#demo` section added to `index.html`
- [ ] Animated flow diagram (CSS/JS, zero dependencies)
- [ ] 5 expandable Space instruction cards with copy buttons
- [ ] Step-by-step guide on the landing page
- [ ] `prompt-sharding-demo` repo created and public
- [ ] `demo/space-instructions/` — all 5 `.md` files written and tested
- [ ] Task 01 seed message written (`task-01-website.md`)
- [ ] Waitlist form connected (replace Tally placeholder in `index.html`)

## v2 Milestone Checklist

- [ ] Multi-provider demo (Claude + ChatGPT slots)
- [ ] Provider badge UI on Space cards
- [ ] Encrypted ledger storage (move beyond plaintext GitHub)
- [ ] Stronger envelope policy enforcement
- [ ] Enterprise guide fleshed out

---

## Design Decisions to Revisit

- **Demo repo vs. fork of main repo** — keeping `prompt-sharding-demo` separate makes forking lighter and keeps the main spec repo clean. Revisit if people want an all-in-one.
- **Copy button implementation** — use `navigator.clipboard.writeText()` with a fallback `<textarea>` select for older browsers. No third-party dependency.
- **"Start Demo" button** — for now this could simply pre-fill the first task into a text block the user manually copies into `ps-architect`'s inbox. No GitHub OAuth needed in v1.
- **Perplexity Spaces vs. other providers** — Spaces is the only current platform with file-uploadable instructions and a persistent system prompt, making it the right starting point. Claude Projects and GPT Custom Instructions are the v2 expansion.
