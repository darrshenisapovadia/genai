# Generative AI - Assignment 1

Darrsheni Sapovadia (27PGAI0063)

Two LangChain pipelines built on top of a Groq-hosted model.

**Part 1** takes the first 30 articles of the BBC News Archive and, for each one, works out
the topic, writes a short summary and pulls out the key people, organisations and places.

**Part 2** takes the first 25 job postings and works out a broad category for the role, then
extracts the skills, education level and experience the advert asks for.

Both notebooks are saved with their outputs, so you can read the results without running
anything.

## Repository contents

```
data/          the two datasets, committed so the notebooks run straight after a clone
notebooks/     Part 1 and Part 2, both saved with outputs
outputs/       the finished dataframes as CSV
.env.example   copy to .env and add your Groq key
```

## Running it

You need Python 3.10 or newer and a Groq API key from https://console.groq.com.

```bash
pip install -r requirements.txt
cp .env.example .env        # then open .env and paste your key in
jupyter notebook
```

Open either notebook from the `notebooks` folder and run all cells. They expect to be run
from that folder, since the paths to the data are relative.

## A note on the model

The assignment brief suggests `llama-3.1-8b-instant`, but Groq has retired the Llama models
and that name now returns a 404. I switched to `openai/gpt-oss-120b`, which is still on the
free tier and handles this kind of work well. If you want a different one, change
`LLM_MODEL` in `.env`.

Everything runs at `temperature=0`. These are extraction tasks, so I want the same answer
every time rather than variety.

One thing worth knowing if you swap the model: `gpt-oss` reasons before it answers, and that
reasoning counts towards `max_tokens`. On the longer articles it used the entire budget
thinking and returned an empty string, which quietly left my entity column blank. Setting
`reasoning_effort="low"` fixes it and makes the run noticeably quicker.

## Rate limits

The free Groq tier allows 8,000 tokens a minute, and Part 1 makes three calls per article.
Both notebooks pause between rows and retry if they get rate limited, reading the wait out
of Groq's own error message, so a full run takes several minutes rather than finishing
instantly. That is expected, not a hang.

There is also a cap of 200,000 tokens per day, and it is counted per model. Part 1 costs
roughly 65,000 tokens for one run, so two or three full re-runs while you are still tweaking
prompts will use the day's allowance up. If you hit that, either wait for it to reset or
point `LLM_MODEL` at a different model, since each one has its own daily budget.
