# pdxterm CHANGELOG

## v1.3.0 -- 2026-09-13 (Wave BBB: 4-issue tail, issues #6/#11/#12/#15)

Landing shape: closes the last four open issues in the tracker.
#6 and #11 are code landings; #12 and #15 are retroactive
documentation closes (the repo has been functionally past a 1.0.0
surface since v1.1.0, but never formally tagged either milestone).

- `src/pty_wire.pdx` (new, #6) -- `Module PtyWire`: R102.M3-001 real
  KIND_PTY spawn + shell attach. `pty_spawn_shell` dispatches the
  proposed KIND_PTY spawn op through the already-landed generic
  `sys_cap_invoke` (SC+ ID 4) against a reserved candidate cap slot.
  **Disclosed kernel gap:** no `kind_pty.pdx` dispatch body exists
  anywhere in the kernel tree yet, so this call is expected to fail
  closed (negative errno) until R101 lands real KIND_PTY substrate;
  the function records the raw result plus a blockage marker
  (`PTY_BLOCKED_KERNEL_GAP = 0x8003`) rather than fabricating live
  fds, per the R102 plan's own §7.3 escalation posture. Two stable
  attach seams are wired now for a future live fd to drop into
  unchanged: `pty_drain_keyboard_to_master` (reads
  `Keyboard::_pty_out_head`, the existing keystroke-ring occupancy)
  and `pty_feed_slave_bytes` (forwards straight into
  `Ansi::ansi_feed_bytes`, the live grid-render path three CLOSED
  fingerprint tests already exercise). argv[0]="sh" staged as a
  `.rodata` fixture (`PROC_ARGV0_SH`) for the eventual `sys_execve` +
  `sys_dup2` spawn (both already-landed kernel sysnos) once a slave fd
  exists.
- `src/syscall.pdx` -- adds `sys_cap_invoke` (SC+ ID 4, R13 legacy)
  wrapper, mirroring `src/user/syscall_shim.pdx` row 5 byte-for-byte;
  pdxterm's sole caller is `PtyWire::pty_spawn_shell`.
- `caps.decl` -- notes that Wave BBB exercises the already-declared
  `KIND_PTY(read, write, mint)` requirement via `pty_spawn_shell`; no
  new capability kind added.
- `tests/scrollback_smoke.pdx` (new, #11) -- `Module ScrollbackSmoke`:
  pushes 300 rows into `Scrollback`'s 256-slot ring and asserts (a)
  `head` wraps to 0 at exactly 256 pushes, (b) `head` ends at 300 mod
  256 = 44, (c) slot 0 (oldest row) is overwritten by push index 256,
  proving the overwrite-oldest policy. Diverges from the plan doc's
  PageUp/PageDown scroll-view scope by design -- no scroll-view API
  exists anywhere in this tree yet, so this smoke targets the
  primitive `Scrollback::sb_push_row` actually provides (see the
  file's header for the full rationale); a view-level smoke is a
  follow-up once Grid/Scrollback grow a viewport-offset API.
- Retroactive milestone closes: #12 (R102.M5-001, "signed 1.0.0
  release") and #15 (v1.1-C release closer) are closed as
  documentation-only. `manifest.pdxsig`'s dual-sign note now records
  that ed25519/ML-DSA signing tooling is unbuilt org-wide, so the two
  signature slots stay `unsigned` placeholders rather than a
  fabricated signature; `README.md` gains a "Release status" section
  cross-referencing this history.
- `manifest.pdxproj` / `manifest.pdxsig` -- version 1.2.0 -> 1.3.0;
  sources list gains `src/pty_wire.pdx`; tests list gains
  `tests/scrollback_smoke.pdx`.
- `src/tool_ident.pdx` -- `PDX_TOOL_VERSION` bumped to `1.3.0\0`.

## v1.2.0 -- 2026-09-13 (Wave OO: M1/M2 cohort, issues #1-#5)

Landing shape: fills in the M1/M2 surface Wave P's skeletons
deferred, without disturbing the already-CLOSED M3/M4/v1.1-B work.

- `caps.decl` -- add `KIND_INPUT_EVENT(read)` for keyboard input
  (#4); `manifest.pdxsig` added as the M5 dual-sign source-form draft
  (#1).
- `src/window.pdx` (new, #2) -- `Module Window`: `--rows=<N>`
  (default 24) / `--cols=<N>` (default 80) argv parser,
  cols*8 x rows*16 ARGB32 geometry, and an honest-blockage
  `window_request_surface` stub (no live KIND_SURFACE mint until
  libpdx-gfx.M2-001 lands). Not yet wired into `Main::pdxterm_main`
  pending a confirmed argv ABI.
- `src/grid.pdx` (#3) -- adds `_cell_char`/`_cell_attr` (24x80 byte
  planes) alongside the existing `_grid_cells` cursor-oriented
  surface, plus `grid_clear`, `grid_put_char(row, col, ch)`,
  `grid_render_all` (blits via the new `pdxterm_glyph_blit`
  WEAK-in-spirit stub -- libpdx-font not linkable yet). Kept as a
  separate surface rather than reshaping `_grid_cells`, since the
  latter is the live target of three CLOSED-issue tests.
- `src/keyboard.pdx` (new, #4) -- `Module Keyboard`: 128-entry
  keysym->byte LUT (identity for printable ASCII 0x20-0x7E, built at
  `kbd_reset` time rather than as a literal), `kbd_handle_keydown`
  writing into a 4096-byte `_pty_out_ring`. No live KIND_INPUT_EVENT
  subscription yet (no input satellite past its own M1).
- `src/scrollback.pdx` (new, #5) -- `Module Scrollback`: 256-row x
  80-byte `_scrollback` ring, `sb_push_row` (copy + advance `head`
  modulo 256). Not yet called from `Ansi`'s newline path (deferred
  until a build/debugger pass can confirm the existing ANSI
  fingerprints survive the wire-up).
- `manifest.pdxproj` -- version 1.1.0 -> 1.2.0; sources list gains
  the three new files.
- `src/tool_ident.pdx` -- `PDX_TOOL_VERSION` bumped to `1.2.0\0`.

Note on versioning: the dispatch for this cohort named tag `v0.5.0`;
that would be a semver regression behind the already-tagged `v1.1.0`.
This landing instead bumps forward to `v1.2.0` and tags that -- flagged
for confirmation rather than silently applying a backward tag.

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
