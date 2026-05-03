# Support Triage Agent

This folder contains a terminal-based support triage agent built for a hackathon-style challenge.
It processes incoming support tickets and generates grounded responses using a local knowledge base of Markdown articles.

## Project Overview

The agent supports three product domains:
- `HackerRank` — hiring and coding assessment platform
- `Claude` — Anthropic AI assistant and API support
- `Visa` — global payment network and card support

It is designed to:
- classify each ticket into a product domain and intent
- determine whether a ticket can be answered automatically or should be escalated
- retrieve relevant content from a local Markdown corpus using TF-IDF
- generate a strict, grounded response using Anthropic Claude
- avoid hallucination by forcing answers only from the corpus

## Files in this folder

- `main.py` — main application and triage pipeline
- `README.md` — this documentation
- `__pycache__/` — compiled Python cache files

## How it works

Main pipeline steps:
1. Load corpus files from `data/hackerrank/`, `data/claude/`, and `data/visa/`.
2. Convert Markdown into plain text and split it into overlapping chunks.
3. Build a low-memory TF-IDF index over the chunks.
4. For each support ticket:
   - run a hard safety check for prompt injection or harmful content
   - classify the ticket into domain, intent, product area, request type, and escalation decision
   - if safe and not escalated, retrieve relevant corpus chunks
   - generate a grounded JSON response using Anthropic Claude
5. Write results to a CSV and save a transcript log.

## Requirements

Dependencies are tracked in the repository root `requirements.txt`:
- `anthropic>=0.25.0`

Python environment requirements:
- Python 3.11+ is recommended
- `ANTHROPIC_API_KEY` environment variable must be set before running

## Configuration

Key constants in `main.py`:
- `MODEL` — Claude model name used for classification and response generation
- `CORPUS_DIRS` — local domains under `data/`
- `INPUT_CSV` / `OUTPUT_CSV` — default ticket input and output paths
- `CHUNK_SIZE` / `CHUNK_OVERLAP` — corpus chunking settings
- `TOP_K` — number of chunks retrieved per ticket
- `STOP_WORDS` — used by TF-IDF tokenizer

## Data expectations

The project expects:
- `data/hackerrank/` — HackerRank markdown knowledge base
- `data/claude/` — Claude markdown knowledge base
- `data/visa/` — Visa markdown knowledge base
- `support_tickets/support_tickets.csv` — incoming ticket list

The ticket CSV should contain columns like:
- `Issue`
- `Subject`
- `Company`

## Running the agent

From the `code/` folder or repository root, run:

```bash
# with the local Anthropic API key set
python main.py
```

Optional flags:

```bash
python main.py --dry-run
python main.py --input support_tickets/sample_support_tickets.csv --output support_tickets/output.csv
```

The agent prints ticket summaries to the terminal and writes `output.csv` with these fields:
- `issue`
- `subject`
- `company`
- `response`
- `product_area`
- `status`
- `request_type`
- `justification`

## Safety and grounding

`main.py` enforces strong guardrails:
- hard safety checks reject prompt injection or explicit harmful requests
- classifier decides whether a ticket should be escalated
- response generation is told to use only provided reference articles
- out-of-scope tickets receive a safe fallback reply
- no emails, phone numbers, URLs, or contact details are invented

## Code structure

Sections in `main.py`:
- Section 1: Configuration constants
- Section 2: Logging and transcript utilities
- Section 3: Corpus loader and Markdown cleaner
- Section 4: TF-IDF retrieval index and scoring
- Section 5: Safety gate rules
- Section 6: Ticket classifier prompt and logic
- Section 7: Response generation prompt and logic
- Section 8: Ticket processing pipeline
- Section 9: CLI entrypoint and CSV I/O

## Usage notes

- This project is built to operate on the local corpus only; it does not use external search engines.
- The agent is intentionally conservative: if the corpus does not support an answer, it returns an out-of-scope fallback.
- The response generator asks Claude for JSON output only, reducing free-form answer risk.

## Troubleshooting

- If `ANTHROPIC_API_KEY` is missing, the script exits immediately.
- If corpus folders are missing or no Markdown is loaded, the script exits with an error.
- If the classifier or response generator fails, a safe escalation or fallback is used.

## Notes

This project is tailored for the provided hackathon corpus and support ticket workflow. The code is optimized for clarity and correctness over production-grade scaling.
