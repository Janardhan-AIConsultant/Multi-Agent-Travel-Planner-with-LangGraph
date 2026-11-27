# Multi-Agent Travel Planner with LangGraph

**TripMate AI** turns a natural-language trip request into a travel plan with flight information, hotel suggestions, a day-by-day itinerary and an estimated budget.

## How it works

Four LangGraph agents run one after another and share a common state:

| Step | Agent | What it does | Service |
|------|-------|--------------|---------|
| 1 | Flight agent | Works out the route from the request and fetches live flight status | AviationStack |
| 2 | Hotel agent | Searches the web for hotels | Tavily |
| 3 | Itinerary agent | Writes a practical, budget-aware itinerary | Groq LLM |
| 4 | Final agent | Combines everything into the final response | Groq LLM |

LangGraph checkpoints are saved to PostgreSQL under a thread ID.

> AviationStack provides flight status data, **not ticket prices**.

## Project structure

```
app.py              FastAPI app: serves the web page and the API
backend.py          LangGraph agents, graph and PostgreSQL checkpointer
tools/
  flight_tool.py    AviationStack search and route parsing
  tavily_tool.py    Tavily web search
templates/          HTML page
static/             CSS and JavaScript
test.py             Command-line run of the full agent workflow
demo.excalidraw     Design diagram (may include tools that are not implemented)
requirements.txt    Python dependencies
```

## Prerequisites

- Python 3.12
- A PostgreSQL database that accepts SSL connections (for example, Render PostgreSQL)
- API keys for [Groq](https://console.groq.com), [Tavily](https://tavily.com) and [AviationStack](https://aviationstack.com)

## Setup

```bash
git clone https://github.com/Janardhan-AIConsultant/Multi-Agent-Travel-Planner-with-LangGraph.git
cd Multi-Agent-Travel-Planner-with-LangGraph
python -m venv .venv
```

Activate the virtual environment:

```bash
# Windows (PowerShell)
.\.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

## Configuration

Create a `.env` file in the project root. It is listed in `.gitignore`. **Never commit it, and never paste real keys or connection strings into tracked files.**

```env
GROQ_API_KEY=your_groq_api_key
GROQ_MODEL=your_groq_model_name
TAVILY_API_KEY=your_tavily_api_key
AVIATIONSTACK_API_KEY=your_aviationstack_api_key
DATABASE_URL=postgresql://USER:PASSWORD@HOST/DATABASE

# Optional
GROQ_TEMPERATURE=0
GROQ_BASE_URL=https://api.groq.com
DEFAULT_ORIGIN_IATA=DAC

# Optional: LangSmith tracing
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_API_KEY=your_langsmith_api_key
LANGSMITH_PROJECT=your_project_name
```

- `sslmode=require` is added to `DATABASE_URL` if it has no `sslmode` setting.
- `DEFAULT_ORIGIN_IATA` is the departure airport used when only a single destination can be matched from the request. It defaults to `DAC` (Dhaka).

## Run

```bash
python app.py
```

This starts the web page and the API together. Open http://127.0.0.1:8000.

To run the agents from the command line instead:

```bash
python test.py
```

## API

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | Web interface |
| POST | `/api/travel` | Generate a travel plan |
| GET | `/health` | Health check |

Request body for `POST /api/travel` (`thread_id` is optional):

```json
{
  "message": "Plan a 5 days Dubai trip from Dhaka with flights, hotels and sightseeing.",
  "thread_id": null
}
```

A successful response contains `success`, `thread_id`, `answer`, `flight_results`, `hotel_results`, `itinerary` and `llm_calls`.

## License

[Apache License 2.0](LICENSE)
