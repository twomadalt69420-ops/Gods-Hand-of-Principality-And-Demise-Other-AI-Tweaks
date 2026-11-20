# Explicit note events — no virtual multiplicity

This branch adds an **opt-in strict mode** for Psych-like JSON charts.

## JSON contract

Set `"ghRealNotes": true` in the `song` object. Every element in `notes[].sectionNotes` must be one real event, such as `[timeMs, noteData, sustainMs]` or `[timeMs, noteData, sustainMs, noteType]`. Two identical entries still count as two independent events. Each imported note has `stack == 1`; sorting may change order, but neither the JSON DOM nor the streaming importer collapses entries within 1 ms. `ghStack64` and fifth-column multiplicities are explicitly rejected in this mode.

**Note:** The historical stacking import path remains enabled for older charts *without* `ghRealNotes`. To guarantee no merging, explicitly opt in. The original stack64 example and mod must **not** be used as proof of actual 3.56-trillion individual entries.

## 64-bit indexing and memory

`StreamedNotes::rangeStart` now stores `uint64_t` offsets instead of `uint32_t`. Binary `.ghc` v3 already encodes section-offset deltas with variable-width unsigned integers; its reader and writer now preserve 64-bit values instead of truncating them. The on-disk layout is unchanged. The chart stream-cache version was bumped to 3 because its in-memory offsets widened.

This does **not** prove practical loading of trillions of simultaneous C++ `Note` objects. `Chart::notes` and `StreamedNotes::notes` are still RAM-resident vectors. An external-memory, on-demand page reader, indexed storage, and judging/rendering architecture would be required to serve the whole expanded playlist within realistic RAM. `ghRealNotes` is an event-preservation behavior, not a performance or capacity guarantee.

## Current source and export status

The separate, user-owned Drive directory `CHART REAL COM TODOS OS PROCEDIMENTOS` contains a lossless MIDI24/FL Studio Playlist source: 57 patterns, 50,056 clips, 168,623,461 original 24-byte note records, and a **3,560,023,825,730-event Playlist expansion**. The source is a recipe for expansion — it is not itself a materialized 3.56T-event Psych JSON.

As of 2026-11-20 the previously published 8 ZIPs declared **219,671,530** nonoverlapping individually serialized notes. Two further distinct clips from pattern 54, indexes 893 and 894, each added **72,427,348** serialized events; the latest clip includes `ghRealNotes`. **364,526,226** have thus been exported as individually serialized entries; **3,559,659,299,504** have not. The current files are separate **partial** ZIPs, not one integral playable chart.

The independent Python converter `exportar_playlist_REALMODE_GH.py` expands each Playlist clip from native 24-byte FL Studio records into individual Psych-like triples. It preserves one triple per record, but uses a **provisional RAW100K pitch mapping**, not the original 4-direction layout. In the engine, `keyCount` is currently limited to fewer than 100 keys; therefore these RAW100K files are primarily **source/audit fragments**, not accurately playable song charts. A verified mapping from FL Studio voices/instruments to four on-screen directions is not supplied. Pitch modulo 4 would be an invented mapping; none was performed.

The Rust SNIFF source (`JordanSantiagoYT/SNIFF`) was used as a reference to the FLP 24-byte note representation. **The SNIFF executable was not run** for the exports. Do not label them 'made by SNIFF'.

## Validation and limits

New unit tests in `tests/psych_tests.cpp` check DOM and streamed strict mode, equal-timestamp duplicates, 1-entry/1-note accounting, and refusal of `ghStack64` and fifth note columns. These tests were **added but have not yet been executed in a compiled build**. There was no demonstration of end-to-end playback in the target engine.

For each ZIP: validate the CRC, count *actual array elements*, validate compressed and decompressed SHA-256 manifests and verify no fifth-column multiplicity. Never equate virtual pattern repetition or a summed count field with physically stored note entries.

## Correct use

See `examples/real-notes-5-events.json` as the executable schema example (five individually written notes). Until the complete Playlist is exported, audited and all source-to-animation correspondences are demonstrated, do not describe the result as the exact original 3.56T-note 4K Overkill chart.
