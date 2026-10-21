# LedgerLine — Requirements

## 1. Scope (v1)

LedgerLine is an offline .NET console application. It imports a bank CSV export, categorizes each transaction using configurable rules, and produces a monthly spending summary per category, shown as a console table and exported as JSON.

## 2. User stories

Each story has acceptance criteria that will become tests.

**US-01 — Import.** As a user, I want to import transactions from a bank CSV file so that I don't have to enter them manually.

- Given a valid file, every row becomes a transaction with date, description, amount, and balance.
- Given a file that doesn't exist or can't be read, the program exits with a clear error message and a non-zero exit code.

**US-02 — Validation.** As a user, I want invalid rows to be detected and reported so that bad data never silently corrupts my reports.

- A row is invalid if it has the wrong number of fields, an unparseable date, or an unparseable amount.
- Invalid rows are reported with their line number and the reason.
- Valid rows are still processed (see open question 2 for the proposed default).

**US-03 — Categorization.** As a user, I want transactions categorized by rules I configure so that I can see where my money goes.

- A rule maps a description pattern to a category.
- If several rules match, the first one in the configuration file wins.
- Transactions that match no rule go to the category `Uncategorized`.

**US-04 — Monthly report.** As a user, I want a monthly summary per category so that I can understand my spending.

- The report shows, per month and category, the total of expenses and of income.
- It is printed as a console table.
- It can also be exported as JSON.

**US-05 — Reconciliation.** As a user, I want the program to check the file's running balance (`SALDO`) so that I can trust the parsed amounts.

- For each row, the balance should equal the previous balance plus the amount.
- Mismatches are reported as warnings with their line numbers.

## 3. Input assumptions

Source: a real export from my bank. Test data in the repository must be **sanitized** (fake merchants and amounts).

### Columns

| Column        | Meaning                               |
| ------------- | ------------------------------------- |
| `FECHA`       | Transaction date                      |
| `DESCRIPCIÓN` | Free-text description                 |
| `VALOR`       | Signed amount                         |
| `SALDO`       | Running balance after the transaction |

### Date format

`DD/MM`, for example `21/10`. **The year is not part of the value.**

- The year is supplied as a CLI argument (`--year 2026`). A file that crosses a year boundary is out of scope for v1.

### Number format

- Two decimal places, `.` as the decimal separator, for example `-2456.85`.
- The currency symbol is absent.
- no thousands separators are accepted; a value that contains one is an invalid row.

### Sign convention

One signed column (`VALOR`). Expenses are negative and income is positive.

### File-level assumptions

- Delimiter: comma.
- Encoding: UTF-8.
- Header row: present, with the four column names above.
- Quoting: fields containing the delimiter are wrapped in double quotes.
- Currency: a single currency per file (no conversion).

## 4. Security and robustness

- The input file is untrusted: malformed rows must never crash the program or produce wrong totals silently.
- Maximum accepted file size: 10 MB. Larger files are rejected before parsing.
- The program never executes or evaluates anything from the file.
- No real financial data and no secrets are committed to the repository.

## 5. Non-goals (v1)

1. No multiple currencies.
2. No GUI.
3. No bank API sync.
4. No editing or deleting individual transactions.
5. No persistent database. Each run reads the CSV and produces its report.
6. No duplicate detection across runs (depends on persistence; see open question 1).

## 6. Open questions

Each question has a proposed default, so the decision can be challenged.

1. **How are duplicates detected?** The file has no transaction ID. Proposed: within a single file, a row is a duplicate only if date, description, amount, **and balance** are all identical. The balance column distinguishes two legitimate identical purchases on the same day. Cross-run detection is deferred with persistence.
2. **Reject the whole file or import valid rows?** Proposed: import the valid rows, report every rejected row, and exit with a non-zero code if any row was rejected, so scripts can detect it.
3. **Where do categorization rules live?** Proposed: a JSON file next to the executable, loaded through `IConfiguration`.
4. **Should persistence come later?** Proposed: yes, as a possible future project. v1 stays stateless.
