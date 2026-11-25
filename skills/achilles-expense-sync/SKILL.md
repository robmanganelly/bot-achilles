---
name: Achilles expense sync
description: How one Achilles sync run pulls, maps, records and reports transactions; also the Dec 31 year rollover.
---

# Achilles expense sync

## Read first (memory)
`achilles-tracker`, `achilles-last-sync`, `achilles-category-list`, `achilles-category-map`, `achilles-unsettled`, `achilles-duplicate-decisions`, `achilles-backlog`, `achilles-email-settings`.

## 1. Pull
- Pull transactions since `achilles-last-sync` from **every** account linked in Finance, loans included.
- Loan payments are recorded as expenses.
- **Exclude** transfers between the user's own linked accounts and credit-card payments. They're a net zero and would add a fake expense plus a fake income.
- **Skip already-written items**: same bank transaction id, or the same date + account + amount + description already in the sheet.

## 2. Possible duplicates
Two or more charges that look alike (same merchant and amount, close dates):
- If `achilles-duplicate-decisions` already has a decision for that merchant + amount pattern, apply it silently.
- Otherwise alert the user right away and let them decide. Two identical gym memberships, for example, can both be real.
- Save the answer as `merchant + amount pattern = legit | duplicate`, so the same recurring pair isn't raised every month.

## 3. Map
Follow **Achilles categories** for matching and any new categories. Ask about unclear items with small, focused decision cards. Note the answers, and write **nothing** to the sheet or memory until every card is answered.
If today is an email day in `achilles-email-settings`, ask through **Achilles email channel** instead.

## 4. Pending charges
- Write pending charges as if settled (map and apply), and log each one in `achilles-unsettled`: `date, account, expense|income, amount, description`.
- On every run, re-check the log. Settled: remove the line. Refunded, declined or rolled back: find the row in the sheet, change the description to `[Refunded/declined]-<previous description>` and the amount to 0.00, then remove the line.

## 5. Write (two checked steps)
- **Step A: categories only.** Add or rename any categories (see **Achilles categories**). Formula edits are allowed only here.
  - Snapshot all totals first and compare them after: no number or total may change. Also check that the group SUM range covers the new row.
  - If anything differs, roll back and report.
- **Step B: entries only.** Append the expense and income rows to the month's transactions tab (Date, Amount, Description, Category). Never edit formulas.
  - Check that the month and year totals moved by exactly the amounts added.

Only ever write to the monthly transactions tabs, plus `01-year-setup!C13` and the header year. Never type values into the yearly tabs.

## 6. Report
Update `achilles-last-sync`, then send one short line, e.g. "2 expenses added, 1 income added, 1 charge still pending." If there's nothing new, say so in one line.

## Year rollover (Dec 31 run)
After the normal run:
- Copy the public template into **Achilles Expense Tracker** as `[next year] Budget and Expense Tracker`, and set its header year.
- Put that day's ending balance in the new file's `01-year-setup!C13`.
- Tell the user which value you used, so they can adjust it.
- Add the new link to `achilles-tracker`. The category list and map carry over, and the old file stays as it is.
