---
name: Achilles first-run setup
description: One-time Achilles setup (getting-started skill). Connect apps, create the tracker, import this year. Skip if setup is already done.
---

# Achilles first-run setup

This runs once, as Achilles' first conversation. Talk to the user in short, friendly steps, and use small decision cards whenever you need input.

## Guard (always first)
Check memory for `achilles-setup: complete`. If it's there, don't run setup again. Tell the user setup is already done, and offer to run a sync now or change a routine. Re-run setup only if the user explicitly asks for it.

## 1. Google Sheets
Check that the Google Sheets connector is installed and signed in. If not, stop and help the user install and connect it (show the connect card). Don't continue until it works.

## 2. Google Drive
Check the Google Drive connector the same way. When prompting, tell the user: "If you have more than one Google account, connect the one where you want your budget tracker stored." Continue only once Drive is signed in with that account.

## 3. Finance
Check the Finance connector (bank links through Plaid). If it's missing, help the user install it, and remind them to link **every** account they want tracked, loans included. Achilles can't see anything that isn't linked.
Once it's connected, list the linked accounts in a confirmation card and ask: "Is any account missing?" Wait until the user confirms the list is complete.

## 4. Create the tracker
- Use the folder **Achilles Expense Tracker** in the root of the user's Drive, and create it if it doesn't exist. Don't ask where; the folder is fixed. Changing it later is a user customization (update this skill or memory when the user asks).
- Copy the public template spreadsheet https://docs.google.com/spreadsheets/d/17TKTfYg6BM-kjq7qoVn-Zqgx4137l5jJkLGfvMeJ3ko into that folder as `[year] Budget and Expense Tracker`, using the current year.
- Set the month header dates on the yearly tabs (02-year-expenses, 03-year-income, 04-year-summary) to the current year.
- Save the file link and id to memory as `achilles-tracker` (one line per year: `year: link`).

## 5. Starting balance
Ask: "How much money do you want to start the year with?" Make clear it's optional: if they skip it, the year starts at 0. If they give a value, write it to `01-year-setup!C13`. That's the only cell a human value goes into.

## 6. Category list
Read every category name from the copied sheet (the expense categories on 02-year-expenses and the income categories on 03-year-income). Save them to memory as `achilles-category-list`, one per line, spelled exactly as in the sheet. Create an empty `achilles-category-map`.

## 7. First import (one-time backlog)
Tell the user you're now pulling this year's transactions from their accounts.
- Pull everything from Jan 1 to today, following the skill **Achilles expense sync** (same exclusions, duplicate checks and mapping rules).
- Auto-map everything that's clear. Count the unclear items.
- If there are many unclear items, tell the user the count and ask whether they have time now, or when. Make it very clear that this happens only once: after the backlog is cleared, Achilles runs with minimal interruption.
- Save the unclear items to `achilles-backlog` (one line per item) and cross each one off as it's resolved.
- Work **month by month**, starting with January. Resolve that month's unclear items, write that month to the sheet (both checked write steps), report the milestone ("January done: 42 expenses, 3 incomes"), then move to the next month.
- If the user stops, keep the backlog in memory and pick it up next session. Don't mark setup complete until the backlog is empty.

## 8. Chat or email
Ask: "Do you want my daily check-ins here in chat, or by email on some days?" If they choose email, follow **Achilles email channel**, setup section.

## 9. Turn on the routines
The routines ship paused. Resume:
- `achilles-daily-sync`: daily at noon in the user's timezone. Tell the user they can change the time.
- `achilles-monthly-budget`: on the 28th of each month. It stays quiet until there are 2 full months of data.
- `achilles-email-reply`: only if the user chose email. Fill in the bot's own Grok Bot inbox address and the user's address first.

## 10. Done
Write `achilles-setup: complete (date)` and `achilles-last-sync: (today)` to memory. Send the user the tracker link and a one-line summary of what happens next.
