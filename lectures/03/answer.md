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

### Basic Level

- Level attempted: Basic

- What you did:
  * Modified the tool docstring for `search_glossary` in `app/tools.py`.
  * Changed the instruction from `'Put only one term to look up in query...'` to `'Put several terms together separated by commas in query (for example, "LoRA, QLoRA").'`.
  * Ran the agent with `--trace` on the same prompt: `uv run python -m app.agent --trace "What is the difference between LoRA and QLoRA?"` using `gemini-3.1-flash-lite`.

- Results:
  * **Tool calls & arguments**: The model respected the modified docstring and batched both terms into a single query: `search_glossary({'query': 'LoRA, QLoRA'})`.
  * **Number of tool calls**: Dropped from 2 calls down to 1 (`tool_calls=1`).
  * **LLM requests & tokens**: Remained at `requests=2` (1 request to invoke the tool and 1 request to produce the final response). Input text tokens decreased from 1,125 to 1,041, resulting in lower cost (`cost=Decimal('0.00064875')`).
  * **Answer quality**: In a single search call, the tool found and returned definitions for both terms. The agent then used those definitions to write a clear, accurate comparison and included the correct citation for `(Lecture 02)`. Note!: I modified `print_trace(result)` function to visualize how many terms were fletched from `glossary.json` in that single call and fed into the model.

- Observations and questions:
  * The tool docstring serves as the primary interface description and behavioral guideline for the LLM. 
  * By modifying only one sentence in the docstring, the agent shifted from invoking separate parallel tool calls to a single batch call. This demonstrates that tool descriptions can be intentionally engineered to optimize token usage, reduce tool invocations, and lower execution costs.

### Implementation Level

- Level attempted: Implementation

- What you did:
  * Created `save_learning_log(topic: str, content: str) -> str` in `app/tools.py` to write study records to files in `learning_log/`.
  * Configured `Agent` in `app/agent.py` with `output_type=[str, DeferredToolRequests]` and registered `save_learning_log` with `requires_approval=True`.
  * Updated CLI execution in `main()` of `app/agent.py` to detect `DeferredToolRequests`, display the requested tool name and arguments, prompt the user for confirmation (`[y/N]`), and resume the agent loop with `DeferredToolResults` via `run_sync`.

- Results:
  * **When approved (`y`)**: The agent paused before executing `save_learning_log`, prompted for confirmation, and upon entering `y`, executed the tool and wrote the learning summary to `learning_log/LOG_NAME.txt`.
  * **When denied (`n`)**: The agent paused for confirmation, and upon entering `n`, passed `False` approval back to the agent loop. Pydantic AI returned `The tool call was denied.`, and no file was written to disk.
  * **Automated testing**: `uv run pytest -q` passed 15/15 tests cleanly.

- Observations and questions:
  * Human-in-the-loop (HITL) tool approval provides an essential security and safety guardrail for irreversible operations (like file writes or database modifications).
  * Pydantic AI's two-step `DeferredToolRequests` / `DeferredToolResults` mechanism allows pausing and resuming multi-turn tool calling without losing conversation state (`message_history`).

- Log:
```
❯ uv run python -m app.agent --trace "What is the difference between LoRA and      
    QLoRA?. Give me one review question and save the log."
[model] gemini-3.1-flash-lite
                 pydantic-ai v2.57.0 • Python 3.14.3

       / \        agent: target model: google:gemini-3.1-flash-liteoutput: str | DeferredToolRequests
     /   \         tools: 4 • capabilities: 0
   /___.___\
  /    |    \    observability: off — see every model and tool call live, with cost
/      |      \    set it up free with Logfire and a GitHub login: https://pydantic.dev/ai-setup.md
`---.._|_..---'    or use any OpenTelemetry backend: https://pydantic.dev/docs/ai/logfire/#otel

                 goes away once observability is on — or PYDANTIC_AI_NO_BANNER=1

[Tool Approval Required] Tool: save_learning_log
Arguments: {
  "content": "The user learned about the differences between LoRA (Low-Rank Adaptation) and QLoRA (Quantized Low-Rank Adaptation). They also received a review question regarding Lecture 02.",
  "topic": "LoRA vs QLoRA and review quiz"
}
Do you approve executing this tool? [y/N]: y
To understand the difference, consider how both methods approach memory efficiency during fine-tuning:

-   **LoRA (Low-Rank Adaptation):** This method focuses on efficiency by freezing the pre-trained model weights and injecting small, trainable "adapter" matrices into the architecture. Instead of updating all billions of parameters, you only train these small matrices, significantly reducing the number of trainable parameters.

-   **QLoRA (Quantized LoRA):** This builds on top of LoRA by adding **quantization**. It compresses the base model's weights to a lower precision (such as 4-bit) before applying LoRA. This dramatically reduces the memory footprint required to load the model, allowing you to fine-tune much larger models on consumer-grade hardware.


**Think about it this way:** LoRA reduces the number of parameters you need to update, while QLoRA additionally reduces the amount of memory needed to store the base model itself.

***

### Review Question for Lecture 02
**Question:** What does "Learning rate" mean in the context of Lecture 02?
**Answer:** A hyperparameter that determines how far parameters move in one update. Too large a value makes training unstable; too small a value slows progress.

***
I have saved this progress to your learning log.

(Lecture 02)

--- trace ---
[tool call]   search_glossary({'query': 'LoRA, QLoRA'})
[tool return] [found terms: 'LoRA' (Lecture 2), 'QLoRA' (Lecture 2)] ; search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "A PEFT method that freezes pretrained weights, represents weight updates …
[tool call]   make_quiz({'lecture': 2})
[tool return] ; make_quiz -> {"question": "What does 'Learning rate' mean in the context of Lecture 02?", "answer": "A hyperparameter that determines…
[tool call]   save_learning_log({'content': 'The user learned about the differences between LoRA (Low-Rank Adaptation) and QLoRA (Quantized Low-Rank Adaptation). They also received a review question regarding Lecture 02.', 'topic': 'LoRA vs QLoRA and review quiz'})
[tool return] ; save_learning_log -> Successfully saved learning record for 'LoRA vs QLoRA and review quiz' to learning_log/LoRAvsQLoRAandreviewquiz.txt.…
[usage] RunUsage(cost=Decimal('0.0006580'), details={'text_prompt_tokens': 958}, input_text_tokens=958, input_tokens=958, output_tokens=279, requests=4, tool_calls=3)
```



