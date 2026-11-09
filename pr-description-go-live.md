Makes the downstream keyers a "set up, then take" control that works with OBS Studio Mode, the way a hardware switcher's downstream keyers do. Selecting keyer scenes no longer puts them on air. Scenes are checked, then taken live with a button, cued on the Studio Mode preview, or taken live or off together with the program scene transition. Several scenes can be live on one keyer at once.

**Preview a keyer on a scene in Studio Mode, then take both live together.** With **Go to Preview**, the keyer is shown on top of the Studio Mode preview scene, so the operator can see exactly how the lower third will look over that scene before it goes out. When the preview scene is transitioned to Program, the keyer goes live with it in the same transition. No second click is needed, and nothing is on air until the scene transition. This isn't possible today, because keyer output never appears in the Studio Mode preview.

The goal is the same as Load On Scene: let a busy operator set up the next graphic without it going on air by accident, and then take it at the right moment, ideally together with the scene change.

**This builds on #67 (Load On Scene).** The first seven commits are that PR's. Once #67 is merged, this PR only contains the Go Live commits.

## Usage
- **All Keyers row** (above the tabs, acts on every keyer):
  - **All Keyers Off**
  - **Keyer Off at Scene Change**: one-shot, clears every keyer except the Load On Scene keyers of the new scene on the next transition
  - **Hold Keyers**: keyers stay as they are on transitions and Hide After stops
  - **Transition Keyers** (Studio Mode): the program keyers become what the preview shows, without a scene transition
- **Keyer Action column** (each tab, acts on that keyer only):
  - **Go Live** (Studio Mode off)
  - **Go to Preview** and **Go to Program** (Studio Mode on)
  - **Keyer Off**
- **Check boxes** on the right of each scene name choose what is taken.
- **Include Audio** in the gear menu.

## Behaviour
- **Tally.**
  - Keyer tabs are filled red while on program and green while shown on the Studio Mode preview. The selected tab is gray otherwise and has a white underline.
  - In the list, live scenes are red bold and cued scenes green bold.
- **Several scenes at once.** Checked scenes go live together, layered in list order with the top of the list on top. More than one scene is put on the output as a private scene holding them as layers, so the keyer's transition blends the group as one picture.
- **The Studio Mode preview shows the next look of every keyer:** cued keyers, Load On Scene keyers of the previewed scene, held keyers, and keyers that stay. A plugin can't draw over the Studio Mode preview, so each keyer's next look is a temporary, locked scene item at the top of the preview scene, using a private source and tagged in its private settings.
  - On a program transition, cued keyers ride in with the scene and are cut onto their keyer output, with the same picture, when the transition ends. The other copies are removed when the transition starts.
  - **Duplicate Scene** is handled: copies are also removed from the private program copy OBS makes, and any copy left in the program scene is removed after each transition.
  - Copies are never added to the scene that is on program, and leftovers in a saved scene collection are purged on load.
- **Scene transitions.** On the main output, keyers change when the program transition starts, so keyer and scene transitions run together. OBS's scene transitions are private sources, so the plugin connects to each transition's own `transition_start` signal and reconnects when the transition list changes. Per keyer, in order:
  1. Hold Keyers keeps it as is.
  2. Exclude Scene hides it.
  3. A cue for the scene goes live.
  4. Keyer Off at Scene Change, or another keyer's cue being taken, clears it unless it is a Load On Scene keyer of the new scene.
  5. Load On Scene hides it on unlisted scenes and loads it on listed ones.
  6. Otherwise it stays.
- **Keyer transitions.** A keyer without its own Transition follows the OBS scene transition and its duration, as a private copy kept in step with the Scene Transitions dock. The gear menu shows this as **Same as Scene Transition**.
- **Include Audio, off by default.** Without it, the keyer output is wrapped in a small private video-only source (`downstream_keyer_video_only`, hidden from the source list), so the keyer scenes' audio isn't mixed into the output.
- **Hide After** starts when scenes go live, never during a cue, and is stopped by Hold Keyers.

## Compatibility
- **Old settings:** the scene that was on air stays live and nothing is checked. **Tie** is ignored.
- **New keys:**
  - per keyer: `checked` on each scene, `live_scenes`, `include_audio`
  - per dock: `downstream_keyers_hold`, `downstream_keyers_off_at_scene_change`
  - `scene` is still written.
- **Behaviour changes:**
  - Include Audio defaults to off, so keyers that played audio are silent until it is checked.
  - Keyers without a Transition now fade with the scene transition instead of cutting.
  - Selecting a keyer scene no longer puts it on air.
- **Removed:** the Tie check box, its hotkeys, and `dsk_set_tie` (it now returns `success: false`).
- **Existing websocket requests:**
  - `dsk_select_scene` checks only that scene and puts it live.
  - `dsk_get_scene` returns the first live scene.
  - `dsk_scene_changed` reports the live scenes, separated by commas.
- **Existing hotkeys:**
  - The per-scene **Show on** / **Hide on** hotkeys add the scene to, or remove it from, the live scenes right away.
  - The keyer **Hide on** hotkey is that keyer's Keyer Off.

## obs-websocket
New vendor requests on the `downstream-keyer` vendor. Each also takes an optional `view_name`.

| Request | Fields |
|---|---|
| `dsk_go_live` | `dsk_name` (required), `scene` or `scenes` (optional: check exactly these scenes first) |
| `dsk_go_to_preview` | `dsk_name` (required), `scene` or `scenes` (optional) |
| `dsk_all_keyers_off` | none |
| `dsk_set_hold_keyers` | `hold` (required, bool) |
| `dsk_set_keyer_off_at_scene_change` | `enabled` (required, bool) |
| `dsk_transition_keyers` | none (main output in Studio Mode) |

`scenes` is an array of `{ "name": "..." }` objects. The response has `success`, plus `error` when `view_name` is unknown, a required field is missing, the keyer or a scene is not found, or Studio Mode is not active for `dsk_transition_keyers`.

## Changes
- `downstream-keyer.cpp/.hpp`:
  - the Keyer Action column and the right-hand check box delegate
  - live and cue state, and building the layered source
  - the preview next look and cue take
  - the scene change rules
  - the default scene transition
  - the video-only wrapper source
  - save/load and the new hotkeys
- `downstream-keyer-dock.cpp/.hpp`:
  - the All Keyers row and its saved settings
  - the tab tally
  - the transition start hooks
  - preview and Studio Mode events
  - cleanup of keyer copies on program and on load
  - the **Include Audio** menu item and registering the video-only source
  - the websocket requests
- `data/locale/en-US.ini`: new strings

## Testing
**Tested on Windows 11 x64 with OBS 33.0**, building against the OBS 31.1.0 deps with MSVC 2022 (RelWithDebInfo, no warnings):
- Installed the build into OBS and checked the dock layout, button labels and tab tally with the default theme.
- With **Include Audio** off, put a keyer scene with audio on air and recorded. The recording was silent.
- In Studio Mode, used **Go to Preview** on a keyer with **Hide After** set. The keyer stayed on the preview scene with no timer running. After the scene was transitioned to Program, the keyer went live with it and turned off after the set duration.
- Swapped keyers on a scene change. One keyer was live on Program and another was cued with **Go to Preview**. On the scene transition, the live keyer faded out with the scene and the cued keyer went live with it.
  - Checked with temporary debug logging: every keyer change started in the same millisecond as the program transition.
  - Ran it with keyers applied manually, and with a Load On Scene keyer plus **Keyer Off at Scene Change**.
- With **Duplicate Scene** on, checked that **All Keyers Off** clears a Load On Scene keyer from Program and that no keyer stays behind after transitions.
- Code is clang-format clean against the OBS style with `ColumnLimit: 132`.

**Not yet tested:**
- **macOS and Linux.** I haven't built or run it on these; I'm relying on this PR's CI builds.
- **Not systematically tested on Windows yet:**
  - the obs-websocket requests (built and compiled, not yet called from a client) and hotkeys
  - keyers on extra views/canvases
  - **Swap Scenes**
  - stinger transitions (keyers change when the transition starts, not at the stinger's transition point)

## Sign-off

Signed-off-by: Micah Harman \<micahharman20@gmail.com\>

Assisted-by: Claude:claude-opus-5-5
