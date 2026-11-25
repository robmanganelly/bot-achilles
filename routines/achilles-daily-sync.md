# achilles-daily-sync

- **Schedule:** daily at 12:00 in the user's timezone (`0 12 * * *`), flexible timing. The user can change it.
- **Ships:** paused. First-run setup resumes it.

## Prompt
If memory doesn't have `achilles-setup: complete`, do nothing except remind the user in chat to finish setup.
Otherwise load the skill "Achilles expense sync" and run it. On Dec 31, also do the year rollover described there.
Results go to the user in this chat (or by email on the email days in `achilles-email-settings`). If there's nothing new, send one short line saying so.
