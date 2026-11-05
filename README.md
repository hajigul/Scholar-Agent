# ScholarAgent

A local, free research assistant that finds scientific papers and summarizes them with a small language model running on your own computer. No API keys, no accounts, no cost, and your queries never leave your machine.

It searches **arXiv** and **OpenAlex**, shows verified paper cards (title, authors, date, abstract, citation counts), and adds a short AI-written overview. The cards come straight from the sources. Only the overview is written by the model.

![Home screen](docs/screenshot-home.png)

![Search results](docs/screenshot-results.png)

## Features

- **Local and private.** Runs on [Ollama](https://ollama.com); no cloud LLM and no API key.
- **Three tools** the agent can call:
  - `search_arxiv`: preprints, with exact-phrase search and labeled fallbacks (relaxed or broad match).
  - `search_openalex`: published literature across venues, with citation counts and a "most cited" sort.
  - `get_paper_details`: full abstract, authors, categories and PDF link for one arXiv paper.
- **Two modes.**
  - *Fast:* a rule-based router calls the tool directly, then the model streams a short overview.
  - *Agent:* the model chooses the tools itself and handles follow-ups like "the second paper".
- **Honest results.** Each result is labeled (exact phrase, relaxed, broad, fallback), and each paper gets a keyword-based relevance badge.
- **Research workflow.** Save papers to a library, copy BibTeX per paper or for all results, and export the library as BibTeX or CSV.
- **Fast.** Disk cache for searches, model keep-alive and prewarm, and a timing breakdown (model time vs search time) under every answer.
- **Evaluation suite.** `eval.py` runs 13 test cases, compares modes and models, and exports a sheet for hand-labeling relevance.
- **Web UI.** Dark and light themes, model picker, source selector, streaming output, and chat export to Markdown.

## Quick start

**Requirements:** Python 3.9+ and [Ollama](https://ollama.com/download).

### Mac / Linux

```bash
git clone https://github.com/hajigul/ScholarAgent.git
cd ScholarAgent
chmod +x run.sh
./run.sh
```

`run.sh` creates a virtual environment, installs the dependency, pulls the model if missing, and starts the app at http://127.0.0.1:8000.

### Manual setup (any OS)

```bash
python3 -m venv agent-env
source agent-env/bin/activate        # Windows: agent-env\Scripts\activate
pip install -r requirements.txt
ollama pull llama3.2:3b
python server.py
```

## Usage

**Web app**

```bash
python server.py            # http://127.0.0.1:8000
python server.py 8010       # another port
```

Example questions:

- `Find recent papers on knowledge graph completion.`
- `Highly cited papers on link prediction.`
- `Find papers on retrieval augmented generation with knowledge graphs.`
- `Get the full details of arXiv 1706.03762.`

**Command line**

```bash
python agent.py "Find papers on drug target prediction"
python agent.py --agent "Find papers on drug target prediction"   # agent mode
```

## Configuration

| Variable | Default | Purpose |
|---|---|---|
| `SCHOLAR_MODEL` | `llama3.2:3b` | Ollama model to use |
| `SCHOLAR_PORT` | `8000` | Server port |
| `SCHOLAR_EMAIL` | empty | Optional contact email sent to OpenAlex (it serves identified clients more reliably) |

Other settings (cache lifetime, keep-alive) are in `config.py`.

## How it works

```
Question
   |
   +-- Fast mode:  rule-based router --+
   +-- Agent mode: LLM picks the tool -+--> tool call (arXiv / OpenAlex / details)
                                              |
                                  verified paper cards (shown first)
                                              |
                                  LLM writes a short streamed overview
```

The key design choice is that the model never re-types the paper list. The cards are built from the source data, which keeps answers fast and avoids invented titles, authors or links.

## Project structure

```
ScholarAgent/
├── server.py        # local web server (streams events to the UI)
├── index.html       # front end
├── agent.py         # fast router, agent loop, CLI
├── tools.py         # arXiv, OpenAlex, paper-details tools + caching
├── config.py        # settings
├── eval.py          # evaluation suite
├── run.sh           # one-command setup and launch
├── requirements.txt
└── docs/            # screenshots
```

## Evaluation

```bash
python eval.py --limit 3          # quick check
python eval.py --mode both        # fast vs agent, all 13 cases
python eval.py --model llama3.1:8b --mode both
python eval.py --score FILE_labels.csv   # score your hand labels
```

Metrics per case: pass/fail, right tool called, keyword precision, `sort=date` used when "recent" was asked, expected paper found, invented arXiv IDs, time to first result, and total latency. Searches are not cached during evaluation, so timings are honest.

**Results on the built-in test set (`llama3.2:3b`, Apple M1 Pro):**

| | Fast mode | Agent mode |
|---|---|---|
| Pass rate | 13/13 | 13/13 |
| Mean keyword precision | 0.978 | 0.978 |
| Invented IDs | 0/13 | 0/13 |
| Mean latency | 4.3 s | 5.3 s |

An earlier run, before fixes to query handling and the citation-sorted search, scored 11/13 (fast) and 10/13 (agent).

**Limitations of this evaluation:**

- The test set was written by the author and the tools were tuned against its failures, so these scores are optimistic and do not measure generalization.
- Keyword precision only checks whether the topic words appear in the title or abstract. Hand-labeled relevance (via the `_labels.csv` export and `--score`) is the more reliable measure.
- 13 cases and a single run cannot support claims about small differences between modes or models.

## Limitations

- The overview is written by a small local model and can be generic or wrong. Treat the cards and links as the source of truth.
- Fast mode extracts topics with simple rules, so very conversational questions may need Agent mode.
- "Most cited" means most cited among the best text matches from OpenAlex, not among every paper on the topic.
- arXiv requests are paced 3 seconds apart, as arXiv's API terms request.

## Security note

By default the server listens on `127.0.0.1` only. The `--lan` flag exposes it to your local network with **no login**, so only use it on a network you trust. Do not put it on the public internet.

## Roadmap

- Full-text PDF reading tool
- Local paper library with embeddings and retrieval
- Larger, independently labeled evaluation set
- Knowledge-graph query tool (e.g., KEGG)

## Author

**Haji Gul**: PhD candidate (AI), Universiti Brunei Darussalam.
Research interests: knowledge graph completion, link prediction, LLM-augmented graph reasoning.

GitHub: [@hajigul](https://github.com/hajigul)

## License

MIT License. See `LICENSE`.

## Acknowledgments

[Ollama](https://ollama.com), [arXiv API](https://info.arxiv.org/help/api/index.html), [OpenAlex](https://openalex.org).
