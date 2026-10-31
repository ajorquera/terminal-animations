# How to control third-party Animations

Research for the ticket [How to control third-party Animations](https://github.com/ajorquera/terminal-animations/issues/3), part of the [Terminal animations spec](https://github.com/ajorquera/terminal-animations/issues/1). Facts are as of 2026-10-31, on tmux 3.7c on the owner's Mac.

## Answer

Let tmux do the input handling and the terminal cleanup. Don't sit between the terminal and the Animation.

- **Capture keys with tmux bindings.** A PTY wrapper can't work for Ambient, because the Ambient pane isn't focused and the keys never reach it. Skip, Favourite and Ban use one key chord from any pane. Break can also have bare keys.
- **Shut the Animation off from input.** Run `select-pane -d` on its pane. A stray key can't quit the tool, or drop cellscape into its menu. Bound keys still work, and the setting survives a respawn.
- **Stop and swap with `respawn-pane -k`.** It gives a fresh pty and a fresh screen, so it doesn't matter that most tools leave a mess when killed. This one mechanism worked on all six tools.
- **Run each Animation under a short-lived runner.** This small Launcher process lives for one Animation. It holds the Rotation timer, sees how the Animation ended, and starts the next one. If `waitpid` returns first, the Animation ended by itself. If SIGHUP arrives first, it was stopped.
- **Take a lock around each swap.** Overlapping `respawn-pane -k` calls sometimes leave a stray process. With `wait-for -L` there were none in 1,600 swaps.
- **Accept the short blank between Animations.** It is under 30 ms for most tools. cmatrix takes about 270 ms and sneakers_auto about 100 ms, which is mostly the tool starting up. Preparing the next Animation in a hidden session removes the gap, but it costs more than the gap does, so keep it as a later option.

## How this was checked

- **Experiments:** throwaway scripts ran on private tmux servers (`tmux -L <name> -f /dev/null`) with `default-terminal tmux-256color`, `mouse on` and `remain-on-exit on`. Key tests went through a real `tmux attach` client on a Python-owned pty, so bindings fired the normal way.
- **Pane state** came from tmux formats (`alternate_on`, `cursor_flag`, `mouse_any_flag`, `pane_dead_status`, `pane_dead_signal`), `capture-pane -e` and `stty -a` inside the pane.
- **Source read at the versions in use:** tmux 3.7c, [cmatrix 2.0](https://github.com/abishekvashok/cmatrix/blob/v2.0/cmatrix.c), [cbonsai 1.4.2](https://gitlab.com/jallbrit/cbonsai/-/blob/v1.4.2/cbonsai.c), [tarts 0.1.25](https://github.com/oiwn/tarts/tree/v0.1.25), cellscape 0.1.0, [bubbletea 1.3.5](https://github.com/charmbracelet/bubbletea/blob/v1.3.5/tea.go), [no-more-secrets 1.0.1](https://github.com/bartobri/no-more-secrets/tree/v1.0.1) and [ternimal](https://github.com/p-e-w/ternimal/blob/master/ternimal.rs). The ternimal source is upstream master, because the installed build carries no version.
- **Not checked:** nobody watched any of this. Blank times come from pane captures, not from a real terminal or a human eye. Only the six tools in `screensaver` were tested, so the tools recommended by [What existing Animations are worth adding](https://github.com/ajorquera/terminal-animations/issues/4) are untested here.

Each finding below is marked **Tested** (seen in an experiment), **Source** (read in code), **Docs** (tmux man page) or **Inferred**.

## What the tools do

### Signals

**Tested.** Each cell shows how the tool ended and whether its alternate screen was left on. A tool was killed directly and the dead pane was inspected, so the hidden cursor can't be seen in this table: tmux clears that flag on any dead pane ([`server-fn.c` line 370](https://github.com/tmux/tmux/blob/3.7c/server-fn.c#L370)).

| Tool | SIGTERM | SIGINT | SIGHUP | SIGQUIT | SIGKILL |
|---|---|---|---|---|---|
| cmatrix | exit 1, clean | exit 0, clean | killed, alt left | exit 0, clean | killed, alt left |
| cbonsai -S | exit 1, clean | exit 1, clean | killed, alt left | killed, alt left | killed, alt left |
| tarts | killed, alt left | killed, alt left | killed, alt left | killed, alt left | killed, alt left |
| sneakers_auto | still alive at 2.5 s | still alive at 2.5 s | exit 1 at once | still alive at 2.5 s | killed |
| cellscape run … -f | exit 0, clean | exit 1, clean | killed, alt and mouse left | exit 2, alt and mouse left | killed, alt and mouse left |
| ternimal | killed | killed | killed | killed | killed |

- **cmatrix** catches SIGINT and SIGQUIT and runs its cleanup (**Source:** [lines 387-390](https://github.com/abishekvashok/cmatrix/blob/v2.0/cmatrix.c#L387-L390)).
- **cbonsai** has no signal handlers (**Source**). It and cmatrix still come out clean on SIGTERM, most likely from ncurses's own handler (**Inferred:** ncurses itself wasn't read).
- **tarts** only cleans up in a `Drop` guard, which no signal reaches (**Source:** [`main.rs` line 74](https://github.com/oiwn/tarts/blob/v0.1.25/src/main.rs#L74)).
- **cellscape** gets SIGINT and SIGTERM handling from bubbletea, but not SIGHUP (**Source:** [`tea.go` line 287](https://github.com/charmbracelet/bubbletea/blob/v1.3.5/tea.go#L287)).
- **sneakers_auto** is a zsh script whose trap only runs after the current foreground command finishes, so a signal waits out `nms` or `sleep 5` (**Inferred** from the delay and the script). Its SIGQUIT result wasn't explained.

No single signal is clean on all six, and exit codes follow no pattern. A Launcher can't rely on signals to leave a tool tidy.

### Keys

**Tested** through `send-keys`, plus **Source** where noted.

| Tool | What keys do |
|---|---|
| cmatrix | `q` and Ctrl-C quit cleanly. Other letters change its settings. |
| cbonsai -S | Any key quits. |
| tarts | `q`, Esc and Ctrl-C quit, but keys get lost. In one run `q` was ignored and Ctrl-C worked; in another `q` worked and Ctrl-C was ignored. A second loop reads and drops events ([`common.rs` line 83](https://github.com/oiwn/tarts/blob/v0.1.25/src/common.rs#L83)). |
| sneakers_auto | Ctrl-C stops it. `q` does nothing. |
| cellscape | `q`, Esc and Ctrl-C do not quit. They return to the menu and stay there, and `f` turns off focus mode ([`simulation.go` line 98](https://github.com/ashish0kumar/cellscape/blob/v0.1.0/internal/ui/simulation.go#L98)). |
| ternimal | Ctrl-C stops it. Other keys do nothing. |

Quit keys differ by tool and aren't reliable, and in cellscape a stray key leaves a static menu on screen. Sending keys can't be the stop mechanism, and the Animation shouldn't receive keys at all.

### What a killed tool leaves behind

**Tested.** This only matters when something outlives the tool on the same pty, as a long-lived supervisor does. The same tool was run as a child, killed, and the pane inspected while the parent was still alive.

| Tool, signal | Alternate screen | Cursor | Mouse | Terminal mode |
|---|---|---|---|---|
| cmatrix or cbonsai, SIGKILL | on | hidden | off | `-echo -icanon -icrnl` |
| tarts, SIGTERM or SIGKILL | on | hidden | off | also `-isig -opost` |
| cellscape, SIGKILL | on | hidden | on | also `-isig -opost` |
| cmatrix, cbonsai, cellscape, SIGTERM | off | shown | off | normal |
| ternimal, SIGTERM or SIGKILL | off | shown | off | normal |

- **Colours:** no stale colour showed in the probe text that was printed afterwards. This was a small sample.
- **ternimal** hides and shows the cursor around each frame ([line 252](https://github.com/p-e-w/ternimal/blob/master/ternimal.rs#L252)), so a kill mid-write could leave it hidden. That wasn't seen.
- **Resetting by hand is not simple.** `ESC c` restored the cursor and mouse but left the alternate screen on. It took `ESC[?1049l` followed by `ESC c`, and the terminal mode needs a separate `stty sane`.

## Stopping and swapping

### `respawn-pane -k`

- **What it does (Source):** tmux sends no signal. It closes the pane's pty, then calls `screen_reinit` and frees the input parser ([`spawn.c` lines 258-263](https://github.com/tmux/tmux/blob/3.7c/spawn.c#L258-L263)). The new command gets a new pty. The SIGHUP comes from the kernel when the pty closes (**Inferred**, and it fits the HUP-ignoring test below).
- **On all six tools (Tested):** the old process and its children were gone by the first check after the 6 ms call. The new pane had the alternate screen off, cursor shown, mouse off and normal `stty`, even after tools that had left the alternate screen and mouse on.
- **Across swaps (Tested):** pane options persist, including `@` user options, `window-style`, and the `select-pane -d` input flag.
- **Limit (Tested):** a process that ignores SIGHUP survives. None of the six do. The runner closes that gap: its SIGHUP handler runs `killpg(0, SIGKILL)`, which cleared a HUP-immune script and its children.

`kill-pane` also took a runner and its Animation down, so turning Ambient off leaves nothing behind (**Tested**).

### Stray processes under rapid swaps

**Tested.** When `respawn-pane -k` calls overlap, a just-started process can outlive the swap with parent PID 1. It looks like it loses the pty before it gets SIGHUP (**Inferred**).

| Calls | Swaps | Result |
|---|---|---|
| 8 at once, no lock | 300 per round | 3 of 11 rounds left 1 to 3 strays, and one earlier run left 27 |
| 1 at a time, no lock | 300 per round, 7 rounds | none |
| 8 at once, with `wait-for -L` around each swap | 200 per round, 8 rounds | none |

Starting the command with or without a shell made no difference. The prototype's 12-way parallel Skip left one stray `zsh`, found afterwards. Its runner would have kept its timer, so a stray could disturb Rotation later. `run-shell -b` bindings can overlap, so the lock is needed.

### Blank time between Animations

**Tested.** Each cell is median/worst milliseconds with under 20 visible cells on a 100x30 pane, over 5 swaps from a full tarts plasma screen. Samples came from `capture-pane` called in a tight loop.

| Method | cmatrix | cbonsai -S | tarts plasma | sneakers_auto | cellscape | ternimal |
|---|---|---|---|---|---|---|
| `respawn-pane -k` | 270/275 | 6/11 | 28/29 | 104/107 | 27/27 | 7/8 |
| In-pane supervisor | 261/273 | 7/8 | 25/29 | 30/97 | 15/24 | 0/8 |
| PTY wrapper, holding the old frame | 270/314 | 0/0 | 19/25 | 98/102 | 21/21 | 0/0 |
| Prepare hidden, then `swap-pane` | 0/49 | 0/0 | 0/0 | 0/0 | 0/0 | 0/0 |

- **Reading it:** the first three rows are close. The gap is mostly how long the next tool takes to draw, so a wrapper in the data path gains little. cmatrix starts from an empty screen in all of them.
- **Caveats:** the runner wasn't in these runs, so a real Launcher adds its own start-up time, and a slow start-up language would show. Some other experiments ran at the same time as the first, second and fourth rows.
- **Preparing hidden has real costs (Tested):** it needs a detached session, created at the pane's exact size. The owner's `destroy-unattached on` kills such a session at once unless `destroy-unattached off` is set in the same command, and then it outlives every client unless removed. The visible pane also gets a new ID, and `@` options and the input flag stay behind on the old pane, so they must be set again.

Three smaller points:

- **Dead pane message:** an empty `remain-on-exit-format` removes the "Pane is dead" line (**Tested**).
- **Last frame:** a respawn wipes the screen, so the old frame can't be held on screen by tmux alone (**Source**).
- **Background colour:** setting `window-style bg=black` on the pane should keep the blank from flashing a different colour on a non-black terminal. The option persisted; the visual effect wasn't checked.

## Telling "ended" from "stopped"

| What happened | Runner sees | `pane-died` hook (no runner) |
|---|---|---|
| Animation exits 0 | `waitpid` returns 0 | fires, status 0 |
| Animation exits non-zero | `waitpid` returns that code | fires, that status |
| Command not found | return 127 | fires, status 127 |
| Killed by a signal | negative return | fires, `pane_dead_signal` set |
| Skip, or `respawn-pane -k` | SIGHUP | does not fire |
| `kill-pane` | SIGHUP | does not fire |

**Tested,** except that the runner's signal-death row wasn't run. `pane-exited` did not fire in any of these cases with `remain-on-exit on` (**Tested**), which matches the source ([`server-fn.c` lines 354 and 377](https://github.com/tmux/tmux/blob/3.7c/server-fn.c#L354-L377)).

Both ways can tell the cases apart. The runner is better because the Rotation timer lives in the same process, so a stopped Animation can't leave a timer behind. With hooks, a separate `sleep` would need a token to avoid skipping the wrong Animation.

- **Failures:** a quick non-zero exit counts as a failure. The prototype moved on after each one and parked after 3 in a row, so a broken tool can't cause a tight respawn loop (**Tested**).
- **Input-off matters here:** with input on, a stray `q` would look like a normal end, with exit 0.
- **Natural ends are rare today:** none of the six ended on its own during these runs, which lasted a few seconds each. sneakers_auto loops by design.

## Capturing Skip, Favourite and Ban

**Tested,** through an attached client.

| Technique | Result |
|---|---|
| Prefix chord: `bind a switch-client -T ta`, then `s`, `f`, `b` in table `ta` | Works from any pane. Focus stays put and nothing leaks into either pane. |
| `bind -r` in that table | `s` repeated three times within `repeat-time`, for several Skips in a row. |
| Bare root key: `bind -n s if -F '#{...}' '<skip>' 'send-keys s'` | Fires only in the Animation pane. Typing `sssxs` in another pane arrived intact and in order. |
| Session option `key-table` | Works, and the prefix still works, but unbound keys reach the pane unless `Any` is bound. It's per session, while a Break can be a pane. |
| `select-pane -d` | Keys and mouse clicks did not reach the pane, bound keys still fired, and cbonsai -S survived a keypress. sneakers_auto still ran normally. |
| `list-panes -s -f '#{@ta}'` | Finds the Animation pane from a binding. |
| PTY wrapper | Saw `s` only when focused. Unfocused, it received nothing. |

| | Ambient | Break |
|---|---|---|
| **Skip** | Chord from any pane. Focus stays on the work. | Chord, or bare `s` |
| **Favourite** | Chord. Records the mark and keeps playing. | Chord, or bare `f` |
| **Ban** | Chord. Records the mark, then Skips. | Chord, or bare `b` |

The current Animation is kept in a pane option, which the bindings read. The prototype set it from the runner just after start-up, which leaves a moment where it is out of date. Set it before the respawn instead.

The end-to-end prototype ran all of this against the six real tools: Rotation on a 3-second timer, Skip, Favourite and Ban in Ambient and Break, Loop, a natural end, failures, and Ambient off. Work-pane input stayed clean, and no banned Animation came up in the Break check after a Ban. No tool process was left over at the end, though see the stray `zsh` above.

## Approaches compared

| | Bindings, respawn, runner | Bindings, respawn, `pane-died` hook | Long-lived supervisor in the pane | PTY wrapper |
|---|---|---|---|---|
| Keys in Ambient | Yes | Yes | Yes, via bindings | Only with bindings as well |
| Terminal reset | tmux does it | tmux does it | Launcher must do it | Launcher must do it, or run a terminal emulator |
| Stopping | One way for all tools | One way for all tools | Tool signals, then cleanup | Kill, then cleanup |
| Rotation timer | In the runner | Separate timer needing a token | In the supervisor | In the wrapper |
| Ended or stopped | Clear | Clear | Clear | Clear |
| HUP-immune tools | Handled | Not handled | Handled | Handled |
| Extra code | Small | Smallest, but spread across tmux hooks | Moderate | Largest |

The supervisor and PTY wrapper were each built as a prototype far enough to swap and measure. The supervisor needs the three-part reset above, and getting it wrong leaves the next Animation in a bad terminal. The PTY wrapper hit a deadlock in about 40 lines: waiting for a killed tool hung until the pty was closed first. Its first timing run was invalid because of this, and its pty start-up also needed rework. A wrapper also needs bindings for Ambient anyway, so it would be a second input path.

**Recommended: bindings, respawn and a runner.**

## Recipe for the spec

This is a sketch of what the prototype did, not final code. The key names are placeholders, and `a` is unbound in the owner's `.tmux.conf`.

```tmux
# Installed once
bind a switch-client -T ta
bind -r -T ta s run-shell -b "launcher skip '#{session_name}'"
bind    -T ta f run-shell -b "launcher favourite '#{session_name}'"
bind    -T ta b run-shell -b "launcher ban '#{session_name}'"

# Set on each Animation pane
set -p @ta ambient                      # or break
set -p remain-on-exit on
set -p remain-on-exit-format ''
select-pane -d
```

1. **Swap:** take `wait-for -L`, record the current Animation in a pane option, run `respawn-pane -k -t <pane> <runner> <animation>`, release the lock.
2. **Runner:** start the Animation as a child in the runner's process group, so it keeps the tty foreground. Wait for it with a timeout.
3. **On SIGHUP:** `killpg(0, SIGKILL)`, so nothing outlives the swap.
4. **On timeout or a natural end:** record the outcome and do a Swap. A failure counts towards parking.

## For the spec to settle

- [What the Launcher is built with](https://github.com/ajorquera/terminal-animations/issues/6): the runner starts for every Animation, so start-up time adds to the blank gap. This favours a compiled language or a thin shell script. A zsh runner has a known trap: tarts exited at once when started with `&` from a zsh script, while it ran fine under a Python parent.
- **Key names:** the chord and whether Break gets bare keys. Bare keys need root-table bindings, which touch every keypress in tmux.
- **Which pane a chord acts on** when there is more than one. A sensible order is the active pane if it's an Animation, then the current window, then the session. This wasn't tested.
- **Skip and Ban during Loop:** the charted rules don't say whether either leaves Loop.
- **A stray runner:** the lock should prevent these. As a backstop, a runner could check a token in a pane option before acting on its timer. This wasn't tested.
