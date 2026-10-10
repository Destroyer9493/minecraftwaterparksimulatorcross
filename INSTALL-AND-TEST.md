# Install and test (no coding needed)

There are three pieces, and all three run on the same PC:

| Piece | File | Where it goes |
|---|---|---|
| Minecraft mod | `wpmc-0.0.1.jar` | your Minecraft `mods` folder |
| Waterpark mod | `WaterparkBridge.dll` | `...\WaterPark Simulator\BepInEx\plugins\` |
| Bridge (the "phone line" between the two games) | `wpmc-bridge.exe` | anywhere, e.g. your Desktop |

## 1. Minecraft side (one-time setup)
1. Install **Fabric Loader 0.19.5** for **Minecraft 26.3** using the official installer from https://fabricmc.net/use/. This adds a "fabric-loader-0.19.5-26.3" profile to the Minecraft Launcher.
2. Download **Fabric API** for 26.3 (version `0.162.0+26.3`) from Modrinth or CurseForge.
3. Open your mods folder. You use Prism Launcher: right-click the **WaterparkSim** instance, then **Folder**, then open
   `minecraft\mods` (that is `C:\Users\Kraft\AppData\Roaming\PrismLauncher\instances\WaterparkSim\minecraft\mods`).
   With the official launcher it would be `%appdata%\.minecraft\mods`.
4. Put **both** `wpmc-0.0.1.jar` and the Fabric API jar into that folder. **Milestone 6 changed the protocol (v2):
   replace all three files together** (jar, DLL and `wpmc-bridge.exe`). Mixed old/new pieces refuse to talk to each other.

## 2. Waterpark side (one-time setup)
BepInEx 6 is already installed (your log shows `BepInEx 6.0.0-be.755`). Copy `WaterparkBridge.dll` into:
`C:\Program Files (x86)\Steam\steamapps\common\WaterPark Simulator\BepInEx\plugins\`
You can put it in its own subfolder if you like (for example `plugins\WaterparkBridge\`).

This DLL is the **GameHooks** build. Once it is linked to Minecraft, it stops the Waterpark character walking on its own
and moves it to wherever your Minecraft player is.

## 3. Start everything, in this order
1. **Bridge:** double-click `wpmc-bridge.exe`. A black window should show `listening on 127.0.0.1:47560`. Leave it open.
   If Windows SmartScreen warns you, click "More info", then "Run anyway". It's an unsigned program built for you.
   If the Windows Firewall asks, you can click Cancel, because the bridge only talks to your own PC.
2. **Minecraft:** start the Fabric profile and create a new world for the bridge (keep reusing it afterwards):
   - **Game Mode: Survival**, **Difficulty: Peaceful**, **Allow Commands: On**.
   - **World type: Superflat**, then click **Customize** > **Presets** and pick **The Void**. In a void world,
     Minecraft's own ground can't get in the way: the only floor is the park's, so you can clearly see collision working.
     (A normal Superflat world also works, but its grass sits at height -60 while the park is at about 0 to 20, so
     anything that goes wrong is harder to spot.)
   - Once in the world, type `/gamerule fall_damage false` in chat, so a missed floor never hurts you.
   - Why Survival rather than Creative: Survival can't fly, so there's no way to accidentally double-tap Space into
     flying. Creative also works if you never double-tap Space (press it twice quickly and you're flying again; double-tap to stop).
   - Press **F3+P** once, so Minecraft keeps running when you click into another window. The chat should show "Pause on lost focus: disabled".
   - The bridge window should now show `minecraft: connected ...`.
3. **Waterpark Simulator:** start it and load your park. The bridge window should show `waterpark: connected ...`.
   Once the park has loaded (not in the main menu), Minecraft teleports your player to where your Waterpark character stands,
   and the park's floors, walls and slides around you (about 40 m) are sent to Minecraft first.

Order doesn't strictly matter, because both mods retry the connection every couple of seconds. The order above is just the easiest to follow.

## 4. What to expect (milestone 4: collision, 4b: no more loading stutter)
* **New in 4b:** loading collision no longer freezes the game. Waterpark now spreads the work over many frames
  (about 1.5 ms per frame, adjustable, see section 7), remembers objects it has already converted, and only re-sends what
  changed, nearest first. Minecraft builds the shapes on a background thread. A big area (spawn, or after a respawn)
  can take about a second to fill in. Teleports wait for the floor at the destination (up to 3 s) before moving you.
* Play from the **Waterpark window**. WASD, Space, Shift, Ctrl, the number keys and mouse clicks are sent to Minecraft, which does the walking and jumping.
  Your Waterpark character and camera follow the Minecraft player. Mouse-look stays with Waterpark.
* **No flying needed any more.** Minecraft now has the park's real collision shapes: you stand on the paths and pool decks,
  walk up stairs and ramps, jump onto low things, and walls and buildings block you. Nothing of the park is *drawn* in
  Minecraft (it looks like you are standing in mid-air there); the Waterpark window is where you see it.
* Things to try, in order:
  1. Stand still: you should not sink or fall. Walk around on flat paths.
  2. Walk into a wall or building: you stop, you don't pass through.
  3. Walk up a staircase or a gentle ramp: you go up smoothly (Minecraft climbs steps up to about 0.6 m by itself).
     Steep slopes feel a little "steppy" (the shapes are made of 1/8-block pieces).
  4. Walk 50+ m in one direction: new geometry streams in as you move (every ~6 m or 5 s).
  5. Ride a slide: while you slide (or get knocked over/ragdolled), Waterpark takes over your character and Minecraft
     just follows it. When the ride ends, Minecraft is put back where your character got up and you walk normally again.
* If you ever fall out of the park, Waterpark's own "fell out of the world" respawn now also moves your Minecraft player
  back, instead of the endless respawn loop seen in the first test.
* Build mode still works as normal. While it's open, your keys stay with Waterpark.
* Known rough edges: water surfaces aren't solid (you walk on the pool bottom), moving objects and guests aren't
  collision, and slides are only "followed" (untested in the real game). If anything looks wrong, close Waterpark,
  remove the DLL (see section 6) and send both logs.

## 4c. What to expect (milestone 6: Minecraft's HUD, hand and body in Waterpark)
* **Minecraft's HUD is drawn over the Waterpark window** the whole time you are linked: hotbar, hearts, hunger, XP bar,
  chat, action-bar messages, and the crosshair. Minecraft renders it to an invisible image at Waterpark's screen size
  (30 times a second by default) and hands it over through shared memory; Waterpark shows it on top of everything.
  Minecraft does **not** draw its world any more while linked (only the HUD and the hand), and its window is hidden
  (it comes back by itself when Waterpark disconnects or closes).
* **Keys 1-9 always pick a Minecraft hotbar slot**, in normal play and in Minecraft UI mode. Waterpark's own uses of
  1-9 are switched off while linked (the log lists which ones, see section 5) and come back when you unlink.
  The mouse wheel stays with Waterpark until you turn on Minecraft mode (below).
* **Right Alt = Minecraft mode (changed in 6b).** Nothing opens and the mouse stays locked: you keep looking around
  and walking as normal. Only two things change while it is on:
  * the **mouse wheel scrolls Minecraft's hotbar** (Waterpark's own wheel item switching is blocked);
  * **E opens Minecraft's inventory** (Waterpark's own E action is blocked so both don't fire). While the inventory is
    open the cursor appears and mouse/keys go to Minecraft (move items, crafting); Waterpark ignores them. Close it
    with E or Esc: the cursor locks again and you are back in Minecraft mode (wheel still on the hotbar).

  * **T opens Minecraft's chat, / opens it for a command** (e.g. `/gamemode spectator`). The cursor appears and
    everything you type goes to Minecraft (Enter sends, Esc cancels, Tab completes commands, arrow keys work).
    When the chat closes you are back in Minecraft mode with the mouse locked. The T or / you pressed is not typed
    into the chat. On keyboards where / needs Shift (e.g. German Shift+7), typing / works too, but the 7 also
    selects hotbar slot 7.
  * **F3 combos go to Minecraft:** hold F3 and press N (spectator toggle), F4 (game mode switcher), B, H, G, ...
  * Waterpark's own actions on E, T, / and F3 are switched off while Minecraft mode is on (the log lists them).
  * **Waterpark's own item hotbar is hidden** so only Minecraft's hotbar shows (setting `HideWaterparkHotbar`).

  Press Right Alt again to give the wheel and E back to Waterpark (and show its hotbar again; also on disconnect). A short hint appears in the middle-lower part of the
  screen for 3 seconds whenever the mode changes, and the log says `wpmc: Minecraft mode ON` / `OFF`.
* **When Minecraft opens a screen by itself** (chat with T, a chest you right-clicked, the death screen) the cursor
  appears and input goes to Minecraft automatically, in or out of Minecraft mode; it ends when that screen closes.
* **First person:** Waterpark's own arms are hidden; you see Minecraft's hand and the item you hold (it swings when
  you click). **Third person** (the game's camera switch): Waterpark's character is hidden and a blocky Minecraft player
  with **your Minecraft skin** (slim or wide arms, hat/jacket layers) stands where you are, walking, turning its head,
  swinging its arm and crouching like in Minecraft.
* **Placing blocks on the park (new):** Minecraft's crosshair now "sees" the park's collision. Look at a floor, wall,
  slide or stair in Waterpark and right-click with a block in hand: Minecraft places it against that face. The block is
  real and solid (you can stand on it), and collision refreshes never remove it. Limits:
  * Minecraft blocks sit on Minecraft's 1 m grid, so on a curved slide or a floor that isn't at a whole-metre height
    the block snaps to the nearest grid cell: it can sink into or float above the surface by up to half a block.
  * **You can't see the placed blocks in Waterpark yet** (Waterpark doesn't draw Minecraft blocks). They are there in
    Minecraft and you collide with them. There is also no block-outline highlight.
  * Water and invisible trigger zones are never targets. If a mob/item is closer than the park surface, that wins.
* Things to try, in order:
  1. Look at the bottom of the screen: the Minecraft hotbar and hearts should be there, crisp, with Waterpark visible around them.
  2. Press 1 to 9: the selected slot moves. Waterpark should not react to those keys.
  3. Press Right Alt: a hint "Minecraft mode ON" appears, no cursor. Scroll the wheel: the Minecraft hotbar slot moves
     and Waterpark's held item does not change; Waterpark's own hotbar is gone. Press E: Minecraft's inventory + cursor; drag an item; press E: cursor
     gone, you can look around, the wheel still scrolls the hotbar. Right Alt again: hint "OFF", wheel and E are Waterpark's.
  4b. In Minecraft mode press / and type `gamemode creative`, Enter: you're in creative, the chat closes, mouse locked.
     Press T, type a message, Enter. Hold F3 and press N: spectator; again: back.
  4. Press T in normal play: Minecraft chat opens (UI mode turns on), type a message, press Enter.
  5. Switch to third person: you should see yourself as a Minecraft player with your skin. Walk, crouch, punch.
  6. Hold a block, aim at a park floor, right-click: walk into the spot afterwards, you should bump into it.

## 4d. What to expect (milestone 7: Minecraft blocks and entities in Waterpark)
  1. Link everything as usual. The Waterpark log shows `wpmc: asked Minecraft for its world: blocks within 48 m, entities within 32 m (max 64)`
     and, a moment later, `wpmc: Minecraft block materials ready (shader 'Universal Render Pipeline/Lit', ...)`.
  2. In Minecraft mode, place a few blocks in front of you (stone, glass, a slab, stairs, leaves, a water bucket).
     Each appears in Waterpark within about a second, textured, in the same spot as in Minecraft.
  3. Break one: it disappears. Walk away more than about 60 m: far blocks disappear; come back: they return.
  4. Spawn a pig (spawn egg) and drop an item (Q): a pink box walks around, the item spins and bobs.
  5. If a friend joins your Minecraft world, they appear with their skin.
  6. Every 10 s the log has `wpmc: Minecraft blocks (10 s): ...` and `wpmc: Minecraft entities (10 s): ...` lines with
     counts and timings. If the game stutters while blocks load, send those lines.
  7. Turn it off in the config (`[MinecraftWorld] ShowMinecraftBlocks` / `ShowMinecraftEntities` = false).

## Changelog
* **Slides, Minecraft water, guests around blocks, missing ground (2026-10-10, `WaterparkBridge.dll` + `wpmc-0.0.1.jar` + `wpmc-bridge.exe`, replace all three; they only work together):**
  - *Slides:* while you ride a slide (or are ragdolled) the Minecraft body copies Waterpark's ragdoll (lying back, feet first,
    head first, tumbling), and Minecraft follows smoothly 30 times a second (new message `PLAYER_FOLLOW`): no pull-backs,
    no fall damage, the park collision is paused for you until you get off. The camera is the game's own slide camera.
    Not done: swimming with Minecraft's swim animation in the splash pool (park pools aren't Minecraft water).
  - *Minecraft water and the park:* water you pour in Minecraft no longer flows through park walls or floors; it spreads on
    park floors and pours off their edges (Minecraft's own flow rules, with the park collision as walls).
  - *Guests walk around Minecraft blocks* standing on the park (fences, walls, glass, stone...; not plants, torches, rails,
    carpets). Ragdolled guests bump into them. Your own Waterpark player is not blocked by them (Minecraft already is).
    Off switch: `[MinecraftWorld] GuestsAvoidBlocks` (section 7). Log line every 10 s: `wpmc: guests avoid Minecraft blocks: N obstacle boxes`.
  - *Spots with no ground:* the collision is sent in 16 m cubes of at most 26213 triangles. A cube over that limit was cut
    short, and the park terrain was always the part that got cut, so a 16 m patch of ground could go missing. Ground and
    terrain now always go first; if something must be left out, the log says what (`collision cube at Unity x .. has more
    than 26213 triangles: kept .., left out ..`). **F8** also prints a `wpmc: ground under (...)` section (what is below
    your feet, terrain holes, whether that cube was sent). Already in the 08:15 hotfix DLL.
  - *Guests react to arrows, hits and TNT again* (same fix as the 08:20 hotfix below), plus Minecraft log lines for
    every step: `wpmc: Waterpark guests (10 s): N in the last GUESTS, N hittable stands (.. spawned, .. failed), .. hurts,
    .. hits sent; nearest guest at (...)`, `wpmc: guest N hit (projectile ...) -> GUEST_HIT`, `wpmc: explosion sent to
    Waterpark ...`, and a warning if the world isn't singleplayer/LAN-host (guests can only be hit there).
  - *Includes the 08:15 hotfix:* Minecraft blocks, beds and entities render with depth again (they were drawn over the
    park and in the wrong order with water) and cast shadows again.
* **Guest hotfix (2026-10-10 08:20, `WaterparkBridge.dll` only; includes the 08:15 hotfix):** arrows, hits and TNT
  didn't knock guests over. Waterpark told Minecraft where the guests were using the position of the guest's AI object, which
  doesn't move with the walking guest, so Minecraft's invisible stand-ins stood in the wrong places and nothing ever hit them
  (the log showed `0 hits from Minecraft`, and `14 near you now` never changed while you walked across the park).
  Guests are now placed at their navigation/pelvis position. New log lines: `wpmc: guest position check #1..3` (where a
  guest's parts are and which one was used), `wpmc: guest hit from Minecraft: '<guest>' kind ...` for every hit, and a
  one-time check that the ragdoll really started (`... knocked the guest over`, or a direct `BaseRagdoll.EnableRagdoll`
  if the network call did nothing).
* **Rendering hotfix (2026-10-10 08:15, `WaterparkBridge.dll` only):** the 23:25 build copied a game particle material
  (cherry petals) for Minecraft blocks, which made them ignore depth (drawn over the park, beds behind blocks, water in the
  wrong order) and stopped their shadows. Blocks now use the plain URP Simple Lit shader with solid/cut-out depth settings;
  the `Minecraft block materials ready` log line shows `depth: blocks queue 2450 ZWrite 1 ZTest 4 shadow pass True`.
* **See-through blocks fixed + TNT (2026-10-09 23:25, `WaterparkBridge.dll` + `wpmc-0.0.1.jar` + `wpmc-bridge.exe`, replace all three):**
  - *Plants, torches, rails, fences, panes, doors, ladders, vines, leaves* draw with their real Minecraft shapes and with the
    see-through parts actually see-through (no black/coloured squares around a flower, no solid box for a pane). Glass,
    stained glass, ice, slime and honey are see-through with their tint; blocks behind them are drawn in the right order.
  - *Why it went wrong before:* the game's shader may not include the "alpha clip" variant the mod relied on, so cut-out
    pixels came out solid. The mod now tests the shader once when the world loads (log line
    `wpmc: Minecraft block materials ready ... see-through texels: alpha clip / cut out as geometry ...`). If the shader can't
    do it, see-through pixels are cut out as real geometry instead, which works with any shader. Force either way with
    `[MinecraftWorld] CutoutMode` (section 7).
  - *Water and waterlogged blocks:* the water surface shows from below too, a waterlogged slab/stair shows its water, and water
    no longer hides the faces of glass or slabs next to it.
  - *TNT works.* Lit TNT appears in the park as the TNT block, flashing white and swelling before it goes off; TNT minecarts show
    the TNT inside the cart. It falls and lands on the park floor like any other entity. The explosion shows Minecraft's
    explosion particles plus a short bright flash in the park, and you hear it from Minecraft. Waterpark guests nearby are
    knocked over (ragdoll), harder the closer they were. You get knocked back and stop at park walls. The park is never damaged
    (explosions can only break Minecraft blocks). Log: `wpmc: Minecraft explosion at (...), radius 4.0, ...`.
  - Limitations: plants don't get Minecraft's small random position offset; no camera shake; entity textures (mobs) keep the
    shader's alpha clip and don't get the geometry fallback.
* **Real mob models, block entities, park collision for mobs/arrows, hittable guests, jump height, hand lighting, particles, sneak/sprint (2026-10-09 22:51, `WaterparkBridge.dll` + `wpmc-0.0.1.jar` + `wpmc-bridge.exe`, replace all three):**
  - *Mobs look like Minecraft.* Pigs, cows, zombies, creepers, every mob and other entity now shows its real Minecraft model
    with its texture (from your resource pack), animated: legs walk, zombie arms are raised, heads turn, babies are small,
    hurt mobs flash red. Anything that can't be exported still shows the old box. Other players still use the player model
    with their skin as before. Not exported yet: extra layers (sheep wool, armour, saddles, held items on mobs).
  - *Chests, shulker boxes and other block entities show.* Chests (single, double, trapped, ender), shulker boxes (all colours),
    banners, bells, heads etc. appear with their Minecraft model; chest and shulker lids open and close with the real
    animation. Signs and decorated pots are normal blocks in this Minecraft version and already showed (sign text isn't
    drawn). Block entities without a model (e.g. end gateway) show as a cube. Banner patterns and player-head skins are
    plain colours for now.
  - *Mobs, items and arrows collide with the park.* On your own (singleplayer/LAN-host) world, mobs and dropped items stand
    on the park's floors, arrows stick in park walls and stay there, snowballs break on them. (Mob pathfinding still
    doesn't "see" park floors, so mobs don't deliberately walk on them.)
  - *Arrows and hits knock guests over.* Waterpark guests within 32 m become invisible stand-ins in your Minecraft world: an
    arrow, a snowball or your punch on one makes the real guest ragdoll in Waterpark (pushed the way the hit went), and
    visitors get the "injured" thought. Strength: `GuestHitForce`. Singleplayer/LAN-host worlds only.
  - *Same jump height as Waterpark.* While linked, your Minecraft jump reaches the height of Waterpark's own jump
    (read from the game's player controller, sent once on link and when it changes); unlinked it's vanilla again.
  - *Hand lighting.* Minecraft's first-person hand and held item were lit like an inventory icon. They're now lit by
    Waterpark's sun: the side facing the sun is bright, the other side darker, and the hand goes flat when the sun is
    off. (Day/night time sync comes later; the hand's colour doesn't follow a sunset tint yet.)
  - *Sneak and sprint.* Minecraft's own sneak / sprint keys (Left Shift / Left Ctrl, or whatever you rebound them to in
    Minecraft) work while Minecraft drives your character; double-tap W to sprint should work too (vanilla behaviour, not tested separately). Waterpark's own bindings on Left
    Shift / Left Ctrl are switched off meanwhile (logged by name; back on when unlinked, in build mode and in Minecraft
    mode). Your Minecraft avatar crouches and the Waterpark character moves at sprint speed. Sneaking stops you at the edge of
    park floors like at a block edge (before, sneaking on a park floor froze you in place, because Minecraft's edge check
    only looked at blocks). Park collision is in 1/8 block steps, so the edge can be up to 1/8 block further out.
  - *Particles.* Minecraft's particles near you (smoke, flames, crits, dust, splashes...) are drawn in the park as
    camera-facing sprites with your resource pack's textures; park walls hide them. Up to `MaxParticles` (1000).
  - Unchanged: the 22:03 ground fix, terrain holes, blocks/entities streaming, hotbar hiding, chat and F3.
* **Main park ground fixed (2026-10-09 22:03, `WaterparkBridge.dll` only):**
  - *What was wrong.* The 13:12 build took the pool holes from the game's terrain data (`TerrainData.IsHole`). In your park
    that data says "hole" for **every** spot of the terrain, although the ground is solid (your Waterpark character stands on it).
    So the mod sent Minecraft no terrain at all, and also cut the big flat ground plates ('BuildingPlane', `LVLPlug1-7`) near
    ground level everywhere. Your log shows it: exactly one "terrain-hole area" per 16 m terrain tile (24 tiles -> 24 areas)
    and "terrain hole cells" equal to all terrain cells. In Minecraft you were walking on whatever was left underneath
    (e.g. the `BackgoundFloor` at y 4.9, about half a metre below the real ground at 5.5), or falling where nothing was left.
  - *Fix.* The mod now checks the terrain data against the terrain's real collider (576 test rays per terrain). If they
    disagree, as in your park, it ignores the data and finds the real holes (pools) with rays against the collider: about
    300 rays per 16 m tile, plus extra rays along hole edges, redone only when the game re-bakes its holes or every 2 minutes.
    The log says which way it went: `wpmc: terrain '...' holes: TerrainData.IsHole says hole at N of 576 test points, the
    TerrainCollider has no surface at M ...`. The scan line now also shows `(N hole raycasts)`.
  - Unchanged: Minecraft blocks/entities (milestone 7), hotbar only in Minecraft mode, chat, F3. Jar and bridge exe stay as they are (13:22).
* **Minecraft blocks and mobs inside Waterpark - milestone 7 (2026-10-09 13:25, `WaterparkBridge.dll` + `wpmc-0.0.1.jar` + `wpmc-bridge.exe`, replace all three):**
  - Blocks you place in Minecraft (and any other Minecraft blocks near you) now appear inside Waterpark with Minecraft's
    own textures from your resource pack: full blocks, slabs, stairs, glass, leaves, water. Faces hidden between two
    blocks are left out. They're lit by Waterpark's sun like everything else. "Near" means within `BlockRadius` (48 m)
    of your Minecraft player, counted per 16x16x16 chunk section. They're visual only: Waterpark guests and objects
    don't collide with them (you do, in Minecraft).
  - Minecraft entities within 32 m (the 64 nearest) appear too: other players with their skins, dropped items (small
    spinning blocks or flat item pictures), and mobs as simple boxes in the mob's colour with a lighter head (not real
    mob models yet). They move smoothly. You never see yourself twice.
  - The bridge keeps a copy of the world, so restarting Waterpark redraws everything straight away.
  - **The new bridge exe is required:** the 11:48 bridge does not know the new messages.
  - Unchanged: pool holes, Minecraft hotbar only in Minecraft mode, chat, F3, everything from 13:10.
* **Pools follow the game's own holes + Minecraft hotbar only in Minecraft mode (2026-10-09 13:10, `WaterparkBridge.dll` + `wpmc-0.0.1.jar`):**
  - *Pools.* The 13:00 cut used a box around each pool's water and also took pool floors away, and its edges were
    off. It is now off (`CutGroundOverWater = false`). Instead the mod copies what the game itself does: Waterpark Simulator
    digs a **hole in its terrain** under each pool (`BuildingSystem.UpdateTerrainHoles` / `CreateTerrainHoles`, from the
    pools' `TerrainMask` objects), and its terrain collision has no surface there. The terrain is now copied on its own
    height grid, so the hole edges are exact (before this, sampling every 1 m blurred the edges by up to 1 m). The flat 'BuildingPlane'
    ground plates (layer 28, the park-level floors) get the same hole, but only near ground level, so a pool floor
    one metre or more down is never touched. When you build, move or remove a pool, the collision follows within a few seconds.
    In the log: `N terrain-hole areas in range (the game's pool holes)`, and in the scan line `cut N ground tris over terrain holes, terrain hole cells N`.
  - *Hotbar.* With Minecraft mode OFF (Waterpark mode), the Minecraft overlay no longer shows the hotbar, hearts, hunger,
    armour, XP bar or held item name. They come back when you press Right Alt. Chat, the crosshair, action-bar messages and the hand stay.
    Setting: `ShowMinecraftHotbarInWaterparkMode`. With an older jar the hotbar just stays visible.
* **Ground over pools (2026-10-09 13:00, `WaterparkBridge.dll` only):** the park ground (terrain and the ground-only
  layers 'Draggable'/'BuildingPlane' that 12:36 made solid again) went on over the pools, so Minecraft stood on
  it while Waterpark's player dropped into the water. Now the game's pool water volumes (pool water colliders, also
  when they are triggers, the pool water surfaces, and anything on a water layer) are cut out of terrain and
  ground-only surfaces, from 0.5 m below the water up to 2 m above it. Pool floors/walls and normal floors are not
  touched; oceans and lakes over 5000 m² are ignored so the sea floor stays solid. Log: `wpmc: N water volumes in
  range ...`, `game water objects: N PoolInteraction (M pool water volumes)`, and `cut over water N tris` in the scan line.
* **Chat and commands (2026-10-09 12:44, only `WaterparkBridge.dll` changed):** in Minecraft mode T = chat,
  / = command, F3 + key = Minecraft debug combos. Log lines: `wpmc: Minecraft UI mode ON (T pressed: Minecraft chat)` /
  `(/ pressed: Minecraft command)`, and `wpmc: Minecraft mode: E = inventory, T = chat, / = command, F3 = debug keys go
  to Minecraft; disabled N game bindings on E/T/slash/F3: ...`.
* **Milestone 6b** (2026-10-09, only `WaterparkBridge.dll` changed):
  * **Right Alt = Minecraft mode** (wheel = Minecraft hotbar, E = Minecraft inventory, Waterpark's hotbar hidden, mouse
    stays locked); see 4c. If the log says `InventoryUI not found; hotbar-like UI objects: ...`, send that line.
  * **Fix: invisible walls near trigger areas.** Your log showed `triggers skipped 0` on every scan: triggers were in
    fact never exported (the scan ignores them), so they were not the cause. The invisible walls came from objects whose
    mesh the game doesn't let mods read (157-178 per scan in your log). For those the mod used to shoot rays straight
    down and built "top surface + walls down to the bottom of the object's box", which closed every opening underneath:
    under umbrellas, canopies, arches, gates, entrance roofs - exactly where entrance/shop trigger zones usually sit.
    Now those objects are sampled with rays from above, below and from the sides, so openings stay open.
  * Collision now only exports colliders the **player actually collides with**: the game's layer collision settings for
    the player's own collider, per-collider include/exclude layers and ignored collider pairs. Before, every layer in the
    game's ground/building masks was solid, even layers the player walks through. The log lists those layers once.
  * Scan line: `triggers in range N` (counted on the 1st and every 10th scan, never exported), `not touching the
    player N`, `floating props N`.
  * **Fix: water false positives.** The name pattern `ocean` matched beach decorations (OceanFence, OceanBench,
    OceanWhale...) and made them non-solid. Now only exact names like `Ocean`/`Sea`/`Water` or specific ones like
    `OceanWater` match. An old config with the old default is updated automatically (the log says so).
  * **Floating props are not exported:** beach balls, liferings, water balloons and anything else with a `WaterChecker`
    (they move and float), like other physics-driven objects. They were wrongly counted as water before.
  * New debug options `LogLargestColliders` and `DumpCollidersKey` (F8), see section 7.
  * **Hotfix 12:27:** the 12:19 build's player-collision filter rejected every object in the park (`not touching the
    player 1066`, collision gone). It now checks itself before each scan: if it would drop more than half, it's switched
    off for that scan and the log shows `wpmc: player-collision filter would drop N of M colliders - not trusted ...`
    with the reasons and an example object. **Please send that line.** Setting `MatchPlayerCollision = false` turns it off.
  * **Hotfix 12:36 ("most floors I fall straight through"):** the 12:27 log showed the cause of the filter problem:
    the game answers "ignored pair" for every object when asked about the player's CharacterController, so that check is
    gone and the filter works again. Floors: 6b had stopped treating the game's `GroundLayers`/`BuildingLayerMask` layers
    ('Draggable', 'BuildingPlane') as solid because the player's collider doesn't physically hit them, but the game uses
    them as ground. They are solid again, like before 6b. Also, objects the mod can't read now always keep a walkable top
    (narrow planks, edges, steps), and slides are no longer processed twice.
* **Milestone 6** (2026-10-09): Minecraft HUD/hand overlay, Right Alt UI mode, Minecraft player model with your skin,
  keys 1-9 always for the Minecraft hotbar, placing Minecraft blocks on park surfaces. Protocol v2.
* **Fix: "Is Trigger" colliders were solid in Minecraft.** Invisible trigger zones (entrances, slide sensors, shop
  areas...) became walls/floors. Now every collider's `isTrigger`, `enabled` and active state is checked on **every**
  scan (also slides, terrain and cached objects), so a collider that becomes a trigger or gets disabled later is
  removed from Minecraft at the next scan. The scan line says `triggers skipped N`.
* **Fix: water and pools were solid.** Water colliders are not always triggers. The mod now recognises water by layer
  name (any layer with "water" in its name, setting `NonSolidLayers`), by the game's water components (pool and
  water-surface/water-checker objects) and by name patterns (`NonSolidNamePatterns`), and sends them to Minecraft as
  **non-solid** water. You walk on the pool floor, not on the water surface. The scan line says
  `water skipped N (sent non-solid)`, and the log lists the water layers/patterns once per session.

## 5. Checking the Waterpark log
Open `...\WaterPark Simulator\BepInEx\LogOutput.log` in Notepad and use Ctrl+F to search for **`wpmc`**. A good run looks like this (numbers will differ):
```
[Info   :Waterpark x Minecraft bridge] wpmc plugin loaded; bridge port 47560
[Info   :Waterpark x Minecraft bridge] wpmc: connected to bridge on port 47560
[Info   :Waterpark x Minecraft bridge] wpmc: linked; collision + spawn follow once the player is in the park
[Info   :Waterpark x Minecraft bridge] wpmc: collision reset, epoch 1
[Info   :Waterpark x Minecraft bridge] wpmc: scene changed to 'GameplayScene_Beach'; resending collision and spawn
[Info   :Waterpark x Minecraft bridge] wpmc: collision layer mask 0x...
[Info   :Waterpark x Minecraft bridge] wpmc: collision scan #1 at (...): 900 ms wall over 400 frames, main thread 560.0 ms total / max 1.70 ms per frame (...); worker 250.0 ms. Colliders 1150 (box 400, ...), reused 0, rebuilt 1150 (...). In range: 45 cubes / 190000 tris; queued 45 changed (7400 KB) + 0 removed; ...
[Info   :Waterpark x Minecraft bridge] wpmc: spawn: teleport #1 Minecraft to (35.20, 5.60, -51.70) yaw 90 (collision ready after 900 ms)
[Info   :Waterpark x Minecraft bridge] wpmc: Minecraft acked teleport #1
[Info   :Waterpark x Minecraft bridge] wpmc: PlayerMovementController.Move prefix active (game movement skipped, Minecraft drives the player)
[Info   :Waterpark x Minecraft bridge] wpmc: PlayerMovementController.JumpAndGravity prefix active (game gravity skipped)
```
Milestone 6 adds (numbers will differ):
```
wpmc: Minecraft UI mode blocks 30 game input handlers while active
wpmc: water / non-solid rules: layers [...] patterns [...]
wpmc: game water objects: ... PoolInteraction, ... WaterSurface, ... WaterChecker, ...
wpmc: collision scan #3 ...; water skipped 12 (sent non-solid); triggers skipped 85
wpmc: overlay shared memory 'Local\wpmc_frames_v1_1920x1080' created (23 MB, 3 slots)
wpmc: overlay config sent: 1920x1080 @ 30 fps, flags 0xF (...)
wpmc: Minecraft overlay status: shared memory open, ...
wpmc: keys 1-9 belong to Minecraft: disabled N game bindings: ...
wpmc: hid Waterpark's first-person arms ...
wpmc: Minecraft skin received: 64x64, slim arms
wpmc: Minecraft player model built (slim arms, shader '...')
wpmc: overlay shown 300 new frames in 10 s (290 partial uploads), upload avg 0.40 ms, max ... ms
wpmc: Minecraft mode ON (RightAlt pressed): mouse wheel scrolls Minecraft's hotbar, E opens Minecraft's inventory
wpmc: Minecraft mode: E opens Minecraft's inventory; disabled N game bindings on E: ...
wpmc: Waterpark's hotbar hidden: InventoryUI.InventoryRoot '...' (CanvasGroup added)
wpmc: Minecraft UI mode ON (Minecraft opened InventoryScreen); Waterpark input suspended, cursor free
wpmc: Minecraft UI mode OFF (Minecraft closed its screen); Waterpark input back
wpmc: Waterpark's hotbar shown again (RightAlt pressed): InventoryUI.InventoryRoot '...'
wpmc: Minecraft mode OFF (RightAlt pressed): wheel and E back to Waterpark
```
Milestone 6b adds (please send these back):
```
wpmc: collision layers (player layer N '...', from its CharacterController): solid [...]; NOT solid any more although in GroundLayers/BuildingLayerMask: [...]
wpmc: collision scan #1 ...; water skipped N (sent non-solid); triggers in range N (counted on scan #1, never exported); not touching the player N; floating props N; ignored N), ...
wpmc: collision scan #1: 15 biggest solid colliders exported to Minecraft (of N):
    1. '.../Parent/Object' [MeshCollider, Mesh non-readable] layer N '...', size 12.0 x 4.0 x 8.0 m at (x, y, z)
```
**If you still hit an invisible wall:** stand right at it and press **F8**. The log gets a
`wpmc: colliders within 3 m of the player ...` list with every object there and whether it was exported as solid,
trigger, water, or skipped. Send that block with the log.
**If you fall through the ground somewhere:** stand next to the spot (or on it in Waterpark) and press **F8**. The
`wpmc: ground under (...)` section lists what is below you (every hit straight down, with the mod's verdict), whether the
terrain has a hole there, and whether that collision cube was sent to Minecraft or was over the triangle limit.
The `overlay shown ... upload avg ... ms` line (every 10 s) is the cost on Waterpark's side: the time per frame to copy
Minecraft's image into the game's texture. Please send it back. Under 1 ms is fine; several ms means try a lower `OverlayFps`.

When you ride a slide you should also see `wpmc: player is sliding; Minecraft follows the game player` and afterwards
`wpmc: slide/ragdoll ended: teleport #...`. During the ride the Minecraft body lies on the slide like Waterpark's ragdoll. A Waterpark respawn shows `wpmc: game teleport: teleport #...`.

Useful to send back:
* The `collision scan #...` lines. Since milestone 4b each scan is spread over many frames. The important numbers:
  * `max ... ms per frame`: the most a scan took from any single frame. It should stay near `FrameBudgetMs` (1.5 ms by default).
    Before 4b a scan froze the game for about 1 second (900-1250 ms) in one go.
  * `reused` vs `rebuilt`: after the first scan, almost everything should be `reused` while you walk around.
  * `queued N changed`: how many 16 m cubes actually had to be re-sent (normally only new ones at the edge).
* Any `cubes over the 26213 cap` note (too much geometry in one place).
* With `VerboseLog = true` (see section 7): `MeshCollider '...' uses non-readable mesh` lines (those objects get a simpler
  top-surface-plus-walls shape) and `copied mesh ...` lines for slow mesh copies.

Bad signs to report:
* `Error loading [Waterpark x Minecraft bridge` or a Harmony / `HarmonyException` error means a game hook failed (for example, after a game update).
* `wpmc: no ack for teleport` repeating means Minecraft isn't answering (is the world open and F3+P set?).
* Many `[Player] Player fell out of the world` lines in a row means the floor isn't reaching Minecraft; send both logs.
* `wpmc: Input System read failed` or `legacy UnityEngine.Input unavailable` means your keys aren't reaching Minecraft.
* `Built without WPMC_GAME_HOOKS` means you have the wrong DLL (the test build without hooks).
* No Minecraft HUD visible: look for `overlay` lines in both logs. `Minecraft overlay status: shared memory NOT open`
  means Minecraft couldn't attach to Waterpark's shared memory: send both logs.
* `building the Minecraft player model failed` or `no usable shader` means the body stays hidden; send the log.
* The cursor doesn't appear in the inventory, or Waterpark still moves while it is open: send the log (look for `UI mode ON`).
* Waterpark's item switches or its E action fires in Minecraft mode: send the log (look for `Minecraft mode` and `OnSwitchItem`).

Minecraft writes its own lines starting with `wpmc:` to `%appdata%\.minecraft\logs\latest.log`. **Send the `latest.log` of the
Fabric 26.3 profile** you played with, straight after closing Minecraft. The Minecraft log from test 2 was from a different
profile (vanilla 1.20.4, no mods), so it showed nothing about this mod. If you use a launcher with separate instances
(Prism, MultiMC, CurseForge...), the log is inside that instance's folder: `...\instances\<name>\.minecraft\logs\latest.log`
(Prism: right-click the instance, then **Folder**). For milestone 6 look for `wpmc: overlay config ...`, `wpmc: overlay shared memory ... open`, `wpmc: skin sent to
Waterpark ...` and the performance line printed every 10 s:
```
wpmc: overlay 1920x1080: 300 captured / 200 published in 10 s; render thread 0.54 ms/frame, worker 1.39 ms/frame, 590 KB per published frame
```
`render thread` is what the overlay costs Minecraft's own frame; `worker` runs on a separate thread. For collision, look for:
```
[Render thread/INFO] (wpmc) wpmc: collision cleared, epoch 2
[Render thread/INFO] (wpmc) wpmc: collision 1s: 12 chunks (60000 tris) built in 40.0 ms on the worker thread, 1 publishes (3.1 ms), max receive->usable 45 ms; now 190000 tris in 45 cubes, 9000 blocks (epoch 2); queries 120 (avg 3.0 us, max 0.02 ms)
```
That summary line appears at most once per second, and only when collision changed. The shapes are built on a separate
"wpmc-collision" thread, so `built in ... ms` is NOT time the game froze. `max ... ms` at the end is the slowest
collision lookup the game itself did in that second, and should stay far below 1 ms. Also report any
`Can't keep up!` lines and any `wpmc: collision still processing after 2 s; teleporting anyway` lines.
If you get kicked with "Flying is not enabled on this server", tell me: that check is supposed to count the park as ground.

## 6. Turning it off
* Waterpark: delete or move `WaterparkBridge.dll` out of `BepInEx\plugins`. The game then runs exactly as before. Removing BepInEx isn't needed.
* Minecraft: remove `wpmc-0.0.1.jar` from the mods folder, or just play with a non-Fabric profile.
* Bridge: close its window.

## 7. Tuning without rebuilding (Waterpark config file)
After the first start with the mod, BepInEx creates `...\WaterPark Simulator\BepInEx\config\dev.wpmc.waterpark.cfg`.
Open it in Notepad **with the game closed**, change a value, save, and start the game again (values are read at startup).
The first `wpmc plugin loaded` line in the log shows the values in use.

| Setting | Default | What it does |
|---|---|---|
| `[Collision] Radius` | 40 | Metres around you whose collision is sent to Minecraft. Lower (e.g. 24) = less work and data; too low and you can walk off the edge of what's loaded when moving fast. |
| `[Collision] FrameBudgetMs` | 1.5 | Most milliseconds per frame the collision scan may use on the game's main thread. Lower = smoother, but new areas take longer to load. 0.5-3 is sensible. |
| `[Collision] SendBudgetKB` | 512 | Kilobytes of collision data handed to the bridge per frame (nearest cubes first). |
| `[Collision] RescanDistance` | 6 | Start a new scan after moving this many metres. |
| `[Collision] RescanInterval` | 5 | Also rescan every N seconds, to notice moved or new objects (unchanged objects are cached, so this is cheap). |
| `[Collision] SnapMeters` | 0.0625 | Corners are snapped to this grid and duplicate/tiny triangles dropped. 0 = off. |
| `[Collision] TerrainRefreshSeconds` | 20 | How often terrain heights are re-read (for terrain you edit in build mode). |
| `[Collision] MatchPlayerCollision` | true | Only send objects Waterpark's player really collides with. If that would drop more than half the objects in a scan, the mod ignores it for that scan and logs a warning. `false` = send every non-trigger object on a solid layer (if collision ever "disappears", try this). |
| `[Collision] CutGroundOverWater` | false | The 13:00 rule (cut ground in a box around every water volume). Off: it removed pool floors. Pools now follow the game's own terrain holes, which is always on. |
| `[Collision] NonSolidLayers` | auto | Unity layers treated as water (non-solid). `auto` = every layer whose name contains "water". Or a comma list of layer names/numbers, e.g. `Water, 4`. Empty = none. |
| `[Collision] NonSolidNamePatterns` | watersurface, waterplane, ..., =ocean, =sea, =water | Objects whose name contains one of these are water (case, spaces and `_` ignored). A pattern starting with `=` must be the whole name (ignoring `(Clone)` and trailing numbers), so `=ocean` matches `Ocean (2)` but not `OceanBench`. Names containing "slide" never match. Empty = off. |
| `[MinecraftUI] Enabled` | true | Show Minecraft's HUD, screens and hand over Waterpark. `false` = old behaviour (no overlay, no UI mode). |
| `[MinecraftUI] ToggleKey` | RightAlt | Key that turns Minecraft mode on/off (wheel = Minecraft hotbar, E = Minecraft inventory). Any Unity key name, e.g. `F6`, `Backquote`, `RightCtrl`. |
| `[MinecraftUI] OverlayFps` | 30 | How often Minecraft redraws the HUD for Waterpark (10-120). Lower = less work for both games. |
| `[MinecraftUI] GuiScale` | 0 | Minecraft GUI size while linked (1-6). 0 = Minecraft's own setting. |
| `[MinecraftUI] ShowMinecraftHand` | true | Minecraft's hand/held item instead of Waterpark's first-person arms. |
| `[MinecraftUI] ShowMinecraftBody` | true | Minecraft player model with your skin instead of the Waterpark character in third person. |
| `[MinecraftUI] HotbarKeysInGame` | true | Keys 1-9 always select Minecraft hotbar slots; Waterpark's own 1-9 actions are blocked while linked. |
| `[MinecraftUI] HideWaterparkHotbar` | true | Hide Waterpark's own item hotbar while Minecraft mode is on. `false` = keep it visible. |
| `[MinecraftUI] ShowMinecraftHotbarInWaterparkMode` | false | Show Minecraft's hotbar (with the hearts, hunger, armour and XP bar that sit on it, and the held item's name) while Minecraft mode is OFF. Default: it only appears in Minecraft mode. Chat, crosshair and hand always stay. Needs the 13:10 jar. |
| `[MinecraftUI] ScrollHotbarInGame` | false | `true` = the wheel always scrolls Minecraft's hotbar, as if Minecraft mode were always on for the wheel (E stays Waterpark's). Normally leave it off and use Right Alt. |
| `[MinecraftUI] HideMinecraftWindow` | true | Hide Minecraft's window while linked. Set `false` to watch it for debugging. |
| `[MinecraftWorld] ShowMinecraftBlocks` | true | Draw the Minecraft blocks near your Minecraft player inside Waterpark (visual only, no Waterpark collision). |
| `[MinecraftWorld] ShowMinecraftEntities` | true | Draw nearby Minecraft players (with skins), dropped items and mobs (coloured boxes). |
| `[MinecraftWorld] BlockRadius` | 48 | Metres around your Minecraft player whose blocks are drawn (16-128). Larger = more to load and draw. |
| `[MinecraftWorld] EntityRadius` | 32 | Metres around your Minecraft player whose entities are drawn (8-96). |
| `[MinecraftWorld] MaxEntities` | 64 | At most this many entities, nearest first. |
| `[MinecraftWorld] ShowMinecraftModels` | true | Real Minecraft models for mobs and block entities (chests, shulker boxes...). `false` = the old coloured boxes. |
| `[MinecraftWorld] HittableGuests` | true | Waterpark guests near you can be hit in Minecraft (arrows, snowballs, punches) and ragdoll in Waterpark. |
| `[MinecraftWorld] GuestsAvoidBlocks` | true | Guests walk around Minecraft blocks standing on the park (navmesh obstacles) and ragdolled guests bump into them. `false` = guests walk through Minecraft blocks. |
| `[MinecraftWorld] GuestHitForce` | 8 | How hard a hit pushes a guest (0-60); arrows add their damage on top; explosions push from half (edge of the blast) to 3x (point-blank). |
| `[MinecraftWorld] MatchJumpHeight` | true | Your Minecraft jump reaches Waterpark's jump height while linked. |
| `[MinecraftWorld] ShowMinecraftParticles` | true | Draw Minecraft's particles near you in the park, and the light flash of Minecraft explosions. |
| `[MinecraftWorld] CutoutMode` | Auto | How see-through block pixels (plants, rails, doors, leaves) are cut out. `Auto` tests the shader once and falls back to `Geometry` if it can't. `AlphaClip` = always the shader; `Geometry` = always real geometry (more triangles, works everywhere). Try `Geometry` if plants show black or coloured squares. |
| `[MinecraftWorld] MaxParticles` | 1000 | At most this many particles drawn at once, nearest first (0-1000). |
| `[Overlay] MatchSunLighting` | true | Light Minecraft's hand and held item from Waterpark's sun. |
| `[Debug] LogLargestColliders` | 15 | On the first scan of each park and every 10th scan, log the N biggest colliders sent to Minecraft as solid (path, type, layer, size), to find invisible walls. 0 = off. |
| `[Debug] DumpCollidersKey` | F8 | Press it in the park to log every collider within 3 m of you and what the mod does with it. `None` = off. |
| `[Debug] VerboseLog` | false | `true` adds extra `wpmc: [debug]` lines (non-readable meshes, slow mesh copies, abandoned scans). |
| `[Bridge] Port` | 47560 | Must match the bridge. |

If it still stutters: first set `FrameBudgetMs = 0.75` and `Radius = 28`, test again, and send both logs.
On the Minecraft side there is nothing to tune: collision is built on a background thread.

## 8. How to report a problem
1. Close both games (so the logs are complete).
2. Collect: `...\WaterPark Simulator\BepInEx\LogOutput.log` and
   `C:\Users\Kraft\AppData\Roaming\PrismLauncher\instances\WaterparkSim\minecraft\logs\latest.log`,
   plus a copy of the text in the bridge window if it showed errors.
3. Write down what you did and what you saw, e.g. "pressed E in Minecraft mode, inventory opened but the cursor was invisible",
   roughly when (clock time), and a screenshot if something looks wrong on screen.
4. Your config file (`BepInEx\config\dev.wpmc.waterpark.cfg`) if you changed any setting.
