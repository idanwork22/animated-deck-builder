# Embedding draw.io AWS architecture diagrams in the deck

Diagrams are produced as native `.drawio` XML, rendered to PNG, cropped, and
embedded as a base64 data-URI so the deck stays one offline file. Do **not**
hand-write `.drawio` XML from memory: AI-guessed stencil names render as empty
boxes. Use the verified-catalog companion skill.

## 1. Get the companion skill (once)

```
npx skills add vidanov/aws-architecture-diagram-skill --agent kiro-cli
```

Or vendor it into the deck project (as the reference deck does) under
`.tools/aws-architecture-diagram-skill/` and read its `kiro/SKILL.md`. It carries
270+ verified stencil names, left-to-right flow, 78px icons, 220px spacing.

## 2. Generate the diagram

Ask the companion skill in plain language, e.g.:

> Create an AWS architecture diagram: Users to CloudFront to ALB to a Catalog API
> on ECS, MongoDB as the main store, Debezium/MSK CDC to OpenSearch and Aurora,
> Firehose to S3 to Athena. Left to right.

It writes `my-arch.drawio`.

## 3. Render to PNG (both themes)

Use draw.io desktop or the drawio CLI headless export. Render a **dark** and a
**light** variant so each matches its deck theme:

```bash
# dark (default)
drawio --export --format png --scale 1.5 --output my-arch.drawio.png my-arch.drawio
# light: set the diagram's page background / theme to light, then export
DRAWIO_THEME=light drawio --export --format png --scale 1.5 \
  --output my-arch.light.drawio.png my-arch.light.drawio
```

(If you generate diagrams programmatically like the reference `build/drawio.py`,
honour a `DRAWIO_THEME=light` env to pick a light palette.)

## 4. Crop the title band and whitespace

The reference crop (from `build_html.py _slide_crop_light`): drop the top ~100px
title block, then auto-trim to content bbox with ~30px padding. Python/Pillow:

```python
from PIL import Image, ImageChops
im = Image.open("my-arch.drawio.png").convert("RGB")
body = im.crop((0, int(100*1.5), im.width, im.height))          # drop title band
bg = Image.new("RGB", body.size, body.getpixel((2, 2)))
bbox = ImageChops.difference(body, bg).convert("L").point(lambda v: 255 if v>18 else 0).getbbox()
l,t,r,b = bbox; pad = 30
body.crop((max(l-pad,0), max(t-pad,0), min(r+pad,body.width), min(b+pad,body.height))).save("slide_my-arch.png")
```

## 5. Embed as a base64 data-URI

```python
import base64
def data_uri(p): return "data:image/png;base64," + base64.b64encode(open(p,"rb").read()).decode()
# in the dark page:  <img class="visual" src="{data_uri('slide_my-arch.png')}">
# in the bright page: swap to slide_my-arch.light.png
```

Give the `<img>` `max-width:100%;max-height:100%;object-fit:contain` inside a
`.stage` flex container so it scales within the 1920x1080 slide.

## Stencil gotchas (the empty-box causes)

- **Service-level** icon (coloured square + white glyph):
  `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.<name>` with
  `strokeColor=#ffffff` **required**.
- **Resource-level** icon (silhouette): `shape=mxgraph.aws4.<name>` with
  `strokeColor=none` **required**. Confusing these two is the #1 broken-icon cause.
- Renamed services keep old stencil ids: OpenSearch is still `elasticsearch_service`.
- Known broken: `dynamodb_table` (use `dynamodb`), `dynamodb_stream` (use
  `dynamodb` + label), `general_saml_token` (use `traditional_server`).

Let the companion skill own this mapping; it is why we delegate instead of guessing.
