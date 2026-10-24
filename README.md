
# SanHolo

An autonomous client for [replicant.space](https://replicant.space) — plays
the game on your behalf and gives you a live mission-control dashboard
to watch what it's doing.

> **Status**: pre-release. SanHolo is distributed as pre-built binaries and a
> container image. Run with `--dry-run` (the default) until you trust it.

## What it does

- **Mines** the belt resources your current build goal actually needs, and
  adapts as resources deplete; opens new sites with survey drones; consolidates
  loot home with transport drones. Salvage is mined rares first, then the
  minor resources home is not in surplus of, and transports load rares first.
  Any home-system stockpile holding rares is hauled home even when no salvage
  report names it, and in other systems a free transport carries stranded
  rares to the stop your cross-system haul names there (else that system's
  best-stocked belt), where the freighter picks them up. A stockpile an
  autofactory or system hub prints from is left alone, transports pause while
  a relocation, colonise recall or hop involves their system, a refused
  deposit is retried after ten minutes, and a transport stranded in another
  system is never sent home, since it cannot cross systems. When a belt is depleted and miners sit idle,
  the Mining panel calls out that blocker directly. No free drone, on the
  ship's belt or at a remote supply source, is started or retargeted onto a
  resource home already holds more than 50,000 of, and a free drone found
  mining one is moved to something home is short of. Once drones are off a
  resource it stays off until home falls below 40,000, so a stockpile
  hovering at the line doesn't swap drones back and forth; among the rest drones
  take whatever home holds least of (usually rares), and they stay idle rather
  than spend a scarce site on surplus.
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
- **Stations a maintenance drone wherever you mine.** A system with your
  mining drones and no maintenance drone gets one, carried in the vessel's
  hold from a system that holds two or more (a system's last one never
  leaves). It rides along with a mining ferry already going there, or, when
  that system's miners are worn down to 20 percent capacity or less, the idle
  vessel makes the trip itself while no directive is set, deploys the drone,
  sets it patrolling and flies home. Systems with the most worn miners go
  first. Only when no spare exists anywhere does it queue one maintenance
  drone at home, keeping home rares above the reserve and carbon and
  structural above 1,000 and 2,000. At most four drones moved, two trips and
  two prints a day; a trip that keeps failing or runs past 12 hours ends and
  asks for help. Turn the `maintenance-station` strategy off to place them
  yourself.
- **Cleans up stranded extractor drones** conservatively: when a live scan proves
  the current non-home system has no asteroid belt, SanHolo can decommission
  idle mining/survey drones there in place while leaving comms anchors and all
  protected device types alone. The Fleet posture panel calls this out before
  or while the cleanup runs.
- **Brings miners home on request.** Set the `recall-miners` directive with
  a source belt (for example `NEARBY-BELT-1`) and SanHolo releases that
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
  system can be commanded, and you can see it in the Bridge's jobs panel
  alongside any directive you set yourself.
- **Keeps every system commandable.** A beacon lets you *see* a system; only a
  relay or hub lets you *command* one. SanHolo tracks both per star, and when a
  system holds devices you cannot reach, it builds a comms kit, carries it
  there, deploys it and comes home. Devices stranded without a relay cannot be
  recalled at all — the server refuses the command — so this is the only way to
  get them back. The cockpit's Comms Network graph reads the same per-star model
  the bot plans with, so a system it marks commandable is one the bot can
  actually command, not merely one the ship happens to be visiting.
- **Keeps your remote system hubs stocked.** A hub's shield lasts 7 days,
  after which it loses 10% integrity a day unless a maintenance drone in its
  system has carbon and structural beside it to repair it with. For every
  hub of yours in another system SanHolo reads that stockpile every quarter
  hour, and when it covers less than two days of upkeep the idle cargo
  freighter collects one load at home (in the hub's own upkeep ratio when the
  game reports it, else 83 carbon : 143 structural), delivers it and flies
  home. It never takes home carbon below 1,000 or structural below 2,000,
  only starts with the freighter idle and empty at home while no
  cross-system haul or trade expedition is using it (a `hub-deploy` run only
  takes it for its final drop, which waits for an upkeep trip), makes at most one trip per hub a day and three a week, and asks
  for help when commands cannot reach the hub, when home is too low, or when
  the weekly cap is reached. If home is at its floor for a resource the hub
  itself is short of, it sends no short load and asks for help naming that
  resource. While a cross-system haul or trade expedition holds the freighter
  and a hub with its shield down is under the line, it asks for help. A
  refused freighter command waits 10 minutes before the same step is tried
  again, and three refused loads in six hours stop new trips and ask for
  help. A trip that cannot deliver within a day brings its load home. The cockpit's Comms Network menu lists each remote hub with
  its integrity, days of shield left, stockpile and when it was read, refill
  line, days of cover, the freighter's trip and trips this week. Turn the `hub-upkeep` strategy off to feed your hubs yourself.
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
  spend scarce rares or volatiles. A routine print paid from home's
  stockpile waits if it would leave home with fewer than 150 rares: unattended
  fleet growth, extra survey drones, and the beacon dropped after an asteroid
  is diverted. Prints paid from a stockpile elsewhere are not held. Propulsors
  printed against an active asteroid threat are never held.
- **Re-arms logistics** when a real build goal is short on resources and a
  cached remote anchored stockpile looks useful, preferring the best haul
  source instead of waiting only for the last manually-armed supply line. When
  discovery pressure is low, the Progress view can now promote that haul move
  directly as the best near-term climb. A `cross-system-haul` directive can
  name the resources to fetch after a `#`, for example
  `NEARBY-BELT-1,OTHER-BELT-1#rares`: the freighter then collects only
  those resources, moves on from a source once it holds none of them, and
  re-arming the line keeps the filter. A filtered line is re-armed only over
  its sources in systems with a comms anchor, once those hold at least 100 of
  the named resources summed across them, and is checked at most every 30
  minutes; the standing line keeps every source, so one that gains an anchor
  rejoins on the next re-arm.
- **Diverts incoming asteroids** before impact: prints propulsors,
  deploys + activates them at the asteroid, drops a beacon at the
  impact target once diverted. Active in-system asteroids at or above
  50% impact likelihood are handled early instead of waiting for a late
  high-certainty scramble. Once a rock in your home system is diverted,
  idle propulsors left on it cruise back to your home base one at a time,
  ready for the next threat, instead of sitting on the finished asteroid.
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
- **The Bridge: a living star map as the home screen.** http://localhost:8080
  opens on a full-screen 3D map of the space around your home system: your
  systems glow (home in white, yours in cyan, ones another player has claimed
  in purple), hub rings mark the systems you hold with a hub, an amber pulse
  marks the system your current directive is working on, and lines show the
  real comms links between your relays and hubs. Each system is labelled with
  what matters there (devices at home, miners working, claimed or unlinked),
  hovering shows a card and clicking opens the system's overview. Drag to
  rotate, scroll or pinch to zoom. Zoom out to the galaxy to see every
  catalogued star, where your other replicant is heading with its arrival
  time, and the Ark's current stop; zoom into a system to see its planets
  (estimated ones drawn faint until scanned), its belt with sites and how
  much is left, and where your devices are ("78 miners · 41 working").
  Ships between systems show as dashed routes with an ETA, and an incoming
  asteroid shows as a red marker with a live impact countdown; click it for
  its card. The system panel has an Overview and a Devices tab. **List**
  (or `L`) shows the same map as tables: your systems, nearby stars with
  whether a relay there would link, transits, threats and discoveries. In a
  browser that cannot draw 3D the list opens by itself. Each system also
  carries a small six-colour **resource ring**: one segment per resource,
  brighter the more of it the sites your surveys opened still hold (summed
  site quantities, never the scan's "scarce" or "rich" words). The game
  states a site's size only in the message announcing it, so SanHolo keeps
  those amounts once seen; where a site is open but its size was never
  announced, the ring follows what the open sites still hold and the card
  reads "Open sites: 3 · structural, conductive". SanHolo reads each known
  system's belt once a day, spread out, so a system with nothing open says
  "belt known, sites not opened", "no asteroid belt" when its scan found
  none, or "not surveyed yet" when nothing is known. All of this survives a
  restart. A hub ring
  turns amber when the hub has under two days of carbon and structural
  beside it and red when it is inactive or under one day. Something that
  happened in the last 15 minutes (a site worn out, a search finished, a
  repair done, a hub going inactive, an asteroid) shows as a short **ping**
  on its system, a still dot if you prefer reduced motion. A running trip
  (hub deploy, scout, ferry miners, hub upkeep, fetch) draws its **route**:
  solid behind the ship or freighter, dashed ahead, labelled with the job
  and its ETA, gone when the job ends. The hover card
  adds the hub's integrity and days of cover, the stock there, the three
  biggest resources, what the fleet is doing (mining, searching, hauling,
  idle) and the latest pings, and the list's Systems tab has the same
  columns. The **map key** button beside History explains the rings and
  switches the resources, hubs, activity and events layers on and off (your
  browser remembers the choice). The top bar adds a **standing chip** for
  each leaderboard you are on ("Distance #6") or within 30% of rank 10
  ("XP 54,316 / 62,406"); the board the Climb goal works on also shows
  its gap, aimed rank and ETA ("climb: 113 to #10, about 12 d"); click it
  for Jobs > Progress. A top bar shows
  each resource's home stock and net change over the last hour (`--` when
  there is no reading yet, never a made-up 0), an amber ring when a resource
  is under the rares reserve or short for the next build, and an **autopilot
  pill** (AUTOPILOT, DRY RUN or STOPPED, with a red dot when the bot asks for
  help) that opens Start, Stop, dry run, the current directive with Clear,
  and Dismiss for a help request. Alerts at the bottom left list what needs
  you, in order: an inbound asteroid with its countdown, a blocked job, the
  bot's help request, a stopped or dry-run bot, unread mail, an idle pool
  of ten or more devices that has a fix, a system another player claimed
  while your devices are still in it, devices out of comms reach, and
  paused automation. Each card carries one button that does the fix: Show
  it on the map, Open the job, Ferry the miners or Bring them home (an
  order preview that says what it replaces before you confirm), or
  Dismiss, which hides that card for an hour. A claimed system offers one
  button per kind of device to bring home: the miners, and each device
  type that crosses systems only on a carrier (belt surveyors, propulsors),
  fetched by the surge carrier; relays, beacons and hubs stay. A blocked
  job offers Resume and a stalled directive offers Clear, each through its
  preview. A paused AMI controller gets its own card with **Resume <type>
  at <location>** (a preview, then the controller's current directive is
  sent again); after two failed resumes in 24 hours the card says so and
  offers **Recall the controller** instead. The list stays short: two or
  more alerts of one kind become one card with their count and **Show
  all**, at most three cards show at once, and quiet items (unread mail,
  recently paused records) wait in a small "N quiet" tray that opens the
  full list. The unread card has **Mark all read**, and a **Progress** link
  when an achievement is among them. A paused record nobody has touched
  for 72 hours counts as stale: it leaves the alerts and shows in Jobs as
  "N stale, cleared", whose drawer lists them by kind. The Jobs panel lists every
  job the bot is running or holding (the errand, a scout, a hub delivery,
  a hub upkeep trip...) with its step, what it waits for and an ETA, or
  "ETA unknown" when there is none, plus the vessel's own trip and print,
  every autofactory that is printing (with its queue and when the print
  completes), paused records folded into one line, and how many miners are
  working; click a card to see every step, when it started, its deadline,
  the last error and Cancel where cancel exists; the Jobs button carries
  the count. Systems, the Fleet table and a device's inspector say why idle
  devices are idle and what would fix it, and the map marks systems with
  fixable idle devices in amber. Devices crossing between systems show as
  dashed routes with their arrival time; click one for its inspector. A
  green dot in the top bar means the data is live; it turns
  amber when the poller stops refreshing and red when the game cannot be
  reached. Labels never overlap: the home system, hubs, jobs and claims win
  a crowded spot, and the rest appear as you zoom in. Keys: `G F P O C`
  for the screens, `/` or `Ctrl+K` to search systems and screens, `L` for
  the list, `1`/`2`/`3` for galaxy, our space and the open system, arrow
  keys to step through systems (or device groups inside a system) with
  `Enter` to open, `Esc` to close the top panel and then back out, `?` for
  the full list. **Orders preview before they act.** The system panel's
  Orders tab lists what you can do there (claim it with a hub, send idle
  miners, bring the miners home, haul the stock home, link it with a
  relay chain, deliver a relay, rescue stranded transports, fly the vessel
  there, make it the home base, scan it, resolve an event, queue a print at
  an autofactory, print on the vessel; a star you hold nothing in offers
  scout, link and move base), and every order first shows a preview: its
  cost per resource from the blueprint against the stock that pays, what is
  left after and whether that dips under your reserve floors, when a
  shortfall would close at the current rate, a print's time including the
  queue ahead, what it waits for (a hub to be printed, the vessel held by
  another job), and whether it queues behind the running job, runs beside
  it or replaces it; a note says when the bot is stopped or in dry run.
  Confirm sends it; Cancel does nothing. Queueing a print carries a
  one-time confirm token, so a double click queues once, and no autofactory
  takes more than three queue orders in ten seconds. "Bring every device
  home" is shown greyed out with the reason (no order moves surveyors and
  other non-miners home yet; bring the miners home instead). The palette
  (`/` or `Ctrl+K`) now finds every catalogued star by name or designation
  as well as your systems, screens and zoom levels, and lists the orders
  that apply right now ("Claim NEARBY with a hub", "Bring the miners
  home from SOL"); choosing one opens its preview. **Goals** (settings or
  the palette) edits the standing rules: the resource reserve floors, the
  claim-systems caps and order, hub supply cover, the miners and
  recycling limits, and bringing stranded devices home (on by default:
  belt surveyors and propulsors left in a claimed system, or one with no
  work for them, come home on the surge carrier, one trip at a time), and
  opening new worlds (on by default: where a belt you can command has a
  free site slot and its open sites lack something you need, a survey
  drone already there, one elsewhere in that system, or a spare from home
  carried in your vessel starts searching; your hub systems first, then
  relay-reached ones, home while the vessel is away; one run at a time, at
  most two a day, nothing printed, asteroid defence first, a run that
  takes over 6 hours or has 3 commands refused flies home and asks for
  help, the two drones kept for exploring stay home, never a drone a
  controller adopted, you
  hold, or another job, reservation or replicant owns; once sites open,
  Keep miners busy sends miners there), each
  saved after a check, with a pause switch for all
  goal-made jobs. On a phone the rail becomes a tab bar, alerts fold
  into one bar with their count and the most urgent one (tap to open), jobs
  fold into a one-line bar above the tabs, the map opens framed on all of
  your systems, and every close button is a large round X. **Fleet,
  Build and Comms open on the Bridge** (`F`, `P`, `C`, or the rail): Fleet
  has an Overview (counts, posture, what is idle and why), the Roster (every
  device with its commands; a code opens its inspector), Mining, and
  Reserved (devices kept away from the bot, other replicants with their
  errand); Build has the Factories with their queues (Remove, or queue a
  print per factory), Blueprints ("can build now", "next"), the Catalogue,
  Unlocks and Megastructures; Comms has the Inbox (decisions waiting for
  your approval first, then your messages with mark read), BobNet with
  send, the Network and an Activity log. Every order there previews before
  it acts, including queueing a print. The Comms button shows your unread
  count, and a new activity line shows briefly at the top. Old `?open=`
  links and `/fleet`, `/build`, `/inbox`, `/messages` land on these
  screens. **Jobs** (`O`) has your Goals (the current objective, every
  goal with its settings and the pause switch, a directive order for every
  directive kind plus Clear, each with its preview, the strategy switches
  and the copilot), Progress (growth, rankings, achievements) and History
  (what finished, failed or was cancelled in the last two weeks).
  **Settings** (the gear beside SanHolo top left, then "All settings")
  has your name, language and home base, a high-contrast and larger-text
  switch this browser remembers, where your ship and home are (with a
  line of API requests per minute for the poller, the bot, the errand and
  the rest, how much of the API's one-minute window is left and how many
  requests it refused), and the data links. Everything on one token shares
  that window: requests wait for it rather than run into a refusal, the
  errand gives way to the bot when it runs low, and a refused request is
  never retried at once; the next one waits out the time the API asks for, at most a minute. Clicking SanHolo itself takes you back to the plain Bridge
  with every screen closed. **History** under the map (or `H`) replays your space over
  the last day, 3 days or week: drag the slider or press play, then Live
  to return; time SanHolo did not record says "no data" rather than
  showing your current state. Without WebGL the Bridge opens the list,
  with a Retry button, instead of a blank screen.
- **A cockpit that starts you off.** On a fresh install the Bridge's alerts
  say what is happening (connecting, bot stopped, running in dry-run) and
  the one next step, with a **Start** or **Go live** button; that alert
  goes once the bot is running live. Alerts also carry a help request from
  the bot (with **Dismiss**, for one that outlived the job that raised it;
  if the problem is still there, the bot raises it again), an incoming
  asteroid with its impact time, a stalled directive and unread messages,
  each with the button that deals with it. Jobs > Goals states the one
  thing the bot is working towards, and the top bar shows each resource's
  home stockpile with its **net** change over the last hour (net because
  printing spends the same stockpile; with no reading from the start of the
  hour it says so rather than guessing). Empty screens say what fills them,
  in English, German and French; game data stays as the game sent it
  (device types, designations, resource names, API errors, log lines and
  message bodies). Resources and devices carry icons beside their names.
- **Galaxy map at true coordinates.** Zoom out from your space and the
  galaxy level draws every catalogued star (tens of thousands) where it
  actually is, with your systems, your home and your ship marked, including
  its course while travelling. The **List** button beside the levels is the
  plain-table equivalent. **History** (or `H`) replays your space over the
  last 24 hours, 3 days or 7 days from SanHolo's own recorded history:
  drag the slider or press play, then **Live** to return. Time SanHolo did
  not record says "no data", never your last known state, and times before
  presence history began show your ship's track only.
- **Classic cockpit retired.** The older cockpit at
  http://localhost:8080/classic is gone; that address and its
  `/?open=<menu>&tab=<tab>` links now open the Bridge on the screen that
  carries the same content (Engine → Jobs > Goals, Intel's progress,
  rankings and achievements → Jobs > Progress, System → Settings).
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
  steps. The current step is the objective in Jobs > Goals, with contextual
  guidance beside the relevant Fleet and Comms screens. If the tutorial
  service is temporarily unavailable, the rest of the dashboard and bot
  continue to operate normally.
  Accounts created before the game added tutorials never get one started,
  so the game reports them stuck on the first step for good. When the game
  shows no tutorial progress at all but the account has already earned
  achievements, SanHolo hides the step and its hints.
- **Safe tutorial actions**: Jobs > Goals can offer a one-click action only
  for the current `bootstrap` / `scan_system` step. That action reuses the
  existing validated scan path and requires a fresh, matching server step;
  arbitrary fields, stale steps, destructive commands, travel, printing, and
  strategic actions are rejected. Every other tutorial step is guidance-only.
  Tutorial guidance never starts the bot or changes dry-run/live state: SanHolo
  remains stopped and in dry-run until you explicitly change those controls.
  Every screen and tab has a shareable link, `/?screen=<screen>&tab=<tab>`
  (for example `/?screen=fleet&tab=roster`); the address follows the open
  tab, so a reload keeps it. Older `/?open=<menu>` links and `/inbox`,
  `/fleet`, `/intel`, `/build` and `/messages` land on the matching screen.
  **Play by hand.** The **Fleet → Roster** tab and every device's inspector
  list the commands that device accepts now — deploy, mine, send to a
  location, recall, stow, stop and **Decommission** — and say why one is
  off. Each opens a preview first (what it does, the flight time of a trip,
  "the controller will re-task it" for a drone an AMI adopted); only its
  Confirm sends anything. A device you command this way is **held by hand
  for 2 hours**: no strategy, goal or job touches it until the hold ends,
  you press **Give back to bot**, or you extend it with **Hold 2 h more**.
  Holds survive a restart. **Decommission** shows the refund (about 60% of
  the print cost, from the blueprint, landing where the device stands) and
  only goes ahead once you type the device code; hubs, relays, beacons,
  ships, AMI controllers and autofactories can never be decommissioned.
  Right-click (or long-press on a phone) a system, belt or device on the
  map for a short menu of its orders; a belt's rows in the system drawer
  have **Mine here**, which picks a free drone and deploys and starts it.
  With the bot stopped the status pill reads **MANUAL**: you have the con,
  your orders run at once, and your holds still apply when the bot starts.
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
  cockpit — handy for hub-and-spoke play as your galaxy footprint grows.
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
- **Game Director readout** — Jobs > Goals leads with the current
  objective: the directive with its phase and how long since it last moved
  (flagged when it has stalled for half an hour, with its phase), otherwise
  the official tutorial's current step, otherwise what the bot is doing,
  with its blocker and next step. A help request shows its reason there and
  as an alert with **Dismiss**. Long lines end at a whole word, with the
  full text on hover. The strategy list in Jobs > Goals marks the most
  recently firing strategy and shows the last fire time per strategy, so
  you can tell what is actually hot without reading the full activity feed.
  In a scanned system with no asteroid belt, the exploration loop keeps
  survey drones free from non-belt survey AMIs and can print a local survey
  drone when needed so the bot can keep scanning planets instead of
  silently stalling.

## Download

Pre-built binaries are on the [Releases page](https://github.com/TydeWhatMay/SanHolo/releases/latest).
Each release is also published as a container image,
`ghcr.io/tydewhatmay/sanholo:<version>` (linux amd64 and arm64, non-root,
no shell). It is built for the hosted service and starts in hosted mode by
default; to run it yourself, override the command, for example
`docker run ghcr.io/tydewhatmay/sanholo:<version> serve -addr 127.0.0.1:8080`
with your credentials passed in through `SAN_HOLO_TOKEN` and
`SAN_HOLO_CODE`.

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
  https://github.com/TydeWhatMay/SanHolo/releases/download/vX.Y.Z/san-holo-vX.Y.Z-linux-amd64
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
   Open http://localhost:8080. The Bridge opens on a live 3D map of your
   space — hover or click a system, check the alerts for anything waiting
   on you, and use the rail on the left (or `F`, `P`, `O`, `C`) for the
   fleet, build, jobs and comms screens. `?` lists every shortcut.

3. **Go live** when you're comfortable: click the **STOPPED** pill in the
   top bar, click **Start**, then **Go live** — or start with
   `san-holo serve -live`.

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

`/state.json` also lists what the bot is working on under `Jobs`: one entry
per job (each directive you or the bot set, a carrier still being brought
home after an errand ended, a hub upkeep delivery, a maintenance drone being
carried to a system that has none, the second-replicant errand), with its
current step, what it is waiting for and an ETA when one is known. It also
carries scout campaigns: active ones, and paused or finished ones for 14
days after their last change.

Every directive is a job, kept in the catalog, so jobs and what they hold
survive a restart. Up to three jobs run at once. `hub-deploy`,
`fetch-attached` and `cross-system-haul` can run beside other jobs; every
other kind needs the vessel and runs alone, so a second one waits
(`queued`, waiting for `role:vessel`, naming the job that holds it).
`Leases` shows which job holds the vessel and which devices: a haul holds
the cargo freighter, a carrier errand its surge carrier, and no other job or
routine work touches a device a job holds. While an asteroid threatens home,
a job that flies the vessel pauses (`preempted_by_defence`) and resumes once
the threat has passed. A job whose device disappeared while SanHolo was
stopped comes back paused (`device_missing`).

`POST /actions/directive` works as before: a new vessel errand replaces the
running one, and clearing stops the job the cockpit shows. `POST
/actions/job` (form fields `kind` and `param`) adds a job without replacing
anything, and `POST /actions/job/cancel` (field `id`) cancels one.

**Goals** turn standing rules into jobs without you arming anything.
`/state.json` lists them under `Goals` (with `GoalsPaused` and the spend
ledger `GoalSpend`), and the bot seeds a fresh catalog with these:

| Goal | Default | What it does |
|---|---|---|
| `resource-reserve` | on, floors rares 150 / carbon 1,000 / structural 2,000 | No bot spend takes home stock below a floor: hub prints, hub upkeep loads, maintenance drone prints and routine rares prints all read it. A floor you leave out of an edit keeps its default. |
| `claim-systems` | off | Places your system hub in mining systems: prints one (`print-hub` job, cost reserved against home stock until the print starts; the game takes it then, and a print that has not started in 24 hours asks for help naming what the autofactory waits for) and chains a `hub-deploy` job that starts when the hub stands idle at home, then the next system. `max_total` counts every system hub you own (placed anywhere, including the home system, in flight or waiting at home), hubs printing and prints pending; `max_per_week` and `rares_budget_per_week` are checked against the ledger, so a restart cannot lift them. `order` is `"ranked"` (mining intel) or a list of systems; home, claimed systems, systems with any hub, excluded ones and ones commands cannot reach are never targeted. A hub you print or deploy by hand gets its deploy job too, never a duplicate. |
| `keep-hubs-supplied` | on, `days_cover` 2, `max_trips_per_week` 3 | Every upkeep delivery to a remote hub is a `hub-upkeep` job that leases the freighter until it is home. |
| `keep-miners-busy` | off, `max_ferries_per_day` 2, `accept` empty | Acts on the mining drones' idle reasons. Idle home miners whose reason names a belt of ours with room (a fresh site reading, a relay or hub of ours commanding there, nobody else's claim) are ferried there as a `ferry-miners` job; miners stranded in a claimed system come home as a `recall-miners` job (a belt-less system only when the vessel is already there). One move at a time, never while another job or an asteroid holds the vessel, a maintenance-station trip flies it or it is away from home; the home salvage reserve of miners stays. When nothing can move the goal says why (`no_room`, `home_reserve`, `daily_cap`...). `accept` lists idle reasons to leave alone. While it is on, `mining-network` stops arming ferries and claimed-system withdrawals itself. `clear_sites` (empty by default; `*` for every belt) lists belts where idle miners mine whatever is left, surplus included: a site stays active, and holds one of the belt's five slots, until every resource on it is gone, so mining the leftovers lets the sites close and survey open fresh ones; when a resource worth mining comes back, the miners move onto it. The vessel's own belt is left to the mining strategy. |
| `recycle` | off, `min_idle_hours` 24, `max_per_day` 5, `mode` `ask` | Decommissions clear surplus for the refund of about 60 percent: a device idle for the minimum that no job or strategy can use (its idle reason offers no job; a miner a ferry could take is not surplus), in a system with a deployed autofactory. Hubs, relays, beacons, ships, AMI controllers and autofactories are never recycled whatever `never` or `types` say; `never` can only add types. In mode `ask` (default) each device is a decision in the inbox, approve creates one `recycle` job, deny or expiry leaves it alone for seven days; `mode` `auto` acts without asking inside the same bounds. `max_per_day` counts the ledger, approvals included, so a restart cannot lift it. How long a device has been idle is tracked per device (`StatusSince` on `/state.json` devices) and kept across restarts. |
| `climb` | off, `boards` [fleet, xp, megastructure], `max_prints_per_day` 10 | Works toward the nearest reachable leaderboard rank: the next rank up when the board shows you, else rank 10. Fleet (the board counts active devices) closes by printing the cheapest unlocked device you already field whose cost holds no rares and no volatiles and that Recycle could not scrap as surplus (never mining drones), one at a time on a home autofactory, at most `max_prints_per_day` a day (10 at most, counted from the durable goal job log and spend ledger), never below a `resource-reserve` floor; each print carries a `climb-<unix>` tag. XP closes on its own and spends nothing, so the print budget always goes to the fleet board while it has a gap. The target the top bar shows is the board with the soonest ETA (XP from its measured rate, about an hour after a start), not the smallest raw gap. More prints than the budget, and any megastructure contribution (offered while that board has fewer than ten entries), are decisions, never acted on alone. Recycle keeps running: Climb counts its removals and prints to make up, and Recycle never decommissions a device Climb printed within a week. Every 10 prints it checks, an hour after the last, that your fleet board value rose by at least half of them; if not (or the board does not show you, so the rise cannot be read) it stops with `climb_no_effect` and asks a decision to resume. A megastructure contribution you approve runs as an `ark-contribute` job like one you arm yourself. Nothing in dry-run (it says what it would print); `climb_measuring`, `climb_passive`, `climb_reached`, `climb_checking`, `climb_no_effect`, `daily_cap`, `await_decision`, `below_reserve`... |
| `keep-supply-line` | on, `sources` [], `filter` [] | The standing cross-system haul as a goal: haul-rearm re-arms it once its remote source has refilled with something you need, exactly as before. Empty `sources` keep the last haul you armed; naming `sources` (remote locations) sets the line here and bounds the refill search to them, and `filter` (with sources) limits the haul to those resources. Off, nothing is re-armed. The goal says what the line does: `supply_line_none`, `supply_line_standing`, `supply_line_hauling`. |

`POST /actions/goal` edits one: `id` (the goal), `enabled` (`true`/`false`,
optional) and `params` (JSON, validated, optional; an unknown field or an
out-of-range limit is refused and nothing changes). `POST
/actions/goals/pause` (`paused=true|false`) stops every goal creating new
jobs while running ones finish. In dry-run goal jobs are created but never
admitted (`dry_run`), so nothing is spent until you go live. Each goal says
why it created nothing (`Waiting`: `total_cap`, `weekly_cap`,
`budget_exhausted`, `below_reserve`, `no_target`...).

```sh
curl -s -X POST http://localhost:8080/actions/goal -d id=claim-systems -d enabled=true \
  -d 'params={"max_total":4,"max_per_week":2,"order":["NEARBY","OTHER"],"rares_budget_per_week":1000}'
curl -s http://localhost:8080/state.json | jq '.Goals[] | {ID, Enabled, Params, Waiting: .Waiting.Code}'
```

**Decisions** are how the bot asks before acting outside a goal's budget.
When ClaimSystems would print the next hub but only a cap stops it
(`max_total`, `max_per_week` or `rares_budget_per_week`), it raises one
decision for that system (`/state.json` `Decisions`, the goal reads
`await_decision`) instead of printing, and asks nothing else about it. A
decision shows the cost and expires after 24 hours; expiry counts as deny.
`POST /actions/decision` (`id`, `choice=approve|deny`) answers it. Approve
creates the print and deploy jobs charged to a one-off allowance (the
weekly caps do not count it; the reserve floors still apply, a decision
never lowers one). An approval the bot cannot carry out yet (a floor in
the way) stays `approved` and pending until it can; one whose target can
no longer take a hub is recorded as `lapsed`, which is not a deny: the
question may be asked again. Deny, or expiry, creates nothing and the
same question is not asked again for seven days. Recycle asks the same
way for each device it would decommission (`recycle_device`, showing the
device, where it stands, how long it has been idle and the refund); an
approval it cannot carry out yet (the day's cap) waits, one whose device
went back to work lapses. `change_owner` and wipe are never automated,
whatever a goal says.

A job the bot cannot carry on with is `blocked` (a device gone after a
restart, a parameter that no longer parses); a job merely waiting for a
print, a transform or range is `waiting` and needs nothing from you. A job
blocked for a missing device is re-queued by itself when the device is seen
again; `POST /actions/job/resume` (`id`) re-queues any other blocked job
once you have fixed its cause. `NeedsHelp` now points at the worst blocked
job or the oldest open decision (`JobID` / `DecisionID`) rather than
holding free text, and dismissing it does not raise the same pointer again.
Outbound alerts fire for a new decision and for a job entering `blocked` or
`failed`, never for `waiting`. The dismissal and the alert cooldowns live
in memory, so after a restart the same pointer and the same alert are
raised once more.

```sh
curl -s http://localhost:8080/state.json | jq '.Decisions[] | {ID, Key, Status, Cost, ExpiresAt}'
curl -s -X POST http://localhost:8080/actions/decision -d id=dec-1760000000-claim-systems:NEARBY -d choice=approve
curl -s -X POST http://localhost:8080/actions/job/resume -d id=job-1760000000-hub-deploy
```

```sh
curl -s http://localhost:8080/state.json | jq '.Jobs[] | {ID, Kind, Status, Step: .Steps[.Step].Code, Waiting: .WaitingFor.Code, ETA, Leases}'
curl -s http://localhost:8080/state.json | jq '.Leases | map_values(.JobID)'
```

`IdleReasons` says why idle devices are idle. Every idle or stowed device
except beacons, relays, hubs, the observatory and sensors is in exactly one
entry: a group of one device type at one place, a reason code (for example
`home_surplus`: the belt offers only resources home already has plenty of),
the strategy that declined the work, a suggested fix where there is one, and
since when the reason has held. The running bot refreshes the list every 30
seconds, so it is empty until the first tick after a start. Devices no
strategy accounts for are listed as `unexplained`. A worn mining drone (20
percent capacity or less) reads `low_capacity` when a maintenance drone stands
in its system, `maintenance_due` while a maintenance-station trip is bringing
one (naming that drone), `maintenance_held` while something keeps that
strategy from starting a trip (an asteroid, a directive, the vessel away, a
busy printer or the daily caps; `blocked` names it), and `no_maintenance`
when nothing does. A maintenance-station trip that ends for a routine reason
(the ride fell through, the drone was no longer a spare, the ferry left
without it) shows as a cancelled job with that reason, not a failed one.
The cockpit's belt resource bars read "unknown" rather than 0 percent while
no site reading stands, for example after a read the game refused.

```sh
curl -s http://localhost:8080/state.json | jq '.IdleReasons[] | {Count, Subject, Code, Strategy, Fix: .Fix.Action, Since}'
```

## Log output

`serve -log-format json` also writes every activity-log line to stdout as one
JSON object (`time`, `level`, `strategy`, `message`). Messages over 2 KB are
truncated and your API token is never logged. The default is `off`.

`serve -hosted`, `-hosted-allow-any-bind`, `-hosted-tenant`,
`-hosted-proxy-pubkey`, `-hosted-vapid-public-key`, `-hosted-alert-url`, `-hosted-alert-identity-file`,
`-credentials-source` and the `-qm-*` flags are for the
hosted service only; without `-hosted` they do nothing and all but `-hosted`
itself are refused.

`serve` now starts listening before its first full poll finishes: the map shows
"Loading the galaxy..." until it does, and the bot waits for that poll before
acting. On SIGTERM or SIGINT the bot finishes the action it is running, then
the process exits within 18 seconds.

## Health check

`GET /healthz` answers `200 ok` while the bot loop ticked and the poller read
your replicant within the last 2 minutes, and `503 bot stalled` or
`503 poller stalled` otherwise. A bot you stopped yourself is not stalled. It
returns no game data, so it is safe to point your own uptime monitor at. While
the first poll after start-up is still running it answers `200 starting` (for
at most 10 minutes, then `503 warm-up stuck`).

If you put the Bridge behind a reverse proxy on the internet, `serve -csrf`
makes every POST carry a CSRF token (a `__Host-sanholo_csrf` cookie, sent back
as `X-CSRF-Token`; the Bridge does this itself). The cookie is `Secure`, so it
needs HTTPS or `localhost`.

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
  "release_url": "https://github.com/TydeWhatMay/SanHolo/releases/latest",
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
| `SAN_HOLO_ALERT_NTFY_URL` | a topic URL on an [ntfy](https://docs.ntfy.sh/install/) server; self-host your own, or use any ntfy server you trust |
| `SAN_HOLO_ALERT_DISCORD_URL` | a Discord channel webhook URL |
| `SAN_HOLO_ALERT_WEBHOOK_URL` | any URL; receives `{"kind","severity","text","key","at","link"}` as JSON |
| `SAN_HOLO_PUBLIC_URL` | optional: where the Bridge is reachable from your phone (or `serve -public-url`) |

With none set, alerts are off and nothing is logged about them.

With `SAN_HOLO_PUBLIC_URL` set (for example the address of your VPN or
reverse proxy), every notification carries a link that opens what it is
about on the Bridge: the job, the system, or the alert's preview. ntfy
uses it as the tap action, Discord puts it in the message, the webhook
sends it as `link`. Without it, notifications are text only.

Settings > Notifications shows which services are configured (by name;
the URLs are never shown), lets you turn each kind of alert on or off and
choose which service it goes to, sets quiet hours (in the server's time
zone or one you name, such as `Europe/Berlin`) and sends a test, at most
one a minute. During quiet hours only the kinds ticked "In quiet hours"
are sent; out of the box those are an inbound asteroid, a stuck bot, a
request for help and a decision about to expire.

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
- the bot is stuck: the same error at least three times in a row, over at
  least five minutes, with no XP gained (the Bridge offers to switch that
  strategy off, through a preview);
- a decision waits for you, and again when it has two hours left before it
  is denied by default; a job is blocked or has failed;
- the game pushes an event to SanHolo's `/webhook` endpoint (signed; see
  below).

Alerts are rate-limited: the same condition repeats at most once per
cooldown, a condition that stays clear for two checks and then comes back
alerts again straight away, and an hourly cap stops a flood with a single
notice. Critical alerts (an inbound asteroid) keep some room past the cap,
and webhook events have a small hourly budget of their own.

The game's webhook is optional (the game marks it deprecated in favour of
its event stream, which SanHolo already reads). If you register one
(`POST /v1/accounts/webhook` with your Bridge's public HTTPS URL plus
`/webhook`), SanHolo answers the game's verification challenge, and you
then set the `webhook_secret` the game returns in
`SAN_HOLO_WEBHOOK_SECRET`. Every delivery must carry a valid
`X-Replicant-Space-Signature`; without the variable set, `/webhook`
refuses every event.

## Safety defaults

- Bot starts **stopped** and in **dry-run** — you have to explicitly turn
  it on. Until then it polls your account, populates the dashboard, and
  does not send any state-changing commands.
- Every action is logged to the in-cockpit Activity panel before it fires.
- The copilot's advice is **suggestive by default** — you decide whether
  to act on it. A directive it picks that spends (a `build`), hops the
  vessel (`relocate`) or sits outside its menu waits in the decisions inbox for your approval
  (`-copilot-mode ask`, the default); `-copilot-mode apply` lets it set
  them directly as before. You can also set an explicit directive from the cockpit
  (focus_resource / build / explore / relocate / scout /
  observatory-expedition / ferry-miners / recall-miners / trade_expedition /
  fetch-attached / hub-deploy /
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
  `ferry-miners` (param: a target belt designation like `NEARBY-BELT-1`) ships
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
  `fetch-attached` (param: `SYSTEM:device_type`, for example
  `SOL:belt_surveyor`) brings home devices that cannot cross systems on
  their own and do not fit in a hold, such as belt surveyors. It takes an
  idle surge carrier with nothing attached from your home system, checks
  the route with a dry run, flies to where the devices stand, attaches
  them one at a time (idle ones first) up to what the carrier holds, flies
  home, detaches them, and repeats until none are left there. It refuses to
  start unless a relay or hub gives command in that system and at least one
  of the devices reads in control range, and it only ever attaches devices
  that read in range. A device that refuses to attach is skipped; after
  three refusals, or if nothing can be attached when the carrier arrives,
  the carrier flies home with whatever it holds, the run stops there and
  asks for help. An asteroid threatening home brings the carrier straight
  back. A run stops after 48 hours. If a run ends, or you clear or replace
  the directive, while its carrier is still out or loaded, SanHolo keeps
  bringing that carrier home and unloading it, and asks for help naming the
  carrier and where it is while it cannot be reached. Your vessel stays put,
  and only you can set it (the copilot cannot).
  `hub-deploy` (param: a target system like `NEARBY`, optionally with a
  Lagrange point: `NEARBY:NEARBY-6-L5`) places a newly printed system
  hub in another system to claim it. It waits until a newly printed hub
  stands idle at an autofactory in your home system (so you can set it
  while the hub still prints). A hub ever seen active or at a Lagrange
  point is never taken, whatever it reads later, so your placed hubs stay
  where they are. It compacts the hub (a hub compacted by an earlier run is
  picked up where it stopped), waits out the compaction (30% of the hub's
  print time), attaches it to an idle surge carrier at home, and tries to attach a
  spare maintenance drone standing beside it (one always stays home; if the
  carrier refuses the drone, the run goes on without it). After a dry run it
  flies to the Lagrange point, detaches, sets the drone patrolling, sends the
  carrier home, unfurls the hub, waits out the unfurl and activates it.
  Without a Lagrange point in the param it uses the system's catalogue entry
  point when that is an L4/L5 (the game puts it at the planet nearest the
  belt), else the L4/L5 where one of your relays or beacons stands. Last, the
  cargo freighter brings one load of upkeep stock (185 carbon and 315
  structural, less if home holds less) from home and drops it at the hub;
  if the freighter is out carrying another load, it skips the drop and asks
  for help instead. Nothing happens in the home system while an asteroid
  threatens it.
  After that one drop the `hub-upkeep` strategy keeps the hub stocked (see
  below). It refuses to start unless a relay or hub gives
  command in that system with a device of yours there in control range, the
  star catalogue shows no hub there, no other hub has claimed it, and home
  holds at least 150 rares. It prints nothing. Any step the game refuses
  (compact, unfurl, activate) stops the run and asks for help naming the
  step; a hub that refuses to attach three times, an hour apart, does too.
  The directive replaces any other, so a running cross-system haul pauses
  until you set it again. Runs stop after 7 days; the carrier is looked
  after the same way as a fetch-attached carrier. Only you can set it.
- **Creating a second replicant from the Bridge.** Print an empty replicant
  matrix and stow it in a vessel, matrix container or system hub that stands
  where your own matrix is. Build > Factories then offers **Create a second
  replicant** for it. The preview shows both matrices, the cradle and where,
  whether the game's requirements are met, and that it cannot be undone; you
  can name the new replicant, and the order only goes through once you type
  the empty matrix's device code. The game states no resource cost beyond the
  matrix and cradle you already hold. A matrix an armed errand protects is
  refused. Only you can do this: no strategy, goal, errand or copilot ever
  creates a replicant. The new one appears in the other-replicants list at
  once; reserve its vessel (below) so the bot leaves it alone.
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
  step, post `resume=true` to restart a stopped errand where it stopped
  (its plan, print history and caps are kept), or post `clear=true` to
  remove it.
- **Sending the other replicant to scout nearby stars.** A scout errand
  looks for a new base around an anchored harbour without you watching. It
  stops the replicant mining, stows a spare beacon into its vessel (flying to
  the beacon first), then visits each star you list in turn: it checks that
  the harbour beacon is still deployed (if not, it stops and asks before any
  departure), previews the trip, travels, runs a system scan, records the
  star's belts (and their open sites), salvage sites, shops and whether a hub
  or ward claims it, and flies back to the harbour. A leg that is still at
  its star 25 minutes after arriving heads back regardless, as does one that
  has not arrived 20 minutes after leaving or turns up somewhere else. The
  trip back is never abandoned: a refused trip is retried every minute or
  two for as long as the vessel is away, with a needs-help notice that
  leaves the errand running. A star qualifies
  when it has a salvage site holding structural, conductive or carbon, or a
  belt with an open site. Scouting stops once two stars qualify; if none of
  your stars qualifies it tries the fallback list once. It then raises a
  needs-help alert with the stars ranked and stops: it never settles, you
  pick the base. It only ever sends travel, a system scan, stop mining and a
  stow of the spare beacon, and never addresses the devices you list under
  `protect` or any vector charge or replicant matrix (refused by device
  type, listed or not). Arm it with:

  ```sh
  curl -X POST http://localhost:8080/actions/errand -d kind=scout \
    -d 'replicant=<replicant code>' -d 'harbour=<STAR-N>' \
    -d 'harbour_beacon=<beacon deployed at the harbour>' \
    -d 'stow_beacon=<spare beacon to carry>' \
    -d 'stars=<STAR>,<STAR>' -d 'fallback=<STAR>,<STAR>' \
    -d 'protect=<device code>,<device code>'
  ```

  The cockpit's Fleet screen (Reserved tab) shows the step, the star being
  scouted and each star's result beside the replicant.

- **Sending the other replicant to found a base.** Once you have picked a
  star from the scout report, a settle errand moves the replicant there and
  starts a base without you watching. It stops the replicant mining, loads
  its vessel from the harbour stockpile (rares first, up to 200), checks
  the harbour beacon is still deployed, previews and makes the trip, and
  deploys the founding beacon the moment it arrives, before anything else.
  The beacon deploy is retried every 30 seconds and counts only once a read
  shows it deployed; if it is not down within 22 minutes, the replicant
  flies back to the harbour (the help then lists any cargo still aboard) and
  keeps retrying that trip until it is home rather than stopping away from
  an anchor. Once anchored, it deposits the cargo at the base, mines the
  salvage site for
  structural and then conductive until a survey drone's cost is covered
  (checking the stockpile grows, restarting a mine that stops after landing
  some, and moving on only when the salvage reports that resource gone),
  flies to the belt and tries once to mine a site there that reports carbon
  (a belt is mined only from the belt itself) and records the result, mines
  until the belt stockpile covers the carbon the print lacks, hauls it back
  to the base, and prints one survey
  drone when the base stockpile covers the full blueprint and keeps at least
  20 rares. Prints are capped at one a day and three in all. It then raises
  a needs-help report and stops. It only ever sends travel, a deploy of the
  founding beacon, collect, deposit and a survey-drone print on its own
  vessel, and mine or stop mining at the named salvage and belt, and never
  addresses the harbour beacon, the devices you list under `protect` or any
  vector charge or replicant matrix (refused by device type). Arm it with:

  ```sh
  curl -X POST http://localhost:8080/actions/errand -d kind=settle \
    -d 'replicant=<replicant code>' -d 'harbour=<STAR-N>' \
    -d 'harbour_beacon=<beacon deployed at the harbour>' \
    -d 'stow_beacon=<founding beacon stowed in the vessel>' \
    -d 'base=<BASE-N>' -d 'salvage=<BASE-N-SAL-N>' -d 'belt=<BASE-BELT-N>' \
    -d 'protect=<device code>,<device code>'
  ```

  The Fleet screen (Reserved tab) shows the step, the base, whether it is
  anchored, the belt test and the prints sent.

- **Growing the other replicant's base.** Once a base is anchored, a grow
  errand keeps it growing for as long as it stays armed, one action every
  30 seconds at most. It works toward two survey drones and two mining
  drones: it picks the next print (survey drones first), works out what the
  base lacks for it (reading the cost from the blueprints), mines that where
  it can be found (the salvage while it lasts, else a belt site that reports
  it, with the replicant at the belt), hauls belt stock back to the base and
  prints there. Prints are capped at one a day and four in all, never leave
  fewer than 20 rares, and the next print waits until the last one has shown
  up as a drone. It counts and commands every survey and mining drone of
  the replicant's at the base or the belt, listed under `drones` or not
  (`drones` is optional), and leaves anyone else's alone. Each one is put
  to work: survey drones fly to the belt and search for new sites while
  the belt has fewer than five, mining drones fly to the belt and mine what
  the base needs most. With nothing to print, the replicant keeps mining the
  scarcest resource at the belt and the errand checks in every 30 minutes.
  A failed read (a rate limit or a server error) never stops it: it waits a
  minute more after each one in a row, at most ten, with a notice until a
  read works again. It never leaves the base's system, never
  addresses the anchor beacon, the devices under `protect` or any vector
  charge or replicant matrix (refused by device type), and only ever
  sends travel inside the system, mine and stop mining at the named salvage
  and belt, collect, deposit and a survey or mining drone print on its own
  vessel, and deploy, travel, search and start_mining to its drones. A step
  refused three times stops it with a needs-help alert, but only while the
  anchor beacon reads deployed in the base's system; otherwise it raises a
  notice and keeps going. Arm it with:

  ```sh
  curl -X POST http://localhost:8080/actions/errand -d kind=grow \
    -d 'replicant=<replicant code>' -d 'base=<BASE-N>' \
    -d 'salvage=<BASE-N-SAL-N>' -d 'belt=<BASE-BELT-N>' \
    -d 'anchor=<beacon anchoring the base>' \
    -d 'protect=<device code>,<device code>'
  ```

  Add `-d 'drones=<drone code>,<drone code>'` to name drones up front; the
  errand finds the replicant's drones in the system either way.

  The Fleet screen (Reserved tab) shows the base, the survey and mining
  drones against the target, the base stockpile, the next print (or a print
  still landing), what the replicant mines and the prints sent. Clear the
  errand to stop it.

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
     sites. Open **Comms → Activity** to see recent activity — if you
     see survey drones cycling "searching", that's the AMI working. To
     force a fresh scan, open the system on the map and use the
     **Scan** order in its drawer.
  2. **Dry-run is still on.** The autopilot pill in the top bar reads DRY-RUN — if it's
     there, the bot is choosing actions but not sending them. Open
     the pill and click **Go live**.
  3. **A device got paused.** Open **Fleet → Roster** — paused devices
     show in the roster (and AMI status on Fleet → Overview). Resume the device
     from the API docs (auto-resume from the cockpit is on the roadmap).

## Roadmap

- Code-signed macOS/Windows binaries
- `.deb` / `.rpm` / Homebrew tap / Scoop bucket
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

Third-party components shipped inside the binary: [three.js](https://threejs.org)
0.186.1 (MIT licence) draws the map; the Chakra Petch, IBM Plex Sans and
JetBrains Mono fonts are under the SIL Open Font License 1.1. Their licence
texts ship beside them in the binary's static assets.
