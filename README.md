# Sector Sweep

Combat behaviour mod for X4: Foundations.

It adds a **Sweep Sector** default behaviour for player fleets: pick a sector,
and the fleet keeps it clean. It sweeps the whole sector using shared faction
sight - optionally shared allied sight - weighs its own fleet against the
enemy's before committing, picks fights it can win, joins attacks already in
progress, and lets capital groups open assaults on hostile stations. Between
fights it roams the sector instead of parking on one waypoint. No vanilla
decision-making is touched - NPC factions keep their vanilla behaviour.

## Why

Not everyone wants a mod reaching into vanilla AI. **Enhanced Patrol AI** gets
its results by rewriting vanilla **Patrol** in place: it injects into a handful
of vanilla combat and movement scripts, most heavily `move.seekenemies`. That is
the whole point of it - and it is also the reason some players will not touch
it. Injected scripts can file-conflict with any other mod editing the same ones,
and they are the first thing to break on a game patch. EPAI can be narrowed down
to player ships only through its options, but the injections are still there
either way; that is not something an option can switch off.

This mod is the same engine with none of that. **Sweep Sector** is a new
behaviour in its own script - nothing about vanilla's own decision-making is
rewritten. Only the ships you explicitly set to sweep use the mod's logic, and
every other ship in the universe, yours or NPC, runs exactly the code Egosoft
shipped.

So if you looked at Enhanced Patrol AI and thought *"I like the shared allied
vision, the fleet-vs-fleet decision making, the combined-force attacks and the
approach around hostile stations - I just don't want vanilla scripts patched to
get them"* - this is that.

## Game version compatibility

- **9.00 release (build 611726)** - developed and tested here.

The order itself is a new script. Its Combat Logic dependency does inject into two
vanilla scripts, but none of those injections are critical: if they stop applying
on a future patch, the mod loses some features and vanilla carries on as usual.
Nothing here can break a save or a game - worst case is the mod quietly doing less.

## Dependencies

- **[VAS] Combat Logic** - hard dependency. The scan engine, force math and
  attack issuing all live there. It has no options of its own; install it and
  forget it.
- **SirNukes Mod Support APIs** ([link](https://www.nexusmods.com/x4foundations/mods/503)) - hard
  dependency. Provides the Simple Menu / Options helpers used by the options menu.

## Usage

**Sweep Sector is a default behaviour only.** Open a ship's Behaviour tab, set
its default behaviour to **Sweep Sector**, and choose the sector to sweep. It is
not available as a one-off order in the order queue, and there is no intent to
make it one.

Assign it to a fleet commander and the whole fleet participates: subordinates
follow their commander into the fights it picks, and the force math counts the
fleet, not the single hull.

## Per-fleet options (on the order itself)

| Param | Default | What it does |
|---|---|---|
| Sector | - | The sector to sweep. Required. |
| Attack ships | on | Include hostile **ships** as sweep targets. |
| Attack stations | on | Include hostile **stations** as targets, so capital groups can open assaults. |
| Bypass civilian ship attack restriction | off | Engage hostile ships even when vanilla fire authorization says no (traders, miners, builders). Off keeps the sweep off the civilian economy. |
| Bypass civilian station attack restriction | on | Engage hostile stations even when fire authorization would normally hold fire - useful for countering invasions. |
| Max pursuit travelled distance | 800 km (50–500000) | Travelled-distance budget for a pursuit before the ship reconsiders. This is **not** vanilla straight-line `pursuedistance` - movement inside combat range is not counted. |
| Max attack initiation distance | 800 km (50–500000) | Maximum distance at which a *new* attack may be started. |

Everything above is per fleet, so one fleet can be told to leave civilian traffic
alone while another hunts it.

## Mod-wide options (Extensions -> Mod Options -> Sector Sweep)

| Option | Default | What it does |
|---|---|---|
| All-seeing - ignore own line of sight | off | Treat every hostile as visible, bypassing the sight check entirely. |
| Use allied shared vision, not only same faction | on | Accept spotting from any allied sensor, not just same-faction ones. |
| Scan interval | 30 s (1–60) | Minimum time between sector scans for a given sweeping ship. Also drives the Combat Logic wake heartbeat. |
| Max station-avoidance waypoints | 4 (0–10) | How many detour waypoints may be inserted to fly *around* a hostile station on the way to a target. 0 disables detouring. |
| Ship join threshold | 10 % (0–100) | Minimum committed friendly force required before this ship joins an existing ship attack. |
| Ship overcommit limit | 10 x (1–50) | Stop sending more friendly force at a ship target once committed force exceeds this multiple, so one target does not pull the whole sector. |
| Station assault join threshold | 3 % (0–100) | Minimum already-committed force on a station before this ship will pile on. |
| Station assault init threshold | 15 % (1–100) | Combined capital-group force required to *start* a fresh station assault. |
| Potential ally combatant force coefficient | 50 % (0–100) | How much nearby friendly combatants that have not committed yet count toward the thresholds. |
| Force estimation deviation | 10 % (0–90) | Randomness applied to each force comparison, so identical situations do not always resolve identically. |
| Debug logging | off | Writes per-ship logs under `VAS_SectorSweep/`. |

## How it works

A sweeping ship flies to its assigned sector and then loops:
**scan -> attack -> roam -> scan**. It is vanilla Patrol's skeleton with the
sector combat scan wired in and everything not relevant to this mod stripped out.

- **Scan.** On a heartbeat from Combat Logic (throttled to the scan interval),
  the ship scans the whole sector rather than waiting to bump into something. A
  target counts as "seen" if this ship can see it, or if a friendly ship,
  station or satellite can - optionally widened from same-faction to all allied
  sensors, or made all-seeing.
- **Decide.** Each candidate is scored by a fleet-vs-fleet force comparison
  (DPS × durability × mobility, whole visible fleet branch on both sides), and
  penalized when enemy force sits on the route to it. A ship engages alone only
  if it expects to come out ahead; otherwise it can still pile onto a fight
  already live, up to the overcommit limit.
- **Attack.** Combat Logic hands the target to the game's own Attack orders, so
  the actual fighting is vanilla - with detour waypoints around hostile stations
  that cannot be overpowered on the way. When the attack finishes, the order
  re-enters and rescans immediately instead of flying back to a waypoint.
- **Roam.** With nothing worth attacking, the ship wanders between known zones
  of the sector, and asks for resupply when it needs it. Any wake interrupts the
  roam for the next scan.

Same-tick scans are staggered by a few milliseconds per ship, so ships in the
same sector can see the commitments their siblings just made instead of all
piling onto the same target.

## Credits

- Built on **[VAS] Combat Logic** and **SirNukes Mod Support APIs**.
- Grown out of **Enhanced Patrol AI**, which was in turn inspired by
  **Shibdib's Improved Patrol**
  ([Nexus](https://www.nexusmods.com/x4foundations/mods/1712) ·
  [source](https://github.com/shibdib/X4Mods/tree/master/shib_improvedpatrol))
- By VasiliyTemniy.

## Source

https://github.com/VasiliyTemniy/x4-foundations-sector-sweep

