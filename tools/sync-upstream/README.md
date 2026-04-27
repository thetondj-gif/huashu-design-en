# Sync from upstream

This is **maintainer-only** tooling. Regular users of this repo never need to run it.

## What it does

Pulls the latest commit from the upstream Chinese repo
(`alchaincyf/huashu-design`), translates any new or changed text files into
English, and writes them into this repo so you can review and commit.

The translation is content-addressed: each upstream file's bytes are hashed,
and the cache (`translation-cache.json`) maps that hash to the English
translation. Unchanged files reuse their cached translation, so syncs are stable
and only modified files round-trip through the LLM.

## How to run

```bash
export ANTHROPIC_API_KEY=sk-ant-...
pip install anthropic   # one-off
python tools/sync-upstream/sync.py
git status
git diff
git add -A && git commit -m "sync: upstream <short-sha>"
```

If upstream hasn't moved, the script exits with "Already synced".

## Files

| File | Purpose |
|---|---|
| `sync.py` | Pipeline implementation |
| `upstream-state.json` | Records `upstream_url` + `last_synced_sha` |
| `translation-cache.json` | `sha256(upstream bytes) → english content` |
| `.upstream-clone/` | Local mirror of upstream (gitignored) |

## Why no `git merge upstream`?

A line-by-line merge between Chinese upstream and English fork produces
conflicts on essentially every translated line — there's no common ancestor
on the text. This pipeline treats upstream as an *input*, not a git parent:
no merge ever happens, so no conflicts ever happen.

## When translation goes wrong

If a translated file looks broken in the diff, just delete its entry from
`translation-cache.json` and re-run `sync.py` — only that file will be
re-translated. To force-retranslate everything, replace `translation-cache.json`
with `{}`.
