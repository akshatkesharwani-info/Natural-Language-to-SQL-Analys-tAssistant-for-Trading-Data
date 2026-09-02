# Natural-Language-to-SQL Analyst Assistant for Trading Data

**You ask:** *"Which trader had the largest total buy volume in AAPL last month?"*

**It generates and runs, verbatim from the actual run:**

```sql
SELECT trader
FROM trades
WHERE symbol = 'AAPL'
  AND side = 'buy'
  AND timestamp >= date('now','start of month','-1 month')
  AND timestamp < date('now','start of month')
GROUP BY trader
ORDER BY SUM(quantity) DESC
LIMIT 1;
```

**And explains in plain English:** *"It looks at all the 'buy' trades for Apple that happened during
the previous calendar month, adds up how many shares each trader bought, and picks the trader with
the highest total."*

(The sample data in this run had no trades that specific month, hence an empty result — the SQL
logic itself is correct, which is the part that matters.)

## Why this needs a safety layer

Letting an LLM write and run SQL against real trading data unsupervised is a bad idea if you don't
also check what it wrote. This notebook adds:

1. **Automatic schema introspection** — the tool reads your actual table structure, so it works on
   any trade blotter without hardcoded column names
2. **A hard SELECT-only validator** — anything that isn't a clean `SELECT` (`INSERT`, `UPDATE`,
   `DELETE`, `DROP`, etc.) is rejected before it ever touches the database
3. **A plain-English explanation** of the generated query, so a non-technical trader can sanity-check
   it before trusting the result

## Why local SQLite

Trading desks are often not permitted to send transaction data to a third-party API. Everything here
runs on a local SQLite database — only the natural-language question and the generated SQL (not the
underlying data) go anywhere near the LLM call.

## Stack

SQLite (local, free) · `call_llm()` (Cerebras → Groq → Ollama) · `pandas`

## Confirmed working

Both Cerebras calls in the run hit the free-tier quota and fell back to Groq successfully — the query
generation and the plain-English explanation both completed.

---
*Educational project. Always review generated SQL before running it against production data.*
