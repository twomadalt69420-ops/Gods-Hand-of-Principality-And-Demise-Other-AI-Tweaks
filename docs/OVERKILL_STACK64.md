# Psych Overkill — 64-bit stacked-note charts

This fork supports large 64-bit virtual note stacks without allocating an individual C++ object for every logical note.

## Extended Psych syntax

Standard Psych note: `[timeMs, noteData, sustainMs, optionalType]`.

Overkill extension: `[timeMs, noteData, sustainMs, optionalType, "stackCount"]`.

- The **fifth element is a decimal string** representing a positive uint64_t value. Keep it quoted to avoid IEEE-754 rounding.
- The fifth element is optional; omitted means exactly one note.
- `song.ghStack64: true` is a marker for mod authors. The importer recognizes the fifth field in either DOM or streaming mode.
- Global chart counts, streamed-note counts, and session lane-note stacks use uint64_t. A press judges a stack as one visual note while score/combo/accuracy count its full multiplicity.
- Binary `.ghc` **v3** encodes stacks as unsigned LEB128 varints. Older v1/v2 charts remain readable; the cache version is bumped to avoid incompatible layouts.
- All other Psych fields remain available, including the original player, stage, note-type and event data.

## Reference example

```json
{
  "song": {
    "song": "amphotercity",
    "bpm": 295,
    "speed": 6,
    "player1": "bf",
    "player2": "gena",
    "stage": "ampho",
    "ghStack64": true,
    "notes": [
      {
        "mustHitSection": true,
        "lengthInSteps": 16,
        "sectionNotes": [
          [26033.898305, 0, 0],
          [26034.03072, 4, 0, "", "19313960"]
        ]
      }
    ]
  }
}
```

## AMFOTERNOST Overkill chart

The full projected chart is published separately in the existing user-owned Google Drive project folder:

- 3,560,023,825,730 total logical notes (sum of stacks), no inserted extra notes.
- 4,107,821 time/lane grouped visual note events; 492 Psych sections; BPM 295.
- 196,608 stacks have counts >65,535; maximum stack size 19,313,960.
- Real chart times use **60,000/(295 × 768 × 4) milliseconds per FLP tick**, confirmed by music/stage alignment. The published visual chart spans about 26–400 seconds.
- Psych-like note side mapping: when `mustHitSection=true`, noteData 0–3 plays BF, and 4–7 Gena.
- **Warning:** the extreme MIDI source contains 30 chromatic pitches, and no exact pitch→4-lane mapping can be proved from a musical animation or waveform. The 4K lane mapping is explicitly a provisional projection, not a lossless musical conversion.
- The companion native-MIDI JSON retains all 57 patterns, 168,623,461 original 24-byte note records, 50,056 Playlist pattern clips and the exact 3,560,023,825,730-note expanded total, including full pitch, velocity, channel and note-length data.

### Install

Place the mod ZIP's contents in a God's Hand mods directory, retaining the `data/amphotercity/amphotercity-hard.json` chart and supplied `stages/`, `images/`, `characters/`, `songs/`, and `data/` assets. Select Hard.

### Validate

Run `gh_tests` for the DOM and streaming >2^53 stack tests and the binary .ghc v3 round-trip; then `gh_tests --keys path/to/amphotercity-hard.json` and `gh_tests --chart path/to/amphotercity-hard.json`.

**Development limitations:** the extension changes the importer, chart state and game session. It does not replicate the separate animation video's dialogue/cinematics or prove exact 30-pitch-to-4-key mapping. Performance of the full 4.1-million-event mod must be tested on a target machine.
