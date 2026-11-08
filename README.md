# Abigail Ray || Econ 3916 || SQL for Economists: Joins, Windows and the Cardinality Trap

## Objective
I translated pandas operations into SQL with DuckDB to show that they are the same relational algebra, and to see how a one-to-many join can quietly corrupt a statistic.

## Methodology
- Translated select, filter, sort, groupby and merge from pandas into SQL with DuckDB and checked that both give the same rows.
- Joined 600 customers to their orders with a one-to-many LEFT JOIN and ran the same Welch t-test of East against West revenue on the one-row-per-customer table and on the joined table.
- Used window functions (`AVG ... OVER (PARTITION BY ...)` and `RANK`) to compute region averages and ranks without collapsing rows.
- Ran `WHERE income != 50000` on a small survey table with missing incomes and compared it with the same filter in pandas.
- Queried a public Parquet file of about 3 million NYC taxi trips over HTTP without downloading it, reading only the columns the query needed.
- Read a query I did not write, predicted which of its columns the join corrupts, and rewrote it by summing orders per customer before joining.
- Asked an AI to translate three of my pandas steps into SQL, revised my prompt once, and compared the row counts of its queries with my pandas results.

## Key Findings
- A one-to-many LEFT JOIN turned 600 customer rows into 3,612 rows. The same Welch t-test on the same revenue figures went from t = 0.07 (p = 0.9413) to t = 0.71 (p = 0.4749). The standard error shrank from 9.35 to 3.75 and the East-West difference moved from 0.69 to 2.68, because each customer was counted about six times and heavy orderers got more weight.
- In the query I inherited, `customers` and `total_orders` were correct, but `rows_out` (1,688 East and 1,924 West) counted customer-order rows. `avg_revenue` was 493.91 and 491.24 against the true 490.91 and 490.22. `total_orders` summed to 291,178.06, the same as the sum over the orders table.
- After summing orders per customer before the join, the average revenue matched the true values (490.91 East, 490.22 West). `COUNT(DISTINCT ...)` was not enough on its own, because it only fixes the count it wraps.
- `WHERE income != 50000` kept 3 of 5 survey rows, because a comparison with a missing income is unknown in SQL and WHERE drops those rows. The same filter in pandas kept 5.
- The AI's first translation of Step 7 had that problem. After I told it to return exactly what the pandas line returns and keep the missing incomes, all three row counts matched my pandas results (2, 3,612 and 5).
