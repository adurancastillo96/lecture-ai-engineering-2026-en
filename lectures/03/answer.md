# Lecture 03 Answers

## Exercises

- Differences between the Lecture 01 (bare LLM) and agent answers:

  1. The standard model (`app.ask`) only knows what it learned during original training. It cannot look up course materials. When asked about AIE2026 homework, it admits it does not know and suggests checking a syllabus.

  2. The agent (`app.agent`) uses a search tool called `search_glossary` to look up information inside `data/glossary.json`. It finds the real definitions for the three levels (Basic, Implementation, and Advanced). It explains that finishing any one level completes the homework, and it cites Lecture 01 at the end.

- Tool calls shown by `--trace` (tool names and arguments):
- Number of LLM calls for one question (`requests`):
- Tool added and the description in its docstring:

## Homework

- Level attempted: Basic / Implementation / Advanced
- What you did:
- Results:
- Observations and questions:
