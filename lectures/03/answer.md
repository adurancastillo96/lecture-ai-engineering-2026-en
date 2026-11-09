# Lecture 03 Answers

## Exercises

- Differences between the Lecture 01 (bare LLM) and agent answers:

  * The standard model (`app.ask`) only knows what it learned during original training. It cannot look up course materials. When asked about AIE2026 homework, it admits it does not know and suggests checking a syllabus.

  * The agent (`app.agent`) uses a search tool called `search_glossary` to look up information inside `data/glossary.json`. It finds the real definitions for the three levels (Basic, Implementation, and Advanced). It explains that finishing any one level completes the homework, and it cites Lecture 01 at the end.

- Tool calls shown by `--trace` (tool names and arguments):

  * First run with `gemini-3.5-flash-lite`: The model made only 1 tool call: `search_glossary({'query': 'LoRA'})`. Even though the prompt asked to compare two concepts, it only looked up the first term.
  * Other runs with `gemini-3.1-flash-lite`: The model made 2 tool calls side by side as expected:
   `search_glossary({'query': 'LoRA'})`
   `search_glossary({'query': 'QLoRA'})`

- Number of LLM calls for one question (`requests`):

  * **In the first run:** The model (`gemini-3.5-flash-lite`) invoked the tool only once: `search_glossary({'query': 'LoRA'})`, inferring the QLoRA part on its own. (input_text_tokens=897, input_tokens=897, output_tokens=211, requests=2, tool_calls=1)
  * **In the second run:** The model (`gemini-3.1-flash-lite`) generated two separate calls in parallel: one for `'LoRA'` and one for `'QLoRA'`. (input_text_tokens=1125, input_tokens=1125, output_tokens=284, requests=2, tool_calls=2)
  * **Behavior - stochastic variability:** for the exact same prompt, generation is non-deterministic, and tool-calling plans can differ between runs and models. The docstring for `search_glossary` instructs entering only a single term per query; depending on the model's probabilistic sampling, it may decide to launch both searches simultaneously or settle for just one. Because of this natural variation, the steps the model takes to call tools can change from run to run.

- Tool added and the description in its docstring:
  * **Tool Name**: `make_quiz(lecture: int) -> str`
      - **Docstring**: """Pick one term from the given lecture and return a review question and its answer.Use when asked for review questions or quizzes, such as 'Give me a review question for Lecture 03'."""

--- 

## Homework

### Basic

- Level attempted: Basic
- What you did:
- Results:
- Observations and questions:




