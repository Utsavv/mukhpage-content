# mukhpage-content

Public static content published by the MukhPage pipeline
([`Utsavv/MukhPage`](https://github.com/Utsavv/MukhPage) →
`sara_content_product/core/publishing/github_static.py`). Nothing here is
hand-written: the generator pushes files through the GitHub Contents API, and
they are servable immediately — no build, no deploy step.

## How the word gets on the website

```
main.py / web API  ──►  publish_story_files()  ──►  <topic>/<cache_key>.json
                                                    <topic>/<cache_key>.html
                        publish_pointer()      ──►  <topic>/latest.json
                                                    <topic>/index.json
                                                          │
                       mukh-page-ui  ◄── jsDelivr CDN ◄────┘
                       index.html    ◄── GitHub Pages ◄────┘
```

`latest.json` is the contract the website reads. It is rewritten on every
publish for the default (anonymous) audience, so a consumer never needs to know
cache keys — one fixed URL always holds the newest word.

## Serving URLs

| What | URL |
|------|-----|
| Latest word (JSON) | `https://cdn.jsdelivr.net/gh/Utsavv/mukhpage-content@main/word_of_the_day/latest.json` |
| Published archive index | `https://cdn.jsdelivr.net/gh/Utsavv/mukhpage-content@main/word_of_the_day/index.json` |
| One story (JSON) | `.../word_of_the_day/<cache_key>.json` |
| One story (shareable HTML) | `.../word_of_the_day/<cache_key>.html` |
| Live page | `https://utsavv.github.io/mukhpage-content/` (once Pages is enabled) |

jsDelivr caches `@main` for ~12 hours; the publisher purges each path it
overwrites, so `latest.json` refreshes as soon as a new word goes out.

### Enabling the live page

`index.html` renders `word_of_the_day/latest.json` client-side. To serve it:
**Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)`**. No
workflow file is needed — every publish commit redeploys the page, because the
publisher commits straight to `main`.

## File layout

```
index.html                     live page; fetches word_of_the_day/latest.json
.nojekyll                      serve files starting with "_" verbatim
word_of_the_day/
  latest.json                  newest published word (story + metadata)
  index.json                   catalogue of everything published for the topic
  <cache_key>.json             one story; key is <word>__<gender>__<age>__<exam>[__hash]
  <cache_key>.html             self-contained shareable page for that story
```

### `latest.json` shape

```jsonc
{
  "topic": "word_of_the_day",
  "item": "ephemeral",          // the word itself
  "word": "ephemeral",          // legacy alias the deployed UI reads
  "cache_key": "ephemeral__other__18_21__none",
  "personalized": false,        // true when the pointer holds a profiled story
  "profile": { "gender": "other", "age_group": "18-21", "exam_focus": "none" },
  "published_at": "2026-07-26T09:00:00+00:00",
  "static_url": "https://cdn.jsdelivr.net/.../<cache_key>.json",
  "static_html_url": "https://cdn.jsdelivr.net/.../<cache_key>.html",
  "story": { "word": "...", "hindi_meaning": "...", "quiz": {} }
}
```

The full story is embedded so the website renders from a single request.

## Publishing config (in `MukhPage/sara_content_product/core/.env`)

| Key | Default | Meaning |
|-----|---------|---------|
| `GITHUB_CONTENT_TOKEN` | — | Fine-grained PAT, Contents read/write on this repo. Unset ⇒ publishing is a logged no-op. |
| `GITHUB_CONTENT_REPO` | `Utsavv/mukhpage-content` | Target repo. |
| `GITHUB_CONTENT_BRANCH` | `main` | Target branch. |
