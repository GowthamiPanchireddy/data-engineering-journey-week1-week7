# Relational Banking Database – Week 4

## Project Overview

This project builds a relational banking database using Python and SQLite.

The database stores customer, branch, account, and transaction information in separate related tables.

Python is used to create the database, create the tables, load the supplied CSV files, and test database constraints.

SQLite is used because it provides a lightweight relational database without requiring a separate database server.


## Project Architecture

The project is organized into separate folders based on their purpose.

```text
banking-database-week4/
│
├── banking.db
│
├── input/
│   ├── customers.csv
│   ├── accounts.csv
│   ├── branches.csv
│   └── valid_transactions.csv
│
├── src/
│   └── database.py
│
├── tests/
│   └── test_database.py
│
├── docs/
│   └── ER_Diagram.png
│
├── sql/
│   └── queries.sql
│
└── README.md

Folder Description:-
input/ – Contains the supplied CSV input files.
src/ – Contains the Python database creation and data loading code.
tests/ – Contains pytest tests for database behavior and constraints.
docs/ – Contains the ER diagram.
sql/ – Contains the SQL queries used for analysis.
banking.db – SQLite database created by the Python program.
README.md – Project documentation.


****Database Schema:-

The database contains four main relational tables:

   1. Customer

Stores customer information.

- `customer_id` – Primary Key
- `customer_name` – Customer name
- `email` – Customer email
- `customer_segment` – Customer category

  2. Branch

Stores bank branch information.

- `branch_id` – Primary Key
- `branch_name` – Branch name
- `city` – Branch city
- `state` – Branch state

   3. Account

Stores bank account information.

- `account_id` – Primary Key
- `customer_id` – Foreign Key referencing `customer.customer_id`
- `branch_id` – Foreign Key referencing `branch.branch_id`
- `account_type` – Checking or Savings
- `account_status` – Active or Inactive

   4. Transactions

Stores banking transaction information.

- `transaction_id` – Primary Key
- `account_id` – Foreign Key referencing `account.account_id`
- `transaction_date` – Transaction date
- `transaction_type` – Credit or Debit
- `amount` – Transaction amount
- `currency` – Transaction currency


****Relationships

- One customer can have many accounts.
- One branch can have many accounts.
- One account can have many transactions.

The database is normalized so that customer, branch, account, and transaction information is stored in their appropriate tables without unnecessary repetition.


****ER Diagram:-

The ER diagram shows the relationships between the four database entities.

The relationships are:

- Customer → Account: One-to-Many
- Branch → Account: One-to-Many
- Account → Transactions: One-to-Many

The ER diagram is available in:

`docs/ER_Diagram.png`


****Data Flow:-

The data flows through the system in the following order:

1. Customer, branch, and account master data are read from the supplied CSV files.
2. Valid transactions are read from `valid_transactions.csv`.
3. Python creates the SQLite database and required tables.
4. Foreign key enforcement is enabled.
5. Customer and branch data are loaded first.
6. Account data is loaded using customer and branch foreign keys.
7. Valid transaction data is loaded using the account foreign key.
8. SQL queries are used to retrieve and analyze the banking data.
9. Pytest verifies the database structure, row counts, constraints, and rerun behavior.



****Build and Load the Database:-

The database can be created and loaded using the Python program in `src/database.py`.

From the project root directory, run:

```powershell
python src/database.py

The program:

Creates banking.db.
Enables SQLite foreign key enforcement.
Creates the Customer, Branch, Account, and Transactions tables.
Loads the supplied customer, branch, and account CSV files.
Loads the valid transaction records.
Commits the data to the database.
Handles database integrity errors using a controlled sqlite3.IntegrityError exception.

The expected baseline data loaded into the database is:

Customers: 6
Branches: 3
Accounts: 10
Valid Transactions: 10


****Data Integrity and Constraints:-

Database constraints are used as a second layer of protection in addition to Python validation.

The database uses:

- **PRIMARY KEY** – Prevents duplicate IDs.
- **FOREIGN KEY** – Ensures that referenced customer, branch, and account records exist.
- **NOT NULL** – Prevents missing required values.
- **CHECK** – Ensures that transaction amounts are greater than zero.

Foreign key enforcement is enabled using:

```python
connection.execute("PRAGMA foreign_keys = ON")
The project deliberately tests invalid database operations.

The following integrity failures were tested:

Transaction referencing a nonexistent account.
Account referencing a nonexistent customer.
Duplicate primary key.
Negative transaction amount.
Missing required customer name.

SQLite correctly rejected these invalid records with sqlite3.IntegrityError.

Using both Python validation and database constraints provides defense in depth. Python can validate data before loading, while database constraints protect the stored data even if invalid data reaches the database layer.


****SQL Analysis:-

The project contains 12 SQL queries in:

`sql/queries.sql`

The queries demonstrate different SQL concepts:

- SELECT queries
- INNER JOIN queries
- GROUP BY and aggregation
- CASE expressions
- CTE (Common Table Expression)
- Window functions
- RANK()
- Running total using SUM() OVER()
- HAVING and conditional aggregation

The queries are used to answer meaningful banking questions such as:

- Which customers own which accounts?
- How many accounts exist for each account type?
- What is the total transaction amount by transaction type?
- Which transactions are classified as high, medium, or low?
- What is the total transaction amount for each customer?
- Which customers have total transactions of at least 500?
- How are transactions ranked by amount?
- What is the running total of transaction amounts?
- Which customers have more debit transactions than credit transactions?


*****Indexing and Query Plan:-

An index was created on the `account_id` column of the `transactions` table because transaction lookups commonly use the account ID.

Index:

```sql
CREATE INDEX IF NOT EXISTS idx_transactions_account_id
ON transactions(account_id);
The query plan was checked before and after creating the index.

Before the index

SQLite used:

SCAN TABLE transactions AS t

This means SQLite scanned the transactions table to find the required account.

After the index

SQLite used:

SEARCH TABLE transactions AS t USING INDEX idx_transactions_account_id

This shows that SQLite used the new index to locate matching transaction records.

The dataset is small, so the project does not claim a measurable runtime speed improvement. The EXPLAIN QUERY PLAN output demonstrates the change in SQLite's access strategy.


****Testing:-

The project uses `pytest` to test the actual database-building and data-loading code.

Tests are located in:

`tests/test_database.py`

Run the tests from the project root:

```powershell
python -m pytest -q
The test suite verifies:

Expected database tables exist.
Foreign key enforcement is enabled.
Expected row counts are present.
Duplicate primary keys are rejected.
Invalid foreign keys are rejected.
Invalid transaction amounts are rejected.
Re-running the data loading process does not create duplicate records.

Test result:

7 passed

The tests use the production database functions from src/database.py rather than recreating the production database logic separately.


*****Conceptual Questions:-

1. Why should customer information not be repeated on every transaction?
Customer information is stored once in the `customer` table and connected using `customer_id`.
Repeating customer information in every transaction would cause data duplication and could create inconsistent values if customer details change.
Normalization avoids unnecessary repetition and keeps each fact in its appropriate table.

2. What is the difference between Python validation and database constraints?
Python validation checks data before it is inserted into the database.
Database constraints protect the data at the database level.
For example, Python can check whether an amount is valid, while the database `CHECK (amount > 0)` constraint ensures that a negative amount cannot be stored.
Both provide different layers of protection.

3. What happens if foreign key enforcement is disabled in SQLite?
If foreign key enforcement is disabled, SQLite can allow records that reference nonexistent parent records.
For example, a transaction could reference an account ID that does not exist.
Therefore, the application explicitly enables:
```python
PRAGMA foreign_keys = ON

4. Why is creating indexes on every column not always a good idea?
Indexes can make searches faster, but they also require additional storage.
They can also make INSERT, UPDATE, and DELETE operations more expensive because indexes may need to be updated.
Therefore, indexes should be created on columns that are frequently used for searching, joining, filtering, or sorting.

5. What is the difference between a relational banking database and an analytical data warehouse?
A relational banking database is designed mainly for operational transactions such as creating accounts and recording banking transactions.
An analytical data warehouse is designed mainly for reporting and analysis across large amounts of historical data.
The banking database focuses on reliable transactional operations, while a data warehouse focuses on analytical queries and reporting.


*****Conclusion:-

This project demonstrates how a relational banking database can be designed, created, loaded, tested, and queried using Python and SQLite.

The database maintains relationships between customers, branches, accounts, and transactions using primary keys and foreign keys.

Database constraints protect data integrity, while SQL queries provide meaningful banking analysis.

Pytest verifies that the database structure, relationships, constraints, and rerun behavior work as expected.