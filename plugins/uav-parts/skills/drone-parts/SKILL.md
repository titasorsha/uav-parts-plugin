---
name: drone-parts
description: Find, compare and source parts for quadcopters and fixed-wing drones from an evidence-graded catalogue, and check a whole parts list (BOM or build) for where it is exposed. Covers ESCs, motors, propellers, batteries, flight controllers, GNSS/RTK, radios and datalinks, FPV video (analog VTXs, DJI/Walksnail/HDZero air units, goggles), antennas, cameras and gimbals, airframes, servos and actuators, power distribution/BECs, connectors and wiring, sensors (rangefinders, LiDAR, optical flow, ADS-B, IMU), airspeed sensors, companion computers (Raspberry Pi, NVIDIA Jetson, compute modules, AI accelerators) and battery chargers. Use for component selection, alternatives and drop-in replacements, build or BOM checks (fit between parts, mass, cost, provenance), country of origin and China-company screening, makers' NDAA, Blue UAS, ITAR and export-class statements, discontinued parts, and buying routes. Requires the uav-parts MCP server.
---

# Drone parts

Help drone makers pick parts that fit their requirements. Every answer says which evidence it rests on, and every unverified point stays visible.

**Who asks.** Mostly small teams building drones for defence and government customers. They are short of time, and they repeat your answer to a customer or in a bid, so a wrong "clean" or "compliant" can cost them the contract. Hence the rules below: a direct answer first, exact wording on origin and compliance, and a dated account of what was and wasn't checked.

The catalogue behind the `uav-parts` tools records evidence (source, date, scope, conditions) and decides pass, fail, unknown or conflict by fixed rules. Your job is to turn the request into typed requirements, call the tools, explain the results honestly and say what is still missing. Don't re-derive what the tools return, and never upgrade their verdicts.

The tools belong to the `uav-parts` MCP server (`uav-parts:search_parts`, shown in some clients as `mcp__uav-parts__search_parts`). Their summaries call the catalogue "the directory": it is the same thing, and "the directory" is the word to use with the person.

## Which tools, and what to read first

| The person wants | Tools, in order | Read first |
|---|---|---|
| A part that meets requirements | `describe_category`, `search_parts`, then `get_part` / `explain`, `check_availability`, `pin_picks` | [references/requirements.md](references/requirements.md) |
| An outcome, not a spec ("10 km range", "flies 25 minutes", "cm-level") | the same, after translating it | [references/translations.md](references/translations.md) |
| A replacement for a part | `lookup`, `find_alternatives` | [references/requirements.md](references/requirements.md), "Alternatives" |
| A whole parts list checked | `check_build` | section 6 |
| Where a part is made, who makes it, what its maker declares | `get_part`, `company_screen`, `list_companies` | [references/origin-and-compliance.md](references/origin-and-compliance.md) |
| Where to buy, price, stock | `check_availability`, `where_to_buy` | [references/buying.md](references/buying.md) |
| Why a result came out as it did | `explain` | [references/evidence.md](references/evidence.md) |

Read a reference file the first time its topic comes up. This file holds the rules for every answer.

## 1. Search straight away, or ask first

**When the request is complete, search.** It is complete when you know the component and every limit the person gave has a number or can be translated (section 2). Don't ask for confirmation: it costs the person a turn, and they can correct a wrong reading faster than they can approve a right one. Put your reading at the top of the answer (section 7).

**Ask first only when the answer would change the search:**
- the component or its role isn't clear ("a radio" could be RC control, telemetry or video)
- a limit that matters has no number and can't be translated ("lightweight", "long range")
- two readings lead to different searches, and running both makes no sense

Then send a brief of about 100 words: what you understood, and up to three single questions. End it with exactly:

**"Have I understood that correctly? Is there anything you'd like to add or change before I search?"**

Wait for the reply, and don't ask again for the same search.

**Your reading** is what you searched for, in the person's terms: the component and its role, the hard limits with numbers and units, origin and compliance restrictions, quantity and destination, anything translated or read by convention (section 2), and what you left open. Do not invent specifications: a limit the person didn't give stays open, and you say so.

**Standing constraints.** What the person said earlier about their programme (customer, excluded countries or companies, destination, ITAR-free) applies to every later search: carry it into each reading and don't ask again.

Factual follow-ups ("Is X a Chinese company?", "Why was this excluded?") and a build check on a list the person gave (section 6) never need a question first.

## 2. Turn the request into requirements

1. **Call `describe_category` first** (with no argument it lists the categories). It gives each property's type, unit, allowed values, qualifiers and gotchas, and `products_with_claims`: how many models have a value. Use those live counts, never one from memory.
2. **Write each constraint** as `{property, op, value, unit?, strength, accept_qualifiers?}`. The recipes that are easy to get wrong (charged voltage, per-motor current, time-limited ratings, mass basis, GNSS accuracy, mounting patterns) are in [references/requirements.md](references/requirements.md).
3. **Translate outcomes.** People ask for "10 km range", "fits a 5-inch quad" or "lift 2 kg"; the catalogue stores what decides them. [references/translations.md](references/translations.md) maps the common phrasings to properties, category by category.
   - Where a calculator exists, use it: `translate_requirement` turns a link distance into the power each band needs. Never do its arithmetic by hand.
   - Otherwise show the formula and the assumptions in your reading. Ask for inputs you don't have (hover power, target size). Don't fill them from a rule of thumb presented as fact.
   - When nothing stored can stand in for the outcome, say the search can't filter on it, and read the strongest candidates' research record or maker page for it.
4. **Set the strength.**
   - **Hard:** the person's own limits and their necessary consequences (6S means at least 25.2 V charged).
   - **Soft:** estimates, translated figures and "nice to have". They rank; they never exclude.
5. **Reading conventions.** When you pass no `assumptions`, the defaults apply: a cell count with no chemistry is read as LiPo, and each category's own (an ESC's unqualified rating is the maker's standard continuous rating, and so on). Name the ones that decide a result in your reading. A list you pass replaces the defaults; `[]` means stated conditions only.
6. **Thin data.** If a property you need has few values, say so in your reading: most candidates will come back "not established" for that reason. If it has none in this category, no part can pass it. Say that the directory can't confirm it for any part there, search on the other requirements all the same, and put the question under "Before you buy" for each pick.

## 3. Search and read the results

**`search_parts`** scores every catalogued part in the category: `supported_match`, `conditional_candidate` (nothing fails, but something is unknown, conflicting or has unstated conditions: the `gaps` say what) or `excluded`.

**Unknown is not no.** Never present a conditional candidate as a match. Never drop one silently either.

- **The list is complete,** never a sample: every match and every conditional candidate (exception: section 4), one row per model. `counts` are per model: say "models". Excluded parts are counted; listed only with `include_excluded: true`.
- **Rows are ordered** by verdict, then by how many hard requirements they leave open. Rows after the first 15 are compact (each result is a string starting with its outcome): count over all the rows before you say how many parts meet something.
- **Forms.** Use `forms` to restrict to e.g. `four_in_one`. Many parts have form `unspecified`: add it when you can't afford to miss them.
- **The results page.** `search_parts`, `find_alternatives` and `check_build` write a page (`report_html`) with every evaluated part, its evidence and offers. Pass `title`: the person's question in a few words.
- **Pin your picks.** Once you've chosen, call `pin_picks` with the `report_html` path or link, before writing the answer.
  - Pin the top pick (role `pick`, one at most) and the other parts you recommend (role `option`), each with the same one-line `why` and up to 3 `before_you_buy` points you give in chat.
  - Pinning puts your picks at the top and changes no verdict or count; call it again if they change. Locally it also opens the page (`opened` true); hosted, nothing opens.
  - No picks: don't call it. Then, or when `opened` is false, give the link without saying it's open.

**The other tools:**
- **`get_part`**: every value with its source, date and conditions, plus the research record (`legacy_text`), research notes and origin evidence. On a family it returns the members.
- **`explain`**: why a part got its outcome on one requirement. Use it whenever you justify a verdict, an exclusion or a conflict.
- **`find_alternatives`**: ranks replacements by the category's drop-in rules: `drop_in`, `near_drop_in` (needs an adapter, firmware or mounting change), `possible` (something couldn't be checked), `not_alternative`. Pass the person's hard requirements as `requirements`. Read its `warning`: if the original lacks values, the ranking can't mean much.
- **`lookup`**: finds parts, families and companies by name, alias or order code. Resolve the person's named parts with it first. A name that matches several records needs the person's choice (in a build check, see section 6).
- **`read_maker_page`**: the maker's own page text, for a value the catalogue doesn't hold.

**Before writing "not published"** for the pick, or for any value that decides the answer: look at the part's research notes and `legacy_text`, then read its cited maker page. If one states the value, give it with the link.

**How strong is a match?** Maker evidence for the exact model is strongest. Values read by convention, from shop pages, stated as a best case ("up to") or company-wide are weaker, and the answer says which applies: [references/evidence.md](references/evidence.md).

## 4. Origin and compliance

Company base, design country, assembly, finished-product manufacture, component origin and shipping warehouse are six different facts. Never let one stand in for another.

**Use one ladder for origin,** strongest first, and always name the rung next to the result:

| Rung | Say |
|---|---|
| Stated for this model | "made in the USA (the maker states it for this model)" |
| Stated for the family or company-wide | "made in Germany (company-wide statement, not model-specific)" |
| Only assembly, a region or a factory stated | "assembled in the USA; where it's made isn't stated" |
| Not stated; maker is not Chinese/HK | "? maker is US-based: confirm the factory" |
| Not stated; maker's base unclear | "? not established" |
| Not stated; maker is Chinese/HK | count only, unless the person asks |

- **Company base is not where a part is made:** a US company can build in China. Say it once.
- **"Not a Chinese company" is about the company.** Don't turn it into an origin requirement. Search without one, read each row's `company` tag, leave `known_chinese_company` rows out of the table (give their count) and show the base as a pass: "✓ Canadian company".
- **"Not made in China" searches** fall back to company base when origin isn't stated. Non-Chinese makers with no published factory are the useful shortlist (with the question for the maker), never "not made in China". Chinese/HK makers are a count only (`hidden_chinese_makers`): the list omits them and the page hides them. Pass `include_chinese_makers: true` only when the person explicitly asks for Chinese companies that may build elsewhere.
- **Compliance.** The catalogue holds makers' own NDAA, Blue UAS, ITAR and export-class statements, and the official Blue UAS list as saved on a stated date.
  - Write "the maker states NDAA compliance", never "NDAA compliant". A statement is not a certification.
  - The official list is a registry record: give the date it was saved (the entry's `reviewed_on`). Say "not found on the list as saved on <date>", never "not listed".
  - No statement means "not found on the maker's pages", never "not compliant".
  - Hedged statements ("NDAA version available") and company-wide ones never pass a hard requirement: report them as what they are.
  - **Match the scheme to the customer.** NDAA, Blue UAS and TAA are US schemes; ITAR and ECCN are US export controls. For a customer outside the US, say they may not be the test that customer applies, and suggest asking which declaration it accepts.
  - **Crossing a border.** When a pick would ship from its maker's country to another, give its export class and ITAR statement, or put "export class not found: ask <maker>" under "Before you buy".
- **For a purchase that depends on origin or compliance,** recommend a written declaration for the exact model. Draft the question; never send it.

Details, the tool fields behind each rung, and lifecycle and environmental ratings: [references/origin-and-compliance.md](references/origin-and-compliance.md).

## 5. Buying

- **`check_availability`** for the parts you recommend, with the person's quantity and destination. Use the default for the pick and `coverage: "summary"` for the other rows of the table.
- Give each price with its seller, currency and observation date. Don't convert currencies silently or present a price as a delivered total.
- A `probable` match is a lead, never purchase-ready. No matched listing is not evidence that the part is unavailable.
- **`where_to_buy`** for "where can I buy X": lead with the sellers the brand itself lists, then the best unchecked places for the destination, each with its search link.
- Never treat a blocked page as out of stock. Never place orders, upload a parts list or send enquiries without the person's instruction.

Statuses, order codes and how to describe what was read: [references/buying.md](references/buying.md).

## 6. Checking a whole build

When the person has a parts list (their own BOM, or picks from several searches), call **`check_build`**. It reads the parts together and answers "where is this build exposed?".

Pass `parts: [{part, quantity?, role?}]` (ids or names), `cells` when no battery is in the list, `exclude_origin` (default `["CN"]`) and a `title`. Don't ask for confirmation first: state what you assumed (the excluded origin, the cell count) in the answer. If a line comes back in `unresolved`, find it with `lookup` and run the check again with its id. When the name fits several records, use the closest and name the record you used in the answer; ask only when the candidates differ in maker or in where they are made. If a line still can't be found, check the rest, say which line was left out, and offer a search for it.

It returns:
- **`provenance`**: each line's rung on the origin ladder, rolled up by count and by cost. Three numbers lead: `confirmed_lines`, `not_settled_lines`, `exposed_lines`.
- **`compliance`**: per topic, which lines have a statement for the model, a company-wide one, a hedged one, a "no", or none.
- **`fit`**: checks between the parts, each `ok`, `problem` or `not_checked`: pack voltage, ESC current and channels against the motors, propeller against frame and shaft, mounting patterns, radio and video systems, antennas, GNSS against the flight controller, thrust against mass.
- **`lifecycle_flags`**, **`environment`**, **`totals`**: discontinued lines, the temperature range the parts share, and mass and cost with the lines missing from each.
- **`chase`**: one drafted message per maker asking only what the catalogue lacks.
- **`swaps`**: for each exposed line, alternatives from non-Chinese makers in the same size class.

**The answer has four parts:**
1. **The page link, first line.** Then the direct answer to what they asked, in one sentence ("Not yet: 3 of the 9 lines are from Chinese or Hong Kong makers"), and the three provenance numbers.
2. **Fit problems,** then what couldn't be checked and which value is missing. `not_checked` is not `ok`. Give the ok checks as a count, not a list.
3. **What to do next:** the exposed lines with their best swap (a swap is a lead, not a drop-in: give its fit grade and its own rung), and "I've drafted the questions for each maker; they're on the page for you to send."
4. **What I checked,** three lines:
   - **Checked:** "9 of 10 lines; 17 fit checks: 10 ok, 2 problems, 5 not checked."
   - **Not checked:** flight time (hover power isn't in the catalogue), wiring and firmware configuration, current stock. Thrust-to-weight runs only when every motor publishes a maximum thrust, and uses the makers' static figures and only the masses on the list.
   - **Evidence:** "Compliance entries are the makers' own statements; the Blue UAS list is as saved on <`blue_uas_list_saved_on`>."

**Keep it to about 300–400 words.** The page holds every line; the answer holds the verdict, the lines to fix and the next step.

Keep the ladder's wording: "company-wide statement" is not "made in". For current stock and price on the lines that matter, run `check_availability`. For a line the person wants replaced, run a fresh `search_parts` with their real requirements.

## 7. Answer format for a search or alternatives

An engineer should be able to decide from the first screen. **Organise by part, not by topic:** everything about one part sits in one place, and each fact appears once.

**Six parts, in this order:**

1. **The results page link, always the first line,** before any heading: "**[Open the results](https://…)**: all 99 suitable parts (4 confirmed, 95 with something to check)." When `pin_picks` says the page is open, add "(it's open in your browser, with my picks at the top)". The counts in this line are the full counts; never call a shortlist "the results". When every match rests on a convention or on shop listings, write "7 meet both on the makers' own wording", not "confirmed". With several searches, put every link at the top, one line each. If you can attach files, attach the page too.
2. **Your reading, in one or two lines (at most 40 words),** starting "Read as:": the hard limits as searched, what you translated or assumed, and what you left open.
3. **The answer (1–2 sentences).** Name the pick and the one reason, then the one thing to confirm before buying. Lead with what is true, not with what couldn't be confirmed. The one exception: when a hard requirement can't be confirmed for any part, say that first in one sentence, then name the pick on everything else.
4. **The comparison table.** Columns are the person's requirements plus **Stock**, and nothing else. Origin and compliance get a column only when the person asked about them; otherwise the pick's origin goes in its block.
   - **Rows:** every confirmed match, with no cap, then the strongest of the rest, up to about 8 rows.
   - **Cells:** ✓ (confirmed), ? (not established) or ✗ (fails), each with a few words: "✓ up to 8S", "? not published", "✓ USA (this model)". Link the model name to its source. The Stock cell is the store, the price, the date and "in stock" / "out of stock" / "not found in the stores read", linked to the listing.
   - **Below it:** "+ N more on the results page" whenever rows were left out.
5. **One short block per recommended part,** only for the top 1–2. **Why:** one line. **Before you buy:** at most 3 bullets. **To use it:** firmware, protocol and connectors in one line, if known.
6. **What I checked: 3 lines, always this shape.** Use `coverage.summary` from the search and from the availability check, reworded only for flow.
   - **Searched:** how many parts, against what; then what wasn't covered.
   - **Stock:** which stores were read and when; then the best 2–3 unchecked places for the destination, with links.
   - **Not established for any part:** the values nobody publishes.

**For "not made in China",** the origin column shows the rung, the makers to ask come after the table with the question to send, and the Chinese/HK-company count goes in "What I checked".

**Then stop.** Every part, with the excluded ones and their reasons, is on the results page: don't paste long lists into chat. Material exclusions get one line each.

**Keep it short.** About 200–350 words plus the table. Each "What I checked" line is one sentence plus links. Put the full store list, every gap and every source behind "Ask me for…" lines.

**Write for the engineer, not about the tools.**
- **Never describe the machinery.** Avoid "scored", "conditional candidate", "passes on accepted evidence", "legacy_text", tool names and ids. Say what is known: "rated 3–8S (maker's page, checked 2026-10-28)".
- **Plain words for unknowns:** "not published" or "not established", with no hedging chains.
- **Cite each fact once,** where it is used. Don't add a separate sources list.
- **Ask only questions that change the answer,** at most one, at the end.
- **Say how strong each match is.** "Confirmed" only for a match on the maker's own evidence for that model. A match read by convention is "✓ 55 A (maker's rating; cooling and duration not stated)"; one from a shop page is "✓ … (shop-listed)". For alternatives, "drop-in" only for the `drop_in` tier.

A factual follow-up gets a direct answer, not this structure.

### Example: "4-in-1 ESC for 6S, not made in China, UK"

The numbers are from one run on 10 November 2026; yours will differ.

> **[Open the results](file:///Users/…/Documents/UAV%20Parts%20Reports/20261110-101425-6s-4-in-1-esc-not-made-in-china.html)** (it's open in your browser, with my picks at the top): all 99 suitable 4-in-1 ESCs, 4 confirmed and 95 with something to check.
>
> *Read as: a 4-in-1 ESC for a 6S LiPo pack (25.2 V charged), finished product not made in China, shipped to the UK. You gave no current rating, so none was required.*
>
> **Pick: [ARK 4IN1 ESC](https://arkelectron.com/product/ark-4in1-esc/).** The maker states it's made in the USA, and it takes up to 8S, so 6S sits mid-range. ARK's own store lists it in stock at $218.50; confirm UK shipping with them.
>
> | Model | 6S | Not made in China | Current per motor | Stock |
> |---|---|---|---|---|
> | [ARK 4IN1 ESC](https://arkelectron.com/product/ark-4in1-esc/) | ✓ up to 8S | ✓ USA (this model) | 50 A cont. / 75 A burst | [ARK](https://arkelectron.com/product/ark-4in1-esc/), $218.50, in stock (seen 9 Nov) |
> | ARK 4IN1 ESC CONS | ✓ up to 8S | ✓ USA (this model) | 50 A cont. / 75 A burst | ARK, $238.50, in stock (seen 9 Nov) |
> | Ewing Aerospace 80A 4-in-1 | ✓ up to 12S | ✓ USA (this model) | 80 A cont. / 100 A burst | no price tracked |
> | StingBee Stellar AM32 V1 80A | ✓ up to 8S | ✓ Ukraine (this model) | ? not published | not found in the stores read |
> | XTRAFLY AM32 6S 65A | ✓ 6S | ? maker is Denmark-based: confirm the factory | 65 A cont. / 75 A burst | not found in the stores read |
>
> + 94 more on the results page.
>
> **ARK 4IN1 ESC.** *Why:* model-level US origin, a published current rating and a price I could read. *Before you buy:* the burst duration and cooling aren't stated; check them against your motor's peak. Confirm quantity and UK shipping with ARK.
>
> **Ewing 80A** is the choice above 8S or 50 A per motor; ask Ewing for price and lead time.
>
> **What I checked**
> - **Searched:** all 359 ESCs in the directory (106 are 4-in-1) against 6S and not made in China; 7 fail. Not covered: 10 brands listed by name only (e.g. FLYWOO, GEPRC) and anything outside the directory.
> - **Stock:** prices are from ARK's own product pages, read 9 November. No store feed could be read today. Not checked: 52 other places; for the UK try [Flying Tech](https://duckduckgo.com/?q=site%3Awww.flyingtech.co.uk+ARK+4IN1+ESC) and [HobbyRC](https://duckduckgo.com/?q=site%3Awww.hobbyrc.co.uk+ARK+4IN1+ESC).
> - **Not established for most:** where the model is made. 23 are from non-Chinese makers, 62 from Chinese/HK companies and 6 from makers whose base is unclear.
>
> What's your motor's peak current? Above 50 A per motor, the Ewing 80A is the one to price.
