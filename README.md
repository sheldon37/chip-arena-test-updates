# Chip Arena — test update channel

**Test-only.** This public repo is the update channel for Chip Arena **TEST-ONLY** builds.
It is not a player release and is not the live update feed. The game source stays in the
private `sheldon37/Chip-Arena` repo. Anyone with the URL can download these test game files.

## How it works

TEST-ONLY builds made with `--channel` read their updates from this repo:

```
https://raw.githubusercontent.com/sheldon37/chip-arena-test-updates/main/feed/
```

- `feed/latest.json` is the manifest. The Update button and the menu update badge read it.
- `feed/arena-<16 hex>.json.gz` is the payload that `latest.json` names.

The production updater (`launcher/update.mjs`) is unchanged. It verifies the manifest schema,
the payload SHA256 and a 32 MB size cap. A missing or bad channel reports "unavailable" and
never falls back to the live feed. Raw GitHub files can be cached for about 5 minutes, so a
new version may take a few minutes to appear.

Tooling lives on branch `claude/test-update-channel` of the private repo, which is based on
`release/candidate-workflow` (PR #3). See `scripts/release/remote-feed.mjs`,
`test-remote-update.mjs`, `test-remote-status.cjs`, `test-remote-bootstrap.cjs`, and
`docs/RELEASE-CANDIDATES.md` ("Remote test channel").

## Publishing a new test version (Claude or Astra)

1. Build a candidate from an exact commit with the channel flag. This needs Node 24.19.0,
   npm 11.9.0 and Python 3:

   ```
   python scripts/release/candidate.py --commit <40-char SHA> --output <new dir> --test-only \
     --channel https://raw.githubusercontent.com/sheldon37/chip-arena-test-updates/main/feed/
   ```

   You only need the ZIP once per tester, as the seed install. Later versions only need their
   `feed/` files.
2. In this repo, replace `feed/` with the candidate's `feed/latest.json` and its single
   `arena-*.json.gz`. Delete the old payload so exactly one payload remains.
3. Commit, naming the source commit and content version (see `PUBLISHED.md`), then push to `main`.
4. Add a line to `PUBLISHED.md`, and record the version in the private repo's `docs/DEV_LOG.md`.

Rules: never put credentials, player data or signing keys here. Only publish builds from
reviewed branch commits. Moving these builds to the live feed is a separate, owner-approved
release.
