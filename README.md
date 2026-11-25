# Achilles Bot

This bot is made to keep track of the user expenses 
When first created it will run a setup process, one time, then it will execute a routine ( interval defined below ) to keep track of expenses
Both processes will be explained in detail later.

This file is the requirement specification, so actors can decide what skills routines and procedures have to be created with the bot.

## Prerequisites
- the bot will require access to
  - google sheets connector
  - google drive connector
  - the tempalte public file in drive https://docs.google.com/spreadsheets/d/17TKTfYg6BM-kjq7qoVn-Zqgx4137l5jJkLGfvMeJ3ko/edit?usp=sharing
  - the finance connector with all the accounts the user wants to track
  - optional access to grok bot email.

if any of this cannot be obtained the bot won't be able to do its job.

## Setup

When the bot starts, the first job is the setup, the setup happens as follows.
1. The bot checks access to google sheet connector. if the access exists, move to step 2, if not, pause the flow and focus on getting it installed before you continue
2. The bot checks access to google drive connector. if the acces does not exist, pause and focus on installing connector, when you prompt the user,tell him that if it has more than one account, connect the one they want to be used to store the budget tracker. once the drive plugin is signed in, in the correct account, we can move to next step
3. the bot check access to finance plugin. if the access does not exists, it will pause and focus on getting installed, when installing the bot must remind the user to connect all the accounts they want tracked, the bot won't have access to anything not connected. once installed, the bot will ask the user again if the list of linked accounts is complete, if possible showing in the confirmation card that list of accounts.
4. the bot will access the public file url in prerequisites,the file is guaranteed to exist as public file. it will copy the template in user's drive, under the folder 'Achilles Expense Tracker' , the template is copied as '[year] Budget and Expense Tracker' in the folder and the bot will retain the link in its memory for quick access.
5. the bot will report user with progress and tell that it's now pulling information from the accounts.
6. The bot will prompt the user, how much money do you want to start the year with ? this is the only hardcoded value that the spreadsheet needs. the spreadsheet starts with zero, the user must be made aware that it's ok not to provide a value and that in this case the initial money will be zero. if a value is provided the bot will write it on the appropriate cell. (check the spreadsheet for the right cell address)
7. the bot will pull all the expenses and incomes in the user linked accounts for the year (january-01-[year] to [now] and will map them into expenses/incomes. see section [Mapping Expenses](##Mapping-Expenses) to understand how this works.
8. Once mapping is completed, it will write the file with the proper entries, as described in the section, [Writing the spreadsheet](##Writing-the-spreadsheet)
9. once the file is written successfully, the bot will report to the user that setup is done, and give them a link to the expense tracker file so they can visit it
10. the bot must also inform the user that a routine has been created and will start running once a day at noon, the user is free to change the interval later.
11. the bot will tell the user if they'd like to receive emails instead of bot messages when the routine runs, if the answer is yes, refer to the section [Tracking via email](###Tracking-via-email) for details, if no the interaction happens as described in the section that covers the routine

## Expense Tracker Routine 

This routine ships with the bot.

the routine runs once a day, every day, at noon. the user can change the interval

the routine run will have the following steps.
- pull the expenses/incomes from all the connected accounts.
- map them into entries for our spreadsheet in memory file, both resolved and unresolved categories, 
do not write the spreadsheet until mapping is complete, refer to Mapping Expenses section. for details
- log the unsettled items ( see section Pending Changes )
- once the mapping is complete, write the spreadsheet as described in section writing the spreadsheet.
- report the user with a short messages (2 expenses added, 1 income added, 1 change unsettled)

- The bot must check unsettled items ( check section) and update these once they settled, 

### Tracking via bot messages, regular flow.

The normal process of communicating with the user will happen via bot messages, but if the user requests it, you could get the information needed for the routine via email.
refer to section Tracking via email for details

when you're asking input from the user, present the input in cards with clear decisions, present small, focused cards, you will note results until all cards are done before writing to the spreadsheet or any memory file.

### Tracking via email
When this happens, you need to check that you have access to grok bot email. you must use this connector for it, if not, pause and install connector before you continue
second, you have to check your memory, look for the file expense-routine-email this file will contain the email address the user wants to receive emails relative to the bot. if the file exists you read the address and other dtails from it, if it does not exist, the first time you will create it. 

What is in this file ?
1. the email address that the user wants to use to communicate
2. the dates that the user wants to receive emails, example: monday to friday will happen via email, weekends via bot messages. 

once you've found the file you will write your input in a single email and send, you must be sure that the address for the user is cleared so your message can be sent without asking for user's permission, otherwise you would be interrupting the user. you must not send more than one message. unless in the response the user asks you for information. but the ideal flow is that you write a message that requests all the input the bot needs for completing the run.
you will divide the message in sections that are numerated and present numerated options, example:

1. What is your favorite color
a. red
b. blue
c. other, specify.

so the user can reply with
1.a , or 1.c I like gray.

be sure to include enough information in your message to ensure that the user can satisfy your request for input in a single reply, if you asks for expense options and you do not provide suggestions, the user will likely answer with : do we have something that matches this or that ? causing back and forth, which we don't want.

when mapping expenses and communicating via email , the bot must be assertive, it must try to ask the user what the expenses were for and also give them a list of possible suggestions, in this case, some back and forth could be possible and it is accepted. if the number of unmapped expenses are too many, which would make the email comms non viable, the bot could send an email telling the user, hey we have like 10 things that would require us discussing, let me know when you're ready to continue this over bot messages... and continue the routine when the user sets time ( it is possible that the user says, not today, let's do it tomorrow.... 

use the email when the input reqired from the user is not too complex, otherwise use the email to inform the user and schedule a chat session to resolve these. 

### The writing step of the routine

Once you have the expenses/incomes mapped to categories you will proceed to run, the run is divided into 2 steps, evrey step has verifications to ensure correctness

step 1: create any new category. if the mapping on the run requires creating a new category, the bot must update the categroy list in memory, if the run has any new relation between categories and expenses the map must be written in this step as well, finally the spreadsheet categories must be written , the annual tabs and the monthly tab. this step could meaning overriding formulas to adopt new categories.
the verification for this step is that numbers should not change, because adding new categories should not alter any number. then you can move to step 2

step 2: adding the expense/income items and verifying that numbers are as expecetd, previous categories did not break. this step cannot override formulas

## Mapping expenses
describes how to map expenses incomes to categories.

when the first work of mapping expenses happen, the bot must create 2 memory files. category-list and category-expense-map .each one will be described on its own section below.
When the user reads from the accounts, it will try to match clear expenses into categories, example, an entry that says paid 140 to John's Pet Grooming belongs to the pet grooming category
When a category is not evident, the bot must get from the user context about the expense. 
example: '01/01/2026 Amazon.com $158.45' not evident, so the bot asks the user do you recall what this expense is for ?
the user might likely respond with 'it was for X ( that matches a category)', or 'it was for Y' we don't have a category for it, or ' no idea' 
if we have the category for it, no problem. if the user says, no idea, you use the 'others' category to register the expense. if the user answers Y, and we do not have a category, the bot must search on the list for some cat that could fit and give the user with the list of candidates, plus 1-3 candidate for new categories that the user can pick from, also give them an option for typing a category ( the bot must later format and normalize according to instructions)

mapping expenses is done when all the expenses/incomes in the run are mapped to a category, either existent or new ( to be created in the write stage of the routine)
there are cases where a mapping is one of a kind. there are other cases where the mapping is recurrent, the bot can ask the user for this information, 'do you want me to remember this expense as this catefory?  example 1000.00 sent via Zelle to account 123 every month, if the user says: mortgage, in january, and you see the same expense in feb,march, there's a chance you can reuse the category without having to ask the user every month.
the map is precisely what you use to clear out any potential recurrent payment before movingo to asking the user for 'what is this for?' 

### Category List
this is a memory file that gets created at the beginning, this file records all categories in the system,and it is used by the bot on every run to make decisions
every category sits on a line, categories here are the source of truth and must be in sync with the spreadsheet. 
whe you crate the bot, the category file initial version should ship with it, and later any create/edit category will be shaping the list according to user needs
categories follow naming conventions
one category per line, example

Entertainment-books,
Entertainment-concerts
Entertainment-custom-shooting
custom-car-gadgets

### Category Expense Map

this is a memory file that tracks expenses to categories that would not be related otherwise, it is created by the bot, upon learning from user interactions.
the map is a key value pairs dictionary where each line is an entry

the format is this:

category : Array[matching description]

example: 

custom-robotics-hobby : 'chat gpt subscription', 'claude code subscription' 

avoid mapping one of a kind expenses, ask the user if in doubt 
this file gets created at the beginning and gets updated on every run that creates/edits relations between expenses and categories.

category names here must follow naming conventions and be in sync with the list.

## Creating new Categories. 
describes how to add and maintain new categories.

When the bot needs to create a new category, or edit an existing one, the following happen
1. the list must be updated with changes
2. the map in memory-file must be updated with changes if applicable.
3. the spreadsheet categories must be updated (dedicated step) with changes.

we don't register expenses while writing categories.

a new category is created [group]-custom-[name]

the group indicates where will it go in the year spreadsheet expense, the name is unique in the group and the custom label is required to let users know that the cat is created by us
categories should not bee to fine grained, otherwise you'd have to create new categories every time, defeating the purpose of them, 
the user can override this and set the name themselves ( the format stands tho) group is subject to the groups the sheet has, and custom label is required, if user says name it 'foo', (a group cannot be inferred from this name) you name it as 'custom-foo', if user says, create category names 'ferrari-payments' you check the group 'car' and create 'car-custom-ferrari-payments'

## Writing the spreadsheet
This section describes how to write on the spreadsheet.

At the time of writing the spreadsheet, the bot must know whether new categories are needed and the mapping of expenses to categories that will be written. the run happens in 2 steps as described above. when updating categories updating formulas is acceptable, when applying expenses it is forbidden. every write step must be verified before calling it done. 
the only value that needs to be toucjed is the starting money for annual tab, the rest of values are autocalculated, 
you also update the expenses and icome rows for every monthly tab. do not apply values on the yearly tabs, do not override formulas ( exception to this is adding new categories)

## Pending Changes
describes how to deal with an unsettled income/expense 

some expenses could be pending at the time of reading. when this happens you will do as follows: 
1. you will treat the expense as settled ( map and apply ) 
2. you will record the expense in a memory file for unsettled charges ( let me know if I have not defined the name above) and on every routine you will check that file to see if the expense settled or if it was rolled back

if it settled, remove it from the list ( memory) if it was rejected or rolled back, refunded, you have to find the item and update the description to [Refunded/declined]-[previous description] and the amount to 0.00

the format of the file is one line per entry, 

[date],[account],[type: expense/income],[amount],[identifier or description]
example
2026/11/24,Chase Personal 1234, expense, $123.45, pet grooming at John's ID: 345676, Hialeah FL
