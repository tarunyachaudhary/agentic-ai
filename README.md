# 🎬 Movie Info Agent

An agentic AI project that answers movie questions using **LangChain**, **Groq** (LLM inference) and the **TMDB API**. The agent decides on its own when to search for a movie and when to fetch its official details, so ratings, genres and release dates always come from real data instead of the model's memory.

---

## ✨ Features

- **Tool-calling agent** built with LangChain's `create_agent` (runs on LangGraph under the hood).
- **Two TMDB tools** the agent can call:
  - `search_tmdb_movie` – finds a movie by title and returns its ID, release date and overview.
  - `get_tmdb_movie_details` – fetches official genres, rating, vote count and release date by movie ID.
- **Structured extraction** – uses `with_structured_output` (TypedDict + JSON schema) to pull the movie title and any user-mentioned genres out of free text.
- **No hallucinated facts** – the system prompt forbids inventing ratings, genres or release dates, and the agent tells the user clearly when a movie isn't found.
- **Resilient HTTP calls** – a `requests` session with automatic retries and exponential backoff for `429` and `5xx` errors.
- **Release status helper** – plain Python logic to decide whether a movie is already released.
- **Streaming support** – supports both `updates` and `messages` stream modes.

---

## 🧠 How It Works

```
User question
     │
     ▼
┌──────────────────────────┐
│  LangChain Agent         │  LLM: openai/gpt-oss-120b (via Groq)
│  (system prompt + tools) │
└────────────┬─────────────┘
             │ decides which tool to call
     ┌───────┴────────┐
     ▼                ▼
search_tmdb_movie   get_tmdb_movie_details
     │                │
     └───────┬────────┘
             ▼
        TMDB REST API
             │
             ▼
   Grounded, final answer
```

Example flow for *"Find The Dark Knight and give me its genres, release date, rating and vote count."*:

1. Agent calls `search_tmdb_movie("The Dark Knight")` → gets movie ID `155`.
2. Agent calls `get_tmdb_movie_details(155)` → gets genres, rating, votes, release date.
3. Agent writes the final answer using only the retrieved data.

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|------------|
| Language | Python |
| Package manager | [uv](https://docs.astral.sh/uv/) |
| Agent framework | LangChain / LangGraph |
| LLM provider | [Groq](https://groq.com/) (`langchain-groq`) |
| Model | `openai/gpt-oss-120b` |
| Movie data | [TMDB API](https://developer.themoviedb.org/) |
| HTTP | `requests` + `urllib3` retry |
| Config | `python-dotenv` |

---

## 📁 Project Structure

```
movie-info-agent/
├── notebooks/
│   └── 2-Movie_info_agent.ipynb   # Main walkthrough: extraction, tools, agent, streaming
├── src/                           # Source code
├── .env                           # API keys (not committed)
├── .gitignore
├── .python-version
├── pyproject.toml                 # Project metadata & dependencies (uv)
├── requirements.txt
├── uv.lock
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- Python (see `.python-version`)
- [uv](https://docs.astral.sh/uv/getting-started/installation/) installed
- A **Groq API key** – get one at [console.groq.com](https://console.groq.com/)
- A **TMDB API Read Access Token** – get one at [themoviedb.org/settings/api](https://www.themoviedb.org/settings/api)

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/movie-info-agent.git
cd movie-info-agent
```

### 2. Install dependencies

Using uv (recommended):

```bash
uv sync
```

Or with pip:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Configure environment variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key_here
TMDB_READ_ACCESS_TOKEN=your_tmdb_read_access_token_here
```

> ⚠️ Never commit your `.env` file. It is already listed in `.gitignore`.

### 4. Run the notebook

```bash
uv run jupyter lab
```

Then open `notebooks/2-Movie_info_agent.ipynb` and run the cells in order.

---

## 💻 Usage

```python
from langchain.agents import create_agent

agent = create_agent(
    model=llm_model,
    tools=[search_tmdb_movie, get_tmdb_movie_details],
    system_prompt="You are a movie information assistant. ..."
)

result = agent.invoke({
    "messages": [
        {"role": "user", "content": "Find Interstellar and tell me its rating."}
    ]
})

print(result["messages"][-1].content)
```

### Sample output

**Query:** *Find The Dark Knight and give me its official genres, release date, rating and vote count.*

```
The Dark Knight (2008)

- Genres: Action, Thriller, Crime
- Release date: July 16, 2008
- Rating (TMDB average vote): 8.54
- Vote count: 37,343
```

**Query:** *Tell me about a movie called XYZ12345.*

```
I'm sorry, but I couldn't find any movie with the title "XYZ12345."
If you have any other title or additional details, feel free to share them.
```

### Streaming

```python
# Step-by-step updates (model turns and tool results)
for chunk in agent.stream({"messages": [...]}, stream_mode="updates"):
    print(chunk)

# Token-by-token output
for chunk in agent.stream({"messages": [...]}, stream_mode="messages"):
    print(chunk)
```

---

## 🧩 Notebook Walkthrough

| Section | What it covers |
|---------|----------------|
| Setup | Loads env vars, creates a retry-enabled TMDB session |
| List models | Lists the models available on your Groq account |
| Structured output | Extracts `title` and `genres` from natural language with a `TypedDict` schema |
| TMDB integration | `search_movie()` and `get_movie_details()` helper functions |
| Release status | `calculate_release_status()` checks if a movie is out yet |
| Pipeline version | `movie_info_agent()` – a fixed, non-agentic pipeline for comparison |
| Tools | Wraps TMDB helpers as LangChain `@tool` functions |
| Agent | Builds the agent with `create_agent` and a guard-railed system prompt |
| Testing | Invalid title, streaming modes, and a full multi-tool query |

---

## 🔮 Future Improvements

- Add tools for cast, crew, trailers and "similar movies"
- Support year-based disambiguation (e.g. "Dune 1984")
- Add conversation memory for follow-up questions
- Wrap the agent in a Streamlit or FastAPI app
- Move notebook logic into reusable modules in `src/`

---

## 🙏 Acknowledgements

- [TMDB](https://www.themoviedb.org/) for the movie data. *This product uses the TMDB API but is not endorsed or certified by TMDB.*
- [LangChain](https://www.langchain.com/) and [Groq](https://groq.com/)

---

## 📄 License

This project is licensed under the MIT License. Add a `LICENSE` file if you plan to publish it.
