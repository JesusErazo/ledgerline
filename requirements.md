# Requirements

## User stories

US-01: As a user, I want to import banking transactions from a CSV file so that I can bring my existing bank account activity into Ledger Line without entering transactions manually.

US-02: As a user, I want the application to validate the CSV file's structure and transaction data before importing it so that invalid, incomplete, or incorrectly formatted records do not corrupt my ledger.

US-03: As a user, I want the application to identify transactions that have already been imported so that importing the same CSV file more than once does not create duplicate records.

US-04: As a user, I want to view my imported transactions, including their dates, descriptions, and amounts, so that I can review my banking activity from the console.

US-05: As a user, I want to view my account balance based on my imported transactions so that I can understand the financial impact of my recorded banking activity.

US-06: As a user, I want my imported transactions to be stored persistently so that I can access my financial history and calculated balance after restarting the application.

## Input assumptions

### columns

FECHA,
DESCRIPCIÓN,
VALOR,
SALDO

### date format

DD/MM
Example:21/10

### number format

2 decimals
Example: -2456.85

### sign convention

One signed column (VALOR). Expenses are negative and income is positive.

## Non-goals

1. No multiple currencies.
2. No GUI.
3. No bank API sync.

## Open questions

1. Does the CSV include a unique transaction ID? If not, what combination of fields should be used to detect duplicate transactions?
2. Should the application reject the entire file if some rows are invalid, or import valid rows and report the rejected ones?
3. Which storage mechanism should V1 use: a local SQLite database, a JSON file, or another option?
4. Should users be able to edit or delete imported transactions in V1, or should imported records remain unchanged?
