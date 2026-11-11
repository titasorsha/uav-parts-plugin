# Origin, company base and compliance

Read this when a request involves where a part is made, who makes it, or what its maker declares (NDAA, Blue UAS, ITAR, export class), and for discontinued parts and environmental ratings.

## Six different facts

Keep them apart: company base, design country, assembly, finished-product manufacture, component or cell origin, shipping warehouse. A US or German company can build in China; a part assembled in the USA can hold Chinese boards.

## Company base (`company_screen`, `list_companies`)

- `company_screen` answers "Is X a Chinese company?" with yes / no / unclear (Hong Kong counts as yes). Give the country and the evidence link. Do not extend that answer to where products are made.
- Check `screen_evidence_kinds`. If it is only `reported`, the screen rests on a third-party page: say so.
- `list_companies` gives the screen for every brand in a category at once. Use it for "which X makers are Chinese?".
- Every part row carries a `company` tag: `known_chinese_company`, `non_chinese_company` or `company_base_unclear`, with the head-office country in `hq`. It is about the company, not the part.
- **"Not a Chinese company"** has no requirement to pass. Search on the other requirements, then use the tag: keep `non_chinese_company` rows (cell "✓ Canadian company"), show `company_base_unclear` as "? base not established", and leave `known_chinese_company` rows out with their count in "What I checked". If the person also asks where the part is made, add the origin requirement and use the ladder below.
- **Background.** `company_screen` returns `background` (`get_part` has a compact `company_background`): founding year, head-office city, the company's own one-line description, the markets it says it serves, staff count, parent company. Each is quoted from the company's own site with a retrieval date. Present it as what the company says about itself: it is not verified, it never decides a verdict, and a market such as `defence_security` is not a certification. If `background` is null, say nothing is recorded.

## The origin ladder

The tools describe origin evidence in three places with three sets of labels. In answers, use the one ladder in the main file. This table maps the tool fields to its rungs.

| Rung (wording for the answer) | `get_part` `origin_evidence_status` | `search_parts` `origin_tier` | `check_build` `tier` |
|---|---|---|---|
| Stated for this model | `model_claim` | `origin_stated` | `origin_stated` |
| Stated as made in an excluded country | `model_claim` (value is the excluded country) | `made_in_china_stated` | `origin_excluded` |
| Stated for the family (this model is a listed member) | `family_claim` | `origin_stated` | `origin_stated`, with `level: family` |
| Company-wide statement only, not this model | `company_claim` | `origin_stated` when the statement is not hedged (it counts by default, and the row's `assumed` says "company-wide statement is counted"); otherwise one of the three "not stated" tiers | `company_statement`, `company_statement_excluded` |
| Only partly stated | `assembly_claim`, `region_claim`, `mixed_components`, `facility_only` | — | `assembly_only`, `origin_one_of` |
| Not stated; maker is not Chinese/HK | `unknown` | `non_chinese_company` | `non_chinese_company` |
| Not stated; maker's base unclear | `unknown` | `company_unclear` | `company_unclear` |
| Not stated; maker is Chinese/HK | `unknown` | `chinese_company` | `chinese_company` |

What the partial rungs mean:
- `family_claim`: only the listed family members. A shared family name does not share an origin.
- `company_claim`: a company-wide statement for that category. Verify the exact model.
- `assembly_claim`: assembly only, not where the parts are made.
- `facility_only`: a factory location with no product tied to it.
- `region_claim`: e.g. "EU"; never an exact country. It still settles "not made in China": EU, EEA and USMCA are compared through their member countries.
- `mixed_components`: components from several countries; finished origin unknown.

## "Not made in China" searches

Makers rarely publish where each model is made. When a part's origin isn't established, `search_parts` adds `origin_tier` and `origin_fallback`, and `origin_tiers.summary` gives the counts. Present three groups, in this order:

1. **Origin stated for this model:** confirmed. Give the evidence level.
2. **Made by a non-Chinese company, factory not published:** the useful shortlist. Show the company base and its evidence. Give the person the question to send, from `to_confirm`: "Ask <maker> where this exact model (order code) is manufactured, ideally in writing." Never call these "not made in China".
3. **Chinese/HK company, factory not published:** give the count only, from `hidden_chinese_makers`. Don't recommend them for this requirement.
   - When "not made in China" is a hard requirement, `search_parts` leaves these models out of its list, and the results page hides them behind a "Chinese/HK makers" tick box. They still count in `counts` as conditional candidates. They are not excluded and not "made in China": their origin is unknown.
   - Say it in one line: "<N> more are from Chinese/HK makers that don't publish where the model is made; they are hidden on the page."
   - Pass `include_chinese_makers: true` only when the person explicitly asks to include Chinese companies that may build outside China. Then they are listed and shown on the page, still as "needs checking". The search ranks them after the other makers, so give them their own rows in the table, below the non-Chinese shortlist, with the origin cell "? Chinese/HK maker: ask where it's made". Say how many there are, and draft the question for the maker: where the exact model is made.
   - A preference (a soft origin requirement) hides nothing.

Finished-product origin is not component origin. For "no Chinese components", a passing origin claim settles only where the product was made. Receiver chips, cells and antennas need their own evidence: look in the part's `legacy_text` and say what remains unknown.

## Compliance

| Property | What it is | Kind |
|---|---|---|
| `compliance.ndaa_declaration` | the maker says the part is, or is not, NDAA compliant | maker's statement |
| `compliance.blue_uas_cleared` | the part is on the copy of the official Blue UAS Framework list saved on a stated date, matched by exact name | registry record |
| `compliance.blue_uas_statement` | the maker says the part is on a Blue UAS list | maker's statement |
| `compliance.itar_controlled` | `false`: the maker says ITAR-free. `true`: ITAR-controlled | maker's statement |
| `compliance.eccn` | the export class the maker publishes (EAR99, 7A994…) | maker's statement |
| `compliance.taa_declaration` | the maker says TAA compliant | maker's statement |

The maker's quote is in the claim's `scope.statement_scope`.

- **Search:** `{property: "compliance.ndaa_declaration", op: "eq", value: true}`. Only a statement about the exact model passes. A company-wide statement counts only with `broadest_scope: "organization"`, and still comes back conditional.
- **Coverage is thin.** Read the live counts from `describe_category`. No statement means "not found on the maker's pages we saved", never "not compliant". Say so.
- **Hedged statements** ("NDAA version available", "most products", "pending") never pass. Report them as what they are.
- **The official list is dated.** Always give the date it was saved: the entry's `reviewed_on` in `get_part` (its `statement` gives the date the part was added to the list), or `blue_uas_list_saved_on` in a build check. A part missing from it may be listed under another name: say "not found on the list as saved on <date>", never "not listed". When a maker says a part is listed and the saved list doesn't show it, say both.
- **In the answer:** "the maker states NDAA compliance (its own page, dated)", never "NDAA compliant". For a purchase that depends on it, recommend a written declaration for the exact model, and checking the current official Blue UAS list. NDAA or TAA wording is not a country of origin.
- **Which scheme applies.** NDAA (US defence authorisation acts that bar the US defence department from buying drones and certain drone components from covered countries, China among them), the Blue UAS list (the US defence department's list of cleared drones and components) and TAA are US schemes. ITAR and ECCN are US export controls: they matter to anyone, in any country, who buys or re-exports a US-controlled part. A UK or EU customer may apply a different test. When the person's customer is outside the US, say which statements exist, say they are US schemes, give the origin per model, and suggest asking the customer which declaration they accept. Don't say what a non-US customer's rule is: the directory doesn't hold it.
- **Crossing a border.** When a recommended part would ship from its maker's country to another, look at `compliance.eccn` and `compliance.itar_controlled` in `get_part`. Give them if present. Otherwise say "export class not found on the maker's pages: ask <maker>". Never say a part needs no licence.
- **Results pages** show NDAA, Blue UAS, ITAR and ECCN columns whenever a listed part has a value, marking official-list entries, makers' statements and company-wide statements.

## Lifecycle and environment

- `lifecycle.status`: `discontinued`, `eol_announced` or `nrnd`, from the maker's own page. A live product page doesn't prove current production, so no value is unknown, not "active".
- `physical.operating_temperature`: the maker's operating range (qualifier `limit_kind: operating`). Storage and charging ranges are separate facts.
- `physical.ingress_rating`: the IP code as published. Several codes ("IP54/IP67") are options or housings and are recorded as hedged.

A minority of parts publish these. Read the counts from `describe_category`, and say "not found on the maker's pages" for the rest.

Do not contact a maker, send an enquiry or request a declaration without the person's go-ahead. Draft the question for them.
