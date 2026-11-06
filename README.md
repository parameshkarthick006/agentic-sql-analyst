# Agentic SQL Analyst

Ask a business question in plain English. A LangGraph agent writes a SQL query, checks that it is
read-only, runs it, retries with the error message if it fails, and explains the result.

**Live demo:** _add your deployed link here_

## How it works

```
question -> load schema -> generate SQL (Groq LLM) -> safety check -> execute -> explain
                               ^                          |             |
                               +------- retry (max 3) ----+-------------+
```

- **Agent:** LangGraph state machine with conditional retry edges (`app/agent.py`)
- **Safety:** `sqlglot` parses every query; only a single `SELECT` is allowed (`app/guard.py`).
  PostgreSQL also runs each query in a read-only transaction.
- **API:** FastAPI (`POST /ask`, `GET /schema`, `GET /health`)
- **Data:** sample shop database (customers, products, orders, order_items), SQLite locally,
  PostgreSQL with docker-compose
- **UI:** small static page served by FastAPI

## Run it

```bash
cp .env.example .env        # add your Groq API key (free at console.groq.com)
pip install -r requirements.txt
uvicorn app.main:app --reload
# open http://localhost:8000
```

With PostgreSQL and Docker:

```bash
docker compose up --build
```

## Evaluate it

`eval/questions.json` has 8 questions with hand-written gold SQL. The eval runs the agent and
checks that its query returns the same rows as the gold query (execution accuracy).

```bash
python -m eval.run_eval      # writes eval/results.json
```

Add your own harder questions (joins, window functions, date logic) and record the results here.

| Run | Questions | Execution accuracy |
|-----|-----------|--------------------|
| _your first run_ | 8 | _fill in_ |

## Tests

```bash
pytest -q
```

## Project layout

```
app/        FastAPI app, agent, SQL guard, database helpers, seed data, static UI
eval/       Test questions with gold SQL and the eval runner
tests/      Unit tests for the SQL guard and API
```

## Limitations and next steps

- Schema is passed to the model in full; large databases need schema retrieval (for example pgvector).
- Evaluation set is small; extend it before trusting the accuracy number.
- Add authentication and rate limiting before exposing it publicly.
