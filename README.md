# transparency-log

Append-only checkpoints of a hash-chained log, one line per day in
`checkpoints.jsonl`:

```json
{"seq": 1234, "hash": "<sha256 hex>", "at": "2026-10-05T16:00:03.000Z",
 "kid": "<key id>", "sig": "<base64>",
 "peers": {"rekor": {"tree": "<id>", "size": 0, "root": "<hex>"},
           "gosum": {"size": 0, "root": "<hex>"}}}
```

`hash` is the chain's head after entry `seq`, where each entry's hash is
`sha256(previous hash + "\n" + entry text)`. The log can't be rewritten without
breaking a hash already recorded here, so lines are only ever appended. Older
lines have only `seq`, `hash` and `at`.

## Signature

`sig` is an Ed25519 signature over

```
audit-checkpoint/v1
emma-stensland.com/audit
<seq>
<hash>
<at>
```

(each line ending in `\n`) by the key named by `kid`. Public keys
(`alg kid base64`):

```
Ed25519 2d37782cd42b6867 okl47wvhj5KJLSvGCRO0nDm5CMULUImMJFh++vRCHEU=
```

## Witnesses

This repository's history is one witness, but the log's owner can write to it.
These can't be written to by the owner:

- `peers`: the size and root of Sigstore's Rekor log and Go's checksum
  database when the checkpoint was made. A root at a given size can't be known
  before then, so the line is no older than those heads.
- `ots/<seq>-<calendar>.ots`: an [OpenTimestamps](https://opentimestamps.org)
  proof for the sha256 of that day's exact line (without the newline), from two
  independent calendars, anchored in Bitcoin. Check one with
  `ots verify -d <sha256 of the line> <file>` (run `ots upgrade` first if it's
  still pending).
- The Wayback Machine keeps a copy of `checkpoints.jsonl` at each checkpoint's
  commit, under `raw.githubusercontent.com/stenslae/transparency-log/<commit>/`.

Copies of this file kept anywhere else make a rewrite detectable too.
