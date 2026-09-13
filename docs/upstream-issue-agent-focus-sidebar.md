# Upstream issue draft — herdr

To submit: https://github.com/herdrdev/herdr/issues/new
(copy the block below; fill in the version you are running when you submit)

---

**Title:** `agent.focus` (changes_ui=true) does not move the agent panel / sidebar selection in an attached client

**herdr version:** 0.9.0 (stable), Linux, protocol 22
**Setup:** interactive client running inside a terminal (ghostty), one local
session plus one saved SSH machine; sidebar visible with both the machines
section and the agents section.

## What happens

Focusing an agent from outside the client — `herdr agent focus <pane>` on the
CLI, or the same `agent.focus` request over the socket API — moves the
session's focus correctly, and the attached client's *displayed* workspace and
tab follow it. But the client's **sidebar does not follow**:

- the **agents section**'s selected row (inverse highlight) stays on whatever
  was selected by the last in-client interaction,
- the **machines section**'s highlight likewise stays on the old workspace
  row.

The server log shows the focus succeeded (`workspace.focus` + `tab.focus` for
the requested pane, and the request is marked `changes_ui=true`), so the
client clearly receives the UI change — only the sidebar selection does not
move with it.

## Why it matters

An external launcher (in my case an Omarchy bar widget that lists agents and
calls `herdr agent focus <pane>` on a click) can bring the user to the right
agent's pane, but the sidebar still points at the previous agent. Users read
the sidebar selection as "which agent is active", and if the sidebar holds
keyboard focus (common when the last interaction was selecting a row there),
typing or Enter acts on the stale row and jumps the user back — the external
focus appears to "not stick" even though it did.

## Reproduction

1. Open a herdr client with the sidebar visible; select agent **A**'s row in
   the agents section (it highlights).
2. From any shell outside herdr, run:
   `herdr agent focus <pane-id-of-agent-B>`
3. The client's window switches to B's workspace and tab (window title with
   `{workspace}` confirms it), and the server reports focus on B.
4. The sidebar still highlights **A** in the agents section (and the old
   workspace in the machines section). It stays there until you interact with
   the sidebar in the client again.

## Expected

Either (or both):

- focus requests that are marked `changes_ui=true` also move the client's
  sidebar selection to the focused agent/workspace, or
- the socket API exposes a way to drive the selection explicitly — e.g. a
  `source`/client-attribution parameter on focus requests, or a dedicated
  method (something like `sidebar.select` / `agent.select`).

## Notes

- Checked `herdr api schema --json`: there is currently no method for sidebar
  or panel selection. `agent.view.set` configures the view's filter/sort/
  label, not the selected row.
- Same behavior with `workspace.focus` / `tab.focus` requests directly.
- Happy to test a preview-channel build if useful.
