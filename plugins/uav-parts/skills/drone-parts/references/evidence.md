# Reading the evidence

Read this when you explain why a result came out as it did, or when a value rests on something weaker than the maker's own word for the exact model.

## Evidence kinds

- **Accepted for passing:** `manufacturer_claim`, `manufacturer_doc_mirrored`, `integration_doc`, `certification_record`, `independent_measurement`.
- **Never pass on their own:** `reported` (a third-party page) and `reseller_claim`. They can still turn a result into a conflict.

When a result rests on reported evidence, say so.

## Shop-listed values (`shop_listed`, default true)

Hobby FPV makers (props, motors, batteries, frames, small servos, FPV GPS) rarely publish specs, so most of those parts' values come from shop pages. `search_parts` and `find_alternatives` count them by default, only where the maker states nothing.

- Each such value is flagged `shop_listed`.
- A pass that rests on one gives `shop_listed_match` ("Meets spec (shop-listed)"), never `supported_match`.
- A fail that rests on one gives `shop_listed_excluded`: hidden like excluded, counted apart. If the list of matches is short, say how many failed only on shop listings. In alternatives such a part is `not_alternative` with `ruled_out_on_shop_listed`.
- In the answer, keep the two lists apart: "Maker-confirmed: … Shop-listed only: …". Say the shop-listed values should be checked with the maker or on the part.
- For industrial or compliance-driven buys pass `shop_listed: false` (maker evidence only) and say so.

## Best-case figures and rated maximums

- **Best case.** A rating the maker gives as "up to X" or "as low as X" (radio range, RF power, latency, data and update rates, frame rate, zoom, burst current, TOPS) is stored as X with the gap "stated as a best case". It can rule a part out (best case 20 km can't meet 30 km), but a pass stays conditional, and the results page marks the cell. Pass `accept_best_case: true` on a requirement only when the person says maker best-case figures are fine, and say so in the answer. Never present a best-case figure as a guaranteed one.
- **Rated maximum of a range.** A rangefinder range or an airspeed speed range stated only as "up to X" is stored as max X with `range_limit_basis: rated_maximum`. Present it as "rated up to X (maker's figure, favourable conditions)".
- **Maximums by definition** (max prop size, MTOW, payload, motor max power and max thrust, counts such as "up to 8 PWM outputs") store "up to X" plainly: there it is the rating.

## Values the tools work out

- **ESC motor channels** without a claim come from the recorded form: single 1, 2-in-1 2, 4-in-1 4. AIO boards are not derived.
- **Charged voltage limit** comes from the maker's operating voltage maximum when published, otherwise from the cell range.
- **FC IMU count** without a claim is counted from the listed sensor parts and is a floor: it can pass "at least N" and never fails a requirement. Lists giving alternatives ("BMI270 or ICM42688P") are not counted.
- **Battery continuous current** is derived from C rating × capacity when only C is published.
- **A bare GNSS receiver module** (form `smt_module`) is its own receiver.
- **A radio band named on the page** ("2.4GHz", "915MHz") is stored as that band's standard allocation with `band_basis: named_band`. Say so when it decides a match; `stated_range` means the page gave the numbers.

## The directory's research record (`legacy_text`)

`get_part` returns `legacy_text`: researched records that haven't been converted into typed values yet. Each row comes from a maker profile and carries its primary-source links and review date.

- **Use it.** State its figures directly, with their conditions ("66 A continuous, 110 A for under 3 s"). Cite the link in the row.
- **Say what it is,** once per answer: "from the directory record (reviewed <date>), not machine-checked".
- **Respect what the row says it doesn't establish.** "Not captured", "conflict" or "verify" mean exactly that.
- **Don't downgrade a row's figure to "unverified" just because it wasn't converted.** Verdicts and gaps only cover typed values. When a decisive figure exists only in `legacy_text`, give the figure and say the search couldn't score it.
- For a broad search, read the `legacy_text` of the strongest candidates too: a part can be the best fit even when it scored conditional only because its rating wasn't converted.

## Research notes (`research_notes`)

The maker profiles' prose (integration and purchasing sections, firmware documents, rating caveats, sales-route and lifecycle changes) is kept as verbatim notes with links and a research date. They are not typed data and are never scored.

- `explain` adds the notes that mention the property when the result is unknown, conflicting or rests on a convention. `get_part` lists the part's notes, then the maker's notes for its category. `company_screen` lists company, origin, purchasing and lifecycle notes. Search rows show only counts.
- **Read them before writing "not recorded" or "not established".** If a note answers the question, give it, worded as research: "The directory research (2026-10-28) says … ([link])".
- A note never changes a verdict. If a note disagrees with typed data, show both and say which is the maker-page claim.
- Notes about the maker in general may not apply to every model; say so when you rely on one for a specific part.

## Wording the strength of a match

- **"Confirmed"** only for a supported match with no `meets_by_convention` and nothing shop-listed.
- A row with `meets_by_convention` meets the requirement on the maker's unqualified rating: "✓ 55 A (maker's rating; cooling and duration not stated)", never "confirmed".
- Shop-listed passes are "✓ … (shop-listed)".
- **Multi-motor boards and "sustained" currents.** A per-motor rating times the channel count is not a board rating. When the person asks to sustain a current on every motor, through the supplied connectors, or for a duration, check the connector ratings, the total or input current and the thermal or duration basis. Say which are published and which aren't. Don't call a board suitable on the per-motor label alone.
- **Alternatives:** "matches" or "drop-in" only for the `drop_in` tier. For anything else, name what differs or isn't established.
- **Quote recommendations and pairings as written.** "55A+ HVC recommended / 75A+ alternate" is two recommendations with their thresholds: keep both.
- **Per-SKU pages.** When a source gives a page per SKU, cite the right page for each SKU.
- **Numbers in a product name** ("Siege 80A 3-8S") are not ratings unless the tools return them as a value: show them as "? '3-8S' in the name only", once.
