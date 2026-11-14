# 📋 BriefMe: AI Pre-Meeting Briefing Agent

> Walk into every meeting prepared. Give BriefMe a company or person and a meeting time, and get a one-page, source-backed briefing built from the last 30 days of news.

**Hackathon:** Geek Room Global AI Hackathon (Oct 9 – Nov 29, 2026)
**Theme:** AI Agents & Intelligent Systems
**Problem statement:** Tavily Research Agent
**Built by:** Mohd Avaish and Avani D Poojary

---

## The problem

Before an important meeting, people spend 30+ minutes jumping between news sites, press releases and social profiles to find out what changed recently. Most still walk in underprepared, and generic AI answers often rely on stale or unsourced information.

## The solution

BriefMe is a research agent that, given a **company or person** and a **meeting time**, researches the preceding **30 days** and produces a **one-page briefing** with:

1. **Snapshot**: who they are right now, in 2-3 lines
2. **Last 30 days**: news, launches, hires, funding and other updates, each with a date and source link
3. **Suggested discussion points**: tailored to the recent developments (and the meeting purpose, if given)
4. **Open questions**: smart things to ask
5. **Sources**: every claim traces back to a link

## Features

- 30-day research window anchored to the meeting date (it never searches past today)
- Multi-angle search run in parallel: news, product launches, hires/leadership changes, funding/partnerships (different angles for people vs companies)
- Source-backed output where every finding carries a date and citation
- Hallucination guardrails: the LLM may only use the provided sources, and the code removes citations to non-existent sources and drops uncited findings
- Optional meeting purpose to tailor discussion points
- Agent trace in the UI showing the queries run and sources found
- Streamlit web UI, command-line interface, and Markdown export

## Architecture

```
 Company/Person + Meeting time (+ purpose)
                │
        ┌───────▼────────┐
        │  Query planner │  4 angles (news, launches, people, money)
        └───────┬────────┘
                │ parallel
        ┌───────▼────────┐
        │ Tavily Search  │  topic=news, 30-day start/end window
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Filter + dedupe│  drop out-of-window, dedupe URLs, rank by relevance
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ Tavily Extract │  fuller text for the top 5 articles
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │  LLM synthesis │  grounded JSON, cited by source id
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │   Validator    │  strips invalid / uncited claims
        └───────┬────────┘
                │
       One-page Markdown briefing (UI / CLI / download)
```

## How Tavily is used

| Tavily feature | Purpose |
|---|---|
| `search` (`topic="news"`, `start_date` / `end_date`) | Finds recent articles strictly inside the 30-day window, run in parallel across 4 query angles |
| `extract` | Pulls fuller article text for the top sources so summaries rest on more than snippets |

## How AI is used

- **Tavily** provides live, real-time web research (the agent's eyes)
- **Anthropic Claude** reads the numbered sources and produces a structured JSON briefing, restricted to facts found in those sources
- **Code-level validation** checks every citation against the real source list before rendering
- **AI-assisted development:** Claude was used as a coding assistant while building this project

## Tech stack

Python 3.10+ · Tavily (`tavily-python`) · Anthropic Claude · Streamlit · python-dotenv

## Project structure

```
briefme/
├── agent.py            # core pipeline: plan, search, filter, extract, synthesize, validate, render
├── app.py              # Streamlit web UI
├── cli.py              # command-line interface
├── tests/test_logic.py # unit tests for filtering, dedupe, parsing, guardrails, rendering
├── requirements.txt
└── .env.example
```

## Installation

```bash
git clone <your-repo-url>
cd briefme
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env             # Windows: copy .env.example .env
```

### Environment variables

| Variable | Required | Description |
|---|---|---|
| `TAVILY_API_KEY` | Yes | Get one at https://app.tavily.com |
| `ANTHROPIC_API_KEY` | Yes | Get one at https://console.anthropic.com |
| `BRIEFME_MODEL` | No | Override the Claude model used for synthesis |

## How to run

**Web UI**
```bash
streamlit run app.py
```

**CLI**
```bash
python cli.py "Stripe" --when "2026-11-20 15:00" --kind company --purpose "Partnership intro" --out briefing.md
```

**Tests**
```bash
python -m pytest tests
```

## Example output

> Illustrative example of the output format using a fictional company. See the demo video for real runs.

```markdown
# Briefing: Northwind Robotics
*Meeting: Fri 20 Nov 2026, 15:00 · Research window: 2026-10-21 → 2026-11-14*

## Snapshot
Northwind Robotics builds warehouse automation systems. It has expanded into
Europe this month and is hiring aggressively in sales and engineering.

## Last 30 days
- **Opens Berlin office** (2026-11-05): Announced its first European hub to serve EU customers. [1]
- **Names new CFO** (2026-10-28): Appointed a finance chief ahead of a planned funding round. [2]

## Suggested discussion points
1. How the Berlin expansion changes their integration and support needs.
2. Their priorities under the new finance leadership.

## Open questions to ask
- What is the timeline for European rollouts?
- How will the new CFO affect vendor and procurement decisions?

## Sources
[1] Example article title (https://example.com/a), 2026-11-05
[2] Example article title (https://example.com/b), 2026-10-28
```

## Mapping to the judging criteria

| Criterion | How BriefMe addresses it |
|---|---|
| Innovation | Meeting-time-anchored research window plus angle-specific query planning and a citation guardrail |
| Technical execution | Parallel Tavily search, Extract enrichment, JSON validation, unit-tested logic |
| Problem & impact | Saves 30+ minutes of prep per meeting for sales, recruiting, investing and partnerships |
| Use of AI | Tavily for live retrieval, Claude for grounded synthesis and tailored questions |
| User experience | One form in, one page out, with a downloadable Markdown briefing and a transparent trace |
| Scalability & potential | Stateless pipeline; extendable to calendars, email delivery and caching |

## Limitations

- Results depend on what Tavily can find in the window; very obscure names may return thin briefings (the agent says so instead of padding)
- Common names can return mixed results, so adding context in the name (e.g. "Acme Inc.") helps
- Briefings are only as good as the cited sources, so always verify critical facts via the links

## Future scope

- Google/Outlook calendar integration to auto-generate briefings before every meeting
- Email or Slack delivery shortly before the meeting
- Deeper person research (role history, public talks) and multi-language support
- Result caching to reduce Tavily credit usage

