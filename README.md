# CrewAI
crew ai agentic framework

# CrewAI — YouTube-to-Blog Content Generator

A multi-agent content pipeline built with [CrewAI](https://github.com/crewAIInc/crewAI). Given a topic, a **Researcher** agent searches a YouTube channel for relevant video content, and a **Writer** agent turns the findings into a blog post — written to a markdown file.

## How it works

```
crew.kickoff(topic) → Researcher Agent → Writer Agent → new-blog-post.md
```

- **`blog_researcher`** (Agent) — searches the configured YouTube channel for video content relevant to the given topic and produces a research summary.
- **`blog_writer`** (Agent) — takes the researcher's findings and writes an accessible blog post summarizing them.
- Both agents share the same tool (`yt_tool`) and run as a **sequential** CrewAI process — the writer's task depends on the researcher's output.

## Files

| File | Purpose |
|---|---|
| `agents.py` | Defines the `blog_researcher` and `blog_writer` agents — their roles, goals, and backstories |
| `tasks.py` | Defines `research_task` and `write_task`, and wires each to its agent |
| `tools.py` | Sets up `YoutubeChannelSearchTool` from `crewai_tools`, scoped to a specific YouTube channel |
| `crew.py` | Assembles the agents and tasks into a `Crew`, and runs it (`crew.kickoff`) |
| `requirements.txt` | Python dependencies |

## Setup

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Set your target YouTube channel

In `tools.py`, replace the placeholder handle with the channel you want to search:

```python
yt_tool = YoutubeChannelSearchTool(youtube_channel_handle='@your-channel-handle')
```

### 3. Add your API key

Create a `.env` file (**do not commit this — see Security note below**):

```
OPENAI_API_KEY=your-openai-api-key
```

This project uses `gpt-4-0125-preview` via the OpenAI API (set in `agents.py`).

## Running

Set the topic you want researched and written about in `crew.py`:

```python
result = crew.kickoff(inputs={'topic': 'AI vs ML vs DL vs Data Science'})
```

Then run:

```bash
python crew.py
```

The blog post is written to `new-blog-post.md` in the project root, and the full crew output is also printed to the console.

## ⚠️ Security note

This repository currently has a **`.env` file committed to git**, which is how secrets like `OPENAI_API_KEY` end up publicly exposed on GitHub. If that key is real:
1. **Revoke it immediately** in your OpenAI dashboard and generate a new one.
2. Remove `.env` from the repo (`git rm --cached .env`) and add `.env` to a `.gitignore` file so it's never tracked again.
3. Keep the key only in your local, untracked `.env`.

## Notes

- `crew.py` currently hardcodes the research topic — you'd want to parameterize this (CLI arg, input prompt, etc.) for repeated use.
- `memory=True` and `cache=True` are enabled on the `Crew`, so repeated runs on the same topic can reuse cached results.
- `max_rpm=100` caps the crew's requests-per-minute to stay within API rate limits.
