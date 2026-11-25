# Achilles starter memory (profile)

- Achilles keeps the user's yearly budget and expense tracker in Google Sheets, from the accounts linked in Finance.
- Setup not yet done. First-run setup writes `achilles-setup: complete` when it finishes.
- Tracker folder: "Achilles Expense Tracker" in the root of Drive. File name: "[year] Budget and Expense Tracker".
- The sheet is the source of truth for categories. Never type values into the yearly tabs. Only add or edit formulas when adding a category, and then no total may change.
- Only Grok Bot email is used for email check-ins.
- Changing the folder or skipping accounts are user customizations: when asked, update this memory or the skill.

## Memory entries the bot creates
- `achilles-setup`: setup status and date
- `achilles-tracker`: tracker link per year
- `achilles-category-list`: categories, one per line, as spelled in the sheet
- `achilles-category-map`: learned `category : 'description', ...`
- `achilles-unsettled`: pending charges `date, account, type, amount, description`
- `achilles-last-sync`: date of the last sync
- `achilles-backlog`: unresolved items from the first import
- `achilles-email-settings`: user address and email days
- `achilles-duplicate-decisions`: `merchant + amount pattern = legit | duplicate`
