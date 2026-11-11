# Writing requirements

Read this when you build the requirements for a search. `describe_category` is the authority on what exists: each property's type, unit, allowed values, qualifiers, gotchas and how many models have a value. This file holds the recipes that are easy to get wrong.

Each constraint is `{property, op, value, unit?, strength, accept_qualifiers?, required_conditions?, broadest_scope?, accept_best_case?}`.

## Recipes

- **Voltage limits.** Compare charged voltage, not nominal: `power.max_input_voltage_charged` `gte` 50.4 for 12S LiPo (cells × 4.20; LiHV × 4.35). For a supply-range check, use `power.input_voltage` with `accept_qualifiers: {"limit_kind": ["operating"]}`, so an absolute maximum can't pass.
- **Current.** State what kind of current is meant. Phase current is not battery current.
  - **Normal ESC sizing** ("at least 50 A per motor"): `esc.continuous_current` with `accept_qualifiers: {"channel_basis": ["per_channel", "single_channel_device"]}`. ESC figures are often published without saying whether they are per channel; this pair accepts both.
  - **Battery current:** add `current_basis: ["dc_bus"]`.
  - **Through the supplied connectors** (the person asks, or the board ships with motor plugs): `esc.deliverable_continuous_current`. It takes the lowest limit between the board and the motor connector. Solder pads and bare leads don't limit it. A named connector with no published rating (e.g. MR30) leaves it unknown, and so does a connector that isn't recorded. Say which connector a pick uses when it's known.
- **Time-limited ratings.** A current rating with a duration (62 A for 180 s) is a time-limited rating: quote the maker's own label and the duration. Don't call it a plain peak rating, and don't call it continuous without its time limit. Put duration requirements in `required_conditions`, e.g. `{"duration_s": {"gte": 600}}`.
- **Mass.** Pin the basis, e.g. `mass_basis: ["with_leads", "with_connectors", "installed"]`. Bare-board mass is not wired mass.
- **Origin.** `origin.finished_product_country` `not_in` / `in` with ISO codes. Hong Kong (HK) and Taiwan (TW) are separate from China (CN). Company base is a different question, answered by each row's `company` tag and by `company_screen`, never by a requirement.
- **Company-wide statements** ("all our ESCs are made in Germany") count only with `broadest_scope: "organization"`. Even then the result carries a company-claim gap and stays conditional until the exact model is confirmed. Use it when the person asks about compliance, and say the evidence is company-wide. **Origin is the exception:** for `origin.finished_product_country` an unhedged company-wide statement counts by default and can pass; the row's `assumed` says so, and the answer still calls it a company-wide statement. Pass `broadest_scope: "family_member"` when the person wants model-level evidence only. "Made in EU" (also EEA, USMCA) settles "not made in China" through the region's member countries.
- **Firmware support.** `fc.firmware_support` `contains_all` `["px4"]`, and nothing more for a plain "runs PX4". A `support_level` filter the person didn't ask for drops parts whose claim is recorded as the maker's own word, and changes the pick. Only when the person asks for official or stable-release support: add `accept_qualifiers: {"support_level": ["stable_release"]}` and pass `assumptions` without `firmware_claim_convention`. With the convention left on, makers' unlabelled claims still pass that filter.
- **GNSS.**
  - RTK rover: `gnss.rtk_roles` `contains_all` `["rover"]`.
  - Bands: `gnss.frequency_bands` `contains_all` e.g. `["l1","l5"]`. Dual-band isn't RTK by itself.
  - Autopilot: `gnss.autopilot_support` `contains_all` `["ardupilot"]`. Accept `support_level` `stable_release` only for "supported"; add `vendor_claim_only` if a maker's word is enough, and say which it is.
  - Accuracy: `gnss.position_accuracy` `lte` with `accept_qualifiers: {"positioning_mode": ["rtk_fixed"], "axis": ["horizontal"]}`. RTK figures assume a nearby base: "+1 ppm" means 1 mm extra error per km of baseline.
  - Survey work: `gnss.raw_logging` and `gnss.event_input` cover PPK and camera triggering.
- **Mounting, chip family, size.** `physical.mount_pattern` takes "30.5x30.5" (or "20x20 M2"), matched within 0.5 mm. Text fields such as `fc.mcu` take `op: "contains"` ("H7"). For a 2306 motor use `motor.stator_height` `eq` 6 with `tolerance` 0.5, or `gte`/`lte` on separate keys.
- **Operators.** `lt`/`gt` for strict limits ("below 70 g"), `lte`/`gte` for inclusive ones. A property that holds one value takes `eq`/`in`; one that holds a set takes `contains_all`/`contains_any`; a range takes `covers`.
- **Optional extras** are `soft`. Soft requirements rank; they never exclude.
- **When a requirement comes back unknown,** read the result's `note`: it says which claims were not counted and why.

## Reading conventions (`assumptions`)

Makers publish ratings without their conditions. A convention reads such a rating the way an engineer would, and every row lists what was read that way under `assumed`; `assumptions_note` says which conventions applied.

- **Defaults** (applied when `assumptions` is omitted): `unspecified_chemistry_is_lipo` in every category, plus the category's own: ESC `esc_rating_convention`; flight controller `esc_rating_convention` + `firmware_claim_convention`; battery `battery_rating_convention`; radio `radio_range_convention`.
- **A list you pass replaces the defaults.** To add one convention, pass the defaults you still want too. Pass `[]` for stated conditions only.
- **Name the ones that decide a result in your reading:** "ESC 'continuous' ratings read as the makers' standard ratings; 6S read as LiPo."

| Convention | What it reads | Turn it off when |
|---|---|---|
| `unspecified_chemistry_is_lipo` | "3–6S" with no chemistry as LiPo, 4.2 V per cell, and a bare cell count as the operating range (not an absolute maximum) | the person uses LiHV or Li-ion |
| `esc_rating_convention` | an ESC continuous rating with no stated duration, cooling or current basis as the maker's standard continuous rating, and a board's unqualified figure as per motor | the rating must hold for a stated time or with no airflow (fixed-wing cruise, hover-heavy duty). Then give the published figure and say it isn't confirmed for those conditions |
| `battery_rating_convention` | a capacity with no stated basis or discharge rate as the pack's rated capacity | the person needs capacity at a stated discharge rate |
| `firmware_claim_convention` | a maker's "supports ArduPilot/PX4" with no named target or release as stated | the buy depends on a specific target: check the firmware's own board list |
| `radio_range_convention` | a maker's link range as measured with stock antennas in clear line of sight | the person's antennas or environment differ. It never turns an "up to" best case into a pass |
| `rf_power_convention` (never default) | an RF power with no stated region or statistic as the hardware's highest setting | it is set only by `translate_requirement`'s searches |

When a pick passed because of `esc_rating_convention`, add one line: "Current ratings are the makers' standard figures; duration and cooling aren't published." The convention leaves burst ratings, hedged claims and conflicts alone.

## The link-range calculator

`translate_requirement` with `need: "link_range"`, `distance_m`, `link` (`rc_control`, `telemetry` or `video`), the person's bands if any, and `destination`. Per band it returns `summary` (one sentence, ready to quote: the power needed, the assumptions and what the destination allows), the path loss, the RF power each published receiver sensitivity needs (with sources; a maker's "up to" sensitivity is shown but never used), `legal_limit` for the destination's licence-free rules with the range they allow, Fresnel clearance, and `screen.search`: a ready search with the band hard, and power and the maker's stated range soft.

- **The person named a band:** run `search_parts` once per band with its `screen.search`, adding the person's own hard requirements.
- **No band named:** call it without `bands` (it returns every band it has data for), don't filter the search by band, and rank on the makers' stated range as a soft requirement.
- **A link that carries two things** (telemetry and video): call it once for each `link`.
- **Put the result in the answer,** as one short paragraph after the pick's block that starts "On the 10 km:". Build it from each band's `summary`, shortened but with its numbers, margin, antenna gains and limit unchanged: "On the 10 km: with a 15 dB margin, 2 dBi antennas and line of sight, 868 MHz needs about 10 to 26 mW, within the UK's 25 mW ERP only for the most sensitive receiver on file. 2.4 GHz needs about 208 to 416 mW, above the 100 mW EIRP limit, which allows about 5.5 km." In "Read as" write only "10 km treated as a ranking figure; link budget below".
- **Never rework the numbers or supply a limit yourself.** Where `summary` says the power wasn't worked out, or that no limit is on file for the band at the destination, say exactly that.
- **Change the assumptions when the person gives them:** antenna gains, margin, altitude (`rx_height_m`). 10 dB is a thin margin; 20 dB is robust.
- **Which end counts:** RC control is the ground transmitter's uplink; telemetry and video are the air unit's downlink.
- `screen.search` turns on `rf_power_convention` and accepts "up to" power figures. Say once: "power figures are the makers' highest settings; legal power at the destination is separate."

## Alternatives to a part that isn't in the catalogue

1. `lookup` the part first. If it's there, call `find_alternatives` with `part`.
2. If it isn't, get its specs before searching. Never guess them, and never search on the name alone.
   - Find the maker's own page (web search if you have it, or a link the person gave) and read it with `read_maker_page`. That's the text the quote check uses.
   - Take the specs the category's drop-in rules compare (`describe_category` lists the rules). Give each a typed value and a verbatim quote from the page text.
   - Call `find_alternatives` with `reference: {category, name, brand, form, page_url, specs: [{property, value, qualifiers?, quote}]}`. Specs whose quote isn't on the page, or whose value doesn't fit the dictionary, come back in `specs_rejected`: fix them from the page text or drop them.
   - No web access, or the page is blocked: use what the person told you, with their own words as the quote and no url. Ask for the decisive specs that are missing, in at most one question. Otherwise say which rules couldn't be checked.
   - `baseline: "all"` also uses third-party evidence for the original's values. Say so if you use it.
3. In the answer, say where the part's specs came from: "X isn't in the directory; its specs are from [maker page] (read today)" or "…from your description". List any spec that couldn't be used. The person's own requirements still go in `requirements`.

## Category notes

- **FPV video** (`video-link`): forms `analog_vtx`, `digital_air_unit` (DJI O3/O4, Walksnail Avatar, HDZero, OpenIPC), `goggles`, `vrx_module`, `video_kit`. Use `video.system` for the ecosystem and `video.compatible_with` for stated goggles/air-unit pairings. Digital systems only work within a family and generation: say so. VTX power is a list of selectable levels. Industrial video datalinks (Herelink, SIYI HM30, Doodle Labs) are in `radio`.
- **Antennas:** `antenna.frequency_band`, `antenna.polarization` (RHCP and LHCP don't work together), `antenna.connector` (SMA and RP-SMA don't mate), `antenna.gain`. Most values come from shop listings: say so.
- **Chargers:** `charger.charge_power` and `charger.charge_current` carry `channel_basis` (per channel or aggregate). AC and DC input often give different maximum power.
- **Companion computers and vision** (`companion-computer`): forms `sbc` (Raspberry Pi 5), `som` (a module that needs a carrier: Jetson Orin NX, CM5), `carrier_board`, `dev_kit`, `ai_accelerator` (Hailo-8, Coral), `vision_processor`, `flight_computer` (VOXL 2).
  - `cc.ai_performance` is in TOPS. Always give the precision: sparse INT8 is about twice dense INT8. Never compare TOPS across precisions, and never give a TOPS figure for a CPU-only board.
  - `cc.module_standard`: a module only fits a carrier with the same standard (Jetson 260-pin, RPi CM4/CM5).
  - `cc.camera_inputs` are counted with their interface (MIPI CSI, USB, GMSL).
  - `cc.autopilot_integration` holds only what the maker states. Any Linux board can run MAVLink tools: say so instead of treating it as unknown.
  - For "a Raspberry Pi or alternatives", run `find_alternatives` on the specific Pi, then add the person's real needs as requirements: camera count, TOPS for the model they run, weight, power budget, origin.
- **Servos:** hobby servo speed is `servo.speed` (seconds per 60°); industrial actuators use `servo.angular_speed` and `servo.force`. Torque accepts N·m, kg·cm and lbf·in.
- **Airspeed:** accuracy as % of span is `airspeed.pressure_accuracy_fs`; accuracy in pressure units is `airspeed.pressure_accuracy`.
