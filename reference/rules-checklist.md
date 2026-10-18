# Pre-ship checklist (run every one before saying done)

## Themes
- [ ] Both `deck.html` AND `deck bright.html` open and are fully legible.
- [ ] No fixed hex on a surface (bg/panel/card/text). Every surface uses a
      `var(--...)`. Grep: `grep -nE '#(fff|ffffff|000|000000|[0-9a-fA-F]{6})' deck.html`
      and confirm each hit is inside a `:root` block or an accent, not a surface.
- [ ] Every background has a matching text colour set alongside it.
- [ ] Accent hues identical across the two themes (brand holds).

## Typography & copy rules
- [ ] Amazon Ember (or the chosen brand face) on titles/body; Menlo/SF Mono on code.
- [ ] No em dashes. Grep: `grep -nE '—|&mdash;|\u2014' deck.html` → zero hits.
- [ ] No `->` arrow glyphs in prose/titles/takeaways (functional UI/axis cues OK).
      Grep: `grep -n '\->' deck.html` → review each hit.
- [ ] No body text below ~22px at 1920x1080. Footnotes moved to speaker notes,
      not shrunk. Legal footer may stay small.

## Content
- [ ] Visual-first: no slide is a wall of bullets. Each carries a graph/diagram/
      animation the presenter narrates.
- [ ] No count-up (data-count) number animations unless explicitly requested.
- [ ] Reusable visuals authored once, embedded by reference (not pasted twice).

## Motion (the one people get wrong)
- [ ] Animations verified in a REAL-TIME browser reading live DOM — NOT a headless
      screenshot (headless freezes rAF/CSS and shows a static/mid-fade frame).
      Use the `web-verify` skill or a DevTools-driven probe.
- [ ] For any motion captured to a still/GIF, it is driven by a setTimeout loop,
      not requestAnimationFrame.
- [ ] Each animated slide re-triggers its animation on entry (`.replay`/`.play`
      class toggle with `void el.offsetWidth`).

## Diagrams
- [ ] AWS diagrams generated via the verified-catalog companion skill, not
      hand-written XML. No empty-box icons on render.
- [ ] A light variant exists for each diagram, embedded in the bright deck.

## Offline
- [ ] File opens with the network off: fonts + images are base64 data-URIs.

## Navigation
- [ ] Right/Space/click = next, Left = prev, Home/End, F fullscreen, N notes,
      P presenter, `#n` URL jump all work.
- [ ] Speaker notes present for every slide that needs them.
