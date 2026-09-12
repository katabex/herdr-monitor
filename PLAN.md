# herdr-monitor

Project directory for extending the [omarchy-herdr](https://github.com/jankeesvw/omarchy-herdr)
bar widget with remote machines. Forked from upstream `master` at `fa545a4`
(v1.1.0); the work lives on the `remote-machines` branch.

## Goal

List remote agents — herdr sessions running on saved SSH machines (`herdr
machine add`) — on the widget card, next to the local sessions.

## Decisions (agreed before implementation)

- **Widget-side SSH fan-out** (option A): the data script queries each enabled
  machine itself, over plain SSH, running the same herdr CLI remotely that it
  runs locally. This is herdr's documented rule for remote automation
  ("run commands on the intended host against the intended session"), because
  no headless remote query exists in the local CLI (`--remote` is TUI-only).
- **One row per machine**, mirroring herdr's own sidebar model: a machine
  profile targets exactly one remote session. Locals first, machines after.
- **Open-only for v1**: no kill/delete on machine rows. Open = focus the
  attached window or launch `herdr --remote <target>` in foot. Agent-line
  click = remote `agent focus` over SSH, then the same window logic.
- **Same poll cadence** as locals (3s open / 20s closed), affordable because
  SSH multiplexing (`ControlMaster`/`ControlPersist` in the cache dir) makes
  a warm query ~40ms. Every remote call is bounded (`BatchMode`,
  `ConnectTimeout`, `timeout` wrapper) and machines are gathered in parallel.
- **Fix `--remote` window matching** as part of this: a window running
  `herdr --remote <target>` used to be misattributed to the local `default`
  session. It now pairs with the machine's row via a target→row-key map;
  herdr's own `remote-client-bridge` SSH process is skipped like `herdr server`.

## Row identity

Remote rows are named `<machine id>:<session>` — herdr's opaque profile id,
plus the remote session the profile targets. Session names may not contain a
colon, so no collision is possible. Labels are display-only (they can be
renamed); ids are stable.

## State handling

- Machine answered → normal row (`remote: true`, `machine: <label>`), agents
  from its snapshot, window paired by the extended map.
- Machine silent → last-good row from `~/.cache/omarchy-herdr/machines/`,
  `running: false`, `unreachable: true`, note "unreachable" (amber). First
  run with no cache → minimal quiet row.
- Remote herdr path resolved once via `sh -lc 'command -v herdr'` and cached;
  re-resolved automatically whenever the snapshot call fails.

## Future ideas (out of scope v1)

- Upstream a headless machine query to herdr (`herdr api snapshot --machines`
  or similar; the socket API already has subscription events) — would remove
  the SSH layer entirely.
- Event-driven updates via a long-lived per-machine stream.
- Kill/delete for machine rows via SSH signalling.
- `machine` token styling like herdr's sidebar row layouts.

## Install / test on the desktop

The live plugin at `~/.config/omarchy/plugins/jankeesvw.herdr` is untouched.
To try this fork:

```bash
omarchy plugin disable jankeesvw.herdr
omarchy plugin add /home/ptr/Repos/ktbx/herdr-monitor
omarchy plugin enable jankeesvw.herdr
```

(or swap the directory manually and restart the shell). The data script can
be exercised on its own: `bin/herdr-sessions list | jq .`
