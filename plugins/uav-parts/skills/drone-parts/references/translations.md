# Translating needs into searchable requirements

Read this when a request states an outcome ("flies 25 minutes", "fits a 5-inch quad", "cm-level", "works at −20 °C") rather than a stored property. Each row maps a common phrasing to the properties the catalogue holds. How many models publish each property differs widely between categories: before you make one of these a hard requirement, read its `products_with_claims` in `describe_category` for the category you are searching.

**Contents:** strength and units (below) · All categories · Radio · FPV video · Antenna · ESC · Motor · Propeller · Battery · Flight controller · GNSS · Camera · Companion computer · Airframe · Power distribution, servo, charger, wiring, sensor, airspeed.

**Strength:**
- **Hard:** the person's own limits, and their necessary consequences (6S means a charged 25.2 V).
- **Soft:** estimates, rules of thumb and translated figures. They rank parts; they never exclude.
- **Reading:** every translation goes in your reading ("Read as:", or the brief when you ask first) as one line with its assumptions, so the person can correct it.
- **Answer:** keep three things apart: direct evidence (a maker's stated figure), what the translation implies (a candidate on physics, not a confirmed result), and what couldn't be checked (no data for that property).

**Units:** 1 in = 25.4 mm · 1 kg·cm = 0.0981 N·m · 1 oz·in = 0.00706 N·m · dBm → W: 10^(dBm/10)/1000 · charged LiPo 4.20 V/cell, LiHV 4.35, Li-ion 4.20 (nominal 3.6–3.7), LiFePO4 3.65 (nominal 3.2).

## All categories

| Person says | Search | Notes |
|---|---|---|
| "for 6S" (or any cell count) | `power.max_input_voltage_charged` `gte` cells × 4.20 (LiHV × 4.35) — hard | Derived from voltage or cells. Use charged voltage, never nominal. |
| "not made in China" | `origin.finished_product_country` `not_in` ["CN"] — hard | Company base is a separate question (`company_screen`). Say which one was checked. |
| "not a Chinese company", "non-Chinese supplier" | no requirement: read each row's `company` tag | About the company, not where the part is made, and known for most makers. Leave `known_chinese_company` rows out of the table, give their count, and show the base as a pass ("✓ Canadian company"). Add the origin requirement only if the person also asks where it is made. |
| "NDAA compliant" | `compliance.ndaa_declaration` `eq` true | The maker's own statement, not a certification. Company-wide statements need `broadest_scope: "organization"` and stay conditional. No statement is not "non-compliant". |
| "Blue UAS" | `compliance.blue_uas_cleared` `eq` true (official list as saved on a date). Also `compliance.blue_uas_statement` `eq` true (what makers say) | Give the list's date. Not found on the saved list is not "not listed": names differ. Say when a maker claims a listing the list doesn't show. |
| "ITAR-free", "export class" | `compliance.itar_controlled` `eq` false; `compliance.eccn` | Makers' statements, not legal determinations. Other export controls can still apply. Pass `broadest_scope: "organization"` so that company-wide statements show up (the value with a company-wide gap; they stay conditional), and name the makers that have one. |
| "waterproof", "IP67" | `physical.ingress_rating` `contains` "IP67" (text) | The exact code the maker publishes. Several codes ("IP54/IP67") mean options or housings, recorded as hedged. Most parts publish none: unknown, not "unsealed". Connectors also have `wiring.ip_rating`. |
| "works at −20 °C", "hot climate" | `physical.operating_temperature` `covers` −20 °C (range), with `accept_qualifiers: {"limit_kind": ["operating"]}` | The maker's operating range. Storage and charging ranges are separate (`battery.charge_temperature`). Most parts publish none. |
| "still in production", "not discontinued" | `lifecycle.status` `not_in` ["discontinued","eol_announced","nrnd"] | Only what makers' pages say. A live page doesn't prove production, so no value is unknown. `check_build` flags discontinued lines. |
| "lightweight", "small", "cheap" | — | Ask for a number. Never invent a limit. Price is `check_availability`, not a requirement. |
| "drop-in replacement for X" | `find_alternatives` on X | Add the person's hard requirements as `requirements`. |
| "check my build", "here's my BOM", "is this build NDAA-clean?" | **`check_build`** with the parts list | Provenance and compliance roll-up, fit checks, totals, drafted questions, swaps (SKILL.md section 6). |
| "in stock in the UK", "buy in Europe" | `destination` on `search_parts`, then `where_to_buy` | Availability is checked per part, not filtered. |

## Radio

| Person says | Search | Notes |
|---|---|---|
| "10 km range", any distance | **`translate_requirement`** `link_range` | Calculator: sourced sensitivities, power per band, legal limit. How to run it with or without a named band, and where the working goes in the answer: requirements.md, "The link-range calculator". Few makers publish `radio.range` itself. |
| "ELRS", "Crossfire", "works with my TX16S" | `radio.rc_ecosystem` `in` — hard | A radio model (TX16S) names an ecosystem, not a property: look it up first. |
| "telemetry and video in one" | `radio.functions` `contains_all` ["telemetry","video"] | |
| "low latency" | `radio.latency` `lte`; `radio.rc_packet_rate` `gte` — soft | Ask for a number, or say "lowest available" and rank. |
| "encrypted" | `radio.encryption` `contains_any` ["aes128","aes256"] | `proprietary` is not AES: say so. |
| "anti-jam link", "frequency hopping", "LPI/LPD", "mesh" | `radio.interference_resilience` `contains_any` [...] — e.g. ["lpi_lpd","interference_cancellation","adaptive_frequency_hopping"] | Almost every 2.4 GHz RC link hops (`fhss`); that alone isn't military-grade anti-jam. Name the technique each part states. |
| "drive servos directly, no FC" | `radio.pwm_outputs` `gte` servo count | |
| "legal in the EU / UK" | band via `translate_requirement` `destination` | Licence-free power limit per band. Region rules are a check to state, not a catalogue filter. |
| "1 W" | `radio.rf_output_power` `gte` 1 W, `assumptions` incl. `rf_power_convention` | RF watts, not input watts. |

## FPV video (`video-link`)

| Person says | Search | Notes |
|---|---|---|
| distance | **`translate_requirement`** `link_range`, `link: "video"`, `category: "video-link"` | Sensitivity is sourced only for HDZero. For DJI, Walksnail and analog, say the power screen rests on that one receiver figure. |
| "HD", "1080p", "4K live" | `video.live_resolution` `contains_any` [...] | Live feed, not recording. |
| "for DJI Goggles 3 / Walksnail" | `video.system` `contains_any` — hard; `video.compatible_with` | Digital systems pair only within a family and generation: say a pairing isn't confirmed unless stated. |
| "racing / lowest latency" | `video.latency` `lte` — soft | Sparse: most come back unknown. |
| "25 mW for EU", "1 W long range" | `video.rf_output_power` (list of levels) | A list: "has a 25 mW level" vs "can do 1 W". |
| "SmartAudio / MSP control" | `video.control_protocols` `contains_any` | |
| "fibre-optic FPV", "unjammable link" | `product.form` `eq` "fiber_optic_link"; `video.fiber_length` `gte` | Few fibre makers publish model pages: expect a short list and say so. |

## Antenna

| Person says | Search | Notes |
|---|---|---|
| "for my 868 receiver" | `antenna.frequency_band` `covers` 868 MHz — hard; `antenna.connector` — hard | Get the receiver's connector first (u.FL/IPEX vs MMCX vs SMA). SMA and RP-SMA don't mate. |
| "more range" | `antenna.gain` `gte` — soft; forms `patch`, `helical` | Each 6 dB of extra gain doubles the free-space range (`translate_requirement` takes the gains). Directional antennas need pointing. |
| "match my goggles" | `antenna.polarization` `eq` the other end — hard | RHCP and LHCP don't work together. |

## ESC

| Person says | Search | Notes |
|---|---|---|
| "for these motors" / "for a 2207 on 6S" | `esc.continuous_current` `gte` the motor's `motor.peak_current`, else `motor.max_power` ÷ charged voltage — soft | Get the motor's figures with `get_part` first, then say which one sized it. Don't use a rule-of-thumb class ("5-inch = 45 A") as a fact. |
| "4-in-1 for a 5-inch" | `forms` ["four_in_one","unspecified"]; `physical.mount_pattern` `eq` "30.5x30.5" or "20x20" | Ask which stack size if they didn't say. |
| "RPM filtering" | `esc.control_protocols` `contains_any` ["bidirectional_dshot"] or `esc.telemetry_transports` `contains_any` ["bidirectional_dshot_erpm","extended_dshot_telemetry"] | |
| "AM32 / Bluejay / BLHeli_32" | `esc.firmware` `contains_any` | |
| "DroneCAN / CAN ESC" | `esc.control_protocols` `contains_any` ["dronecan","cyphal","kdecan"] | |
| "for fixed-wing cruise / no airflow" | `assumptions: []` | The standard rating isn't confirmed for long duty without cooling. Say so. |

## Motor

| Person says | Search | Notes |
|---|---|---|
| "2207", "2806.5" | `motor.stator_diameter` `eq` 22 `tolerance` 0.5; `motor.stator_height` `eq` 7 `tolerance` 0.5 | Many sizes are only in the name: also try `lookup`/`text`. |
| "1750 KV" | `motor.kv` `eq` 1750 `tolerance` ~5% | |
| "for 6S at 5-inch" | `power.input_cells` `covers` 6; KV — ask or rank | No-load rpm = KV × voltage. Loaded rpm is lower. Check it stays under `prop.max_rpm` when the prop publishes one. |
| "lift 2 kg", "thrust-to-weight 2:1" | `motor.max_thrust` `gte` per-motor thrust in gf (conditions `test_prop`, `supply_v`) — soft | Per motor: AUW × ratio ÷ motors (state the ratio; 2:1 is a common multirotor starting point, not a rule). The figure is static thrust with the maker's test propeller. For the other motors, read thrust tables via `read_maker_page` and say thrust was read, not filtered. `check_build` does the ratio for a parts list. |
| "efficient for long endurance" | `motor.kv` lower, larger `motor.stator_diameter` — soft | Efficiency (g/W) isn't stored: read thrust tables at hover thrust. |
| "T-mount", "M5 shaft" | `motor.prop_attachment`; `motor.shaft_diameter` | |

## Propeller

| Person says | Search | Notes |
|---|---|---|
| "5-inch", "5x4.3x3", "51466" | `prop.diameter` `eq` 127 `tolerance` 3; `prop.pitch` `eq` 109 `tolerance` 3; `prop.blade_count` `eq` 3 | inch × 25.4. Compact names put diameter then pitch: "51466-3" = 5.1 in × 4.66 in, 3 blades. Ask if a name is ambiguous. |
| "fits frame X" | X's `airframe.max_prop_diameter`, then `prop.diameter` `lte` | |
| "for motor with 5 mm shaft / T-mount" | `prop.hub_bore` `eq` 5; `prop.hub_mount` `eq` "t_mount" | |
| "durable / bendy" | `prop.material` — soft | |

## Battery

| Person says | Search | Notes |
|---|---|---|
| "6S" | `battery.series_cells` `eq` 6 — hard | |
| "flies 25 minutes" | `battery.capacity` `gte` Ah, or `battery.energy` `gte` Wh — soft | Energy (Wh) = hover power (W) × minutes ÷ 60 ÷ usable fraction. 0.8 is a common LiPo planning figure; say it. Ah = Wh ÷ nominal pack voltage. Hover power must come from the person, their logs or a motor thrust table. If none is known, ask. Don't use a W/kg rule as fact. More capacity adds mass, which raises hover power: say the estimate ignores that unless iterated. |
| "120 A continuous" | `battery.continuous_discharge_current` `gte` 120 (derived from C × capacity when only C is published) | Say that C ratings are makers' figures and often optimistic. |
| "under 100 Wh for flying with it" | `battery.energy` `lte` 100, or capacity × nominal voltage | 100 Wh is the usual airline threshold for carrying without approval; the airline's rules decide. |
| "long endurance / Li-ion" | `battery.chemistry` `eq` "li_ion" | |
| "XT60" | connector isn't a battery property: read `legacy_text` | |

## Flight controller

| Person says | Search | Notes |
|---|---|---|
| "runs ArduPilot / PX4 / INAV / Betaflight" | `fc.firmware_support` `contains_all` | No `support_level` filter unless the person asks for official or stable-release support: see requirements.md, "Firmware support". |
| "octocopter", "VTOL with 4 servos" | `fc.pwm_outputs` `gte` motors + servos | Count the outputs the airframe needs. |
| "GPS, telemetry, RC and VTX" | `fc.uart_count` `gte` device count | Count serial devices; CAN devices don't use UARTs. |
| "DroneCAN peripherals" | `fc.can_bus_count` `gte` 1 | |
| "redundant", "dual IMU" | `fc.imu_count` `gte` 2 (derived from `fc.sensors`); `fc.redundancy` `contains_any` | |
| "blackbox logging" | `fc.logging` `contains_any` ["onboard_flash","sd_card"] | |
| "H7" | `fc.mcu` `contains` "H7" | |
| "fits a 20x20 stack" | `physical.mount_pattern` `eq` "20x20" | |

## GNSS

| Person says | Search | Notes |
|---|---|---|
| "cm-level", "survey grade" | `gnss.rtk_roles` `contains_all` ["rover"] — hard; `gnss.position_accuracy` `lte` 0.02 m with `positioning_mode` rtk_fixed, `axis` horizontal | RTK needs a base or corrections; "+1 ppm" grows 1 mm per km of baseline. Say it. |
| "fast RTK fix", "works near trees" | `gnss.frequency_bands` `contains_all` ["l1","l5"] or ["l1","l2"] — soft | Multi-band helps; it isn't RTK by itself. |
| "heading without a compass", "near metal" | `gnss.heading_method` `in` ["dual_antenna_single_receiver","paired_receivers_moving_base"] | |
| "PPK mapping" | `gnss.raw_logging` + `gnss.event_input` | |
| "for ArduPilot over CAN" | `gnss.output_protocols` `contains_any` ["dronecan"]; `gnss.autopilot_support` | |
| "10 Hz" | `gnss.update_rate` `gte` 10 | |
| "anti-jam antenna", "CRPA unit" | form `crpa` | Antennas and anti-jam units with their own electronics; check the technique row below too. |
| "anti-jamming", "keeps GPS when jammed" | `gnss.interference_protection` `contains_any` ["crpa_nulling","beam_steering","adaptive_notch_filter","pulse_blanking"] — hard | Detection (`jamming_detection`, `spoofing_detection`) only warns; it doesn't keep the fix. Say which the part has. Bands and constellations aren't jamming resistance. |
| "CRPA", "N-element", "nulls N jammers" | `gnss.antenna_elements` `gte` N; `gnss.nulling_capacity` `gte` N (an N-element array nulls at most N−1) | Few makers state these; unknown is not no. |
| "J/S of X dB" | `gnss.jamming_rejection` `gte` X | Stated floors ("> 110 dB with GNSS receiver") are recorded as the floor; J/S and J/N differ, and a "jammer suppression" figure is not J/S. |
| "anti-spoofing", "OSNMA" | `gnss.interference_protection` `contains_any` ["spoofing_mitigation","signal_authentication"] | `signal_authentication` is a stated authenticated-signal service (OSNMA etc.), not M-code compatibility. |

## Camera

| Person says | Search | Notes |
|---|---|---|
| "see / recognise a person at 500 m" | `camera.fov` `lte` — soft; `camera.resolution` is width × height and takes only `lte`, so a minimum can't be filtered: compare the returned values | Pixels across a target = target size × image width px ÷ (2 × distance × tan(FOV/2)). Johnson criteria (2 px per cycle): detection ≈ 2 px, recognition ≈ 8 px, identification ≈ 13 px across the target's smallest dimension. State them as the criteria used. Zoom: divide the FOV by the zoom (`camera.zoom`). |
| "2 cm/px mapping at 100 m" | `camera.fov` — soft; compare `camera.resolution` values (a minimum can't be filtered) | GSD = 2 × altitude × tan(FOV/2) ÷ image width px. Solve it for the resolution at a given FOV, or the reverse. |
| "mapping without distortion" | `camera.shutter` `in` ["global","mechanical_leaf"] | |
| "thermal, find hot spots" | `camera.thermal_netd` `lte`; `camera.radiometric` | Radiometric = temperature per pixel. |
| "MAVLink / gimbal control" | `camera.control_protocols` `contains_any` ["mavlink2"]; `camera.gimbal_axes` `gte` | |
| "for a Jetson / Pi" | `camera.video_interfaces` `contains_any` ["mipi_csi2","usb3","gmsl2"] | Match the computer's `cc.camera_inputs` interface. |

## Companion computer

| Person says | Search | Notes |
|---|---|---|
| "run YOLO at 30 fps" | `cc.ai_performance` `gte` TOPS — soft | There's no reliable conversion from a model and fps to TOPS. Ask for a benchmark, or rank by TOPS at a stated precision. Never compare across precisions. |
| "two cameras" | `cc.camera_inputs` `gte` 2 | Count by interface. |
| "ROS 2" | `cc.os_support` `contains_any` ["ros2"] | Any Linux board can run ROS 2; a listing only says the maker states it. |
| "talks to the autopilot" | any Linux board (MAVLink over UART/USB) | `cc.autopilot_integration` is only the maker's statement. |
| "under 10 W" | `power.input_power` `lte` 10 — soft | Sparse: read `legacy_text`. |

## Airframe

| Person says | Search | Notes |
|---|---|---|
| "5-inch freestyle frame" | `airframe.max_prop_diameter` `gte` 127; `airframe.layout` `eq` "quad"; `airframe.stack_mount_pattern` | |
| "carry 1 kg" | `airframe.payload_capacity`, `airframe.mtow` | **Nearly no data.** Search on size and layout, then read `legacy_text`, and say payload wasn't filtered. |
| "1.5 m wingspan" | `airframe.wingspan` `gte`/`lte` 1500 | |
| "fits a backpack" | `physical.dimensions` `lte` | Ask for the size; folded vs unfolded is often unstated. |

## Power distribution, servo, charger, wiring, sensor, airspeed

| Person says | Search | Notes |
|---|---|---|
| "5 V 4 A for a Pi from 6S" | `power_dist.output_voltage` `covers` 5; `power_dist.output_current` `gte` 4; `power.max_input_voltage_charged` `gte` 25.2 | |
| "current sensor for ArduPilot" | `power_dist.monitoring` `contains_all` ["current","voltage"] | |
| "aileron servo, 10 kg·cm, 0.1 s" | `servo.torque` `gte` 0.98 N·m; `servo.speed` `lte` 0.1 s/60° | Convert units. The torque a surface needs isn't derivable from the wingspan alone: ask or use the maker's recommendation. Say the voltage the rating is quoted at. |
| "HV servo on 2S" | `power.input_voltage` `covers` 8.4 | |
| "charge four 6S 1300 packs at 1C" | `charger.charge_power` `gte` cells × 4.2 × A × packs (aggregate, with `channel_basis`); `charger.cell_range` `covers` 6 | 6 × 4.2 × 1.3 × 4 ≈ 131 W, plus a parallel board or 4 channels. |
| "charge from the car" | `power.input_voltage` `covers` 12–14.4 V | DC input power is often lower than AC. |
| "XT60 for 60 A" | `wiring.connector_family`; `wiring.current_rating` `gte` 60 | |
| "terrain following to 50 m" | forms ["rangefinder","lidar_scanner","radar_altimeter"]; `sensor.range` `covers` 50 | Range is usually for a stated target reflectivity: say so. |
| "detect manned aircraft" | `sensor.adsb_functions` `contains_any` ["adsb_in_1090","adsb_in_978"] | 978 MHz UAT is US-only. |
| "navigate without GPS", "GPS-denied" | `sensor.kind` `eq` "visual_navigation" | Camera-based positioning modules; `optical_flow` only holds position at low height. Very few are catalogued. |
| "airspeed to 40 m/s" | `airspeed.speed_range` `covers` 40 or `airspeed.pressure_range` | Dynamic pressure q = ½ ρ v² (ρ ≈ 1.225 kg/m³ at sea level): 40 m/s ≈ 980 Pa. |
