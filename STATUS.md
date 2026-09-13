# pdxterm STATUS

Wave: R102 (userland graphical stack) + v1.1 satellite pass + Wave OO.
Version: **v1.2.0** (Wave OO landing 2026-09-13, pending build/debugger
verification -- see note below).

## Issue-level status

NOTE (Wave OO correction): the prior revision of this table marked
#1 LANDED and #2 PARTIAL against v1.1.0, but `gh issue list` for this
repo shows #1-#5 all still OPEN at the time this landing started.
That prior LANDED/PARTIAL marking was inaccurate -- code existed
(tool_ident.pdx, caps.decl, manifest.pdxproj) that partially overlaps
#1/#2's scope, but the issues themselves were never closed. This
table now reports code-landed-pending-verification rather than
issue-closed; only main/debugger should flip these to CLOSED after a
build + smoke pass confirms the diff.

| Issue | Milestone | Status                       | Notes                                         |
| ----- | --------- | ----------------------------- | --------------------------------------------- |
| #1    | M1-001    | CODE LANDED (v1.2.0), pending verify | repo scaffold complete: README/LICENSE/CHANGELOG, caps.decl, tools/build.sh, manifest.pdxsig (source-form) added this wave |
| #2    | M1-002    | CODE LANDED (v1.2.0), pending verify | src/window.pdx: `--rows=`/`--cols=` argv parser + KIND_SURFACE geometry; live mint stubbed (blocked on libpdx-gfx.M2-001) |
| #3    | M2-001    | CODE LANDED (v1.2.0), pending verify | src/grid.pdx: `_cell_char`/`_cell_attr` + grid_put_char/grid_clear/grid_render_all via pdxterm_glyph_blit stub (libpdx-font not linkable yet) |
| #4    | M2-002    | CODE LANDED (v1.2.0), pending verify | src/keyboard.pdx: 128-entry keysym LUT + `_pty_out_ring`; no live KIND_INPUT_EVENT subscription yet |
| #5    | M2-003    | CODE LANDED (v1.2.0), pending verify | src/scrollback.pdx: 256x80 `_scrollback` ring + sb_push_row; not yet wired into Ansi's newline path |
| #6    | M3-001    | DEFERRED       | real KIND_PTY spawn + /bin/shell attach -- blocking on R101 §7.2.3 |
| #7    | M3-002    | **LANDED (v1.1.0)** | src/ansi.pdx: CSI/SGR/cursor-move state machine + 9-case fingerprint matrix at tests/test_ansi.pdx |
| #8    | M4-001    | **LANDED (v1.1.0)** | tests/render_identity.pdx: alphabet-repetition seed + FNV-1a-64 digest vs GOLDEN_RENDER_FP; BLAKE3 swap noted for M5 |
| #9    | M4-002    | **LANDED (v1.1.0)** | tests/shell_echo_roundtrip.pdx: honest-blockage witness; substrate blocked on issues #3/#4/#6 + MON-005; fixture-fed half asserts h/e/l/l/o lands in row > 0 |
| #10   | M4-003    | **LANDED (v1.1.0)** | tests/cursor_move.pdx: `\x1B[5;10Hhello` -> cursor (4,14), glyphs at (4, 9..13) |
| #11   | M4-004    | DEFERRED       | scrollback smoke -- blocking on issue #5 |
| #12   | M5-001    | DEFERRED       | signed 1.0.0 release -- blocking on M4-* + M3-001 substrate |
| #14   | v1.1-B    | **LANDED (v1.1.0)** | src/syscall.pdx + src/schema.pdx: sysno-115 wrapper + TermEventRecord@0.1 (48B) marshal + emit; 3-case composition fingerprint at tests/test_semantic_emit.pdx |
| #15   | v1.1-C    | DEFERRED       | release closer + tag -- follows this landing after debugger pass |

## Downstream deps observed at Wave P

- **sys_semantic_send (SC+ ID 115)**: kernel-side landed R107-M0-001
  (paideia-os #2350). Wrapper here matches src/user/syscall_shim.pdx
  sys_semantic_send byte-for-byte.
- **libpdx-argv 1.1.3+ UND-externs**: PDX_TOOL_NAME + PDX_TOOL_VERSION
  published preemptively per Wave F contract; live libpdx-argv link
  waits for the manifest deps bump at v1.2.

## Encoder discipline (per user memory `pdx encoder pitfalls`)

- Module names PascalCase basename (Ansi, Grid, Schema, Syscall,
  Main, ToolIdent, TestAnsi, TestSemanticEmit, RenderIdentity,
  CursorMove, ShellEchoRoundtrip).
- No `test rN, rN` -- every zero-check is `cmp reg, 0`.
- No 2-op `imul r, imm` -- FNV-1a uses `imul rax, r9` (reg-reg,
  legal); pattern seed multiplies use `shl`+`add`.
- No `and r8..r15, imm64` (no wide-and on high regs anywhere).
- Single-line `pub let ="..."` literals only.
- Array sizes match literal byte count exactly (fixture arrays
  carry no NUL; NUL-terminated PDX_TOOL_NAME carries its 1-byte
  terminator in the declared size).
- SysV rsp%16 == 0 at every nested call (documented per function
  in each justification block).
- All labels prefixed by module convention (`ansi_`, `adf_`, `asgr_`,
  `afb_`, `ta_`, `ri_`, `rs_`, `fnv_`, `cm_`, `ser_`, `tse_`,
  `schema_`, `grid_set_cursor_*`, `grid_write_char_*`, `grid_reset_*`)
  to avoid reserved-keyword collisions.
