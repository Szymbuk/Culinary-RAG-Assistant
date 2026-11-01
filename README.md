# Culinary RAG Assistant

An end-to-end Retrieval-Augmented Generation (RAG) application for finding
recipes based on ingredients available to the user. The main application uses
local Ollama models, a Sentence Transformers embedding model, and LanceDB for
semantic recipe search. An optional Gemini-based variant is also included.

## Features

- semantic recipe search with LanceDB and
  `sdadas/mmlw-retrieval-roberta-large`,
- ingredient normalization and recipe decisions with local Ollama models,
- structured responses validated with Pydantic,
- Scrapy and BeautifulSoup scrapers for recipe websites,
- optional Gemini API integration,
- an evaluation script using Ragas,
- a CustomTkinter GUI.

## Requirements

- Python 3.12 or newer,
- Ollama for the main GUI and Ragas evaluation,
- enough disk space for the embedding model and the generated local vector
  database,
- a Gemini API key only when using the optional Gemini variant.

## Installation

Clone the repository and create a virtual environment:

```bash
git clone https://github.com/Szymbuk/AlgorytmyTekstoweProjekt.git
cd AlgorytmyTekstoweProjekt

python -m venv .venv
```

Activate it:

```bash
# Linux/macOS
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1
```

Install the Python dependencies:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

If you use `uv`, the equivalent setup is:

```bash
uv sync
```

## Ollama setup

Install Ollama using the instructions for your operating system:

<https://ollama.com/download>

Verify that it is available:

```bash
ollama --version
```

Start the Ollama service if it is not already running:

```bash
ollama serve
```

In another terminal, download the models used by the main application:

```bash
ollama pull qwen3:8b
ollama pull phi4:14b
```

The application uses:

| Model | Purpose |
| --- | --- |
| `qwen3:8b` | Normalizing the user's ingredients and identifying crucial ingredients |
| `phi4:14b` | Selecting recipes that can be prepared from the available ingredients |

Ollama normally exposes its API at `http://localhost:11434`. Keep the service
running while using the GUI.

## Optional Gemini configuration

The Gemini variant requires a key from
[Google AI Studio](https://aistudio.google.com/). Create a local environment
file from the provided example:

```bash
cp .env.example .env
```

Then set `GEMINI_API_KEY` in `.env`. The `.env` file is ignored by Git and must
not be committed or shared.

## Running the application

### GUI (recommended)

```bash
python frontend/gui_app.py
```

On the first run, the application downloads the embedding model and builds a
local LanceDB database in `backend/vector_base/vector_db/` from the JSON files
in `json_data/`.

### Experimental CLI variants

The project also contains two experimental command-line scripts:

```bash
python backend/main.py
python backend/main_api.py
```

`main_api.py` is a
Gemini-based command-line experiment.

## Recipe data and scrapers

Full scraped datasets and the generated LanceDB database are intentionally not
stored in Git. The repository contains only
[`json_data/sample_recipes.json`](json_data/sample_recipes.json) as a small
local sample. Scraped content may be subject to the terms and copyright
restrictions of its source websites.

To generate local data, run one of the scrapers, for example:

```bash
python backend/scrapers/spider_pycha.py
python backend/scrapers/kwestia_smaku_spider.py
python backend/scrapers/zolty_zeszyt_spider.py
python backend/scrapers/przepisy_spider.py
```

The scraper output is written locally and ignored by Git. The GUI will use
the JSON files when it creates the vector database.

Experimental notebooks are stored in [`notebooks/`](notebooks/).

## Evaluation

The Ragas evaluation uses Ollama's OpenAI-compatible endpoint and the
`phi4:14b` model. Make sure Ollama is running and the model has been pulled.

The script expects the dataset in `backend/ai/test_dataset.json`, so run it
from that directory:

```bash
cd backend/ai
PYTHONPATH=../.. python agent_evaluation.py
```

The current script prints the evaluation results. The historical notes are
kept in [`backend/ai/evaluation_report.txt`](backend/ai/evaluation_report.txt).

## Project structure

```text
.
├── backend/
│   ├── ai/                  # AI agents, schemas, prompts, and evaluation
│   ├── scrapers/            # Recipe scrapers
│   ├── vector_base/         # Embeddings, parser, LanceDB index and search
│   ├── dataset_generator.py
│   ├── main.py
│   ├── main_api.py
│   └── recipe_logic.py
├── frontend/
│   └── gui_app.py
├── json_data/
│   └── sample_recipes.json
├── notebooks/
├── .env.example
├── pyproject.toml
├── requirements.txt
└── uv.lock
```
