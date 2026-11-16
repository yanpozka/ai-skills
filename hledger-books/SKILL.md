---
name: hledger-books
description: >
  Use when keeping books in hledger: importing a bank or card CSV, writing
  CSV rules, categorizing transactions, reconciling a balance, or reviewing
  a journal. Triggers include hledger, bank CSV, statement import, reconcile,
  chart of accounts, and a balance column that does not match the bank page.
  Use when the user runs /hledger-books.
metadata:
  short-description: Import, categorize, and reconcile hledger books
---

# hledger books

Keep plain-text books in hledger. The institution's page or statement is the balance. A CSV Balance cell is not, until this skill's check proves it. Entity accounts, tax positions, and prior decisions live in the project's own notes. Read those before posting. Do not copy another project's chart into this one.

Run hledger from the directory whose config points at the journal. `hledger check -s` must pass before a commit.

## Layout

Use this when the project has no layout yet. Do not rename directories that already work.

```
journal/     chart, opening balances, transactions, balance assertions
rules/       one CSV rules file per institution
import/raw/  downloads, gitignored
sources/     statements and receipts, gitignored
```

`hledger.conf` is `-f journal/main.journal`. The journal includes the chart and the assertions. Raw downloads and source documents stay untracked. They hold account numbers.

## Import

1. Put the download in `import/raw/`.
2. Dry-run: `hledger import --dry-run --rules rules/INSTITUTION.rules import/raw/FILE.csv`
3. Give each new `expenses:unreviewed` or `income:unreviewed` posting a rule, then dry-run again.
4. Import. Then `hledger check -s`.
5. Add one balance assertion per new statement date. Date it the morning after that day so it sorts before the next day's transactions. The amount is the balance from the institution's page or statement, written `0 = BALANCE`.

The `.latest.*` marker is per filename. Importing a full history under a new name duplicates every row. Add only the new rows, or point the marker at that file on purpose.

## Balance column

Walk the Balance column in chronological order before asserting anything.

- One account produces one running total. A second series in the same column, or a same-day batch whose balance sits on the wrong row, means the cell is not the account balance.
- Ask what balance the institution shows for that same latest transaction. Available balance and the balance printed on the transaction row are the figures that count.
- If the positive series minus the other series equals that figure, every row is the same account. Post them all to that account. A negative series is not a second account and not an amount owed.
- Assert only the figure the institution shows. Do not assert a raw Balance cell.
- Keep the within-day order that makes the proved balance tie. For some newest-first exports that order is the reverse of the file order inside the day. Test it on the latest day. Do not assume it.

## Mistakes

These are the failures this procedure exists to prevent. They name no institution, account, amount, or date.

- A checking CSV printed card charges on a negative running total and left them out of the positive total. That negative total was booked as a liability. The account page showed one account, and its balance was the positive total minus the negative total. The charges belonged on checking.
- The CSV balance was asserted. The institution's page showed a different balance on that same transaction.
- A same-day batch printed the balance on a different row of that day.
- A card payment was posted twice: once from the funding account, and again from the card file.
- Cash transferred to a brokerage was treated as the portfolio value.
- A full history download, saved under a new filename, was going to be imported. The import marker is per filename, so that would have duplicated the journal.

## Posting

- One real account is one hledger account. Checking, a card, and a brokerage cash balance stay separate. Positions, lots, and market value wait for a statement that lists them. Cash transferred in is not the account value.
- Cash basis uses the date the transaction cleared. On a card file that is the post date.
- A payment already posted from the funding account is not posted again from the card file.
- A returned payment that never left the funding account, and nets to zero on the card, is omitted. Say so in the project notes.
- Income and expenses the owner has not named stay in `income:unreviewed` and `expenses:unreviewed`. Do not decide that a deposit is revenue, or that a personal transfer is a draw, without the owner.
- A purchase the owner is keeping as an asset stays on an asset account. Depreciation, amortization, and an election to expense it are not posted until the owner chooses.
- Rules that cannot multiply: import the charge, then split it in the journal.

## Corrections

A reconciled amount stays. A wrong offset account is fixed on that same line. A wrong amount is a new dated entry. Do not rewrite a reconciled transaction to make a later period look right.

## Close

Take figures from hledger, not from memory.

- `hledger check -s`
- `hledger is not:unreviewed` for the reviewed income statement
- `hledger is` for the full income statement
- `hledger bs`
- `hledger bal unreviewed`

Update the project's status note. Commit the journal, the rules, and the assertions. Do not commit `import/raw/` or `sources/`.

## Red flags

Stop if you were about to:

- assert a CSV Balance cell you have not tied to the institution's page
- open a new account because a Balance cell is negative
- post a card payment that the funding account already recorded
- treat cash transferred to a brokerage as the portfolio value
- expense an asset, or post depreciation, before the owner chooses
- re-import a full CSV under a new filename
