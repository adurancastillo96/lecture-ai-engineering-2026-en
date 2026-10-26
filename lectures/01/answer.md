# Lecture 01 Answers

## Exercises

- Model's answer to the course-specific question (the 3 homework levels; a summary is fine):
  When asked without tools, the bare model either politely refused (stating that AIE2026 is a specific course code outside its training data) or hallucinated plausible academic levels ("Beginner / Foundational", "Intermediate / Applied", "Advanced / Expert").

- What you noticed when comparing it with the correct meaning in the glossary:
  In `data/glossary.json`, the three actual levels are:
  1. "Basic" (review lecture content)
  2. "Implementation" (write your own code)
  3. "Advanced" (research and experimentation)
  Completing any one level completes the homework, and assignments are not graded. The bare LLM has no access to course-specific documents, which causes it to either hedge or make plausible guesses based on general academic patterns.

- Token counts you displayed (input / output):
  - Question: "What are the 3 homework levels in the AIE2026 course?" -> input: 18, output: 230
  - Question: "What is LoRA?" -> input: 6, output: 759

## Homework

- Level attempted: Basic / Implementation / Advanced
- What you did: Implemented token count display using `response.usage_metadata` in `app/ask.py`.
- Results: The script successfully outputs `[tokens] input: <N>, output: <M>` before generating the answer.
- Observations and questions: None
