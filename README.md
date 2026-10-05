# transparency-log

Append-only checkpoints of a hash-chained log, one line per day in
`checkpoints.jsonl`:

```json
{"seq": 1234, "hash": "<sha256 hex>", "at": "2026-10-05T16:00:03.000Z"}
```

`hash` is the chain's head after entry `seq`, where each entry's hash is
`sha256(previous hash + "\n" + entry text)`. The log can't be rewritten without
breaking a hash already recorded here. This repository's history is the witness,
so lines are only ever appended.
