# Animated Deck Builder

Build a **single self-contained HTML presentation** with hand-authored JS/CSS
animations and draw.io-style AWS architecture diagrams. Dark **and** bright themes
off CSS variables, Amazon Ember typography, keyboard/clicker navigation, presenter
mode, speaker notes, progress bar. One offline file, no build server required.

A skill for AI coding agents (KiroCrew, Kiro CLI, Amazon Q / Quick, Claude Code,
Cursor). Drop it in and ask for an animated, visual-first deck.

## Install

```bash
# from the skills.sh registry (once published)
npx skills add idanwork22/animated-deck-builder

# target a specific agent
npx skills add idanwork22/animated-deck-builder --agent kiro-cli
npx skills add idanwork22/animated-deck-builder --global
```

Or copy the folder into a skills directory manually:

- KiroCrew: `~/.kiro/crew/skills/animated-deck-builder/`
- Kiro CLI / Amazon Q: `~/.kiro/skills/animated-deck-builder/` (global) or
  `.kiro/skills/animated-deck-builder/` (per project)

## What it gives you

- **1920x1080 scaled stage** that never reflows, fit to any window.
- **Two themes from one deck** — swap only the `:root` palette; every surface is a
  CSS variable so dark and bright both stay legible.
- **Explanatory motion** — a cookbook of paste-ready keyframes (staged reveal,
  travelling dot, pulse ring, morph reveal, type-on), each marked for whether it
  survives a headless screenshot.
- **Presenter mode** — a second window with notes, next-slide, and a timer, synced
  over BroadcastChannel.
- **AWS diagrams that render** — bundled verified 270+ icon catalog
  (MIT, from [vidanov/aws-architecture-diagram-skill](https://github.com/vidanov/aws-architecture-diagram-skill)),
  so icons never come out as empty boxes.
- **A pre-ship lint checklist** — no em dashes, no tiny text, no arrow glyphs in
  prose, both themes verified, motion verified in a real browser.

## Try it

Open `examples/cdc-demo-deck.html` in a browser. Arrow keys or click to move;
`T` toggles dark/bright, `F` fullscreen, `N` notes, `P` presenter.

## Structure

```
animated-deck-builder/
├── SKILL.md                         # the technique + rules
├── skills.sh.json                   # registry manifest
├── reference/
│   ├── deck-skeleton.html           # minimal runnable deck to copy
│   ├── animation-cookbook.css       # paste-ready keyframes
│   ├── drawio-embed.md              # render/crop/embed AWS diagrams, both themes
│   └── rules-checklist.md           # pre-ship lint
├── examples/
│   └── cdc-demo-deck.html           # a working 4-slide demo
└── vendor/
    └── aws-architecture-diagram-skill/   # bundled verified icon catalog (MIT)
```

## License

MIT. Bundled `vendor/aws-architecture-diagram-skill` is MIT, (c) its authors.
