# Bank Transaction Data Quality Pipeline

A Python-based data quality pipeline for validating bank transaction CSV files, handling duplicate transactions, generating valid and invalid outputs, and producing data quality and operational summaries.

## Project Structure

```text
bank-transaction-pipeline/
├── input/
├── output/
├── src/
│   ├── pipeline.py
│   └── validation.py
├── tests/
│   ├── test_validation.py
│   └── test_pipeline.py
├── requirements.txt
└── README.md

*** Required Input Columns:-
 The pipeline expects the following columns:

  transaction_id
  account_id
  transaction_date
  transaction_type
  amount
  currency

*** Validation Rules:-The pipeline checks:

  Transaction ID is not missing
  Account ID is not missing
  Transaction date is present and in the correct format
  Transaction type is CREDIT or DEBIT
  Amount is numeric and greater than 0
  Currency is USD
  Transaction ID is unique across all input files.
Multiple validation errors for the same record are preserved in the error_reason column.


### Week 2 Testing:-
The pipeline was tested with different real-world input scenarios:

  New branch file
  Header-only file
  Missing required column
  Multiple validation errors
  Cross-file duplicate transaction
  Non-numeric amount
  Impossible transaction date
  Different column order
  Rerun with the same inputs
  Unexpected file
During Week 2, a non-numeric amount such as ABC initially caused the pipeline to fail. This was fixed using safe numeric conversion with pd.to_numeric(..., errors="coerce").


### Week 3 Pipeline Hardening:- Week 3 focused on improving the existing Week 2 pipeline rather than rebuilding it.

*** Refactoring:
The pipeline was divided into reusable functions for:

  CSV file discovery
  File reading
  Schema checking
  Row validation
  Duplicate handling
  Separating valid and invalid records
  Output writing
  Summary generation
  Data quality summary generation
Constants are used for important paths and required columns.


*** Data Quality Summary:-
The pipeline generates:

   output/dq_summary.csv

The DQ summary contains:

   Files discovered
   Files read
   Files rejected
   Total records
   Valid records
   Invalid records
   Rejection rate
   Duplicate records
   Rejected file names
This provides visibility into both row-level and file-level data quality problems.

*** Logging:- Python's built-in logging module is used for operational evidence.
Logs are written to:

  output/pipeline.log

The pipeline records:

  Pipeline start and completion
  File discovery
  File reading
  Empty/header-only files
  Schema rejection
  Unreadable files
  No input files
  All candidate files rejected

Different log levels are used:

  INFO
  WARNING
  ERROR

*** Running the Pipeline
From the project root:
    
    python src/pipeline.py


The pipeline creates:

output/
├── valid_transactions.csv
├── invalid_transactions.csv
├── summary.csv
├── dq_summary.csv
└── pipeline.log

*** Running Tests
Use Python 3.10:
   python -m pytest -v

Expected result:
   6 passed

*** Testing Strategy:- 
 The Week 3 test suite contains
   Unit Tests:
    Tests for amount validation:

        Valid amount
        Negative amount
        Non-numeric amount
        Integration Tests

    Tests for:

        Cross-file duplicate transaction handling
        Multiple validation errors


    Regression Test:- The original three input files are processed through the pipeline functions.
     Expected result:

        Total Records: 24
        Valid Records: 10
        Invalid Records: 14
This confirms that Week 3 hardening did not break the original Week 2 behavior.

*** Current DQ Result:-
 With the current test input folder:

   Files Discovered: 11
   Files Read: 9
   Files Rejected: 2
   Total Records: 32
   Valid Records: 14
   Invalid Records: 18
   Rejection Rate: 0.5625
   Duplicate Records: 4
The rejected files are also recorded in dq_summary.csv.


### Week 2 to Week 3 Changes:- Week 2 focused mainly on validation and testing different data scenarios.
    Week 3 improved the pipeline by adding:

   Reusable pipeline functions
   Data quality summary
   Operational logging
   File-level rejection tracking
   Automated pytest tests
   Meaningful regression testing
These changes make the pipeline easier to maintain, test, and monitor.


### Limitations and Future Improvements:-

1.The pipeline currently uses CSV files and local folders only. Future versions could support configurable input and output locations.
2.The DQ summary currently provides run-level metrics. Future versions could provide more detailed validation failure counts by individual rule and source branch.


### Reflection:-

Week 3 helped me understand that a working pipeline also needs good structure, testing, monitoring, and clear data quality information. I refactored the Week 2 pipeline into reusable functions so that file discovery, schema checking, validation, duplicate handling, output writing, and summary creation are separated. I also added logging to provide operational evidence about what happened during each run. The DQ summary makes it easier to understand how many files were discovered, accepted, rejected, and how many records were valid or invalid.

Testing was an important part of the hardening process. I added unit tests for amount validation, including valid, negative, and non-numeric values. I also added tests for cross-file duplicates and multiple validation errors. The regression test processes the original three files and confirms the expected result of 24 total records, 10 valid records, and 14 invalid records.

One important lesson was that defensive handling of bad data is necessary because a single unexpected value should not stop the complete pipeline. Overall, Week 3 improved the reliability, maintainability, observability, and testability of the existing pipeline.

