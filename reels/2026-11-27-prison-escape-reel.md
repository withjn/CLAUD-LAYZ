# layZ 3D reel: "How a Plastic Spoon Becomes a Prison Break" — mistakes, fixes and standing preferences

Made 2026-11-27 with Claude Code + the layZ MCP (3D Animation template, guided mode).
layZ run `35afd02e-e779-44fc-8246-aeea79810204`, editor project `f1088dd3-4cfe-4aec-9bb6-514ea05279d7`.
Final: 1080x1920, 30 fps, 56.4 s, Liam voice only, no captions, no SFX.

Read this first on any new machine before starting another layZ reel.

---

## 1. Standing preferences (apply by default next time)

| Topic | Preference |
|---|---|
| Format | 9:16, 1080x1920, 45–60 s |
| Voice | **Liam** (`TX3LPaxmHKxFdv7VOQHJ`, "Liam - Energetic, Social Media Creator"). There is a second "Liam" (Afrikaans narrator) — not that one. |
| Captions | **None.** No captions, subtitles or on-screen narration text. |
| Sound effects | Asked for quiet/popular SFX at first, then **removed them all** on hearing the plan. Default to **no SFX** unless asked; if asked, keep them very quiet and simple. |
| Script | **No trivia / "earned fact" lines** (e.g. no "In 1962, three Alcatraz prisoners…"). |
| Script | Don't start too many sentences with "But" — mix in "Except", "Trouble is,", "Then". Aim ~30%+ fewer "But"s than the template formula gives. |
| Camera | Lots of movement, but **nothing may block the view of the character** (walls, bars, cut faces). |
| Render fps | User chose **30 fps** for the final render (template default is 60). |
| Bitrate | Wants 15–18 Mbps. layZ export ("high") gave only **~13.4 Mbps** — see §5. |
| Workflow | Guided mode: show script and scene plan, wait for OK. User reviews the live 3D player and gives timestamped notes. |
| Humour | Crude gags are fine when the user asks (pixelated "goon photo", clothed fake-fap gesture), but **nothing explicit** — pixelate, keep clothes on. |

---

## 2. Script mistakes and fixes

1. **Added a trivia fact the user didn't want.** I followed the template's "one earned fact" habit and wrote an Alcatraz 1962 line. User: remove it, no random facts. → Never add fact lines unless asked.
2. **Too many sentences started with "But".** User asked for ~30% fewer. → Replaced 3 of 7 with "Except…", "Trouble is,…", "Then you hit…".
3. **User rewrote lines later** (fine, but plan for it):
   - "you have an escape" → "you might escape"
   - "and a rat who never talks" → "and your treasured goon photo"
   - "The tunnel comes out at a window three floors up." → "You're three floors up, so it's too high to just jump out, so you tie your bedsheets…"
   - "Congratulations, you have built a prison escape." → "Congratulations, you just escaped prison."
   Every script change means a **new voiceover take**, which shifts every word time → the 3D scene must be re-synced (see §4.1).

## 3. Voiceover mistakes and fixes

1. **False alarm: "Congratulations" said twice.** The word-timed transcript from `layz_inspect_audio` showed "Congratulations! Congratulations, …" on several takes, and I paid for **2 unnecessary regenerations** chasing it.
   **Lesson:** check the word timings — both copies together lasted ~0.72 s, the length of *one* spoken "Congratulations". It's a transcription artifact, not a TTS repeat. Ask the user to listen before regenerating.
2. Takes vary ±0.5 s in length even with identical text; always re-read word times from the take you keep.

## 4. 3D scene mistakes and fixes

### 4.1 Re-syncing after a new voiceover
- My scene hard-coded beat times everywhere. When the script changed, instead of rewriting every number I added a **time-warp table** at the top of `scene.js`: pairs of `[newVoiceTime, oldSceneTime]` at each key word, and `update(tv)` does `const t = warp(tv)` before anything else. Worked well — reuse this pattern from the start next time (author the scene on "beat time", map the voice onto it).

### 4.2 Performance
- **Scattering `pine_tree_01` / `fir_tree_01` (multi-variant photoreal models) made frames cost ~700 ms and ~5 billion triangles.** Fix: use rig trees (`innerscene-mature-scots-pine-tree`, `innerscene-mature-norway-spruce-tree`, `innerscene-mature-silver-birch-tree`) with `kit.scatter` → ~15 ms/frame.
- **Low-poly trees** (`innerscene-low-poly-nature-*`) looked cartoonish next to the realistic prisoner — don't mix styles.

### 4.3 Materials / textures
- `concrete_wall_003` / `concrete_block_wall` looked rusty orange; `brushed_concrete` turned dark brown. **`concrete` (tinted ~0xe4e2de) looked right** for interior; outer walls read as wood siding from far away (still a weak spot).
- **Canvas textures on `kit.box` showed as flat white** (mattress stripes) because my helper ran `kit.worldUV` on them, which breaks 0..1 UVs. Fix: build canvas-textured boxes with plain `E.Mesh(new RoundedBoxGeometry(...))`, no worldUV.
- Bright white props (pillow, dust pile) blew out under the lamp/sun — use mid-grey colours.

### 4.4 Camera
- **Camera yaw toward the right put the right-wall cut face in front of the prisoner** (poster shot). Positive yaw moves the camera to the subject's right; for a subject next to the right wall use **negative yaw**. User's rule: nothing obstructs the character.
- Top-down shots: a steep pitch (-60) from in front looked through the ceiling slab; -40 worked.
- Opening shot too wide (big blank facade) → tighter width.

### 4.5 Character animation (kit.person, model `ped-prisoner`)
- **Head clipped the wall while scraping** → kneel the root further from the wall (0.68 m instead of 0.95 m); the `Fixing_Kneeling` clip leans forward a lot.
- **Knee sank through the floor** while kneeling → call `kit.restOnFloor(H, { floor: FY, weight: 1 })` every kneeling frame, before `H.sync()`.
- **Head snapped back to stare at the ceiling (0:20)** — caused by a `kit.look` at a target *behind* him (the corridor flashlight). Never `look` at targets behind the character.
- **"He's sleeping in the chair"** — `Sit_Chair_Idle` on a stool read as dozing. User wanted him to **sleep in the bed**, then sit on the **foot of the bed** to make the dummy. Lying pose that worked: `LayToIdle` held at `speed: 0, offset: 0` (lying on back), root on the mattress, `kit.restOnFloor` with `floor = mattress top`; play `LayToIdle` forward to sit up. With root rot 0, the body extends ~0.55 m behind the root (head) and ~1 m in front (feet).
- **Dummy head stuck to his back when he turned** — props he holds must follow his hands/body each frame, not stay at a fixed point.
- **Dummy head had no face** → user wanted eyes, nose, mouth. Built 3D features (eye whites + pupils, brows, cone nose, torus smile); a texture drawn on a sphere didn't show where expected.
- **Searchlight hit him while hiding** → he'd be caught. Make the sweep pass a few metres away (miss him).
- **Jacket looked bad** three ways (tube shell, slab beside his arm, clipboard on forearm). In the end the user had him drop it before the photo gag. Clothing add-ons on library people are hard; avoid or keep them off-screen.
- Pixelated Polaroid: canvas with 8×10 big mosaic blocks, `NearestFilter`; turned toward himself later.

### 4.6 Dust pile placement
- Pile was behind the character from the camera's view → user asked to **move it into the foreground**. Place story props between the character and the camera.

## 5. Edit / export issues

- **layZ SFX library is thin**: no toilet flush, footsteps, scraping, cloth or fence rattle. I substituted (bubbles for flush, "Rock Slides" for scraping, "Hangar Door Slide" for metal). User then removed all SFX. If SFX matter next time, plan to bring/own sound files.
- **Bitrate**: `layz_export_video` only has quality low/medium/high; "high" = **~13.4 Mbps** (94 MB / 56.4 s), below the requested 15–18 Mbps. Options: re-encode locally with ffmpeg (not installed on the main Windows machine; `winget install --id Gyan.FFmpeg -e`) from the scene render master (~25 Mbps, 178 MB) muxed with the voiceover, e.g. `-c:v libx264 -b:v 16M -maxrate 18M -bufsize 32M -r 30`.
- Don't start an export and then change the timeline — I exported once with SFX right before the user said to remove them (one wasted export).

## 6. Process lessons

- Ask fewer, better intake questions; the user answers tersely and expects action.
- Preview with `layz_preview_scene` at a handful of times after every big edit, before showing the user — it caught most visual bugs early.
- Keep a list of every user note with its timestamp and tick them off; the user gives batched notes (8 fixes in one message).
- Use `layz_set_step_data` to save choices (voice, captions none, SFX none, notes) so the run remembers them.
