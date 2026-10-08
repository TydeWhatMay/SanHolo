
# SanHolo

An autonomous client for [replicant.space](https://replicant.space) — plays
the game on your behalf and gives you a live mission-control dashboard
to watch what it's doing.

> **Status**: pre-release binaries. Source code will be published once the
> client stabilises. Run with `--dry-run` (the default) until you trust it.

## What it does

- **Mines** the belt resources your current build goal actually needs, and
  adapts as resources deplete; opens new sites with survey drones; consolidates
  loot home with transport drones. When a belt is depleted and miners sit idle,
  the Mining panel calls out that blocker directly. Idle drones at a remote
  supply source never mine a resource home already holds more than 100,000
  of; among the rest they take whatever home holds least of, and they stay
  idle rather than spend a scarce site on surplus.
- **Works idle miners wherever they stand.** SanHolo reads the open sites of
  every belt where your drones sit (and of belts it knows about in systems you
  hold a comms anchor in), one belt every few polls. A free, idle mining drone
  beside a live site holding rares, volatiles, carbon, silicates or a resource
  your build goal needs is started on it, rares first, and a belt's idle drones
  are spread across what its sites hold, at most one per worthwhile resource
  per live site. Drones run by an AMI controller, out of comms range, or on
  the belt under your ship are left to their usual owners. Home-system drones
  are worked this way only while the ship is away from home, never while a
  miner ferry is loading from home, and always leaving two free idle miners
  for salvage. A belt whose command fails for a reason other than depletion is left
  alone for five minutes. When home has spare idle miners, the same readings decide where a
  miner ferry goes: the belt with the most minor-resource value left in its
  live sites wins, whether or not it has a mining controller, and a belt whose
  sites read empty is never chosen. The target must have a relay or hub
  (a beacon does not relay commands) and something there must currently read
  in control range, so ferried miners can still be commanded after the ship
  leaves. A belt without a controller is sent one miner per live site, and a
  ferry is only armed for at least three miners.
- **Stays out of systems another player has claimed.** A system hub locks its
  system's belts and salvage to its owner. The first time the game refuses a
  `start_mining` with "Mining restricted - system claimed", SanHolo remembers
  that system for a week (across restarts) and stops starting miners there,
  ferrying miners to it, crewing it with the cargo vessel, mining it as a
  supply source, or flying to validate it. It also stops surveying there:
  no `search` from survey drones or belt surveyors, no new survey drones
  deployed or printed, and a survey controller there adopts nothing, sets no
  `belt_search` and lets go of the drones it holds. Its free survey drones
  ride home with the claimed system's miners and are deployed at home; if
  the game refuses one drone's recall three times in a row, the run stops
  and waits an hour rather than holding the ship there, and that drone is
  left out of withdrawals for a day (across restarts), so it alone never
  sends the ship back. Belt
  surveyors cannot board the ship's hold (they have no stow feature), so they
  stay parked where they are, idle or holding their last site; relays and
  beacons there are left alone. After the week it lapses: if idle
  miners still stand there SanHolo tries one `start_mining` again, and a
  fresh refusal starts a new week (a lifted claim is picked up this way).
- **Keeps the home belt's five site slots worked.** A belt holds at most five
  active sites, and each needs a survey drone searching or tracking it; survey
  controllers do not count. When fewer than five drones are working the home
  belt and no idle drone is waiting to be tasked, SanHolo queues one survey
  drone at a time on the home autofactory, adopted straight into a home survey
  controller, up to five plus the two kept free for exploration, counting
  prints already queued.
- **Explores** the system: scans all planets and moons, surfaces new
  resource sites, salvages, and location events as they appear.
- **Scouts** nearby stars with the home vessel once your account is mature
  enough to support it safely. It now prefers the best nearby unexplored
  system, not just the nearest, and always returns home after the scan-and-
  survey leg. Fresh accounts keep `scout` off by default; developed anchored
  empires auto-enable it unless you've explicitly chosen another stored scout
  setting.
- **Expands frontier worlds** more deliberately: when nearby discovery pressure
  subsides and the best next move is to anchor a world you've already scouted,
  SanHolo can now arm the normal `colonise` flow automatically toward that
  anchor-chain target instead of leaving expansion stuck at the recommendation
  layer. Colonisation keeps the source hub or relay and beacon active, carries
  a bounded bootstrap payload in the vessel's free slots, and leaves excess
  controller-managed workers productively operating at the source instead of
  waiting forever for the whole fleet to become idle. It reuses and stows a
  spare co-located beacon for the destination, and when the source is down to
  its last one — which is its own watch and cannot be taken — it prints exactly
  one more so that the existing beacon becomes spare, rather than stalling.
  It never prints repeatedly while waiting for a beacon to appear in vessel
  cargo. A temporary nearby-star
  refresh failure no longer clears the persisted colonisation target.
- **Salvages** discovered wreck sites automatically.
- **Repairs** worn drones via maintenance drones before the wear costs
  you mining throughput.
- **Cleans up stranded extractor drones** conservatively: when a live scan proves
  the current non-home system has no asteroid belt, SanHolo can decommission
  idle mining/survey drones there in place while leaving comms anchors and all
  protected device types alone. The Fleet posture panel calls this out before
  or while the cleanup runs.
- **Brings miners home on request.** Set the `recall-miners` directive with
  a source belt (for example `SIHAIL-BELT-1`) and SanHolo releases that
  system's idle mining drones from its AMI, carries them home in the ship's
  hold a load at a time, deploys them at your home base for the normal
  strategies to retask, and clears the directive when none are left. From a
  system with no asteroid belt it also brings home each mining controller
  left holding no drone and deploys it at home. If the hold is full of other
  cargo it stops and clears the directive instead of holding the ship.
- **Withdraws miners a system cannot use.** With the ship at home and no
  directive set, SanHolo arms `recall-miners` itself for a system you command
  that either another player has claimed (every idle miner comes home) or
  where a mining controller's belt holds at least three idle free miners more
  than it keeps (ten, or more if its controllers hold more); adopted miners
  stay, and the miners it keeps go on working any open site while the surplus
  is carried home. A system with no belt is only withdrawn from when the ship
  is already there, never with a dedicated trip. If a run has to stop because
  the ship's hold is full of other cargo, SanHolo waits an hour (across
  restarts) before arming another. If an asteroid threatens home while a
  miner ferry or withdrawal has the ship away, the ship flies home at once
  with whatever it carries, so asteroid defence can act; while the threat
  lasts the ship does not leave home, and a threat in the remote system the
  ship stands in pauses the run there.
- **Stops mining an exhausted belt.** When the game reports a belt
  exhausted, SanHolo stops sending drones to it until a survey reopens it,
  instead of retrying it every few seconds.
- **Ferries idle miners to where the work is.** When a system you can command
  has a mining controller and salvage waiting but no drones, SanHolo loads idle
  miners into a cargo vessel and sends them, releasing each from its home
  controller first so the destination controller can adopt it. It uses a cargo
  vessel rather than your flagship, so the ship stays put and you keep command
  of the fleet while the run happens.
- **Builds relay chains to systems a single delivery cannot reach.** A relay
  reaches 7.5 light years and a system hub 15, so a system further out than
  that stays unreachable no matter how many relays you put *in* it — its own
  relay has nobody to talk to. SanHolo reads the star catalogue, finds the
  route that links it with the fewest new relays, and walks the vessel out
  along it, carrying a relay and a beacon to each intermediate star and
  handing over to the normal deploy-and-activate flow on arrival. A chain needs
  one relay per stop and the ship carries one at a time, so between stops it
  returns to base for the next one — and it will not leave a stop until the
  relay it just delivered is active and the beacon deployed. It holds one
  `relay-chain` directive naming the final destination for the whole journey,
  so the ship is not recalled halfway, and releases it only when the target's
  devices actually report back in range.
- **Keeps every commandable belt watched.** A system you can command whose
  belt has nothing searching gets its own idle belt surveyors or survey drones
  sent to the belt to open new sites. It works only with what is already
  there, or a spare drone in the ship's hold when the ship is already in that
  system; it never prints, never moves the ship, and never sends a scanner to
  a system you cannot yet command.
- **Keeps every system commandable.** While a comms delivery is in flight the
  bot holds a `presence` directive naming the target system, so nothing else
  redirects the ship mid-crossing. The directive clears itself the moment that
  system can be commanded, and you can see it in the cockpit's status strip
  alongside any directive you set yourself.
- **Keeps every system commandable.** A beacon lets you *see* a system; only a
  relay or hub lets you *command* one. SanHolo tracks both per star, and when a
  system holds devices you cannot reach, it builds a comms kit, carries it
  there, deploys it and comes home. Devices stranded without a relay cannot be
  recalled at all — the server refuses the command — so this is the only way to
  get them back. The cockpit's Comms Network graph reads the same per-star model
  the bot plans with, so a system it marks commandable is one the bot can
  actually command, not merely one the ship happens to be visiting.
- **Reclaims surplus FTL beacons.** A location only needs one beacon to hold
  comms, so SanHolo decommissions the extras in place — one per tick, keeping
  two at every location — and the refund goes back into your stockpile. Stowed
  beacons (hop and colonisation cargo) and beacons in other systems are left
  alone.
- **Prints** more drones (and AMI controllers, system_hub, etc.) when
  the right resources accumulate — capped so it doesn't run away. When
  discovery is the strongest climb available, unattended printing now
  biases toward survey capacity before raw miner count. It now also notices
  obvious idle slack in the fleet, so it stops assuming that "print more of
  the thing already sitting idle" is the best default growth move. Big idle
  transport pools are now explicitly demoted so the bot stops inflating a role
  that is already underused. For low-risk fleet growth, the unattended
  mining-drone cap is higher than the early-game defaults because miners do not
  spend scarce rares or volatiles.
- **Re-arms logistics** when a real build goal is short on resources and a
  cached remote anchored stockpile looks useful, preferring the best haul
  source instead of waiting only for the last manually-armed supply line. When
  discovery pressure is low, the Progress view can now promote that haul move
  directly as the best near-term climb.
- **Diverts incoming asteroids** before impact: prints propulsors,
  deploys + activates them at the asteroid, drops a beacon at the
  impact target once diverted. Active in-system asteroids at or above
  50% impact likelihood are handled early instead of waiting for a late
  high-certainty scramble.
- **Drives in-game AMI controllers** (transport, survey, mining) so the
  Replicant Space AI handles routine hauling, surveying, and resource
  gathering for you. If home-system mining or survey drones get stranded
  under a remote AMI controller, SanHolo now revisits that controller's
  system, lets the release path run, then returns home so local surveying
  and mining recover automatically.
- **Mining network planner** keeps a relay-aware per-system mining summary
  (staffing, anchors, known resources, freshness). The bot can now return from
  weak spokes when home is the better-managed mining world, and it only
  auto-arms remote miner ferries for anchored worlds with clearly better,
  fresher resource prospects. Persisted catalog site intel now triggers a
  validation visit too, so promising stale remote worlds are revisited and
  rescanned before miners get sent there. Fully depleted remote worlds are now
  treated as “finish the recovery/validation leg, then come home”.
- **A cockpit that starts you off.** The home screen opens with a **Needs
  you** panel: an ordered list of what wants your attention (a help request
  from the bot, an incoming asteroid with its impact time, a stalled
  directive, an unsupported version, unread messages), each with one button
  that opens the menu tab where you deal with it. On a fresh install it says
  what is happening (connecting, bot stopped, running in dry-run) and the
  one next step, with buttons to open Engine or, if you would rather play
  yourself, Play; that row goes once the bot is running live. A help
  request also has a **Dismiss** button, here and on Engine > Overview,
  for one that outlived the job that raised it. Dismiss only hides the
  current request: if the problem is still there, the bot raises it again.
  Beside it,
  **Objective** states the one thing the bot is working towards: the
  directive with its current phase and how long since it last moved
  (flagged when it has stalled), otherwise the official tutorial's current
  step while its tracker can be trusted, otherwise what the bot is doing,
  and "no objective yet" until the bot runs. **Blocked** groups what is stuck
  rather than listing it: paused automation by kind with the count, how
  long the oldest has waited and its most common error, older paused
  records folded into one count, paused AMI controllers, the last hour's
  errors and devices out of control range; "Nothing is blocked" when all is
  well. Fleet > Overview's "Current blocker" card counts the same paused
  automation. **Income** lists the six resources in order with the
  stockpile at your home base and its **net** change over the last hour
  (plus the same pair for wherever the vessel is, when that is somewhere
  else), and which resources are being gathered across the empire. While
  the vessel is away SanHolo reads the home stockpile every two and a half
  minutes so the hourly change stays current. It is net because printing spends
  the same stockpile, so it can go negative; with no reading from the
  start of the hour it says "no reading yet" rather than guessing. The
  panels refresh on their own (Needs you every 5 seconds, Income once a
  minute) and stop polling while the tab is hidden. On a phone the deck is
  one column in that order, Blocked starts folded, the map takes 45% of the
  screen until you press **Full map**, and the status strip keeps only the
  bot's state and any help request; the directive, the other replicant and
  reservations move into Objective. Empty panels say what fills them, every panel shares one look for
  loading and empty states in English, German and French (the objective's
  next step and blocker, Engine > Overview and the map panels follow the
  chosen language too; API errors and log lines stay as the game sent them), resources and
  devices carry icons beside their names, and the top bar stays readable on
  a phone.
- **Galaxy map at true coordinates.** Zoom out from your system and the
  galaxy level draws every catalogued star (tens of thousands) where it
  actually is, with the systems you hold coloured by presence tier (touched,
  observed, controlled, productive), your home ringed and your ship marked,
  including its course while travelling. Drag or use the arrow keys to pan,
  scroll, pinch or +/- to zoom, N and P to step through your systems, Enter
  to open a system's inspector and Z to zoom into the current system. The
  List tab beside it is the plain-table equivalent. Under the map, a
  **replay scrubber** plays back the last 24 hours, 3 days or 7 days of your
  empire from SanHolo's own recorded history: drag, play at up to 60 frames a
  second, or use space, the arrow keys, Home and End. Time SanHolo was not
  running shows as a hatched gap and an explicit "no data", never as your
  last known state, and times before presence history began show your ship's
  track only.
- **Galaxy-first cockpit** at http://localhost:8080 — a pannable, zoomable
  star/system map is the home screen, and a guided **Play** menu translates
  your current game state into a simple early-game loop for manual pilots.
  Click any planet, belt, star, or device to open an inspector with its
  detail and controls (a planet's type and signs of life are shown before it
  is scanned, marked "(est.)" until a scan confirms them). Devices the system
  map cannot place, elsewhere or in transit, are listed below it in a table
  grouped by location (devices with no reported location are counted but are
  not a place; a long list starts folded behind its count), and the fleet glyph legend appears once, above the
  map; a top menu bar (**Play**, **Engine**, **Fleet**,
  **Build**, **Intel**, **System**) holds the guided flow, bot controls,
  fleet roster, blueprints, leaderboards/catalog, and settings; the Intel
  menu now includes a **Progress** view that turns XP pace, build readiness,
  fleet growth, and leaderboard movement into a clearer “what helps next?”
  summary; leaderboard panels call out your visible rank plus recent up/down
  movement, including the account-scoped identity used by Replicant Space v3;
  nearby-star views show system hubs and wards, while the Fleet roster preserves
  attachment, linked-device, and tag metadata. Fleet controls are derived from
  each device's live advertised capabilities, explaining pending or unsupported
  controls instead of guessing; directives, features, and compact settings show
  why devices differ, while decommissioning remains separately confirmed and
  host-safe. The Comms Network graph follows the same palette and remains
  readable in the persisted high-contrast theme. The Build and Mining surfaces
  now render in tighter content shells with scroll wrappers on the truly wide
  tables, so they stop sprawling. System Settings now pre-fills the current
  replicant name before rename. The guided Play surface also explains live
  travel intent more plainly, including remote AMI orphan-recovery trips and
  stale-intel validation legs, and its automation guidance now adapts to
  new-player, dry-run, and already-live advanced-account states. It now includes
  explicit first-run checks for no scan, no deployed devices, empty current
  inventory, no build-ready blueprint, and stopped/dry-run automation. Play
  and the Inbox, Fleet, Build and Intel menus no longer open with text
  about what the page is for; Play leads with your mode and goes straight to
  the current objective, and the Inbox is translated in full. The
  Engine, Intel and BobNet overviews drop their card descriptions, and their
  labels, buttons and the Fleet, Build and Intel menu titles are translated.
  The
  **Comms network** menu
  maps owned FTL beacons, relays, and hubs, labels current
  relay-range edges, and turns nearby unanchored systems into one-click colonise
  directive targets. The guided Play loop now
  also carries its own per-step actions, so scanning, fleet inspection, and
  build follow-through are linked directly from the guided cards instead of
  forcing menu hunting. Play and Progress now also use a stronger guide-layer
  visual treatment, with clearer kickers, card depth, and section hierarchy so
  the “what matters next?” surfaces stand apart from the denser ops panels; the Progress surface now
  also calls out the current **Frontier** target so distance pushes and scout
  goals are visible instead of implicit, and adds a clearer fleet-utilization
  readout so underused capacity is visible without digging through raw device
  tables. It now also surfaces a **Logistics** hint when a remote anchored
  stockpile looks worth hauling and the freighter is sitting idle. Progress also
  ranks evidence-backed trade, XP, reputation, distance, seasonal, and
  infrastructure opportunities with reward, time, cost, and risk notes; these
  recommendations are advisory and never perform irreversible strategic work.
  The larger
  operational menus now use bounded panels and push heavier detail into
  explicit sub-tabs instead of forcing the top-level menu itself to become a
  scroll container, and scrolling over an open menu now stays with the active
  menu instead of dragging the background map. Engine and System now also open
  into short overview tabs first, while Fleet, Build, and Intel keep the raw
  command roster, full blueprint catalog, deeper intel, settings, and bot
  automation as explicit advanced disclosures one step deeper. That cuts
  default information overload without removing detail. System Overview now
  also shows live travel posture, destination/ETA, scan freshness, and local
  planet/moon coverage instead of behaving like a thin utility shell, and it
  now explains the temporary “empty” handoff state while the API clears the
  current location between systems. The Catalog panel
  itself now also opens with a compact summary before the raw salvage,
  depletion, event, and history tables, and Inbox/BobNet now call out quick
  counts and actions before the long feed. The Mining surface now also leads
  with a compact network summary so active production, idle miners, and the
  top blocker are visible before the raw AMI/resource detail. Fleet Overview
  now does the same for overall utilization, idle pools, and the main blocker
  before the deeper readiness and AMI sections. It reports **Production** at
  empire level, read from the mining controllers and their adopted drones
  (which resources, at how many locations); a controller that is stowed or
  travelling is not counted as producing. Its busy share counts **workers**
  only: beacons, relays, hubs, the observatory and static sensors are listed
  separately as infrastructure, and drones a running controller drives count
  as working rather than idle. Intel Overview now acts as a
  real command summary, pulling threat posture, growth focus, leaderboard
  movement, frontier pressure, logistics posture, and inbox pressure into one
  screen before you dive into the specialist tabs. Intel Threats now also opens
  with a compact posture summary before the detailed asteroid bookkeeping.
  The overview cards across Engine, Fleet, Mining, Build, Intel, and System
  now also share a more consistent visual treatment instead of each tab
  carrying its own ad-hoc inline styling.
  Build Queue now also leads with a short queue posture summary before the raw
  autofactory and unlock tables. When nearby
  unexplored stars run dry, the same frontier planning can now promote the best
  already-scouted unanchored world as the next anchor-chain expansion target
  instead of silently dropping that path; a live ticker along the bottom shows
  the latest activity and copilot advice.
- **"While you were away" recap**: reopening the cockpit after an absence of
  half an hour or more shows a dismissible panel summarising what happened
  while you were gone — XP, fleet size, planets scanned, and reputation
  change, plus the net move on each of the six raw resources at your ship's
  current location (so resources hauled in from elsewhere count as a gain
  there), each clearly marked when there isn't enough data yet rather than
  shown as a misleading zero. Repeating errors are grouped by shape instead
  of listed one by one (so a single runaway failure doesn't bury everything
  else), and any decision the bot is waiting on you for is called out
  separately. The panel says how much of your absence it actually covers,
  separately for the empire stats and for the stockpile: stats come from
  recent history, so a very long absence may be only partly covered, and
  stockpile records at a location start only once the ship is there, so
  after a move the stockpile line can cover less than the empire line. The
  activity log keeps only its most recent entries, so the panel also tells
  you how far back the error picture reaches rather than implying it saw
  everything. The recap is shown once per return: reloading
  the tab, or leaving the cockpit for another page and coming back, counts
  as a new visit, and a short one shows nothing at all.
- **Beginner guidance**: the cockpit reads the official Replicant Space
  tutorials for both brand-new accounts and accounts with unfinished tutorial
  steps. The current tutorial is shown as a persistent checklist and as
  contextual guidance beside the relevant Play, Fleet, Intel, and other
  cockpit surfaces. If the tutorial service is temporarily unavailable, the
  checklist reports an unavailable/retry state and the rest of the dashboard
  and bot continue to operate normally.
  Accounts created before the game added tutorials never get one started,
  so the game reports them stuck on the first step for good. When the game
  shows no tutorial progress at all but the account has already earned
  achievements, SanHolo hides the checklist and its hints.
- **Safe tutorial actions**: the checklist can offer a one-click action only
  for the current `bootstrap` / `scan_system` step. That action reuses the
  existing validated scan path and requires a fresh, matching server step;
  arbitrary fields, stale steps, destructive commands, travel, printing, and
  strategic actions are rejected. Every other tutorial step is guidance-only.
  Tutorial guidance never starts the bot or changes dry-run/live state: SanHolo
  remains stopped and in dry-run until you explicitly change those controls.
  Every menu and tab has a shareable deep link, `/?open=<menu>&tab=<tab>`
  (for example `/?open=fleet&tab=roster`); the address follows the open menu,
  so a reload keeps it. The older `/inbox`, `/fleet`, `/intel`, `/build` and
  `/messages` links redirect into the matching menu.
  The **Fleet → Roster** tab also gives per-device manual commands —
  deploy / recall / stow, and **Decommission** (with a confirm prompt) to
  scrap a device you no longer need; the autofactory keeps its blueprint and
  refunds ~60% of the print cost to the device's location.
  Full keyboard navigation and a reduced-motion / high-contrast mode included.
- **BobNet** — the galactic comms layer (channel-based chat over the FTL
  relay network) is surfaced in the cockpit's BobNet panel as collapsible
  per-channel sections: read `#general` / `#trade` chatter, NPC
  announcements, and other replicants' messages each grouped under their
  own channel (with a message count), see sender codes alongside names when
  available, and send your own from the cockpit.
  BobNet rides on your relays — bring one online to start listening.
- **Inbox** — your message inbox leads with a *Mineable sites & salvage*
  panel that pulls newly-discovered resource sites and salvage (with their
  extractable resources) to the top, so actionable mining intel doesn't get
  buried. Remaining messages are tagged by category — discovery, salvage,
  depletion, achievement, blueprint or info — with mining-relevant tags
  highlighted.
  SanHolo marks the routine bulk notices read once it has taken what it needs
  from them: new resource sites, and other players' system hubs coming
  online. Story, event, threat, salvage and achievement messages stay unread
  for you.
- **Optional AI copilot** (Claude CLI or local Ollama) suggests
  next-step directives — always advisory; you decide whether to act.
- **Operator directives** — set a directive from the cockpit (focus a
  resource, build, explore, or relocate) to steer the bot, or clear it to
  hand control back to the default strategies. `relocate` is the explicit
  opt-in that authorises the bot to recall its fleet and hop to another
  star system (abandoning the current system's stockpile and any immovable
  structures) — it never relocates without it. `trade` is the opt-in that
  authorises the bot to inspect relay-visible shop stock and fulfil trades at
  reachable trade controllers — it only ever pays surplus resources (above a
  reserve floor) for current strategic or print-priority needs, and only
  fulfils at controllers at its exact location; it never trades without it. You
  can also set the **home base** (the location the
  bot consolidates resources into and builds its system hub) from the
  cockpit — handy for hub-and-spoke play as your galaxy footprint grows. A
  `relay-supply` upkeep strategy can top up an approved beacon-only system and
  park the vessel there so the relay bootstrap can finish the command anchor.
  `observatory-expedition` carries the galactic observatory only to
  already-anchored frontier systems, prospects for new catalogue targets, and
  returns it home.
- **Cross-restart memory** via a local SQLite catalog of sites,
  salvages, history; queryable from the CLI (`san-holo catalog ...`).
  Belt depletion is remembered (6h TTL), so after a restart the bot
  steers clear of known-dead belts instead of re-learning them by
  trial — until they regenerate and re-open. Strategy enable/disable
  toggles you set from the cockpit also persist, so a strategy you
  switched off stays off across restarts instead of re-enabling itself.
- **Game Director readout** — the Engine panel now leads with the current
  objective, blocker, and next action before the advanced strategy list. The
  same summary is echoed in the top status strip so manual pilots can see why
  the bot is acting, waiting, or blocked without decoding every strategy name.
  LIVE only ever means the data feed is fresh; everywhere the bot's mode is
  shown (the strip, Engine Controls, the bot panel) it reads ARMED (commands
  are real) or DRY-RUN. A help request shows its reason
  in the strip, and clicking it opens Engine Overview with the blocker, what
  to do about it and the actions that clear it. Long lines end at a whole
  word, with the full text on hover.
  Its action chips open the relevant cockpit menu/tab directly. The grouped
  strategy list also marks the most recently firing strategy and shows the last
  fire time per strategy, so you can tell what is actually hot without reading
  the full activity feed. In a scanned system with no asteroid belt, the
  exploration loop now keeps survey drones free from non-belt survey AMIs and
  can print a local survey drone when needed so the bot can keep scanning
  planets instead of silently stalling. The readout now also catches a
  directive that has stopped moving: if the run behind it has paused or has not
  advanced in half an hour, the objective card says so, names how long it has
  been stuck and what the run last failed on, rather than reporting a held
  directive as healthy progress. Objective colour follows that severity, so a
  blocker is never painted in the same green as normal operation.

## Download

Pre-built binaries are on the [Releases page](https://github.com/lhaig/SanHolo/releases/latest).

Pick the one that matches your machine:

| OS | Architecture | File |
|---|---|---|
| Linux | x86_64 | `san-holo-vX.Y.Z-linux-amd64` |
| Linux | ARM (Raspberry Pi, AWS Graviton) | `san-holo-vX.Y.Z-linux-arm64` |
| macOS | Intel | `san-holo-vX.Y.Z-darwin-amd64` |
| macOS | Apple Silicon (M1/M2/M3/M4) | `san-holo-vX.Y.Z-darwin-arm64` |
| Windows | x86_64 | `san-holo-vX.Y.Z-windows-amd64.exe` |

Each release also includes `SHA256SUMS.txt` — verify the download with:

```sh
sha256sum -c SHA256SUMS.txt --ignore-missing
```

## Install

### Linux / macOS

```sh
# Download (replace vX.Y.Z with the latest tag and pick your platform)
curl -L -o san-holo \
  https://github.com/lhaig/SanHolo/releases/download/vX.Y.Z/san-holo-vX.Y.Z-linux-amd64
chmod +x san-holo

# Optional: move it onto PATH
sudo mv san-holo /usr/local/bin/
```

**macOS gatekeeper**: the binary isn't signed yet, so the first launch will
warn that it's from an unidentified developer. Right-click the file → Open
once to get past the warning, or `xattr -d com.apple.quarantine san-holo`.

### Windows

Download the `.exe` and run from PowerShell. Windows SmartScreen may warn
about an unrecognized app — click "More info → Run anyway".

### Verify the install

```sh
san-holo version
# san-holo v0.4.0 (commit abcdef1)
```

## First run

1. **Register a replicant account on the game** (skip if you already
   have one):
   ```sh
   san-holo register -email you@example.com -name "Your Replicant Name" -tz UTC
   ```
   The game emails you a verification token. Then:
   ```sh
   san-holo verify -token <token-from-email>
   ```
   Credentials are saved to `~/.config/san-holo/credentials.json` (Linux),
   `~/Library/Application Support/san-holo/credentials.json` (macOS), or
   `%APPDATA%\san-holo\credentials.json` (Windows).

   **Alternative — env vars** (handy for CI, headless servers, or
   testing): if you already have an API token from elsewhere, skip the
   register/verify flow and just export:
   ```sh
   export SAN_HOLO_TOKEN="<your-api-token>"
   export SAN_HOLO_CODE="<your-replicant-code>"
   ```
   Env vars override the on-disk credentials file.

2. **Start the dashboard** (safe by default — bot is stopped, dry-run on):
   ```sh
   san-holo serve
   ```
   Open http://localhost:8080. The cockpit opens on a live map of your
   system — click a body to inspect it, use the top menu bar for the
   fleet / build / intel panels, check **Needs you** for anything waiting
   on you, and watch the **Activity** panel beside the map for the latest
   log line and copilot advice.

3. **Go live** when you're comfortable: open **Engine ▾** in the top bar,
   click **Start**, then **Go live** — or start with `san-holo serve -live`.

## Keep it running (Linux, systemd)

`san-holo serve` in a terminal stops when the terminal does. To have the bot
keep playing across logout and reboot, install it as a **systemd user
service**. A ready-made unit ships in the repository at
`deploy/sanholo.service`:

```sh
install -D -m644 deploy/sanholo.service ~/.config/systemd/user/sanholo.service
systemctl --user daemon-reload
systemctl --user enable --now sanholo
loginctl enable-linger "$USER"      # keep running when you are logged out
```

Edit `WorkingDirectory` and `ExecStart` in the unit to match where you put the
binary, then:

```sh
systemctl --user status sanholo
journalctl --user -u sanholo -f     # live log
systemctl --user restart sanholo    # after upgrading the binary
```

Two things worth knowing:

- **systemd does not inherit your shell environment.** If your API token only
  exists as an `export SAN_HOLO_TOKEN=...` in your shell profile, the service
  will start and then fail with "no API token found". Either complete the
  `register` / `verify` flow so the token is written to
  `~/.config/san-holo/credentials.json`, or add an explicit `Environment=` line
  to the unit.
- **Run only one live instance per account.** If the service is running, do not
  also start `san-holo serve -live` by hand.

## CLI reference

```sh
san-holo version          # version + commit
san-holo health           # API connectivity check
san-holo me               # show authenticated account
san-holo serve            # run cockpit at http://localhost:8080
san-holo catalog ...      # query the offline discovery catalog
                          # (unsalvaged | best <res> | events | history | depleted)
san-holo clone            # dry-run cost report for cloning a second replicant
san-holo clone -confirm   # execute the clone bootstrap prints
```

Run `san-holo` with no args for the full flag list.

## API compatibility check

SanHolo asynchronously compares its bundled API contract with the official
Replicant Space OpenAPI document on startup when due and every 24 hours while
the process remains running. The cockpit header reports `API OK`, `API CHANGED`,
or `API CHECK?`; automation continues in every state. Completed attempts are
throttled for 24 hours even if the local catalog cannot persist the result.

The bundled v3.3 client includes generated types for listing research and
conducting a research topic. Conducting research grants an achievement and can
unlock blueprints, so SanHolo exposes no game action, bot strategy, cockpit
route, or control for that mutation; it remains operator-directed.

The compatibility request is a public `GET` with a three-second timeout. It
sends no token, cookie, account identifier, game state, or telemetry, and it
never downloads an update or executes a command. Disable it with
`SAN_HOLO_DISABLE_API_COMPAT_CHECK=1` or
`serve -disable-api-compat-check`.

The cockpit header is the compatibility status for the running binary:
`API OK` means the bundled contract matches the official document, `API
CHANGED` means the upstream contract differs, and `API CHECK?` means the
check could not complete. Treat `API CHANGED` and `API CHECK?` as upgrade
signals: read the [Replicant Space changelog](https://replicant.space/changelog/)
before enabling new controls. This monitor never updates the binary or tags a
release.

## Upgrades, backups, and safe diagnostics

Before replacing a binary, stop the running `serve -live` process. Only one
live instance may use an account at a time; two live instances will issue
competing actions. Make a copy of the local SQLite catalog while the process is
stopped, then start the new binary and run a read-only catalog query to verify
that it opens and completes its idempotent migrations:

```sh
# Set this to the catalog path for your platform, or pass -catalog explicitly.
CATALOG="$HOME/.config/san-holo/catalog.db"
cp "$CATALOG" "$CATALOG.before-upgrade.bak"
san-holo catalog history -catalog "$CATALOG"
san-holo version
```

On macOS the default path is under
`~/Library/Application Support/san-holo/catalog.db`; on Windows it is under
`%AppData%\san-holo\catalog.db`. If the migration or smoke query fails,
keep the process stopped and restore the backup before investigating. Do not
copy a live WAL-mode database; stop SanHolo first, or use SQLite's `.backup`
command while no writer is running. Backups contain discovery and campaign
state, not a replacement for the API account.

For support, share only a sanitized summary. The following deliberately omits
account identity, credentials, locations, inventory, messages, and activity
text while retaining useful health and compatibility signals:

```sh
curl -s http://localhost:8080/state.json | jq '{bot: {running: .Bot.Running, dry_run: .Bot.DryRun, last_error: (.Bot.LastError // "")}, api: {connected: .Connected, compatibility: .APICompatibility.Status, compatibility_error: (.APICompatibility.Error // "")}, system: {planets_scanned: .System.PlanetsScanned, planets_total: .System.PlanetsTotal}}'
san-holo version
```

Never include `SAN_HOLO_TOKEN`, `credentials.json`, raw `/state.json`,
`/catalog.json`, email addresses, replicant codes, or unredacted logs in an
issue. Releases are operator-only: users should install the published binary;
only the operator creates release tags and publishes artifacts.

## Optional version checks

SanHolo can call a self-hosted update endpoint, but it is off unless you
configure one:

```sh
export SAN_HOLO_VERSION_CHECK_URL="https://your-server.example/san-holo/version-check"
san-holo serve -version-check-url "$SAN_HOLO_VERSION_CHECK_URL"
```

Opt out with `SAN_HOLO_DISABLE_VERSION_CHECK=1` or
`serve -disable-version-check`. The request is a `GET` with only:
`version`, `commit`, `os`, `arch`, `command`, and `channel`. It never sends API
tokens, email, account name, replicant code, location, inventory, fleet data,
logs, or strategy state. Checks use a one-second timeout, fail open, and `serve`
records the last check in the local catalog so it runs at most once per day.

Endpoint response:

```json
{
  "latest_version": "v0.4.0",
  "minimum_supported_version": "v0.2.0",
  "release_url": "https://github.com/lhaig/SanHolo/releases/latest",
  "message": ""
}
```

`san-holo version` stays local and immediate; update notices surface in the
cockpit when `serve` performs the configured daily check.

## Alerts

`serve` can push alerts to services you already read. Set any of these in
the environment (never in a file you commit, since each URL is a secret):

| Variable | Sink |
|---|---|
| `SAN_HOLO_ALERT_NTFY_URL` | an [ntfy](https://ntfy.sh) topic URL |
| `SAN_HOLO_ALERT_DISCORD_URL` | a Discord channel webhook URL |
| `SAN_HOLO_ALERT_WEBHOOK_URL` | any URL; receives `{"kind","severity","text","key","at"}` as JSON |

With none set, alerts are off and nothing is logged about them.

Sink URLs must be `https://` and public. If your sink is on your own
network (a self-hosted ntfy on `192.168.x.x`, say), set
`SAN_HOLO_ALERT_ALLOW_PRIVATE=1`; that allows loopback and private
addresses, and only those may use plain `http://`. Link-local and cloud
metadata addresses are refused regardless, and redirects are never
followed. A bad setting stops `serve` at startup with a message naming the
variable; the URL itself is never printed or logged.

You are alerted when:

- an asteroid is on course for one of your locations, with the time to
  impact (repeated every 30 minutes until it is diverted or lands);
- a location event opens in your current system (the game publishes no
  expiry time, so this fires when the event is open, not as it closes);
- devices go `paused` and stop working (one alert listing them, not one
  per device);
- the bot needs attention, for example when it is stuck repeating a failing
  action;
- the game pushes an event to SanHolo's `/webhook` endpoint.

Alerts are rate-limited: the same condition repeats at most once per
cooldown, a condition that stays clear for two checks and then comes back
alerts again straight away, and an hourly cap stops a flood with a single
notice. Critical alerts (an inbound asteroid) keep some room past the cap,
and webhook events have a small hourly budget of their own.

## Safety defaults

- Bot starts **stopped** and in **dry-run** — you have to explicitly turn
  it on. Until then it polls your account, populates the dashboard, and
  does not send any state-changing commands.
- Every action is logged to the in-cockpit Activity panel before it fires.
- The copilot's advice is **suggestive by default** — you decide whether
  to act on it. You can also set an explicit directive from the cockpit
  (focus_resource / build / explore / relocate / scout /
  observatory-expedition / ferry-miners / recall-miners / trade_expedition /
  ark-contribute) to
  steer the bot, or clear it. `relocate` is the only way to authorise an
  interstellar hop — the bot recalls + stows its fleet first and will not
  abandon a system otherwise. `scout` (param: a target star like `MENKENTAR`)
  sends the home vessel on a scan-and-return run to a promising nearby
  unexplored star to reveal its resources, then ALWAYS brings it home — your
  base never moves.
  `observatory-expedition` (param: target star) uses a surge carrier to move
  the galactic observatory to an already-anchored frontier, prospect, and bring
  the observatory back.
  Autonomous scouting ships disabled; enable the `scout` strategy from the
  cockpit to let the bot pick and visit nearby stars on its own.
  `ferry-miners` (param: a target belt designation like `ACHURD-8-L4`) ships
  idle mining drones out to a spoke belt via a surge-capable carrier, where
  that system's AMI controller adopts and runs them — the hub-and-spoke
  restaff move. Each controller adopts at most three miners, so the run
  carries only as many as the spoke's controllers can still take, leaves the
  spoke as soon as they are full instead of waiting beside miners nobody will
  adopt, and puts any miner it no longer needs back down at home.
  `trade_expedition` (param: a shop's controller code and location, like
  `043CA0A9@SOL-3-1`) flies a payment basket to a player shop in another
  system, buys up to five useful worker drones, and comes home. Add
  `#device,device` (for example `043CA0A9@SOL-3-1#shield_generator,sensor_array`)
  to buy exactly one of each named device instead, such as a device whose
  blueprint you want. Bought devices cannot leave the shop's system on their
  own, so the run stows them in your vessel's hold (keeping a slot for
  the replicant matrix), carries them home, and decommissions them there so
  your home autofactory learns their blueprints. Only devices with the stow
  feature fit in the hold: cruise-only devices such as `comm_satellite` and
  `aquifer_tap` need a carrier with attach, so the run will not buy them,
  and if one is already bought it asks for help, names it, and ends the
  run. It only ever decommissions
  devices of the named types that the run bought or picked up idle at that
  shop, and it needs an autofactory deployed in your home system. If a
  restart makes it lose track of a device it carried home, it leaves that
  device alone and asks for help, naming it. Arming the
  same directive again fetches purchases an earlier run left at the shop.
  Named devices whose blueprint you already know are skipped. The run ends
  once each one is bought and delivered, or the shop has none left.
  `ark-contribute` (param: `device_type:count`, for example
  `comm_satellite:6`, count 1-9) gives devices to the space station at the
  Exodus Ark's current stop, which the cockpit's Fleet → Megastructures tab
  shows with its deadline and your contribution so far. It prints up to that
  many (never more than the station still needs) at an autofactory in your
  home system, attaches them to an idle surge carrier there, checks the
  flight lands before the deadline, flies the carrier to the station,
  contributes, and sends the carrier home. Your vessel stays put. Only
  devices printed by that run are ever contributed: each print is tagged
  for the run, and it prints one device first and stops if that device
  does not come back tagged. It needs a relay or hub that gives command in
  the stop's system, and refuses to start without one. It only runs when
  you set it yourself (the copilot cannot, and copilot advice never
  replaces it), and anything unexpected, such as a closed stop, a carrier
  out of control range on arrival or a rejected device, stops the run,
  clears the directive and asks for help. Clearing the directive yourself
  while the run's devices still exist also raises a help request.
- **Running a second replicant.** If you create another replicant on the same
  account, keep the bot's hands off its vessel and kit by reserving the
  vessel: `curl -X POST -d 'code=<8-hex device code>&reserve=true'
  http://localhost:8080/actions/reserve-device` (`reserve=false` releases it).
  The bot then ignores that device and everything stowed in or attached to
  it; the reservation survives restarts. Devices whose owner is another
  replicant are ignored automatically. To see the other replicant's status and
  location in the cockpit header, record it once with
  `curl -X POST -d 'code=<replicant code>' http://localhost:8080/actions/other-replicant`
  (add `&remove=true` to stop). The header names up to two reserved devices;
  past that it shows a count, and hovering lists them.
- **Sending the other replicant on an event errand.** SanHolo can finish a
  location-event delivery for the other replicant without you watching:
  once it arrives in the target system it deploys a stowed beacon straight
  away, wherever in the system it arrived (an unanchored vessel returns home
  after about 45 minutes, so it keeps retrying for 35 minutes, moving to the
  spare beacon if one is faulty and on to the target if the arrival point
  refuses it). It then moves to the target, scans, reads
  the events at the locations you list plus the target's moons (scanning
  again for up to an hour if none show yet), and if an event's criteria name
  your payload's device type it moves there, deploys the payload and resolves
  the event. It only ever sends `deploy` to a device and never travels outside
  the target system. Anything unexpected (criteria that do not match, the
  replicant leaving the system, a failed resolve, repeated errors) stops the
  errand and raises a needs-help alert with the details. Arm it with:

  ```sh
  curl -X POST http://localhost:8080/actions/errand \
    -d 'replicant=<replicant code>' -d 'target=<STAR-N>' \
    -d 'event_locations=<STAR-N>,<STAR-N-M>' \
    -d 'beacons=<beacon code>,<spare beacon code>' \
    -d 'payload=<device code>' -d 'payload_type=vector_charge'
  ```

  The errand survives restarts and shows its step (and the arrival time while
  it waits) beside the replicant in the cockpit header. Posting again replaces
  it and clears a stopped errand; add `-d step=<step>` to resume at a given
  step, or post `clear=true` to remove it.

## Troubleshooting

- **`api.replicant.space` connection refused** — check your network /
  firewall.
- **"Invalid API token"** — re-run `san-holo recover -email <yours>` to
  get a fresh token by email, then `san-holo verify -token <token>`.
  Or temporarily set `SAN_HOLO_TOKEN` in your shell to test a specific
  token without touching the credentials file.
- **"no API token; run `san-holo register` or set SAN_HOLO_TOKEN"** —
  the credentials file is missing or your env vars aren't set. Run the
  register flow, or restore the file from a backup (see paths under
  "First run" → step 1).
- **macOS "binary is damaged" prompt** — that's the unsigned-binary
  warning; see Install → macOS gatekeeper above.
- **Cockpit shows the map but no live data** — check the status readout on
  the right of the top bar: `LIVE` / `STALE` / `OFFLINE`. `OFFLINE` means the
  API poll is failing; check the binary's stdout for the actual error.
- **Bot is running but my XP / inventory isn't changing** — usually
  one of three things:
  1. **Belt depleted, waiting on the survey AMI** to open new mining
     sites. Expand the bottom **ticker** to see recent activity — if you
     see survey drones cycling "searching", that's the AMI working. To
     force a fresh scan, click a planet or belt on the map and use the
     **Scan** button in its inspector.
  2. **Dry-run is still on.** The top bar shows a `dry-run` badge — if it's
     there, the bot is choosing actions but not sending them. Open
     **Engine ▾** and click **Go live**.
  3. **A device got paused.** Open **Fleet ▾ → Devices** — paused devices
     show in the fleet roster (and AMI status below it). Resume the device
     from the API docs (auto-resume from the cockpit is on the roadmap).

## Roadmap

- Code-signed macOS/Windows binaries
- `.deb` / `.rpm` / Homebrew tap / Scoop bucket
- Source code release once API stabilises
- Native window app (likely Wails)

## Issues and feedback

Open an issue here on the SanHolo repo. Include:
- OS + architecture
- Output of `san-holo --version`
- Reproduction steps and what the cockpit showed when things went wrong

## License

(c) 2026 Lance Haig. All rights reserved. Free to download and use for
personal play of replicant.space; no warranty of any kind; redistribution
prohibited. Source publication planned — terms may shift to a permissive
license at that point.

## Acknowledgements

Built against the [replicant.space](https://replicant.space) API.
SanHolo is an unofficial third-party client; not affiliated with or
endorsed by replicant.space.
