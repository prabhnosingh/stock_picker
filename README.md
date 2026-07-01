# Stock Picker

This project is a small CrewAI workflow for finding a promising stock idea from current news.

Instead of starting with a huge watchlist, it begins with what is already trending in a sector, researches a few companies in that space, and then picks one with a written rationale. It also has the option to send a push notification once the decision is made.

This is a working prototype for experimenting with agent orchestration, not a production investing tool.

## What the project does

On a normal run, the crew:

1. Looks for 2-3 US companies that are trending in the news for a chosen sector.
2. Researches each company's market position, future outlook, and investment potential.
3. Chooses the best candidate and explains why the others were not selected.
4. Sends a short push notification with the final pick.

Right now, the default entry point uses:

- `sector = "Technology"`
- `current_date = today`

Those values live in `src/stock_picker/main.py`.

## How the crew is organized

The workflow is defined in `src/stock_picker/crew.py` and uses a hierarchical CrewAI process.

- `trending_company_finder` searches the latest news using `SerperDevTool`.
- `financial_researcher` does deeper research on the shortlisted companies.
- `stock_picker` makes the final investment choice and triggers the notification tool.
- `manager` coordinates the flow and delegates work across the crew.

The current agent setup in `src/stock_picker/config/agents.yaml` uses `gpt-4o-mini` for the worker agents and `gpt-4o` for the manager.

## Files that matter

- `src/stock_picker/main.py`: entry points for running, training, replaying, testing, and trigger-based execution
- `src/stock_picker/crew.py`: crew definition, agents, tasks, and output schemas
- `src/stock_picker/config/agents.yaml`: agent roles, goals, and model configuration
- `src/stock_picker/config/tasks.yaml`: task descriptions, dependencies, and output files
- `src/stock_picker/tools/push_tool.py`: Pushover integration for the final notification
- `output/`: saved results from a run
- `knowledge/user_preference.txt`: a small notes file that exists in the repo, but is not currently wired into the crew

## Expected outputs

After a successful run, the project writes:

- `output/trending_companies.json`
- `output/research_report.json`
- `output/decision.md`

The sample files already in `output/` show what the generated artifacts look like.

## Setup

You'll need Python `>=3.10,<3.14`.

Install dependencies with `uv`:

```bash
pip install uv
uv sync
```

Then create a `.env` file in the project root with the keys the workflow expects:

```env
OPENAI_API_KEY=your_openai_key
SERPER_API_KEY=your_serper_key
PUSHOVER_USER=your_pushover_user
PUSHOVER_TOKEN=your_pushover_token
```

Notes:

- `OPENAI_API_KEY` is needed for the CrewAI agents.
- `SERPER_API_KEY` is needed because the research flow relies on live web search.
- `PUSHOVER_USER` and `PUSHOVER_TOKEN` are needed if you keep the notification step enabled as-is.

## Running the project

From the project root, run:

```bash
uv run stock_picker
```

That uses the `stock_picker.main:run` entry point defined in `pyproject.toml`.

If you want to change the sector, edit the `inputs` dictionary in `src/stock_picker/main.py` before running.

## Other entry points

The project also exposes a few extra commands through `pyproject.toml`:

- `uv run run_crew`
- `uv run train`
- `uv run replay`
- `uv run test`
- `uv run run_with_trigger`

The main polished path right now is the standard run flow. The `train`, `test`, and `run_with_trigger` functions still carry some starter-template style inputs in `main.py`, so they would probably need cleanup before being useful in a real workflow.

## A few honest notes

- Results will vary from day to day because the crew depends on live news and search results.
- The notification tool currently posts directly to Pushover using the values from your environment.
- This project is useful as an agent workflow demo, but it should not be treated as financial advice.
