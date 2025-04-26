# RuleSet Documentation

This document is a help guide for RuleSet, a data validation language that non-coding domain experts can use to develop, maintain and communicate data validation rules.
The accompanying data validation program, ruleset-engine, validates a data table against rules written in RuleSet.

## Key Features

- ### Atomic Rules
Each rule in RuleSet targets a single aspect of your dataset. For instance, applying a "is not null" rule to a column will solely check for the presence of null values, without performing any type or format validations.

- ### Rule Stacking
You can specify multiple atomic rules for a data column. Rules are evaluated independently, and their order does not affect the outcome. A data column must pass all applicable rules to be considered valid. Successful validations are recorded with an "all OK" message in the log file.

- ### Second-Order Validation
RuleSet supports cross-field validations, allowing you to define dependencies between columns. For example, you can specify: `if column A contains "John Doe," then column B must contain "1970-12-22."`

- ### No implicit evaluation 
Data is never evaluated implicitly. All validation rules must be explicitly defined to be applied.

With these key features, RuleSet allows you to specify data validation rules at different levels of stringency. See [Appendix C](#appendix-c) for examples.

---

## Table of Contents

1. [Set Value Rules](#set-value-rules)  
2. [Column Rules](#column-rules)  
3. [Value Rules](#value-rules)  
4. [Second Order Validation](#second-order-validation-1)
5. [Using ruleset-engine](#data-validation-using-ruleset-engine)
6. [Appendix](#appendix) 

---

## Set Value Rules

Global dataset settings that apply before column-level validation.

### `allowed null values in [ … ]`  
**Description:** Treat the listed literals as null/missing values.  
**Syntax:**  
```dsl
allowed null values in <list of literals to be treated as null>
```  
**Example:**  
```dsl
allowed null values in ["Not Measured", "-"]
```

### `thousands separator is "…"`  
**Description:** Specify the character used to separate thousands in numeric fields.  
**Syntax:**  
```dsl
thousands separator is <seperator>
```  
**Example:**  
```dsl
thousands separator is "."
```

### `set precision at "…"`  
**Description:** Specify the level of precision to be used for numeric comparisons. The default value is 0.001 i.e. a precision of 3 decimal places.  
**Syntax:**  
```dsl
set precision at  <precision level>
```  
**Example:**  
```dsl
set precision at  "0.00001"
```

---

## Column Rules

Specify which columns must be included, if all named columns are required and if extra columns are allowed.

### `column names in [ … ]`  
**Description:** (Required) List exactly which columns the table must contain. *This is the only required rule in RuleSet. Every set of rules must have the column names list.* 
<br>**Syntax:**  
```dsl
column names in <list of column names>
```  
**Example:**  
```dsl
column names in ['user_id', 'first_name', 'last_name', 'signup_date']
```

### `all columns required`  
**Description:** Fail if any of the named columns is missing.  
**Syntax & Example:**  
```dsl
all columns required
```

### `no extra columns allowed`  
**Description:** Fail if the table contains any columns **not** in the above list.  
**Syntax & Example:**  
```dsl
no extra columns allowed
```

### `check column order`  
**Description:** Enforce that the physical column order matches the listed order.  
**Syntax & Example:**  
```dsl
check column order
```

---

## Value Rules

Apply one or more value-level constraints to a specific column.

**Block Syntax:**
```dsl
column: <ColumnName>
<ValueRule1>
<ValueRule2>
    …
```
**Example:**
```dsl
column: 'age'
has value type integer
is >= 0
is <= 120
```

### Data Type & Format
- **`has value type <ValueType>`**  
  ```dsl
  has value type string
  ```
See [Appendix A](#appendix-a) for accepted value types.


- **`has format <FormatType>`**  
  ```dsl
  has format "YYYY-MM-DD"
  ```
See [Appendix B](#appendix-b) for formats that can be specified. Note: the current formats apply only to the date-time data type.

### Presence & Uniqueness
- **`is required`**  
  ```dsl
  is required
  ```
- **`is unique`**  
  ```dsl
  is unique
  ```
- **`is null`** / **`is not null`**  
  ```dsl
  is null
  is not null
  ```

### Pattern & Membership
- **`has pattern /…/`**  
  ```dsl
  has pattern /^[A-Z]{2}\d{4}$/
  ```
- **`is in [ … ]`** / **`is not in [ … ]`**  
  ```dsl
  is in ['Active', 'Pending', 'Closed']
  is not in ['Cancelled', 'Deleted']
  ```

### Exact Value Comparisons
- **`is "<string>"`** / **`is not "<string>"`**  
  ```dsl
  is "USA"
  is not "N/A"
  ```
- **`is == <NUMBER>`** / **`is != <NUMBER>`**  
  ```dsl
  is == 100
  is != 0
  ```

### Numeric Bounds
- **`is > <NUMBER>`** / **`is < <NUMBER>`**  
  ```dsl
  is > 0
  is < 100
  ```
- **`is >= <NUMBER>`** / **`is <= <NUMBER>`**  
  ```dsl
  is >= 0
  is <= 120
  ```

### Length Constraints
- **`has length <INT>`**  
  ```dsl
  has length 5
  ```
- **`has min length <INT>`** / **`has max length <INT>`**  
  ```dsl
  has min length 3
  has max length 10
  ```

### String Affixes & Substrings
- **`starts with "<string>"`** / **`ends with "<string>"`**  
  ```dsl
  starts with "ABC"
  ends with ".csv"
  ```
- **`includes "<string>"`** / **`does not include "<string>"`**  
  ```dsl
  includes "urgent"
  does not include "spam"
  ```

### Precision Rules
- **`has <INT> significant digits`**  
  ```dsl
  has 3 significant digits
  ```
- **`has <INT> decimal places`**  
  ```dsl
  has 2 decimal places
  ```

---

## Second Order Validation
Define IF-THEN validations that span two columns.
**Syntax:**
```dsl
conditional rule: "RuleName"
if
column: <ColumnA> <ValueRuleA>
then
column: <ColumnB> <ValueRuleB>
```

**Example:**
```dsl
conditional rule: "US-Zip-Validation"
if
column: 'country' is "USA"
then
column: 'zipcode' has pattern /^\d{5}(-\d{4})?$/
```

---
## Data Validation using ruleset-engine
The program ruleset-engine is a command-line utility designed to validate data tables against rules defined in the RuleSet Domain-Specific Language (DSL). Validation outcomes are recorded in a log file for review.

Usage
```dsl
ruleset-engine [-h] --rules-file RULES_FILE --data-file DATA_FILE --reference-file REFERENCE_FILE [--logfile-path LOGFILE_PATH]
```
<br>**Options:**
<br>-h, --help Display the help message and exit.

<br>Required Arguments:
<br>--rules-file RULES_FILE Path to the file containing the data validation rules.
<br>--data-file DATA_FILE Path to the data file to be validated.
<br>--reference-file REFERENCE_FILE Path to the reference file (e.g., RuleSet.tx.enc) that defines the RuleSet grammar.
<br>Optional Arguments:
<br>--logfile-path LOGFILE_PATH Directory path where the log file will be stored. If not specified, the default directory is used.

<br>**Notes**
- Ensure that the ruleset-engine executable is located in a directory included in your system's PATH environment variable to allow for direct command-line invocation.
- The current version of ruleset-engine accepts, .csv, .tsv and .xls(x) format data files. 
- Data files are expected to be tabular with column-headers included. 
- Logfile name includes a timestamp: `YYYY_MM_DD_HH_MM_SS_ruleset.log`. If no log file path is specified, log file is stored in the directory from which ruleset-engine is invoked.
- For a detailed explanation of the log file output, refer to [Appendix D](#appendix-d).

---
## Appendix

### Appendix A

Value types that can be specified in the `has value type` rule.
| Value Type     | Description                                                                                                 |
| -------------- | ----------------------------------------------------------------------------------------------------------- |
| string         | Text-based data (e.g., names, addresses, categories).                                                       |
| integer        | Whole numbers without decimal points (e.g., 1, 42, -10).                                                    |
| floating point | Decimal numbers (e.g., 3.14, -0.01, 2.718).                                                                 |
| boolean        | True/False values (e.g., true, false).                                                                      |
| date-time      | Date or timestamp values (e.g., 2023-12-25, 2025-03-27T14:00:00). See below for supported date-time formats.|
| scientific     | Numbers expressed in scientific notation (e.g., 1.23e-4).                                                   |
| complex        | Complex numbers with real and imaginary parts (e.g., 3+4j).                                                 |


### Appendix B
Supported date-time formats that can be specified in the `has format` rule:
<br> `["HH:mm:ss" , "HH:mm" , "hh:mm:ss AM/PM" , "hh:mm AM/PM" , "HHmmss" , "YYYY-MM-DD HH:mm:ss" , 
"YYYY-MM-DDTHH:mm:ss" , "YYYY-MM-DD HH:mm" , "YYYY-MM-DDTHH:mm", "DD/MM/YYYY HH:mm:ss" , 
"MM-DD-YYYY hh:mm:ss AM/PM" , "YYYY/MM/DD HH:mm:ss" , "YYYY-MM-DDTHH:mm:ss.sssZ" , 
"YYYY-MM-DDTHH:mm:ss+hh:mm" , "YYYY-MM-DD" , "MM-DD-YYYY" , "DD-MM-YYYY" , "YYYY/MM/DD" , 
"MM/DD/YYYY", "DD/MM/YYYY" , "YYYY.MM.DD" , "MM.DD.YYYY" , "DD.MM.YYYY" , "YYYYMMDD", "YYYY"]`
<br>Other data formats for geolocation and currency data will be added in future.

### Appendix C
Let's see how RuleSet can be used to write data validation rules. Given below is a data table with 15 rows and 8 columns.
<br>

| Rank | PSA_ID  | First_Name | Last_Name   | Country | Year_of_Birth | Address                         | Zip_Code |
|------|---------|------------|-------------|---------|----------------|----------------------------------|----------|
| 1    | PSA9057 | Mostafa    | Asal        | EGY     | 2001           | 18 High St, Alexandria           | 11574    |
| 2    | PSA9086 | Ali        | Farag       | EGY     | 1993           | 411 High St, Giza                | 11772    |
| 3    | PSA6447 | Diego      | Elias       | PER     | 2004           | 235 Oak Dr, Cusco                | 15441    |
| 4    | PSA1932 | Paul       | Coll        | NZL     | 1992           | 99 Beach Rd, Auckland            | 1010     |
| 5    | PSA3884 | Marwan     | ElShorbagy  | EGY     | 1993           | 55 Nile Ave, Cairo               | 11311    |
| 6    | PSA7765 | Tarek      | Momen       | EGY     | 1988           | 12 Lotus St, Maadi               | 11431    |
| 7    | PSA8820 | Victor     | Crouin      | FRA     | 1999           | 45 Rue de Lyon, Paris            | 75012    |
| 8    | PSA1399 | Joel       | Makin       | WAL     | 1994           | 73 Cardiff Rd, Newport           | NP10 8XG |
| 9    | PSA6233 | Mazen      | Hesham      | EGY     | 1994           | 31 Zamalek St, Cairo             | 11211    |
| 10   | PSA5431 | Youssef    | Ibrahim     | EGY     | 1999           | 23 Freedom Rd, Alexandria        | 11432    |
| 11   | PSA4738 | Nicolas    | Mueller     | SUI     | 1989           | 90 Lake View, Zurich             | 8004     |
| 12   | PSA3186 | Saurav     | Ghosal      | IND     | 1986           | 77 Residency Rd, Kolkata         | 700025   |
| 13   | PSA2567 | Mohamed    | ElShorbagy  | ENG     | 1991           | 19 Queen's Walk, Bristol         | BS8 1SB  |
| 14   | PSA9012 | Baptiste   | Masotti     | FRA     | 1995           | 12 Place du Marché, Marseille    | 13001    |
| 15   | PSA8777 | Eain Yow   | Ng          | MAS     | 1998           | 38 Jalan Ampang, Kuala Lumpur    | 50450    |
<br>

Here is a simple set of rules that only specify value types for each column. Note that lines starting with `//` are considered comments in RuleSet. Comments may be entered anywhere in the rules file.
```dsl
//Rules to validate squash_playsers.csv
//These rules specify value type for each column, except for Zip_Code. Why do you think value type evaluation has been
//skipped for Zip_Code? How would you do it using RuleSet? Find out in the more complex rule specifications below.
//-------------------------------------

column names in ['Rank', 'PSA_ID', 'First_Name', 'Last_Name', 'Country', 'Year_of_Birth', 'Address', 'Zip_Code']
all columns required
no extra columns allowed

column: 'Rank'
has value type integer

column: 'PSA_ID'
has value type string

column: 'First_Name'
has value type string

column: 'Last_Name'
has value type string

column: 'Country'
has value type string

column: 'Year_of_Birth'
has value type date-time

column: 'Address'
has value type string

column: 'Zip_Code'
has value type string

```

Here is a more complex, fine-grained set of rules that specify additional constraints.
```dsl
//Rules to validate squash_playsers.csv
//In addition to value type these rules specify additional constraints.
// Column ID should be unique; each value should start with 'PSA'; PSA0000 is not a valid value.
// Column Country can have only one of the following values:
//'EGY', 'PER', 'NZL', 'IND', 'MAS', 'WAL', 'ENG', 'FRA', 'SUI'.
//Column Year_of_Birth should have the format 'YYYY'.
//Column Zip_Code is not null.
//Conditional rules:
//Zip_Code for Wales and England should be of type string.
//Zip_Code for all other countries should be of type integer.
//-------------------------------------

column names in ['Rank', 'PSA_ID', 'First_Name', 'Last_Name', 'Country', 'Year_of_Birth', 'Address', 'Zip_Code']
all columns required
no extra columns allowed

column: 'Rank'
has value type integer
is > 0

column: 'PSA_ID'
has value type string
is unique
starts with 'PSA'
is not 'PSA0000'

column: 'First_Name'
has value type string

column: 'Last_Name'
has value type string

column: 'Country'
has value type string
is in ['EGY', 'PER', 'NZL', 'IND', 'MAS', 'WAL', 'ENG', 'FRA', 'SUI'], 

column: 'Year_of_Birth'
has value type date-time
has format 'YYYY'

column: 'Address'
has value type string

column: 'Zip_Code'
has value type string

conditional rule: 'Country-Zip1'
if
column: 'Country' is in ['WAL', 'ENG']
then
column: 'Zip_Code' has value type string

conditional rule: 'Country-Zip2'
if
column: 'Country' is not in ['WAL', 'ENG']
then
column: 'Zip_Code' has value type integer

```

### Appendix D
The log file output of ruleset-engine has three levels of output messages.
- Info: Informational messages that indicate successful operations and provide summary details:
  - Confirmation of successful parsing of rules and data files.
  - Summary statistics of the data table, including row and column counts, and counts of missing values.
  - Notifications that all validation rules have passed without issues.
- Warning: Indicates potential issues that may not halt execution but could affect validation accuracy:
  - Columns listed in the rules file without associated value rules.
  - Value rules defined for columns not included in the column list.
  - Absence of any value rules in the rules file.
  - Mismatch between column names in the rules file and the data file, preventing evaluation based on column order.
- Error: Critical issues that prevent the validation process from proceeding correctly:
  - Invalid or inaccessible file paths for the rules, data, or reference files.
  - Errors encountered while parsing the rules file.
  - Failures during the execution of column-level, value-level, or second-order validation rules against the data.

Here is the log from validating data table in [Appendix C](#appendix-c) against the simple rule set.
```dsl
Info: Rules File ./squash_rules_v.1.0 is OK.
Info: All columns in columns list have associated value rules.
Info: All columns with at least one value rule are included in the columns list.
Info: Data file ./top_15_squash_players_full.csv parsed as csv.
Info: Rows:15
Info: Columns:8
Info: Missing Values:
Info: Missing value count for Rank: 0
Info: Missing value count for PSA_ID: 0
Info: Missing value count for First_Name: 0
Info: Missing value count for Last_Name: 0
Info: Missing value count for Country: 0
Info: Missing value count for Year_of_Birth: 0
Info: Missing value count for Address: 0
Info: Missing value count for Zip_Code: 0
Info: All columns present in data file.
Info: All required columns found in data file.
Info: No extra columns included in data file.
Info: Column: Rank all OK.
Info: Column: PSA_ID all OK.
Info: Column: First_Name all OK.
Info: Column: Last_Name all OK.
Info: Column: Country all OK.
Info: Column: Year_of_Birth all OK.
Info: Column: Address all OK.
Info: Column: Zip_Code all OK.
```
