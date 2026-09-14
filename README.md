# pdxterm

Framebuffer terminal emulator — a **graphical** alternative to the UART TTY, running as a normal windowed app. Renders a terminal grid onto its own `KIND_SURFACE`; forwards keyboard events to a child `/bin/shell` over `KIND_PTY`. 80x25 default, 8-color ANSI palette, 256-row scrollback ring. The moment R102 becomes a usable substitute for the UART TTY.

Part of the **paideia-os** organization. MIT-licensed.

## Wave

R102 (softarch userland graphical stack) — the CPU-side framebuffer stack
that lands the first graphical UI on paideia-os before the G-series
GPU-accelerated compositor matures. Companion to the osarch R101 kernel-side
plan.

## Design reference

- Design lives in the monorepo at [`design/graphics/r102-user-plan.md`](https://github.com/paideia-os/paideia-os/blob/main/design/graphics/r102-user-plan.md) §2.6 / §4.6.
- Kernel-side companion: `design/graphics/r101-kernel-plan.md`.

## Milestones

Per the plan, this repo lands across five milestones:

- **M1** — repo scaffold; caps.decl (KIND_SURFACE, KIND_IPC via libpdx-event, KIND_PTY stub, KIND_PROCESS); argv parser (--geometry, --font); window creation stub
- **M2** — grid math + full-window redraw over libpdx-gfx + libpdx-font; keyboard->bytes pass-through to stub pty; 256-row scrollback ring
- **M3** — real KIND_PTY spawn (R101 §7.2.3); /bin/shell attach; ANSI escape parser (8-color palette + cursor moves)
- **M4** — smokes: rendering identity, shell echo round-trip, cursor movement, scrollback
- **M5** — signed 1.0.0 release

Every issue is filed against one of these five milestones; see the Issues tab.

## Release status

Current version: **v1.3.0**. M1-M4 have landed (issues #1-#11, #14);
M5 (issue #12, "signed 1.0.0 release") and the earlier v1.1-C release
closer (issue #15) are closed as *retroactive documentation
milestones* -- the repo passed a 1.0.0-equivalent surface back at
v1.1.0 without a formal tag/CHANGELOG closure, so #15 and #12 record
that history rather than gating a not-yet-reached 1.0.0. The dual-sign
manifest (`manifest.pdxsig`) stays an unsigned source-form draft:
ed25519 + ML-DSA signing tooling has not been built anywhere in the
org yet, so no real signature can be produced. See `CHANGELOG.md` for
the per-wave detail and `src/pty_wire.pdx` for the one open kernel-side
gap (KIND_PTY has no dispatch body in the kernel yet) blocking a fully
live M3 substrate.

## Scaffolding

No code lands with this repo scaffold — scaffolding lives in the M1
issues (`caps.decl`, `src/` skeleton, public API stubs, argv parsing).
Repo shape mirrors R100 satellites: paideia-as manifest at root,
`caps.decl` at root, `src/` module tree, `tests/`, `release/`,
`doc/<name>.pdxdoc`, dual-signed `manifest.pdxsig` at 1.0.0.

## License

MIT. See [LICENSE](LICENSE).
