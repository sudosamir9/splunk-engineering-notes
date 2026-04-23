# Module 3 — Log Journey: Endpoint to UI (End-to-End Trace)

> **Status: 🔜 Coming**

## Topics Covered Here
- Full trace of a single log event from endpoint disk to analyst screen
- Hop 1: UF reads file, tails it, adds metadata (host, source, sourcetype)
- Hop 2: UF compresses and TLS-encrypts, sends over TCP 9997
- Hop 3 (optional): HF receives, applies props.conf/transforms.conf, re-routes
- Hop 4: Indexer receives the data stream, parses into events, timestamps
- Indexer parsing pipeline: charset detection → line breaking → timestamp extraction → field extraction
- Hop 5: Indexer writes to hot bucket, replicates to peers
- Hop 6: Analyst query hits Search Head → dispatched to all indexers
- Hop 7: Indexers search local buckets, stream partial results back
- Hop 8: Search Head merges, sorts, applies `stats`/`eval`, renders in UI
- What each hop adds or transforms
- Where data can be lost at each hop (troubleshooting map)
- Protocol details: TCP 9997, port 8089, port 8000, TLS handshake

---

*Content added when we reach this module.*
