# Which Animations work at small pane sizes

Measured on 2026-10-31 on the owner's Mac (M2 Max, 12 cores, tmux 3.7c, `TERM=tmux-256color`), against the 25 commands in the `screensaver` function in `~/.zshrc`.

## Answer

All 25 start at 40x12: none crashed, refused or exited, across a 5-minute run. That does not mean all 25 work. Exit status hides most of the failures here, because the bad ones keep running with a blank or broken pane.

| Result at 40x12 | Count | Commands |
|---|---|---|
| Good for Ambient | 8 | `cbonsai -S`, `cmatrix`, `tarts matrix`, `tarts cube`, `tarts crab`, `tarts pipes`, `tarts fire`, `tarts constellation` |
| Works, with a caveat | 5 | `tarts life`, `tarts donut`, `cellscape run life -f`, `cellscape run ant -f`, `cellscape run forest -f` |
| Works only with extra arguments | 3 | all three `ternimal` commands |
| Runs, but too busy for Ambient | 3 | `tarts maze`, `tarts boids`, `tarts plasma` |
| Does not work | 6 | `sneakers_auto`, `cellscape run brain -f`, `cellscape run ltl -f`, `cellscape run belousov -f`, `cellscape run faders -f`, `cellscape run wildfire -f` |

CPU is not the deciding factor. At 40x12, 18 of the 25 use under 2.5% of one core including tmux, and only `tarts donut` (10.0%) and `tarts boids` (7.9%) use more than 6%. The real filter is whether the pane still shows something worth looking at.

"Too busy" and the caveats are judgement calls made from still frames and change rates, not from watching the Animations move. The owner should overrule them freely.

## All 25 commands

CPU is a percentage of one core, so 100% is one full core of the 12. "App" covers the command and its child processes. "tmux" is the tmux server for that pane. KB/s is what the command wrote to the pane, which is the best available proxy for what the terminal has to draw.

The first set of numbers is 40x12, the second is 80x24.

| Command | At 40x12 | App CPU % | tmux CPU % | Total % | KB/s | App CPU % | tmux CPU % | Total % | KB/s | What happened |
|---|---|---|---|---|---|---|---|---|---|---|
| `cbonsai -S` | Yes | 0.09 | 0.09 | 0.18 | 0.2 | 0.10 | 0.10 | 0.20 | 0.5 | Tree grows, holds, regrows. Calmest and cheapest of the 25. Cropped at the top but still reads as a tree. |
| `cmatrix` | Yes | 0.47 | 0.47 | 0.94 | 17.2 | 0.48 | 0.29 | 0.77 | 34.7 | Fills the pane cleanly at both sizes. |
| `tarts matrix` | Yes | 0.38 | 0.75 | 1.13 | 35.7 | 0.28 | 0.94 | 1.22 | 76.7 | Sparse rain, keeps the look at 40 columns. |
| `tarts life` | Caveat | 1.13 | 2.83 | 3.96 | 297.5 | 2.26 | 6.11 | 8.37 | 827.4 | Intact, but about half the cells change each second, so it is busy. |
| `tarts maze` | Break only | 0.28 | 0.57 | 0.85 | 10.3 | 0.28 | 0.37 | 0.65 | 10.8 | Runs, but white walls on a full field of flickering characters, 100% of cells inked. |
| `tarts boids` | Break only | 4.66 | 3.23 | 7.89 | 413.5 | 5.90 | 3.80 | 9.70 | 609.6 | At 40x12 the flock reads as braille noise. Also among the costlier ones. |
| `tarts cube` | Yes | 0.29 | 0.38 | 0.67 | 15.3 | 0.29 | 0.48 | 0.77 | 58.4 | Small wireframe, centred, fully inside the pane. |
| `tarts crab` | Yes | 0.29 | 0.48 | 0.77 | 43.9 | 0.10 | 0.38 | 0.48 | 37.3 | Crabs wander. Stays legible. |
| `tarts donut` | Caveat | 9.47 | 0.48 | 9.95 | 47.0 | 9.08 | 1.31 | 10.39 | 159.8 | Looks right, but the most expensive Animation that renders intact: its CPU does not fall at the smaller size. |
| `tarts pipes` | Yes | 0.28 | 0.47 | 0.75 | 7.9 | 0.28 | 0.47 | 0.75 | 5.8 | Fills and clears. The lowest output of the tarts effects. |
| `tarts plasma` | Break only | 1.14 | 4.65 | 5.79 | 681.0 | 3.84 | 16.96 | 20.80 | 2230.0 | Every cell changes colour every second. Highest output at 40x12, and tmux pays more than the app. |
| `tarts fire` | Yes | 0.57 | 2.00 | 2.57 | 233.8 | 2.06 | 8.81 | 10.87 | 1112.6 | Keeps its shape at both sizes. Moderate output. |
| `tarts constellation` | Yes | 0.39 | 0.97 | 1.36 | 143.0 | 0.66 | 1.70 | 2.36 | 176.1 | Drifting stars, keeps its look at 40 columns. |
| `sneakers_auto` | No | 0.09 | 0.19 | 0.28 | 12.0 | 0.18 | 1.03 | 1.21 | 41.7 | Runs, but its menu is about 73 columns and 19 rows, so at 40x12 every long line hard-wraps and the layout breaks. Fine at 80x24. |
| `cellscape run life -f` | Caveat | 0.57 | 0.19 | 0.76 | 13.9 | 1.60 | 0.56 | 2.16 | 55.6 | Settles into a few small oscillators on an almost empty pane. |
| `cellscape run brain -f` | No | 0.47 | 0.09 | 0.56 | 3.2 | 0.66 | 0.19 | 0.85 | 16.3 | Blank by 3s at 40x12, by 6s at 80x24, and stayed blank for the full 5 minutes while still using CPU. |
| `cellscape run ant -f` | Caveat | 0.89 | 0.30 | 1.19 | 1.5 | 0.48 | 0.19 | 0.67 | 2.5 | Works and changes slowly. Pattern keeps growing (14% to 41% of cells over 5 minutes). Hard multicolour cells. |
| `cellscape run ltl -f` | No | 0.57 | 0.09 | 0.66 | 0.7 | 0.98 | 0.10 | 1.08 | 26.4 | Blank by 3s at 40x12, by 6s at 80x24, and stayed blank for the full 5 minutes. |
| `cellscape run belousov -f` | No | 0.29 | 0.00 | 0.29 | 0.0 | 12.94 | 4.15 | 17.09 | 693.0 | Blank from the start in 7 of 10 runs at 40x12, active in all 10 runs at 80x24. CPU shown at 40x12 is a blank run; active runs were 2.7% in a parallel run. |
| `cellscape run faders -f` | No | 1.24 | 0.10 | 1.34 | 7.4 | 3.19 | 0.39 | 3.58 | 37.3 | Fine until at least 6s, blank by 12s at both sizes, and stayed blank for the full 5 minutes. |
| `cellscape run forest -f` | Caveat | 1.53 | 0.19 | 1.72 | 32.2 | 4.37 | 1.07 | 5.44 | 167.2 | Works, but churn swings between 12% and 61% of cells per second. |
| `cellscape run wildfire -f` | No | 2.18 | 0.09 | 2.27 | 21.6 | 4.97 | 0.29 | 5.26 | 114.8 | Burns out to one or two visible cells within about 10s and stays that way, still using CPU. |
| `ternimal` | With args | 0.88 | 1.08 | 1.96 | 94.5 | 0.58 | 0.49 | 1.07 | 102.0 | Striped as given: every other row blank, worm often off-pane. Use `width=40 height=24 radius=2,4`. |
| `ternimal length=100 thickness=1,4,1,0,0 radius=6,12 gradient=0:#666600,0.5:#00ff00,1:#003300` | With args | 2.34 | 2.25 | 4.59 | 267.0 | 1.65 | 1.55 | 3.20 | 283.2 | Striped as given. Clean with `width=40 height=24 radius=2,4`. |
| `ternimal length=200 thickness=0,4,19,0,0` | With args | 3.97 | 1.51 | 5.48 | 217.0 | 2.51 | 1.16 | 3.67 | 225.4 | Striped as given. Clean with `width=40 height=24 radius=2,4`. |

`sneakers_auto` CPU comes from `/usr/bin/time` over 60s, not from the polling used for the rest (see Method).

## Things the Launcher needs to know

**A blank Animation never hands over.** `cellscape run brain`, `ltl`, `faders` and `wildfire` go blank and keep running, at both sizes. The map's rule that an Animation ending early hands over straight away will not fire, because nothing ends. In Rotation these would show an empty pane for the rest of the 5 minutes. Under Loop they would stay blank. The simplest fix is to leave them out of the collection, or at least out of Ambient.

**`cellscape run belousov` is unreliable at 40x12.** It stayed blank from the start in 7 of 10 runs, writing 563 bytes and then nothing. At 80x24 it was active in all 10 runs. It is also expensive at that size, at 17.1% in total.

**`ternimal` needs a size passed in.** Without `width` and `height`, every other row is blank at 40 columns at every height tested (12 to 24 rows), and at 44 to 52 columns at 12 rows. No stripes appeared at 60 columns or more, even at 12 rows. 56 columns was inconclusive. At 12 rows the worm also spends time out of view: at 40x12 the pane was empty or nearly so in 4 of 9 samples over 5 minutes, and 1 or 2 of 6 samples were empty at 60, 80 and 100 columns. The likely cause is a canvas bigger than the pane, so lines wrap and run off the bottom. That fits the binary's own error text, but I did not confirm it in its source.

The fix is to pass the pane size, with `height` at twice the row count because each row holds two pixels (`height` must be a multiple of 2):

| Command at 40x12 | App CPU % | tmux CPU % | Total % | KB/s |
|---|---|---|---|---|
| `ternimal width=40 height=24 radius=2,4` | 0.96 | 1.15 | 2.11 | 83.2 |
| `ternimal width=40 height=24 length=100 thickness=1,4,1,0,0 radius=2,4 gradient=0:#666600,0.5:#00ff00,1:#003300` | 2.57 | 1.71 | 4.28 | 147.3 |
| `ternimal width=40 height=24 length=200 thickness=0,4,19,0,0 radius=2,4` | 5.24 | 1.62 | 6.86 | 159.4 |

The `radius` is required. `ternimal width=40 height=24` on its own exits with status 1: "For the configured radius and padding, either width or height must be at least 43.6." `ternimal width=30 height=20 radius=2,4` also rendered cleanly at 30x10. The Launcher would have to read the pane size and build these arguments at start-up. These forms were each run for about 12s, not for 5 minutes, and behaviour after a resize was not tested.

**Output rate and CPU are separate costs.** `tarts plasma` looks cheap at 1.14% app CPU but writes 681 KB/s at 40x12 and 2.2 MB/s at 80x24, and tmux then uses more CPU than the app. `tarts donut` is the reverse: 9.47% app CPU and only 47 KB/s. Since the terminal has to draw whatever is written, KB/s is probably the better guide to the cost not measured here.

## Limits

- **Terminal drawing was not measured.** Ghostty's CPU and GPU work, and the tmux client's, are not in these numbers. Sessions were detached, so tmux had no client to draw for. Real totals will be higher, especially for the high KB/s commands.
- **Motion was not watched.** Verdicts come from captured frames, rendered to images for colour, plus change rates. Nothing here is a judgement of how something feels to sit next to.
- **The 5-minute run was 40x12 only, plus the 8 `cellscape` commands at 80x24.** The other 17 commands got 12s at 80x24.
- **The blank `cellscape` results come from one 5-minute run per size,** plus the shorter first run. Only `belousov` was repeated. Those runs used a real terminal device through tmux, but no keys were sent, and sizes above 80x24 were not tested.
- **Sizes below 40x12 were tested only for `ternimal`.**
- **CPU numbers wander by about 2x with machine load,** so small differences mean nothing. Single-command figures here are 10s windows run one at a time. Four were re-run alone for 60s and matched closely: `cmatrix` 0.46, `tarts boids` 4.64, `tarts donut` 8.85, `tarts plasma` 1.46. The 5-minute run had 33 commands running at once and read about 0.5 to 0.7 times lower, so its CPU figures are not used here. The likely reason is faster core clocks under load, which I did not verify.

## Method

Each command ran with the owner's PATH through `zsh -ic`, in its own tmux server on a private socket. The config was minimal because `~/.tmux.conf` sets `destroy-unattached`, which would kill a detached session. It had the status line off, so the pane was exactly the requested size, and `remain-on-exit` on, so a crash would be seen. The owner's own tmux sessions were not touched.

- **First run:** 25 commands at 40x12 and 80x24, one at a time. CPU was measured over 10s after a 2s warm-up, to skip `.zshrc` start-up.
- **5-minute run:** frames captured every 30s, to catch things going blank or exiting.
- **`belousov` repeats:** 8 extra runs of 20s at each size, because the first run and the 5-minute run disagreed at 40x12. That makes 10 runs at each size.
- **CPU** is the change in `ps` CPU time, polled every 0.5s. `ps` was checked against a known workload and agreed within 1%, including short low-duty bursts.
- **Output** was counted with `tmux pipe-pane`, which adds a little tmux CPU to every row.
- **Screens** were captured with `tmux capture-pane -e`, then parsed for character and colour.

The polling gave a wrong answer for `sneakers_auto`, the only command with child processes. It read 1.16% at 40x12 and 1.40% at 80x24 because of a baseline bug that affected only multi-process commands. Those rows use `/usr/bin/time` over 60s instead, which counts waited-for children: 0.09% and 0.18%.

Versions: cbonsai 1.4.2, cmatrix 2.0, tarts 0.1.25, tmux 3.7c. `cellscape` and `ternimal` versions were not recorded. Argument facts come from each tool's `--help` and, for `ternimal`, its own error messages.
