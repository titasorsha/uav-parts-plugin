# Buying and availability

Read this when the person asks where to buy a part, what it costs or whether it is in stock.

## `check_availability`

It reads the tracked offers for one part: store catalogue feeds, and prices and stock read from makers' and stores' own product pages. Pass the person's quantity and destination.

- **Two sizes.** `coverage: "full"` (the default) lists every store read and the places not checked: use it for the pick. `coverage: "summary"` returns the offers, the status and one sentence on what was read, at about a fifth of the size: use it for the other rows of a comparison table.
- **Statuses:**
  - `available_for_quantity`: the exact part, a stated quantity and stated shipping to the destination.
  - `in_stock_unverified`: the exact part is in stock, but no quantity and/or no shipping countries are shown. Say exactly that, give the price, the observation time and the listing link, and tell the person to confirm quantity and shipping.
- **Prices** come from the store's feed or its product page, in the store's currency, with tax unknown. Give each price with its seller, currency and observation date. Don't present it as a delivered total, and don't convert currencies silently.
- **Age.** An offer may be days old when a store rate-limits reading: give the age.
- **Probable matches** are listings whose exact variant or order code isn't confirmed. Mention them as leads ("probably this part"); they never make a part purchase-ready.
- **No matched listing** means only that no tracked store lists it. It is not evidence that the part is unavailable.
- **Order codes.** `get_part` lists each variant's order code with its basis and source. Codes marked "SKU in the maker's own web store" are the maker's store SKU: quote it as the order code, but don't call it a datasheet part number. When a part has several options (region, connector, pack size), name the one that fits the requirement, with its code.

## `where_to_buy`

For "where can I buy X". It lists every relevant place with a search link:

- **`official_distributors`** first, when present: the sellers the brand itself lists on its own dealer page, ordered for the destination, each with a link that searches that seller's site for the part. Their stock and prices were not checked.
- the maker's store;
- tracked stores, with the time they were read and the result;
- other drone shops, electronics distributors, aggregators and marketplaces that were **not** checked, each with the reason.

Recommend the most relevant unchecked places for the destination: region-fit places first, then the maker's own site and its authorised resellers. Say when a channel is marked `added_unverified`.

## Saying what was checked

Take the numbers from the `counts` block (`known_relevant_places`, `checked_live`, `checked_live_complete`, `not_checked_by_reason`) and use `coverage.summary`, reworded only for flow. Always add that sellers outside the directory were not considered.

These mean different things; keep them apart:

- **found (exact):** a listing's SKU or order code equals the part's.
- **found (probable):** the listing title names the part, but its exact variant is not confirmed.
- **carries the brand but not matched:** the store sells that maker, but no listing was matched. Say "may be listed under another name; search link given". It does not mean the store lacks the part.
- **feed lists no products of that brand:** the full feed was read and that maker is absent.
- **partial:** the read stopped early, so absence there proves nothing.

Never say "I searched widely" or "no sellers have it". Say "3 of 9 relevant stores were read; none of the 3 lists it".

- When the counts differ by part, give the pick's numbers.
- A store that was read, carries the brand but didn't match comes with its own search link (`check_here`): put it in the Stock cell, not in a sentence of explanation.
- The full list of places with reasons is available on request. Don't paste it.

## Parts or sellers the tracked stores don't cover

- **If browsing is available and buying matters:** check the actual seller's offer. Confirm the exact SKU or suffix, stock against the quantity needed, the destination, minimum order, the price with its currency and tax basis, and the dispatch basis. Give each observation its date.
- **Otherwise:** give the technical result plus the maker or official buying routes from `get_part`, and mark availability **not checked**.

Never treat a blocked page as out of stock. Never place orders, upload a parts list or send enquiries without the person's instruction.
