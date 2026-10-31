# What existing Animations are worth adding

Research for the ticket [What existing Animations are worth adding](https://github.com/ajorquera/terminal-animations/issues/4), part of the [Terminal animations spec](https://github.com/ajorquera/terminal-animations/issues/1). Facts are as of 2026-10-31.

## Answer

Add these, in order: termflix, asciiquarium, drift, lavat, astroterm, TerminalTextEffects, weathr, chafa. The full ranking is in the [shortlist](#shortlist).

- **Most of the collection can stay simple.** Six of the eight run forever and need no required input, so Rotation can stop them after the set time.
- **termflix adds the most.** It has 60 animations, many of them nature scenes the current `screensaver` list lacks.
- **TerminalTextEffects is the odd one out.** It animates text you give it, then exits. Its Launcher entry has to supply the text and keep the slot filled.
- **Tools already in use overlap with many others.** Matrix rain, pipes and Game of Life are already covered by cmatrix, tarts and cellscape, so tools offering only those are left out.

## How this was checked

- **Primary sources:** project READMEs and docs, the GitHub API for dates and licences, the Homebrew formula API and PyPI.
- **Hands-on:** release binaries or Homebrew bottles were unpacked into a scratch folder and run in a private tmux 3.7c server, with a throwaway `HOME`. Nothing was installed globally.
- **Pane sizes tried:** 40x20 and 120x40, and 30x12 for asciiquarium only. Deeper pane-size work belongs to [Which Animations work at small pane sizes](https://github.com/ajorquera/terminal-animations/issues/2).
- **CPU figures:** rough, from detached tmux sessions over 30 seconds with other tests running. Use them to compare tools, not as real load.
- **Not checked:** nobody watched these. I read pane captures and project descriptions, so every "soothing" call is a judgement for the owner to confirm. Network traffic was not inspected.

"Tested" below means it was run this way. Anything else comes from the project's docs.

## Shortlist

Ranked by fit: soothing, runs unattended, needs no input, works in a small pane, easy to install on a Mac.

| Rank | Animation | Ambient | Break | Ends on its own? | Input needed |
|---|---|---|---|---|---|
| 1 | termflix | Good | Excellent | No | None |
| 2 | asciiquarium | Good from about 40x20 | Excellent | No | None |
| 3 | drift | Good | Good | No | None |
| 4 | lavat | Good, tune for small panes | Good | No | None |
| 5 | astroterm | Excellent, very calm | Fair | No | Optional location |
| 6 | TerminalTextEffects | Fair, needs a wrapper | Good | **Yes** | **Text, required** |
| 7 | weathr | Only if the pane is at least 70x20 | Excellent | No | Optional location, uses the network |
| 8 | chafa | Depends on the GIF | Depends on the GIF | No for one GIF on a terminal | **GIF file, required** |

## Shortlisted tools

### 1. termflix

- **Shows:** 60 procedural animations. Calm picks: `aurora`, `fireflies`, `rain`, `ocean`, `petals`, `ripple`, `snow`, `garden`, `smoke`, `lava`, `flow`, `waterfall`, `campfire`, `rainforest`, `ink_in_water`, `metaballs`, `solar_system`, `physarum`. Tested: `--list` prints 60 on v0.8.0.
- **Install:** not in Homebrew. Use the release binary `termflix-macos-aarch64`, `cargo install termflix`, or the project's curl script.
- **Maintained:** yes, with one contributor. v0.8.0 on 2026-06-18, last push 2026-09-30.
- **Licence:** MIT.
- **Form:** CLI, single Rust binary.
- **Runs:** forever. Tested for 40 seconds at both sizes.
- **Input:** none. It does read NDJSON control data from stdin whenever stdin is not a terminal, so the Launcher should keep that in mind.
- **Ambient:** good. The README says it detects tmux and adapts its frame rate. Tested `aurora` at about 1% CPU at 40x20 and 3% at 120x40, and `fireflies` lower.
- **Break:** excellent, with the widest range here.
- **Launcher notes:** run one animation per entry, such as `termflix aurora --clean`, so Skip, Favourite and Ban work per animation. Leave out its own `--cycle`. Avoid `--screensaver` in Ambient, because it exits on focus loss. A stray `x` did not quit it.

### 2. asciiquarium

- **Shows:** fish, whales and ships in ASCII art.
- **Install:** `brew install asciiquarium`, in Homebrew core.
- **Maintained:** dormant but finished. Upstream is v1.1, and the [GitHub mirror](https://github.com/cmatsuoka/asciiquarium)'s newest commit dates from 2012. Homebrew still ships bottles for current macOS.
- **Licence:** GPL-2.0-or-later, per Homebrew.
- **Form:** CLI, Perl script.
- **Runs:** forever. Tested for 40 seconds.
- **Input:** none.
- **Ambient:** good from about 40x20, where the waterline, castle and fish all fit. At 30x12 it ran, but the castle filled the tank. About 1% CPU.
- **Break:** excellent, and the most obviously soothing of the set.
- **Launcher notes:** `q` quits, `p` pauses, `r` redraws. A stray `x` did not quit it.

### 3. drift

- **Shows:** 14 scenes and 10 themes. New compared with tarts: `waveform`, `orrery`, `particles`, `starfield`, `clock`, `dvd`. The rest overlap with what tarts, cbonsai and cellscape already show.
- **Install:** `brew install phlx0/drift/drift`, a third-party tap, last updated the day of the latest release.
- **Maintained:** yes, and actively. v1.3.0 on 2026-09-10, last push 2026-10-16. It is young (created March 2026) and mostly one author.
- **Licence:** MIT.
- **Form:** CLI, single Go binary.
- **Runs:** forever, changing scene every 60 seconds by default.
- **Input:** none.
- **Ambient:** good. Tested `waveform` at about 1% CPU at 40x20.
- **Break:** good.
- **Launcher notes:** lock each entry with `drift --scene waveform --duration 0`. **Any key quits it** (tested, exit 0), so a key pressed in a focused Ambient pane ends the Animation. Its config has `hide_tmux_status`, which should stay off in Ambient.

### 4. lavat

- **Shows:** a lava lamp made of metaballs.
- **Install:** `brew install lavat`, in Homebrew core.
- **Maintained:** yes. v3.0.0 on 2026-01-27.
- **Licence:** MIT.
- **Form:** CLI, C.
- **Runs:** forever. Tested for 40 seconds.
- **Input:** none.
- **Ambient:** good once tuned. With default ball size at 40x20 the lava nearly filled the pane as one solid block, so a small pane wants a lower `-r` and `-b`. Tested at `-s 2`, it used under 0.5% CPU, among the lightest here. Colours are set with `-c` and `-k`.
- **Break:** good. The README's example `lavat -g -b 8 -r 10 -R 3 -G` adds gradients and gravity.
- **Launcher notes:** a stray `x` did not quit it. Letter keys adjust the lamp while it runs.

### 5. astroterm

- **Shows:** the real night sky, with stars, planets, moon phase and constellations.
- **Install:** `brew install astroterm`, in Homebrew core.
- **Maintained:** yes. v1.2.0 on 2026-04-01, last push 2026-09-11.
- **Licence:** MIT.
- **Form:** CLI, C.
- **Runs:** forever. Tested for 40 seconds.
- **Input:** none required, but the default location is 0,0. Pass `-i <city>` or `-a`/`-o`.
- **Ambient:** excellent and very calm. It fits 40x20 well, and at `-f 2` it used about 0.2% CPU.
- **Break:** fair. At real speed the sky barely moves, so it is more a picture than an animation. `-s` speeds it up.
- **Launcher notes:** suggested entry is `astroterm -c -u -C -f 2 -i <city>`. Only `q` and Esc quit it, per its help text.

### 6. TerminalTextEffects

See [its own section](#terminaltexteffects).

### 7. weathr

- **Shows:** an animated scene of the real current weather: rain, snow, storms, day and night, passing planes.
- **Install:** `brew install Veirt/veirt/weathr`, a third-party tap, or the release binary.
- **Maintained:** yes. v1.4.0 on 2026-02-23, last push 2026-08-12.
- **Licence:** GPL-3.0-or-later.
- **Form:** CLI, Rust.
- **Runs:** forever. Tested for 40 seconds at 120x40.
- **Input:** none required. It needs the network for live weather from Open-Meteo, and a location from `config.toml` or IP lookup through ipinfo.io. `--simulate rain` shows a condition on demand.
- **Ambient:** poor in small panes. **At 40x20 it exited with status 1 and asked for at least 70x20** (tested).
- **Break:** excellent, and the only Animation here that reflects the world outside.
- **Launcher notes:** treat a fast exit in a small pane as "does not fit". Suggested entry is `weathr --hide-hud --silent`, with the location set in the config.

### 8. chafa

- **Shows:** any GIF or image, rendered as terminal graphics. It is a player, so the content is whatever the repo supplies.
- **Install:** `brew install chafa`, in Homebrew core.
- **Maintained:** yes. 1.18.3 on 2026-09-23.
- **Licence:** LGPL-3.0-or-later.
- **Form:** CLI and C library.
- **Runs:** per its man page, one animated file on a terminal plays forever, unless `--duration` is given. With several files, or with no terminal, each plays once.
- **Input:** a GIF file is required.
- **Ambient and Break:** depend entirely on the GIF. Not tested. It is worth having as a slot for looping footage such as rain or a fire, but the GIFs must be found or made first.

## TerminalTextEffects

**The claim holds for the released version, with one correction.** It animates text you give it, then exits. That is true of 0.15.0, the latest release. The next version changes this, and it is also a library.

### Record

- **Shows:** 37 effects in 0.15.0 (`tte -h`), such as `waves`, `smoke`, `rain`, `thunderstorm`, `burn`, `blackhole`, `fireworks`, `matrix`, `decrypt`.
- **Install:** not in Homebrew. Use `uv tool install terminaltexteffects` or `pipx install terminaltexteffects`. `uv` is on this Mac and `pipx` is not. No third-party runtime dependencies, per PyPI.
- **Maintained:** very actively. 0.15.0 on 2026-05-10, last commit 2026-10-31.
- **Licence:** MIT.
- **Form:** both. The `tte` CLI, and a Python library where each effect is an iterator of frames.
- **Ambient:** fair. The motion is short and then the text sits still, so it needs a wrapper to fill the time.
- **Break:** good, especially with `--canvas-width 0 --canvas-height 0 --anchor-text c`, which centres the text in the pane.

### What the tests showed

All of this was on 0.15.0.

- **It ends on its own.** All 111 timed runs exited with status 0 and none hung: 37 effects at each of three setups, running three at a time.

  | Setup | Shortest | Median | Longest |
  |---|---|---|---|
  | 40x20, 3-line text | 0.9s | 5.3s | 54.7s |
  | 120x40, 3-line text | 1.1s | 7.0s | 116.8s |
  | 120x40, 20-line text | 2.4s | 14.4s | 82.1s |

  `swarm` was the longest every time. Without it, the longest runs were 30.2s, 34.9s and 48.2s.
- **Text is required.** Empty stdin, `/dev/null` and a terminal on stdin all print `NO INPUT.` and exit 1 at once. A Launcher that gives it no pipe gets an instant failure.
- **Text must fit.** At 40 columns a 100-character line disappeared from the final frame unless `--wrap-text` was set. A 60-line file in a 20-row pane showed only the last lines.
- **Stopping it matters.** After SIGINT the cursor came back. After SIGTERM it stayed hidden and part of a frame was left on screen. The Launcher should send SIGINT or reset the pane. This is one for [How to control third-party Animations](https://github.com/ajorquera/terminal-animations/issues/3).
- **It leaves its text behind.** It prints inline and exits without clearing, so the Launcher must clear the pane before the next Animation.
- **CPU varies by effect.** `waves` used 2 to 4%. `matrix` used 10% at 40x20 and 37% at 120x40. The default is 60 frames per second, and `--frame-rate` can lower it.

### The next version

The project's README and docs follow unreleased `main`, heading for 0.16.0, so they describe options 0.15.0 does not have. Tested: `tte -h` on 0.15.0 lists none of these.

- `--repeat 0` loops until interrupted, and with `--random-effect` picks a new effect each time.
- `--hold SECONDS` keeps the final frame before the next playback.
- `--export` and `--play` save frames to `.tterec` files and replay them, including a shuffled folder.

Recordings fix the canvas size and fail on a smaller terminal, so they suit Break better than a resizable Ambient pane.

### How it fits Rotation

Rotation hands over straight away when an Animation ends early. A plain `tte` entry would therefore last a few seconds and then jump to the next Animation, which is the opposite of calm.

**Recommendation:** make it one Animation written in this repo, a small script that loops until stopped:

1. Pick a text.
2. Clear the pane.
3. Run a random calm effect, with `--include-effects`, `--wrap-text`, a full-pane canvas and centred text.
4. Hold the finished text for 20 to 30 seconds.
5. Repeat.

To the Launcher this is a normal run-forever Animation. Rotation stops it at the time limit and Loop just works. The script is still useful after 0.16.0, because `--repeat` reuses one input and this picks new text each round.

Starting set of effects: `waves`, `smoke`, `rain`, `thunderstorm`, `colorshift`, `spotlights`, `blackhole`, `burn`. Leave out `swarm` for its length and `matrix` for its CPU use. These picks come from the docs and need the owner's eye.

### What text to animate

Use a small collection of short texts kept in this repo, as the main source: haiku, quotes, or lines the owner likes. It needs no network or extra tools, it can never be empty, and three or four short lines suit every pane size.

Two extras could sit alongside it:

- **The time and date:** `date '+%A %H:%M'`, which gives the pane a quiet use.
- **A line from the current project:** the latest `git log` subject. Keep it optional, since work text may pull attention back to work.

Avoid live command output and long files. They are too large for small panes, and they break the soothing goal. Falling back to a fixed line whenever a source is empty avoids the `NO INPUT.` failure.

## Considered and left out

Facts here come from project pages and the GitHub and Homebrew APIs, unless marked tested. "Ambient/Break" gives fit for each.

| Tool | Shows | Mac install | Latest release or commit | Licence | Runs | Input | Form | Ambient / Break | Why left out |
|---|---|---|---|---|---|---|---|---|---|
| [arttime](https://github.com/poetaman/arttime) | Text-art clock that flips between two pictures | `brew install arttime` | v2.5.0, 2026-08-17 | GPL-3.0 | Forever | Optional art and message | CLI | Possible / weak | Closest to the shortlist. A clock and timer app first, not tested. |
| [cava](https://github.com/karlstav/cava) | Audio spectrum bars | `brew install cava` | 1.0.0, 2026-06-13 | MIT | Forever | Audio | CLI | Good with music / good with music | Nothing to see in silence. |
| [ttysvr](https://github.com/cxreiff/ttysvr) | 3D bubbles, logo, maze | `brew install cxreiff/tap/ttysvr` | v0.3.4, 2024-11-27 | Apache-2.0 | Forever | None | CLI | Poor / fair | Only three scenes. Needs truecolor and efficient rendering. |
| [bash-screensavers](https://github.com/attogram/bash-screensavers) | 12 scenes including rain, stars | Clone the repo | Commit 2026-07-23 | MIT | Forever | None | Scripts | Untested | Overlaps termflix. No package. |
| [rusty-rain](https://github.com/cowboy8625/rusty-rain) | Matrix rain, recolourable | `cargo install rusty-rain` or binary | v0.5.1, 2026-10-04 | Apache-2.0 on GitHub, README badge says MIT | Forever (tested) | None | CLI | Good / fair | Overlaps cmatrix and tarts. |
| [pipes-rs](https://github.com/lhvy/pipes-rs) | Drawing pipes | `brew install lhvy/tap/pipes-rs` | v1.6.4, 2025-08-22 | Blue Oak 1.0.0, per README | Forever (tested) | None | CLI | Good / fair | Overlaps tarts `pipes`. |
| [pipes.sh](https://github.com/pipeseroni/pipes.sh) | Drawing pipes | `brew install pipes-sh` | v1.3.0, 2018 | MIT | Forever | None | Bash script | Fair / fair | Overlaps tarts. Any key quits. |
| [genact](https://github.com/svenstaro/genact) | Fake busy logs | `brew install genact` | v1.6.0, 2026-10-01 | MIT | Forever unless given exit flags | None | CLI | Poor / poor | Looks like real work output, so it distracts. |
| [nyancat](https://github.com/klange/nyancat) | Nyan Cat | `brew install nyancat` | Homebrew 1.5.2, repo push 2024 | NCSA, per Homebrew | Forever | None | CLI | Poor / fair | Loud, not soothing. |
| [sl](https://github.com/mtoyoda/sl) | A passing steam train | `brew install sl` | 5.02, 2014 | MIT, per Homebrew | Ends | None | CLI | Poor / poor | A one-joke animation. |
| [tty-clock](https://github.com/xorg62/tty-clock) | Large clock | `brew install tty-clock` | Push 2024-07-31 | BSD-3-Clause | Forever | None | CLI | Fair / poor | A clock with no animation. drift has one. |
| [zenta](https://github.com/e6a5/zenta) | Guided breathing | Curl script | v1.1.0, 2025-07-11 | MIT | Ends after set cycles | None | CLI | Poor / fair | A breathing exercise, not something to watch. |
| [unimatrix](https://github.com/will8211/unimatrix) | Matrix rain | Python script, no formula | Push 2026-05-20 | GPL-3.0 | Forever | None | CLI | Fair / fair | Overlaps cmatrix. |
| [neo](https://github.com/st3w/neo), [cxxmatrix](https://github.com/akinomyoga/cxxmatrix), [fakesteak](https://github.com/domsson/fakesteak) | Matrix rain | Build from source; fakesteak has a formula | 2022, 2024, 2020 releases or pushes | Mixed | Forever | None | CLI | Fair / fair | Overlap cmatrix, little recent work. |
| [globe](https://github.com/adamsky/globe), [termsaver](https://github.com/brunobraga/termsaver), [gh-screensaver](https://github.com/vilmibm/gh-screensaver) | Globe; mixed screensavers | No formula | 2020 to 2024 | GPL-3.0, Apache-2.0, GPL-3.0 | Forever | None | CLI | Untested | Stale, and termflix covers the same ground. |

## Notes for other tickets

- [How to control third-party Animations](https://github.com/ajorquera/terminal-animations/issues/3): drift quits on any key. TerminalTextEffects needs SIGINT or a pane reset. termflix behaves differently when stdin is not a terminal.
- [Which Animations work at small pane sizes](https://github.com/ajorquera/terminal-animations/issues/2): weathr refuses anything under 70x20. lavat needs smaller settings. asciiquarium is cramped at 30x12.
- [What Ambient should look and feel like](https://github.com/ajorquera/terminal-animations/issues/5): astroterm, lavat and asciiquarium make a calm starting set.
- [What an Animation definition looks like](https://github.com/ajorquera/terminal-animations/issues/7): one tool can yield many Animations, so an entry needs a command with its own flags. Each entry may also need a minimum size and a stop method.
- "How third-party dependencies get installed" (not yet specified): four of the eight are not Homebrew core formulae. drift and weathr need taps, termflix needs a binary or cargo, and TerminalTextEffects needs uv.

## Sources

- Projects: [termflix](https://github.com/paulrobello/termflix), [termflix gallery](https://paulrobello.github.io/termflix/), [asciiquarium](https://robobunny.com/projects/asciiquarium/html/), [drift](https://github.com/phlx0/drift), [lavat](https://github.com/AngelJumbo/lavat), [astroterm](https://github.com/da-luce/astroterm), [weathr](https://github.com/Veirt/weathr), [chafa man page](https://hpjansson.org/chafa/man/), [pipes.sh man page](https://pipeseroni.github.io/pipes.sh/pipes.sh.6.html)
- TerminalTextEffects: [repository](https://github.com/ChrisBuilds/terminaltexteffects), [application guide](https://chrisbuilds.github.io/terminaltexteffects/appguide/), [changelog](https://chrisbuilds.github.io/terminaltexteffects/changelog/), [PyPI](https://pypi.org/project/terminaltexteffects/)
- Package data: [Homebrew formulae](https://formulae.brew.sh/), for versions, licences and bottles
- Discovery only: [awesome-terminal-art](https://github.com/mcthomas/awesome-terminal-art)
