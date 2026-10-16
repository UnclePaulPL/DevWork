# UncleDev Towing v1.5 — Truck & Trailer Edition

A FiveM tow and vehicle-transport resource featuring a configurable registry of **flatbed trucks, traditional tow trucks, and transport trailers**. Uses a custom **NUI control panel** for operations, and **ox_lib** for notifications and progress bars. Operates standalone with optional ESX, QBCore or Qbox job checking and bank payouts.

> This is a complete source-code release but has **not been tested on a live FiveM server**. Custom add-on models require real-world testing and offset adjustments.

## What changed in v1.5?

- **Rollback trucks with a moving gate.** The gate opens before a vehicle is winched on or rolled off, then closes. See [Rollback trucks with a gate](#rollback-trucks-with-a-gate).
- The `rollback` preset is added. Set the gate in `shared/vehicles.lua` or in `/towedit`.

## What changed in v1.4?

- **In-game configurator (`/towedit`).** Add or edit trucks and trailers without touching Lua files. See [Configure trucks in game](#configure-trucks-in-game).
- Saved settings go to `data/carriers.json` and are applied on top of `shared/vehicles.lua`. Built-in models can be overridden and reset.

## Rollback trucks with a gate

A truck's gate is a GTA vehicle door. To use it:

1. Find the door index (0–7) for the gate in the truck's `vehicles.meta`.
2. In `shared/vehicles.lua`, register the truck with the `rollback` preset and set its gate:

   ```lua
   Config.AddTruck('myrollback', 'rollback', { 0.0, -3.0, 1.0 }, 'My Rollback Truck')
   Config.SetGate('myrollback', 5)
   ```

   Or set **Gate door** in `/towedit`. Leave it empty for trucks without a gate.

With a gate set:
- **Winch in** opens the gate, pulls the vehicle up, then closes the gate.
- **Winch out** and **Rollback** open the gate before the vehicle moves, then close it.
- The gate is opened by whoever is operating the truck, so they are given control of it first. If the truck is owned by someone else and they're far away, the gate may not move until they're nearby.

Gate movement uses GTA's door animation. Only the operator's client runs the animation. Other players see the gate move through normal vehicle sync.

## Configure trucks in game

1. Give staff permission in `server.cfg`:

   ```cfg
   add_ace group.admin uncledev.tow.admin allow
   ```

2. Spawn the truck or trailer you want to set up and stand next to it (or sit in it).
3. Type `/towedit`. The panel opens and coloured 3D markers show each loading spot:
   - **Amber** = where the vehicle sits on the deck.
   - **Blue** = where it is dropped when unloaded or rolled back.
4. Set the basics:
   - **Display name**
   - **Type:** Truck or Trailer
   - **Preset:** Flatbed, Wheel-lift, or Transport trailer
5. Adjust the loading spots. Type a number, or use the **+ / −** buttons with the nudge step (0.01, 0.05, 0.1, or 0.5). Watch the markers move as you change them.
   - **Add loading spot** for multi-car trailers (up to 8).
   - **Advanced** shows unload position, rotation, and attach method.
6. Press **Save**. The change applies to everyone right away. No restart is needed.

**Reset / Delete:** for a built-in model you changed, **Reset to built-in** restores the original. For a model you added, **Delete** removes it. Click the button twice to confirm.

**Keys:** Esc closes the editor without saving.

**Notes**
- The server must be able to write to the resource's `data/` folder. If the save fails, the server console and the error message say so.
- `/towedit` needs the `uncledev.tow.admin` ace (checked on the server, so it can't be bypassed from the client). Change it with `Config.AdminPermission`.
- Changing a spot's offset doesn't move a vehicle that is already loaded. Unload and reload it to use the new position.

## What changed in v1.3?

- Winch and rollback controls (below).

## What changed in v1.2?

- **New tow control panel.** F6 / `/tow` opens a dark side panel instead of the old ox_lib list menus:
  - Pick a truck or trailer from the nearby list (when several are close).
  - See every loading spot at a glance, marked **EMPTY** or **LOADED**, with the loaded vehicle's name.
  - Load shows each nearby vehicle with its plate and distance. Unload and Impound sit on the loaded spot.
  - **Keyboard friendly:** ↑ / ↓ to move, **Enter** to select, **Backspace** to go back, **Esc** to close. Mouse works too.
- **Winch in** (empty loading spot): the chosen vehicle is pulled up the deck over a few seconds instead of snapping on.
- **Winch out** (loaded spot): the vehicle is lowered slowly off the back.
- **Rollback** (loaded spot): the vehicle pitches nose-down and slides off the back faster. GTA can't rotate the bed itself, so the tilt is simulated by moving the vehicle.
- Timings are set in `shared/config.lua`: `WinchLoadDuration`, `WinchOutDuration`, `RollbackDuration`, and `RollbackTilt`.
- "Unload" is replaced by Winch out and Rollback. Impound is unchanged.
- Server logic, callbacks, exports, and the rest of the config are unchanged, so existing integrations keep working.
- To style the panel, edit `html/style.css`. Its colours are CSS variables at the top of the file.

## What changed in v1.1?

- Add trucks and car-hauler trailers by **editing one file: `shared/vehicles.lua`**.
- Three presets: `flatbed`, `wheel_lift`, and `trailer`.
- Supports **transport trailers**, even when they are hitched to a separate truck.
- Supports **1–8 independent loading slots** per truck or trailer (optional).
- Tow menu displays trucks and trailers separately when more than one is nearby.
- `/towmodel` displays a vehicle's actual spawn name to make adding add-on vehicles easier.
- Hash-based vehicle detection works with add-on model spawn names, not just display names.
- Expanded network synchronization and server-side slot validation.

## Requirements

- A reasonably current FiveM server with OneSync enabled.
- `ox_lib` installed and started first.
- No database needed for the core tow/transport features.

## Installation

1. Extract the folder named `UncleDev_Towing` into your FiveM `resources/` folder.
2. Ensure `ox_lib` is installed.
3. Add to `server.cfg` in this order:

```cfg
ensure ox_lib
ensure UncleDev_Towing
```

4. Restart the resource/server. Stand near a **registered** truck or trailer and press **F6**, or use `/tow`.
5. For job-only towing, change `Config.RestrictToJobs = true` in `shared/config.lua`. Set `Config.Jobs` there, too.
6. Optional admin bypass ACE in `server.cfg`:

```cfg
add_ace group.admin uncledev.tow allow
```

## Easy one-line setup (edit `shared/vehicles.lua`)

```lua
-- Flatbed truck
Config.AddTruck('myflatbed', 'flatbed', { 0.0, -2.7, 1.0 }, 'My Custom Flatbed')

-- Conventional hook/wheel-lift tow truck
Config.AddTruck('mywrecker', 'wheel_lift', { 0.0, -3.8, 0.2 }, 'My Wrecker')

-- Car-hauler / transport trailer
Config.AddTrailer('mycarhauler', 'trailer', { 0.0, -2.2, 1.3 }, 'My Car Hauler')
```

**Use the exact vehicle spawn name** (e.g. `myflatbed`) used by that vehicle's resource. `/towmodel` identifies the model of a vehicle you are in or standing near. `tr2` is included as a stock GTA V transport-trailer example. `flatbed2` is registered as an *optional custom spawn name*; it only appears if you separately have that vehicle model.

**Offsets:** `{ x, y, z }` means left/right, forward/backward, and height relative to the carrier. Most loading beds are at **negative Y** values. Customize the three numbers for your specific model.

## Custom trailers with 2+ parking spots

Register your model first, then assign its deck locations. This is an EXAMPLE: the numbers are not calibrated for your specific trailer.

```lua
Config.AddTrailer('mydoubletrailer', 'trailer', { 0.0, -2.0, 1.3 }, 'Double Carrier')
Config.SetSlots('mydoubletrailer', {
  { label = 'Front Slot', offset = { 0.0,  1.7, 1.3 } },
  { label = 'Rear Slot',  offset = { 0.0, -3.2, 1.3 } }
})
```

Each spot has its own **Load**, **Unload**, and **Impound** action. To tune drop positions individually, add `unload = {x,y,z}`. To rotate loaded vehicles, add `rotation = {x,y,z}`. For a compatible native trailer, optionally set `method = 'native_trailer'` on a slot to use `AttachVehicleOnToTrailer` instead of the general attachment method.

### Important: real trailer hitch compatibility

This resource manages **vehicles loaded ONTO trailers**. The towing truck still needs a compatible GTA V hitch and correct `vehicles.meta` trailer compatibility to physically connect and pull the trailer. Registering a trailer in this Lua file does NOT magically add an attach point to any random truck or add hitch compatibility to an unsupported add-on vehicle. This is separate from securing cargo onto the trailer. Ask your custom vehicle author about a trailer hitch or update the relevant vehicle metadata yourself.

## How towing works in game

1. Spawn a registered tow truck or car-hauling trailer and stop it.
2. Park a vehicle close enough to the deck or hook. Everyone should exit the vehicle.
3. Exit the tow vehicle, press **F6** (or `/tow`).
4. If multiple trucks/trailers are close together, choose the one you want.
5. Select an empty **loading spot**, pick the vehicle, complete the progress action.
6. To unload or impound, open **F6** again and select its occupied slot.

An attached vehicle is visually secured using FiveM attachments; mechanical ramp movement/winch cables are **not simulated**. A model might need its ramp opened separately (provided by the vehicle's own resource). Custom vehicle geometry determines how natural it looks.

## Commands

| Command | Description |
| --- | --- |
| `/tow` or `F6` | Open tow operations, select nearby truck/trailer, load/unload slots |
| `/towmodel` | Show nearest vehicle's exact spawn name, including custom add-on models |
| `/towrequest Need a tow at 403` | Send a tow assistance request (optional dispatch) |

## Other settings

In `shared/config.lua`, edit `Config.Framework`, `Config.Jobs`, `Config.InteractionDistance`, `Config.MaxAttachDistance`, `Config.CooldownSeconds`, `Config.ImpoundLocations` and dispatch settings.

If using standalone (without configured mechanics/jobs), set `Config.DispatchJobsOnly = false` to send dispatch requests to everyone.

## Impound caution

**Impounding only removes the active networked vehicle entity. It does not update owned-vehicles tables or garage state.** If you use ESX owned vehicles, QBCore player vehicles, JG Advanced Garages, etc., connect their documented persistence/impound API before enabling production use. Until then, an owned car can later respawn or remain marked out in its garage DB. Impound rewards are disabled by default.

## Server exports

```lua
exports.UncleDev_Towing:GetTowedVehicle(carrierNetId) -- first cargo net ID or nil
exports.UncleDev_Towing:IsVehicleTowed(cargoNetId)     -- boolean
exports.UncleDev_Towing:GetCarrierLoads(carrierNetId)  -- map of slot number -> cargo net ID
```

## Troubleshooting

- **Truck/trailer not showing up:** Check its spawn name with `/towmodel` and register it in `shared/vehicles.lua` using `Config.AddTruck` or `Config.AddTrailer`. Restart the resource after editing.
- **Vehicle floating/sunken:** Adjust `offset = {x,y,z}` for the model's deck height and position.
- **Multiple vehicles overlap:** Configure different `Config.SetSlots` positions, sufficiently spaced for the chosen cars. Test each slot with the vehicle sizes you allow.
- **Trailer will not hitch to the truck:** That is GTA V model/`vehicles.meta` hitch compatibility, not cargo-towing script registration.
- **Vehicle not loading:** Ensure it is close enough, vacant, road-class and networked. If it belongs to another player, you may need that player nearby for network control.
- **Menu not opening:** Start `ox_lib` before this resource and check F8 for script errors.
- **Owned vehicle appears again after impounding:** Add your garage integration; entity deletion is not a DB impound.

This initial v1.1 should be tested and tuned on your server before relying on it for a persistent economy or police workflow.
