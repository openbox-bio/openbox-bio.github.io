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

---

## Table of Contents

1. [Set Value Rules](#set-value-rules)  
2. [Column Rules](#column-rules)  
3. [Value Rules](#value-rules)  
4. [Second Order Validation](#second-order-validation-1)
4. [Appendix](#appendix) 

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
- **`in [ … ]`** / **`not in [ … ]`**  
  ```dsl
  in ['Active', 'Pending', 'Closed']
  not in ['Cancelled', 'Deleted']
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
column: country is "USA"
then
column: zipcode has pattern /^\d{5}(-\d{4})?$/
```

---

## Appendix

### Appendix A

Value types that can be specified in the `has value type` rule.
| Value Type     | Description                                                                 |
| -------------- | --------------------------------------------------------------------------- |
| string         | Text-based data (e.g., names, addresses, categories).                       |
| integer        | Whole numbers without decimal points (e.g., 1, 42, -10).                    |
| floating point | Decimal numbers (e.g., 3.14, -0.01, 2.718).                                 |
| boolean        | True/False values (e.g., true, false).                                      |
| date-time      | Date or timestamp values (e.g., 2023-12-25, 2025-03-27T14:00:00).           |
| scientific     | Numbers expressed in scientific notation (e.g., 1.23e-4).                   |
| complex        | Complex numbers with real and imaginary parts (e.g., 3+4j).                 |


### Appendix B
Supported date-time formats that can be specified in the `has format` rule:
<br> `["HH:mm:ss" , "HH:mm" , "hh:mm:ss AM/PM" , "hh:mm AM/PM" , "HHmmss" , "YYYY-MM-DD HH:mm:ss" , 
"YYYY-MM-DDTHH:mm:ss" , "YYYY-MM-DD HH:mm" , "YYYY-MM-DDTHH:mm", "DD/MM/YYYY HH:mm:ss" , 
"MM-DD-YYYY hh:mm:ss AM/PM" , "YYYY/MM/DD HH:mm:ss" , "YYYY-MM-DDTHH:mm:ss.sssZ" , 
"YYYY-MM-DDTHH:mm:ss+hh:mm" , "YYYY-MM-DD" , "MM-DD-YYYY" , "DD-MM-YYYY" , "YYYY/MM/DD" , 
"MM/DD/YYYY", "DD/MM/YYYY" , "YYYY.MM.DD" , "MM.DD.YYYY" , "DD.MM.YYYY" , "YYYYMMDD"]`
<br>Other data formats for geolocation and currency data will be added in future.
