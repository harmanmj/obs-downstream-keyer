# Go Live, Go to Preview and All Keyers controls

These changes turn the downstream keyers into a "set up, then take" control that works with OBS Studio Mode. Selecting keyer scenes no longer puts them on air. You check the scenes you want, then take them live with a button, cue them on the Studio Mode preview, or let a program scene transition take them, the way a hardware switcher's downstream keyers work.

## The dock

```
All Keyers
[ All Keyers Off ] [ Keyer Off at Scene Change ] [ Hold Keyers ] [ Transition Keyers ]
[ Lyrics ][ Speakers ][ Bumpers ]                                              [⚙]
┌──────────────────────────────┐ ┌──────────────────────────────┐
│ Lower Third - Pastor     [✓] │ │         Keyer Action         │
│ Lower Third - Worship    [ ] │ │ [     Go to Preview      ]   │  Studio Mode
│                              │ │ [     Go to Program      ]   │  Studio Mode
│                              │ │ [       Keyer Off        ]   │
└──────────────────────────────┘ └──────────────────────────────┘
[+] [−]  [↑] [↓]
```

- **The All Keyers row** (above the tabs) acts on every keyer in the dock.
- **The Keyer Action column** (in each tab) acts on that tab's keyer only.
- **Check boxes** (right of each scene name) choose the keyer scenes to take. Clicking a scene name only highlights it for the move and remove buttons.
- **Tab tally:** a tab is filled **red** while its keyer is on program and **green** while it shows on the Studio Mode preview. The selected tab is gray when it is neither, and always has a white underline.
- **Scene tally:** in the list, live scenes are red bold and cued scenes green bold.

## Several scenes at once

Any number of scenes can be checked. They go live together, layered in list order with the top of the list on top. Use the move up/down buttons or drag to change the order.

One scene goes on the keyer output as itself. Several go on the output as a private scene that holds them as layers, so a keyer transition blends the whole group at once. This costs about the same as showing those scenes anyway.

## Keyer Action (each tab)

| Button | Shown | What it does |
|---|---|---|
| **Go Live** | Studio Mode off | Puts the checked scenes on the keyer output now, with the keyer's transition. Red when the checked scenes differ from what is live. With nothing checked, the keyer goes off. |
| **Go to Preview** | Studio Mode on | Cues the checked scenes on the preview scene. Click again to clear the cue. Green while scenes are checked. |
| **Go to Program** | Studio Mode on | Puts this keyer's cue, or its checked scenes when nothing is cued, on program now, without a scene transition. Other keyers are not changed. Red while scenes are checked. |
| **Keyer Off** | Always | Takes this keyer off program, with its transition. The check boxes and any cue stay. Other keyers are not changed. |

Go to Preview and Go to Program are greyed out while nothing is checked.

## All Keyers (above the tabs)

| Button | What it does |
|---|---|
| **All Keyers Off** | Takes every keyer off program now, even while Hold Keyers is on. Load On Scene keyers come back the next time a listed scene goes to program. |
| **Keyer Off at Scene Change** | One-shot toggle. On the next program scene transition, every keyer goes off except the Load On Scene keyers of the new scene. It turns itself off when that transition ends. Greyed out while Hold Keyers is on. |
| **Hold Keyers** | Toggle. Keyers stay as they are on scene transitions, ignoring Load On Scene and Exclude Scene, and Hide After does not count down. A cued keyer still goes live with its transition. Turning it off matches the keyers to the program scene right away: the Load On Scene keyers of that scene load, the others go off. |
| **Transition Keyers** | Studio Mode only. Makes the program keyers match what the preview shows, without a scene transition: cued keyers go live, keyers shown on the preview go live with those scenes, and the rest go off. |

## Program scene transitions

On the main output, keyers change when the program transition **starts**, so they move together with the scene. Each keyer follows these rules, in order:

1. **Hold Keyers on:** no change. A cued keyer still goes live.
2. **Exclude Scene:** the keyer output is off on that scene. What was live comes back on the next scene the keyer is shown on.
3. **Cued with Go to Preview, and its preview scene is the one going to program:** the cue goes live with the scene.
4. **Keyer Off at Scene Change is on, or another keyer's cue is being taken:** the keyer goes off, unless it is a Load On Scene keyer of the new scene.
5. **Load On Scene, and the new scene is not listed:** the keyer output is off for now. What was live comes back on a listed scene.
6. **Load On Scene, the new scene is listed, and nothing is live:** the keyer loads its last live scenes, or the first scene in its list.
7. **Otherwise:** the keyer stays as it is.

Keyers on extra views or canvases follow the same rules when their scene change is reported.

## The Studio Mode preview

The preview scene shows what every keyer will show after a transition to it, by the rules above. That includes cued keyers, Load On Scene keyers listed for that scene, held keyers, and keyers that stay. A cue removes from the preview the keyers it will take off.

OBS has no way for a plugin to draw over the Studio Mode preview, so each keyer's next look is added to the preview scene as a temporary, locked source at the top of its source list, stacked in keyer tab order.

- **It updates** as you check scenes, cue keyers, take keyers live or off, change the All Keyers buttons, or pick another preview scene.
- **On a transition,** cued keyers ride in with the scene and move to their keyer output, with the same picture, when the transition ends. The other copies are removed when the transition starts, because those keyers are shown by their own output.
- **Duplicate Scene and Swap Scenes** are supported. With Duplicate Scene, OBS puts a private copy of the preview scene on program, and keyer copies are removed from that copy too.
- **Never added to the scene that is on program,** because that would put it on air.
- **Leftovers are cleaned up.** Any keyer copy left in the program scene is removed after each transition, and copies left in a saved scene collection are removed when it loads. The copies use private sources, so a saved collection never keeps a real lower third item.

## Keyer transitions

A keyer without a Transition of its own follows the OBS scene transition selected in the Scene Transitions dock, with its duration. Changing that dock changes these keyers on their next change. In the gear menu this shows as **Same as Scene Transition**, and Show / Hide Transition show **Same as Transition**.

A keyer uses, in order: Show / Hide Transition when it turns on or off, a Transition Table override between the two keyer scenes, its own Transition, then the OBS scene transition.

## Include Audio

Keyer audio is left out of the output mix by default. Check **Include Audio** in the gear menu to put a keyer's audio back.

- **Off (default).** The keyer output is wrapped in a private video-only source, so the scenes show but their sound is not mixed into the stream or recording. The sources still appear in the Audio Mixer with moving meters, because the mixer shows each source's own level. Right-click a strip and choose **Hide** to hide it there.
- **On.** The keyer's audio is mixed into the output.
- **Shared sources.** If an audio source is in both a keyer scene and the program scene, leave Include Audio off. With it on, OBS mixes it twice and it plays about twice as loud.

## Hide After

Hide After counts down from the moment scenes go live (a button, a scene transition, or a cue being taken). It does not run while scenes are only cued, and Hold Keyers stops it. When it runs out the keyer goes off and its check boxes stay.

## Hotkeys

- **Go Live** and **Go to Preview** hotkeys for each keyer. Go Live works in Studio Mode too.
- The per-scene **Show on** / **Hide on** hotkeys check or uncheck that scene and add it to, or remove it from, what is live right away.
- The **Hide on** keyer hotkey works like that keyer's **Keyer Off**.

## obs-websocket

These vendor requests are added to the `downstream-keyer` vendor. Every request takes an optional `view_name` (the view or canvas name; leave it out or empty for the main OBS output).

| Request | Fields | Description |
|---|---|---|
| `dsk_go_live` | `dsk_name` (required), `scene` or `scenes` (optional) | Same as **Go Live** / **Go to Program** |
| `dsk_go_to_preview` | `dsk_name` (required), `scene` or `scenes` (optional) | Same as **Go to Preview**, but it only starts a cue and never clears one |
| `dsk_all_keyers_off` | none | Same as **All Keyers Off** |
| `dsk_set_hold_keyers` | `hold` (required, bool) | Turns **Hold Keyers** on or off |
| `dsk_set_keyer_off_at_scene_change` | `enabled` (required, bool) | Arms or disarms the one-shot **Keyer Off at Scene Change** |
| `dsk_transition_keyers` | none | Same as **Transition Keyers**. Main output in Studio Mode only |

- `scene` checks only that keyer scene first. `scenes` checks exactly the listed scenes, as an array of objects: `"scenes": [{ "name": "Lower Third - Pastor" }, { "name": "Bug" }]`. Without either, the scenes that are already checked are used.
- The response has `success`, plus `error` when `view_name` is unknown, a required field is missing, the keyer or a scene is not found, or (for `dsk_transition_keyers`) Studio Mode is not active.
- Requests return right away; the action runs on the OBS UI thread a moment later.

Example (obs-websocket v5 `CallVendorRequest`):

```json
{
  "op": 6,
  "d": {
    "requestType": "CallVendorRequest",
    "requestId": "1",
    "requestData": {
      "vendorName": "downstream-keyer",
      "requestType": "dsk_go_live",
      "requestData": {
        "dsk_name": "Lyrics",
        "scenes": [{ "name": "Lower Third - Pastor" }, { "name": "Bug" }]
      }
    }
  }
}
```

Existing requests:

- `dsk_select_scene` checks only that scene and puts it live right away. An empty `scene` turns the keyer off.
- `dsk_get_scene` returns the first live scene.
- `get_downstream_keyers` / `get_downstream_keyer` include the live and checked scenes, and the dock's `downstream_keyers_hold` and `downstream_keyers_off_at_scene_change`.
- `dsk_set_tie` returns `success: false` with "tie is no longer supported".
- The `dsk_scene_changed` event's `new_scene` lists the live scenes, separated by commas.

## Saved settings

Each keyer saves:

```json
"scenes": [
  { "name": "Lower Third - Pastor", "checked": true },
  { "name": "Scripture Overlay", "checked": false }
],
"live_scenes": [ { "name": "Lower Third - Pastor" } ],
"scene": "Lower Third - Pastor",
"include_audio": false
```

`scene` is still written for older versions and websocket clients.

Each dock saves `downstream_keyers_hold` and `downstream_keyers_off_at_scene_change` (prefixed with the view name for extra views and canvases).

**Settings from earlier versions load as follows:** the scene that was on air stays live and nothing is checked. Tie is ignored. Include Audio starts off, so existing keyers stop sending audio until it is checked. Keyers without a Transition now use the OBS scene transition instead of cutting; set Transition to **Cut** to keep cutting.

## Known limitations

- **Layered scenes on extra canvases** are sized to the main canvas. On a canvas with a different resolution, the layers may be positioned wrong.
- **Go to Preview, Go to Program and Transition Keyers** are only available for the main output, because Studio Mode only exists there.
- **Stinger transitions:** keyers change when the transition starts, not at the stinger's transition point. Untested.
- **The All Keyers row does not wrap,** so the dock cannot be narrower than its buttons.
