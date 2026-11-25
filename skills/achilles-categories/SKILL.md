---
name: Achilles categories
description: Achilles category rules - matching transactions, the learned map, and creating or renaming categories without changing totals.
---

# Achilles categories

The sheet is the source of truth. `achilles-category-list` mirrors it, one category per line, spelled exactly as in the sheet. Match names case-insensitively, but write them with the sheet's spelling.

## Matching order
1. **Learned map** (`achilles-category-map`): `category : 'description', 'description'`. Use it first, so recurring payments never get asked about twice.
2. **Obvious match**, e.g. "Example Pet Spa" goes to Pets-Other (or a pet grooming category if one exists).
3. **Ask the user**: "Do you recall what this was for?" with a few candidate categories.
   - "No idea" goes to the closest group's `-Other` category (e.g. `Everyday-Other`). The sheet's Other group has no general category of its own, only `custom-` ones.
   - If their answer fits no existing category, offer the closest existing ones, plus 1-3 new category suggestions, plus a free-text option. Normalize whatever they type to the naming rules.

For a recurring item (e.g. the same amount sent to the same account every month), ask: "Want me to remember this as <category>?" If yes, add it to the map. Never map one-off charges; if in doubt, ask.

## Naming new categories
- Inside a group: `[group]-custom-[name]`, e.g. `Transportation-custom-ferrari-payments`.
- No evident group: `custom-[name]`. It shows under the Other group in the yearly sheet; never `Other-custom-...`.
- The `custom` tag is mandatory for every category we create. The group must be one of the sheet's existing groups.
- Income categories follow the same rules (e.g. `Wages-custom-[name]`).
- Keep categories coarse; don't create one per merchant.
- If the user names it themselves ("call it foo"), keep their name inside the format: `custom-foo`, or `[group]-custom-foo` when the group is obvious.

## Creating or renaming (step A of the sync)
Order: list, then map, then sheet. No entries get written in the same step.
1. Update `achilles-category-list`.
2. Update `achilles-category-map` if needed.
3. In the sheet, insert a row inside the group's block on the yearly tab (02-year-expenses or 03-year-income), copying the neighbouring row's formulas. Add the name in column B of every monthly summary tab, in the right block, with the row's formulas.
4. Verify: every total is identical to the snapshot taken before the change, the group SUM ranges include the new row, and there's no `#REF!` anywhere. If not, roll back and report.
