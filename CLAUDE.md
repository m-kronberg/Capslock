# CLAUDE.md

Personal fork of [Vonng/Capslock](https://github.com/Vonng/Capslock) — a Karabiner-Elements complex modification that turns CapsLock into a Hyper key (`right_cmd + right_ctrl + right_shift + right_option`).

## Source of truth

- `mac_v3/capslock.yml` — authoritative; edit here
- `mac_v3/capslock.json` — compiled output; must stay in sync with the YAML
- Personal customizations live in their own sections appended to `capslock.yml`

## Build & deploy

### Standard flow (requires `yq`)
```sh
cd mac_v3 && make install
```
This compiles YAML → JSON and copies to `~/.config/karabiner/assets/complex_modifications/`.

Use `make compile` to only regenerate `capslock.json` without touching the live
Karabiner config. Never hand-edit `capslock.json` — it is generated output.

### Critical gotcha: assets ≠ active rules
The file at `~/.config/karabiner/assets/complex_modifications/capslock.json` only lists *available* rules for the Karabiner UI to import. Karabiner does **not** read it live. Active rules live in `~/.config/karabiner/karabiner.json` under `profiles[0].complex_modifications.rules`.

To activate a new rule after editing the source files, either:
1. Open Karabiner-Elements → "Complex Modifications" → "Add rule", or
2. Edit `~/.config/karabiner/karabiner.json` directly (matches a more compact inline style with single-line modifiers). Karabiner auto-reloads on save.

Always back up before editing the active config:
```sh
cp ~/.config/karabiner/karabiner.json ~/.config/karabiner/karabiner.json.bak-$(date +%Y%m%d)
```

**Committing a rule does not activate it.** The repo and the active config are
independent; a rule can be committed, compiled, and still not bound to any key.
Verify against `karabiner.json` — never assume the repo reflects live behavior.

**"Add rule" in the UI appends without deduping.** Clicking it twice imports the
entire ruleset again. Karabiner takes the *first* match, so the copies are inert
but double the config and mislead future edits. (Found in this state on
2026-07-15: rules 0-9 duplicated at 10-19; removed.) Audit with:
```sh
python3 -c "
import json; d=json.load(open('$HOME/.config/karabiner/karabiner.json'))
s=[r.get('description') for r in d['profiles'][0]['complex_modifications']['rules']]
print('dupes:', [x for x in set(s) if s.count(x)>1] or 'none')"
```

Prefer editing `karabiner.json` idempotently — match rules by `description` and
replace in place, appending only when absent — so re-running cannot duplicate.

### Validation
```sh
python3 -c "import json; json.load(open('PATH')); print('valid')"
```
JSON-validity is necessary but not sufficient: Karabiner **silently reverts** a
config it rejects. After writing, confirm `pgrep -f karabiner_console_user_server`
is alive, then re-read the file and check your rules survived.

## Custom sections

### Apptivate (Cmd+number app launchers)
Cmd+1..5 → Chrome / Things3 / Claude / Obsidian / iTerm via `shell_command: open -a`. `open -a` natively launches if closed or focuses if open — same behavior as Apptivate.

**The Cmd+1..5 hijack is intentional — do not "fix" it.** It globally overrides
browser tab switching and Slack workspace switching. This is a deliberate,
accepted trade-off, not a bug. Leave it alone unless explicitly asked to scope
it (which would mean `conditions` + `frontmost_application_unless` on a bundle ID).

**Does not collide with Hyper Clipboard, despite appearances.** `Hyper Clipboard`
also binds `1`..`5` with `left_command`, but its `mandatory` list additionally
includes the full Hyper set (`right_command + right_control + right_shift +
right_option`) — i.e. it fires on Caps+Cmd+N, not plain Cmd+N. Karabiner requires
every modifier not listed in `mandatory`/`optional` to be *absent*, so the two
rules are mutually exclusive and their relative order does not matter:

- plain Cmd+N → launch/focus app
- Caps+Cmd+N → copy to clip slot N

**Never add `optional: [caps_lock]` to these rules.** It looks meaningful but is
a no-op: CapsLock is remapped to Hyper (the four `right_*` modifiers) and never
emits a `caps_lock` keycode, so the option can never match.

### Mouse Thumb Buttons to Middle Click
button4/button5 → button3, with `optional: [any]` so it fires regardless of modifiers.

**One-time setup per mouse:** open Karabiner-Elements → "Devices" tab → toggle "Modify events" on for the mouse. Without this, Karabiner won't intercept mouse events. If your mouse uses vendor software (Logitech Options, Razer Synapse, etc.) that may intercept buttons first — set the thumb buttons to "Generic Button 4 / Button 5" in the vendor app.

This toggle is GUI-only and cannot be done from the CLI. It writes a
`profiles[0].devices[]` entry with `is_pointing_device: true, ignore: false`; with
no such entry, pointing devices default to `ignore: true` and the rule is live but
inert — it will never fire and gives no error. Check whether the gate is open:
```sh
python3 -c "
import json; d=json.load(open('$HOME/.config/karabiner/karabiner.json'))
print([(x['identifiers'].get('product_id'), x.get('ignore'))
       for x in d['profiles'][0].get('devices', [])
       if x['identifiers'].get('is_pointing_device')] or 'no overrides -> ignored')"
```

## Git remotes

- `origin` → `m-kronberg/Capslock` (personal fork; push target) over SSH
- `upstream` → `Vonng/Capslock` (read-only; do not push)
- Work happens on `master`, which tracks `origin/master`. Plain `git push` is correct.
- Upstream's default branch is `main`; pull upstream changes with
  `git fetch upstream && git merge upstream/main`.

## Updating from Apptivate

If the Apptivate mapping changes and you want to re-sync, the hotkeys live at:
```
~/Library/Application Support/Apptivate/hotkeys
```
It's an NSKeyedArchiver binary plist. Decode with:
```sh
plutil -convert xml1 -o - ~/Library/Application\ Support/Apptivate/hotkeys
```
macOS virtual keycodes used by Apptivate entries: `18`=1, `19`=2, `20`=3, `21`=4, `23`=5, `22`=6, `26`=7, `28`=8, `25`=9, `29`=0. `mods: 256` = Command-only.
