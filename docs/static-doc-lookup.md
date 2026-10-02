# Static Documentation & Resource Lookup

**Goal:** read static event documentation (rules pages, challenge briefs, API
docs) for the least token cost, so we don't burn AI budget on a 10k-token page
when 800 tokens of clean text would do.

## When to use a text-only wrapper

- The page is mostly prose/tables you just need to *read*, not interact with.
- You want the content in context without screenshots or the full DOM.
- You're doing repeated lookups and want each one cheap.

## Option A — PinchTab

PinchTab is a standalone HTTP server that gives an agent control of a Chrome
tab, with a text-extraction mode built for token efficiency.

```bash
# Navigate, then pull readable text (strips nav/footer/ads)
pinchtab nav https://event-site.example/rules
pinchtab text          # ~800 tokens instead of ~10,000

# Raw innerText (keeps table/block structure via tabs + newlines)
pinchtab text --raw
```

HTTP API equivalent:

```bash
curl /text              # readability mode (default)
curl "/text?mode=raw"   # full innerText
```

`GET /text` returns `{ url, title, text }`. Default mode runs readability to
drop navigation and ads; `mode=raw` gives the unfiltered innerText, preserving
block and table-cell boundaries.

**Efficiency tips (from PinchTab's agent-optimization notes):**
- Use `text` for extraction tasks rather than a full DOM `snap`.
- For multi-step flows, `--snap-diff` returns only changed elements.
- `snap -s <selector>` scopes a snapshot to one section when you know where to look.

Source: https://github.com/pinchtab/pinchtab

## Option B — native WebFetch

For a one-off read of a public static page, the built-in `WebFetch` tool fetches
a URL and returns processed/markdown-ish content directly — no server to run.
Good for: a single rules page or a linked spec. Reach for PinchTab instead when
you need a persistent session, repeated reads, or the same tab across steps.

## Rule of thumb

| Situation | Pick |
|-----------|------|
| One public static page, quick read | WebFetch |
| Many reads / persistent tab / scoped sections | PinchTab `text` |
| Need to *interact*, not just read | see high-level-automation.md |
