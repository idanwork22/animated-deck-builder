---
name: animated-deck-builder
description: >-
  Build self-contained HTML presentation decks with hand-authored JS/CSS
  animations AND draw.io-style AWS architecture diagrams, in the style of Idan
  Chen's conference talks (DataTLV "Live Shift", EMR CR decks). Use when the
  user wants an animated slide deck, a cinematic/visual-first presentation, an
  explanatory motion graphic on a slide, or an AWS architecture diagram inside a
  deck. One offline .html file, dark + bright themes, Amazon Ember, presenter
  mode, speaker notes.
version: 1.0.0
---

# Animated Deck Builder

Build **one self-contained HTML file** that is a full presentation: keyboard/clicker
navigation, dark **and** bright themes off CSS variables, Amazon Ember typography,
hand-authored CSS/JS animations that explain a concept, speaker notes, a progress
bar, and a presenter window. AWS architecture diagrams are produced as native
`.drawio` XML (via the companion skill) and embedded as images.

This skill encodes a technique proven across real conference decks. Follow it; do
not invent a different architecture.

## When to use

- "Make an animated deck / slide / explainer"
- "Turn this concept into something visual that moves"
- "Build a presentation like my DataTLV / CR deck"
- "Add an AWS architecture diagram to a slide"
- "A cinematic, visual-first talk, not text-heavy bullets"

## Non-negotiable rules (learned the hard way)

1. **Dark AND bright.** Everything is driven by CSS variables from a `:root` block.
   Author ONE deck; emit two files by swapping only the `:root` palette. Never a
   fixed hex on a surface. Always set a background together with its text colour,
   or dark mode renders unreadable. Accent hues (green/orange/blue/red/amber) stay
   identical across themes so brand holds.
2. **Amazon Ember everywhere.** Titles weight 800, tight letter-spacing (-1.5px);
   small uppercase kickers wide-spaced (.16em). Code stays Menlo / SF Mono.
3. **No em dashes** (`—`, `&mdash;`, `\u2014`) anywhere in slide text. Use a comma,
   colon, or hyphen.
4. **No tiny text.** Minimum body ~22px at 1920x1080. Relocate footnotes to
   speaker notes rather than shrinking. A legal footer may stay small.
5. **No arrow glyphs** (`->`) in prose/titles/takeaways. Write the word ("NoSQL to
   SQL", "Snapshot, stream, cutover"). Functional UI arrows and in-graph axis cues
   are fine.
6. **Visual-first.** Prefer a graph/diagram/animation the presenter narrates over a
   text-heavy slide. If a slide is all bullets, it is wrong.
7. **No count-up number animations** unless asked. Numbers show their final value
   immediately; reveal-on-click is the preferred progressive disclosure.
8. **Verify motion in a REAL browser** (see Verification). Headless Chrome freezes
   `requestAnimationFrame` and `--virtual-time-budget`, so a screenshot shows a
   static or mid-fade frame and "proves" nothing. For motion you must capture,
   prefer `setTimeout`-based loops over rAF.
9. **One source of truth per visual.** A reusable visual (a timeline, a log strip,
   an architecture block) is authored once and embedded by reference, never pasted
   twice.
10. **1920x1080 fixed stage, scaled to fit.** The deck lives at a fixed
    1920x1080 and is CSS-scaled into the viewport, so layout never reflows.

## Deck architecture

Mirror the reference `build_html.py` structure:

- A `#deck` element fixed at 1920x1080, `transform: scale()` to fit the window
  (`fit()` on resize).
- Each slide is a `<section>`; `.active` fades it in (`opacity .35s`).
- `:root` holds the palette. Build the page twice, substituting the dark vs bright
  `:root` block; write two files (`deck.html`, `deck bright.html`).
- Navigation JS: Right/Space/PageDown/click = next, Left/PageUp = prev, Home/End,
  `F` fullscreen, `N` notes overlay, `P` presenter window, `#7` in URL jumps to
  slide 7. A `BroadcastChannel` syncs the presenter window.
- Speaker notes live in a `NOTES` JS array, shown in an overlay and the presenter
  window.
- Fonts embedded as base64 `@font-face` so the file works offline.

See `reference/deck-skeleton.html` for a complete, runnable minimal deck with the
palette, nav, presenter mode, and ONE example animated slide. Copy it and grow it.

### The palette block (copy verbatim, then tune)

```
_ACCENTS = --green:#3DDC84;--orange:#FF9900;--blue:#60A5FA;--red:#F87171;--amber:#FBBF24;--purple:#C084FC;--ink:#0B1220
dark   :root{--bg:#0B1220;--panel:#131C2E;--panel2:#0E1628;--grid:#24324A;--txt:#F8FAFC;--txt2:#CBD5E1;--mut:#94A3B8; <accents>}
bright :root{--bg:rgb(228,230,234);--panel:#FFFFFF;--panel2:#EEF1F6;--grid:#C3CCD9;--txt:#0B1220;--txt2:#334155;--mut:#5B6B82; <accents>}
```

## Authoring an animated slide

The style is **explanatory motion**: a diagram whose pieces appear, move, or pulse
in sequence to carry one idea, narrated live. Patterns that work:

- **Staged reveal** — elements fade/slide in on a stagger (CSS `animation-delay`).
  Re-trigger on slide entry by toggling a `.play` class (`void el.offsetWidth` to
  restart the animation). The skeleton's `show()` does this via `.replay`.
- **Flow/draining** — a dot or bar travels a path to show data moving (log to sink,
  snapshot draining). Use `setTimeout` loops, not rAF, so it is capturable.
- **Transform** — one shape morphs into another (a document becoming a table) to
  show a schema decision. CSS transitions on width/opacity between two states.
- **Pulse / now-ring** — a soft `box-shadow` pulse marks the "now" element.

`reference/animation-cookbook.css` has ready-to-paste keyframes: `fadeUp`,
`stagger-children`, `travel-dot`, `pulse-ring`, `morph-reveal`, `type-on`. Each is
commented with when to use it and whether it is headless-capturable.

Keep every surface on `var(--panel)` / `var(--grid)` / `var(--txt)` so both themes
work. Test BOTH before claiming done.

## AWS architecture diagrams (draw.io)

Do **not** hand-write `.drawio` XML from memory; AI-guessed stencil names render as
empty boxes. Use the companion skill that carries a verified 270+ icon catalog.
**It is bundled inside this skill** at `vendor/aws-architecture-diagram-skill/`:

- **Bundled (default):** read `vendor/aws-architecture-diagram-skill/SKILL.md` and
  use `vendor/aws-architecture-diagram-skill/references/*.md` for the verified
  stencil names and `templates/*.drawio` as starting points. No install needed.
- **Or install upstream** for updates:
  `npx skills add vidanov/aws-architecture-diagram-skill --agent kiro-cli`.
- Ask it for the diagram ("an event-driven order pipeline with SQS, Lambda,
  DynamoDB"). It emits a `.drawio` file with correct stencils, left-to-right flow,
  78px icons, 220px spacing.
- Render the `.drawio` to PNG (draw.io desktop `--export`, or the drawio CLI), crop
  the title band, and embed as a base64 data-URI `<img>` on the slide — exactly how
  `build_html.py` does it (`resolve_images`, `_slide_crop_light`).
- Produce a **light variant** of each diagram for the bright deck
  (`DRAWIO_THEME=light`), so diagrams match the theme.

`reference/drawio-embed.md` has the full render-crop-embed recipe and the icon
stencil gotchas (service-level vs resource-level `strokeColor`, known broken
stencils like `dynamodb_table`).

## Verification (always, before saying done)

1. **Themes:** open both `deck.html` and `deck bright.html`. Every surface legible
   in each; no fixed-hex surface bleeding through.
2. **Motion:** verify animations in a real-time browser reading live DOM, not a
   headless screenshot. Use the `web-verify` skill or a DevTools-driven probe. If a
   build pipeline blocks screenshots, fall back to measuring the exported PDF
   geometry (see `auto/verify-html-deck-content-swap-by-residual-scan-and-pdf-geometry`).
3. **Lint the rules:** grep the output for em dashes, `->` arrows in prose, and any
   body font-size below 22px. Fix before shipping.
4. **Offline:** the file opens with no network (fonts + images are base64).

## Sharing this skill

- **Local:** copy `~/.kiro/crew/skills/animated-deck-builder/` to another machine's
  skills dir, or zip and hand it over.
- **Publish:** from the dashboard, Settings → Skills → Discover publishes to the
  public skills.sh registry so anyone installs it with one command. Or use the
  `crystallize` skill to package the current session's technique into a candidate.

## Files in this skill

- `reference/deck-skeleton.html` — minimal runnable deck: palette, nav, presenter
  mode, one animated slide. Start here.
- `reference/animation-cookbook.css` — paste-ready keyframes with usage notes.
- `reference/drawio-embed.md` — render/crop/embed AWS diagrams, both themes.
- `reference/rules-checklist.md` — the pre-ship lint checklist.
- `vendor/aws-architecture-diagram-skill/` — bundled verified-catalog draw.io skill
  (MIT, upstream: vidanov/aws-architecture-diagram-skill) so AWS diagrams work
  without a separate install.
