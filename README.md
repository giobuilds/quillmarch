# quillmarch

**The Core**

A small double-entry ledger in SQLite, with accounts (bank, credit card, savings, income and expense categories) and transactions made of lines that must balance. Post entries and reverse them rather than deleting.

**The Interface** 

Simple screens like "Add expense", "Income", "Transfer", plus CSV bank import with categorising rules and reconciliation.

**Reports**

Balances per account, monthly spending per category, and later a budget against actuals.

## Schema sketch

```
accounts (
  id          INTEGER PRIMARY KEY,
  name        TEXT NOT NULL UNIQUE,
  type        TEXT NOT NULL CHECK (type IN ('asset','liability','income','expense','equity')),
  parent_id   INTEGER REFERENCES accounts(id),   -- e.g. Expenses > Food
  archived    INTEGER NOT NULL DEFAULT 0
);

transactions (
  id          INTEGER PRIMARY KEY,
  date        TEXT NOT NULL,                     -- ISO 'YYYY-MM-DD'
  description TEXT NOT NULL,
  posted_at   TEXT NOT NULL,                     -- when it was entered
  reverses_id INTEGER REFERENCES transactions(id) -- set on a reversal
);

lines (
  id             INTEGER PRIMARY KEY,
  transaction_id INTEGER NOT NULL REFERENCES transactions(id),
  account_id     INTEGER NOT NULL REFERENCES accounts(id),
  amount_pence   INTEGER NOT NULL,               -- debit +, credit -
  memo           TEXT
);
```

A few design choices worth thinking about:

- Money is stored as whole pence (INTEGER), never REAL. Floating point can't hold 0.10 exactly, so totals drift.
- Signed amounts in a single column, debits positive and credits negative, so "balances" just means every transaction's lines sum to 0, and an account balance is SUM(amount_pence). The alternative is separate debit and credit columns, closer to D365's look. Which do you find easier to reason about?
- No updates or deletes on posted transactions. A correction is a new transaction with reverses_id set and the amounts negated, like D365's reverse entry.
- Categories are just income or expense accounts, so a £12 lunch on your debit card is two lines: Food +1200, Bank −1200.

Some things to work out yourself:

- SQLite can't enforce "lines sum to 0" with a simple CHECK. Where would you enforce it: in C before the insert, or with a trigger? What happens if the app crashes halfway through writing the lines? (Hint: transactions in the SQL sense.)
- Turn foreign keys on (PRAGMA foreign_keys = ON) when you open the database. Why does SQLite need you to do that every time?
- Which index would make the running balance per account fast?

For later, you could add a bank_imports table for CSV rows and a rules table for auto-categorising. Leave those out of the first milestone.

