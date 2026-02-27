# SQL Experiment: VARCHAR(100) vs VARCHAR(255) Storage Size

This repository contains a simple SQL experiment to compare the physical disk space utilized by PostgreSQL when storing data in tables with different `VARCHAR` column limits.

Specifically, this experiment tests whether defining a column as `VARCHAR(100)` versus `VARCHAR(255)` results in a significant difference in relation size.

## 📝 Experiment Overview

In many database systems, including PostgreSQL, `VARCHAR(n)` stores text up to a maximum of `n` characters. A common misconception is that a larger `n` value (like 255) automatically allocates more disk space per row than a smaller `n` value (like 100).

This experiment creates two identical tables with the exception of their `VARCHAR` constraints, inserts batches of data, and then queries the database engine directly to report the physical size of each table on disk.

## 📂 Repository Structure

* **`table_1.sql`**: DDL to create `table_1` containing a `VARCHAR(100)` column, followed by `INSERT` statements to populate it with data.
* **`table_2.sql`**: DDL to create `table_2` containing a `VARCHAR(255)` column, followed by `INSERT` statements to populate it with data.
* **`table_size.sql`**: A PostgreSQL-specific query utilizing `pg_relation_size()` and `pg_size_pretty()` to easily compare the final sizes of both tables side-by-side.

## 🛠️ Prerequisites

To run this experiment, you will need:
* A running **PostgreSQL** instance (as the size comparison utilizes Postgres-specific internal functions).
* A SQL client (like `psql`, pgAdmin, or DBeaver) to execute the scripts.

## 🚀 How to Run the Experiment

1. **Create and populate the first table**:
    Execute the contents of `table_1.sql` against your PostgreSQL database.
    ```bash
    psql -U your_username -d your_database -f table_1.sql
    ```

2. **Create and populate the second table**:
    Execute the contents of `table_2.sql` against your PostgreSQL database.
    ```bash
    psql -U your_username -d your_database -f table_2.sql
    ```

3. **Compare the table sizes**:
    Execute the `table_size.sql` script to see the results.
    ```bash
    psql -U your_username -d your_database -f table_size.sql
    ```

## 📊 Expected Results & Theory

When you run `table_size.sql`, you will get a result similar to this:

| table_1 | table_2                   |
|:--------|:--------------------------|
| 160 kB  | 320 kB *(example values)* |

**What's actually happening?**
In PostgreSQL, `VARCHAR(n)` does not pad the string with spaces. The space consumed on disk is determined by the *actual length of the string inserted* plus a small amount of overhead (usually 1 or 4 bytes for the length word), not the declared maximum length `n`.

If `table_1` and `table_2` contain strings of the exact same length, their disk sizes will be identical. If `table_2` is populated with longer strings (closer to its 255 characters limit), `table_2` will be larger purely because the inserted payload is larger, not because the schema reserved more space.

This implies that the attribute `length` in `@Column` annotation in Spring Boot (which in turns control the argument for `VARCHAR(x)` in the generated DDL) does not impact on the storage efficiency, it only restricts the maximum length of string that can be inserted.
