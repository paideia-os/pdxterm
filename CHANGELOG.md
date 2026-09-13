# pdxterm CHANGELOG

## v1.1.0 -- 2026-09-13 (Wave P drain)

Landing shape: scaffold + M3-002 ANSI parser + three M4 fingerprint
smokes + v1.1-B semantic-pipe emission wire. First code landing in
the repo (prior state was README + LICENSE only).

### Scaffold (repo-level)

- `.gitignore` -- build-out/, object files, editor noise.
- `manifest.pdxproj` -- paideia-as v0.21+ manifest, name=pdxterm,
  version=1.1.0, entry=Main::pdxterm_main, six sources + five tests.
- `caps.decl` -- requires KIND_USER, KIND_SURFACE, KIND_IPC_ENDPOINT,
  KIND_PTY, KIND_PROCESS(spawn); declares TermEventRecord@0.1 output
  schema.
- `tools/build.sh` -- paideia-as resolver + build loop over src/ and
  tests/ (identical shape to /tools/user/shell/tools/build.sh).
- `src/tool_ident.pdx` -- PDX_TOOL_NAME="pdxterm\0" (8B),
  PDX_TOOL_VERSION="1.1.0\0" (6B), Wave F ENH-032 UND-extern contract.
- `src/syscall.pdx` -- SC+ ID 115 sys_semantic_send wrapper.
- `src/grid.pdx` -- 80x25 cell grid + cursor state + grid_reset,
  grid_set_cursor, grid_write_char primitives.
- `src/main.pdx` -- Main::pdxterm_main entry skeleton.

### R102.M3-002 -- ANSI escape parser (issue #7)

- `src/ansi.pdx` -- state machine for STATE_GROUND / STATE_ESC /
  STATE_CSI. Handles printable bytes, newline, ESC-[ transition,
  digit accumulation with clamp at 9999, semicolon-separated params
  (up to 8), final-byte dispatch on m/H/f/A/B/C/D, unknown swallow
  with ansi_unknown_seq_count observability. SGR reset (0), fg
  30..37/90..97, bg 40..47/100..107.
- `tests/test_ansi.pdx` -- 9-case fingerprint matrix
  (`pdxterm ansi ok`, bit-per-case at _ansi_test_fp; full pass = 0x1FF).

### R102.M4-001 -- rendering identity smoke (issue #8)

- `tests/render_identity.pdx` -- seed grid with alphabet-repetition
  pattern, FNV-1a-64 digest over the surface, compare to
  GOLDEN_RENDER_FP (fingerprint: `pdxterm render-identity ok`).
  BLAKE3 swap noted for M5 hardening pass.

### R102.M4-002 -- shell echo round-trip smoke (issue #9)

- `tests/shell_echo_roundtrip.pdx` -- honest-blockage witness:
  substrate half (KIND_PTY spawn, libpdx-gfx wire, boot_r102_hello
  witness wiring, scripted-HID emit) blocked on four upstream deps
  documented in file header + at _shell_echo_blockage_kind = 0x8001;
  fixture-fed half asserts the ANSI-parser + grid path lands `hello`
  in a row after the prompt (fingerprint: `pdxterm hello ok`).

### R102.M4-003 -- cursor movement smoke (issue #10)

- `tests/cursor_move.pdx` -- pipe `\x1B[5;10Hhello` through the
  parser, assert cursor lands at (4, 14) and glyphs h/e/l/l/o at
  (4, 9..13) (fingerprint: `pdxterm cursor-move ok`).

### v1.1-B -- Semantic-pipe emission wire (issue #14)

- `src/schema.pdx` -- TermEventRecord@0.1 (48B) with PDXKTERM magic,
  version=1, kind field (WRITE/CURSOR/SGR/UNKNOWN), two u64 arg
  lanes, rdtsc timestamp. Schema tag 0x4D5245544B584450 in the
  0xE0** band (pdxterm slot 0xE003 by convention).
  Schema::term_event_emit marshals + dispatches sys_semantic_send
  (sysno 115); observability slots term_emit_ok_count,
  term_emit_err_count, term_last_kind.
- `tests/test_semantic_emit.pdx` -- 3-case composition fingerprint
  (bit-per-case at _tse_fp; full pass = 0x7).
