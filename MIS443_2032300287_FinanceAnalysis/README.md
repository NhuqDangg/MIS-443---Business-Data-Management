# Finance Analysis — Retail Bank Database & SQL Examination

Đặng Huỳnh Quỳnh Như · Business Analytics Student · Eastern International University
IRN 2032300287 · Quarter 4, 2025–2026

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-14%2B-336791?logo=postgresql&logoColor=white) ![pgAdmin](https://img.shields.io/badge/pgAdmin-4-25597e) ![SQL](https://img.shields.io/badge/SQL-finance--analysis-lightgrey) ![Git](https://img.shields.io/badge/Git-tracked-orange?logo=git&logoColor=white)

---

## What this project is

This is the **MIS 443 Final Examination** — a 90-minute, 100-mark SQL assessment built around a retail banking dataset in PostgreSQL. The database tracks customers, branches, accounts, and transactions, and the questions ask progressively harder things of it: from simple filters and aggregates, up through multi-table JOINs, window functions, and CTEs.

The goal isn't just to get the right number — it's to write a query a manager could actually hand to an analyst and say *"run this every month."*

---

## Data model

| Table | Grain | Rows | Primary key |
|---|---|---|---|
| `customers` | one customer | 6 | `customer_id` |
| `branches` | one bank branch | 15 | `branch_id` |
| `accounts` | one account (Checking / Savings / Credit Card) | 15 | `account_id` |
| `transactions` | one transaction on one account | 15 | `transaction_id` |

**Key relationships:**
- Each `account` belongs to one `customer` and one `branch`
- Each `transaction` belongs to one `account`
- Positive balances = funds held; negative Credit Card balances = amounts owed
- Positive transaction amounts = credits; negative = debits

ERD: see `erd/ERD.png` (exported from pgAdmin's ERD Tool — `.pgerd` file also included).

---

## Examination questions & SQL answers

### Question 1 — Database Setup *(10 marks)*

Create a PostgreSQL database named `danghuynhquynhnhu`, connect to it, and execute `01_import_data.sql`. Confirm all four tables are in the public schema.

**Expected row counts:**

| Table | Rows |
|---|---|
| accounts | 15 |
| branches | 15 |
| customers | 6 |
| transactions | 15 |

📸 *Screenshot: ![Uploading image.png…]()
*

---

### Question 2 — Customer and Account Overview *(10 marks)*

**(a)** List customers living in New York — display `customer_id`, `full_name`, `city`, sorted by `customer_id`. *(5 marks)*

```sql
SELECT customer_id, first_name || '' || last_name AS full_name, city
FROM customers
WHERE city = 'New York'
ORDER BY customer_id;
```

Expected result:
```
1 | JohnDoe  | New York
2 | JaneDoe  | New York
```

📸 *Screenshot: `screenshots/q2a-new-york-customers.png`*

**(b)** Calculate the total number of accounts — name the result `total_accounts`. *(5 marks)*

```sql
SELECT COUNT(account_id) AS total_accounts
FROM accounts;
```

Expected result: `15`

📸 *Screenshot: `screenshots/q2b-total-accounts.png`*

---

### Question 3 — Account Balance Analysis *(20 marks)*

**(a)** Total balance across all Checking accounts — name the result `total_checking_balance`. *(10 marks)*

```sql
SELECT SUM(balance) AS total_checking_balance
FROM accounts
WHERE account_type = 'Checking';
```

Expected result: `31000.00`

📸 *Screenshot: `screenshots/q3a-checking-balance.png`*

**(b)** For each customer in Los Angeles, display `customer_id`, `full_name`, and `total_balance` across all account types — sorted by `total_balance` descending. *(10 marks)*

```sql
SELECT c.customer_id, c.first_name || '' || c.last_name AS full_name,
       SUM(a.balance) AS total_balance
FROM customers c
JOIN accounts a ON c.customer_id = a.customer_id
WHERE city = 'Los Angeles'
GROUP BY c.customer_id
ORDER BY total_balance DESC;
```

Expected result:
```
5 | MichaelLee   | 60000.00
6 | JenniferWang | 15000.00
```

📸 *Screenshot: `screenshots/q3b-la-customers.png`*

---

### Question 4 — Branch and Customer Portfolio Analysis *(20 marks)*

**(a)** Branch with the highest average account balance — display `branch_id`, `branch_name`, `city`, `average_balance` (rounded to 2 decimals). Include ties. *(10 marks)*

```sql
SELECT
    a.branch_id,
    b.branch_name,
    c.city,
    ROUND(AVG(a.balance)::numeric, 2) AS average_balance
FROM accounts a
JOIN customers c ON c.customer_id = a.customer_id
JOIN branches b ON b.branch_id = a.branch_id
GROUP BY a.branch_id, b.branch_name, c.city
ORDER BY average_balance DESC
LIMIT 1;
```

Expected result:
```
14 | North Beach | San Francisco | 30000.00
```

📸 *Screenshot: `screenshots/q4a-top-branch-avg.png`*

**(b)** Customer who owns the single account with the highest current balance — display `customer_id`, `full_name`, `account_id`, `account_type`, `balance`. Include ties. *(10 marks)*

```sql
SELECT c.customer_id, c.first_name || '' || last_name AS full_name,
       a.account_id, a.account_type, a.balance
FROM accounts a
JOIN customers c ON c.customer_id = a.customer_id
GROUP BY c.customer_id, full_name, a.account_id, a.account_type, a.balance
ORDER BY balance DESC
LIMIT 1;
```

Expected result:
```
5 | MichaelLee | 10 | Savings | 50000.00
```

📸 *Screenshot: `screenshots/q4b-highest-account.png`*

---

### Question 5 — Customer Value and Activity *(20 marks)*

**(a)** Most active customers by total number of transactions across all accounts — display `customer_id`, `full_name`, `total_transactions`. Include ties. *(10 marks)*

```sql
SELECT c.customer_id, c.first_name || '' || c.last_name AS full_name,
       COUNT(t.transaction_id) AS total_transactions
FROM accounts a
JOIN customers c ON a.customer_id = c.customer_id
JOIN transactions t ON a.account_id = t.account_id
GROUP BY c.customer_id
ORDER BY total_transactions DESC
FETCH FIRST 1 ROWS WITH TIES;
```

Expected result:
```
2 | JaneDoe      | 4
4 | AliceJohnson | 4
```

📸 *Screenshot: `screenshots/q5a-most-active-customers.png`*

**(b)** Customer with the highest total balance across Checking and Savings accounts only — display `customer_id`, `full_name`, `total_deposit_balance`. Include ties. *(10 marks)*

```sql
SELECT c.customer_id, c.first_name || '' || c.last_name AS full_name,
       SUM(a.balance) AS total_deposit_balance
FROM customers c
JOIN accounts a ON c.customer_id = a.customer_id
WHERE account_type IN ('Checking', 'Savings')
GROUP BY c.customer_id
ORDER BY total_deposit_balance DESC
FETCH FIRST 1 ROWS WITH TIES;
```

Expected result:
```
5 | MichaelLee | 60000.00
```

📸 *Screenshot: `screenshots/q5b-top-deposit-customer.png`*

---

### Question 6 — Advanced Finance Analysis *(20 marks)*

**(a)** Branch with the highest total balance across all account types — display `branch_id`, `branch_name`, `total_balance`. Include ties. *(10 marks)*

```sql
WITH ranked_branches AS (
    SELECT
        b.branch_id,
        b.branch_name,
        SUM(a.balance) AS total_balance,
        DENSE_RANK() OVER (ORDER BY SUM(a.balance) DESC) AS rnk
    FROM public.branches b
    JOIN public.accounts a ON b.branch_id = a.branch_id
    GROUP BY b.branch_id, b.branch_name
)
SELECT branch_id, branch_name, total_balance
FROM ranked_branches
WHERE rnk = 1;
```

Expected result:
```
14 | North Beach | 60000.00
```

📸 *Screenshot: `screenshots/q6a-top-branch-total.png`*

**(b)** Rank all customers by total balance across all account types — equal totals share the same rank with no gaps. Display `customer_id`, `full_name`, `total_balance`, `balance_rank`. No CTE. *(5 marks)*

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS full_name,
    SUM(a.balance) AS total_balance,
    DENSE_RANK() OVER (ORDER BY SUM(a.balance) DESC) AS balance_rank
FROM public.customers c
JOIN public.accounts a ON c.customer_id = a.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY balance_rank ASC;
```

Expected result:
```
5 | Michael Lee   | 60000.00 | 1
4 | Alice Johnson | 25000.00 | 2
3 | Bob Smith     | 20500.00 | 3
6 | Jennifer Wang | 15000.00 | 4
2 | Jane Doe      | 11500.00 | 5
1 | John Doe      |  5500.00 | 6
```

📸 *Screenshot: `screenshots/q6b-customer-ranking.png`*

**(c)** Use a CTE to find branches with the highest total transaction count — include branches with zero transactions. Display `branch_id`, `branch_name`, `total_transactions`. Include ties. *(5 marks)*

```sql
WITH branch_transaction_counts AS (
    SELECT
        b.branch_id,
        b.branch_name,
        COUNT(t.transaction_id) AS total_transactions,
        DENSE_RANK() OVER (ORDER BY COUNT(t.transaction_id) DESC) AS rnk
    FROM public.branches b
    LEFT JOIN public.accounts a ON b.branch_id = a.branch_id
    LEFT JOIN public.transactions t ON a.account_id = t.account_id
    GROUP BY b.branch_id, b.branch_name
)
SELECT branch_id, branch_name, total_transactions
FROM branch_transaction_counts
WHERE rnk = 1;
```

Expected result:
```
1 | Main      | 4
8 | South Bay | 4
```

📸 *Screenshot: `screenshots/q6c-top-branch-transactions.png`*

---

## SQL techniques used

| Technique | Where applied |
|---|---|
| `WHERE` filter | Q2a, Q3a, Q3b, Q5b |
| `COUNT()`, `SUM()`, `AVG()` | Q2b, Q3a, Q3b, Q4a, Q5a, Q5b |
| `INNER JOIN` (multi-table) | Q3b, Q4a, Q4b, Q5a, Q5b |
| `GROUP BY` | Q3b, Q4a, Q4b, Q5a, Q5b, Q6b |
| `ORDER BY` + `LIMIT` | Q4a, Q4b |
| `FETCH FIRST n ROWS WITH TIES` | Q5a, Q5b |
| `DENSE_RANK() OVER (...)` | Q6a, Q6b, Q6c |
| CTE (`WITH ... AS (...)`) | Q6a, Q6c |
| `LEFT JOIN` (zero-transaction branches) | Q6c |

---

## How to run this project

1. Create a PostgreSQL database named `danghuynhquynhnhu` in pgAdmin 4
2. Open and run `sql/01_import_data.sql` — this creates and populates all four tables
3. Open `sql/02_practice_exercise.sql` and run each question's block individually
4. Verify row counts match the table above

---

## Skills this project covers

| Foundations | Querying | Aggregation & Joins | Window Functions | Business Framing |
|---|---|---|---|---|
| ✅ | ✅ | ✅ | ✅ | ✅ |

---

## Repo structure

```
MIS443_2032300287_FinanceAnalysis/
├── README.md
├── erd/
│   ├── ERD.png
│   └── ERD.pgerd
├── sql/
│   ├── 01_import_data.sql
│   └── 02_practice_exercise.sql
└── screenshots/
    ├── q1-database-setup.png
    ├── q2a-new-york-customers.png
    ├── q2b-total-accounts.png
    ├── q3a-checking-balance.png
    ├── q3b-la-customers.png
    ├── q4a-top-branch-avg.png
    ├── q4b-highest-account.png
    ├── q5a-most-active-customers.png
    ├── q5b-top-deposit-customer.png
    ├── q6a-top-branch-total.png
    ├── q6b-customer-ranking.png
    └── q6c-top-branch-transactions.png
```

---

## GitHub

🔗 [MIS-443---Business-Data-Management](https://github.com/NhuqDangg/MIS-443---Business-Data-Management/tree/main)
