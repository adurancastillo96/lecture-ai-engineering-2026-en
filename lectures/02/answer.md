# Lecture 02 Answers

## Exercises

- Model and settings used (from `config` in `result.json`): check out the folder notebooks/submission_exercise/result.json
    - Base model: `Qwen/Qwen3.5-0.8B`
    - Revision: `2fc06364715b967f1860aea9cf38778875588b17`
    - Seed: `42`
    - Epochs: `20` (`num_train_epochs`)
    - Warmup steps: `6`
    - Learning rate: `0.0002`
    - LoRA rank (r): `16`
    - LoRA alpha: `32`
    - Max sequence length: `512`
    - Max new tokens: `96`
    - Generation: `do_sample: false`
- Did you run training yourself (`executed_training_myself`): Yes, on google colabb with T4 GPU.
- Before training: `format_rate` = 0.9524 (95.2 %) / `content_rate` = 0.3810 (38.1 %)
- After training: `format_rate` = 1.0 (100.0 %) / `content_rate` = 0.8571 (85.7 %)
- Questions that regressed: none.
- Failures remaining after training and their types (assertion without evidence / evasion through a confirmation request despite evidence being present / broken format):
    - Evasion through a confirmation request despite evidence being present (2 cases):
        - Row 6: *"Can I submit a format other than PDF?"* — The FAQ explicitly states that assignments must be one PDF, but the model evaded with `"This FAQ does not contain that information. Confirmation from the course organizers is needed."` and `"needs_review": true`.
        - Row 12: *"Tell me how submissions after the deadline are handled"* — The FAQ specifies that late submissions are accepted within 48 hours with a penalty, but the model replied with the ungrounded confirmation template and `"needs_review": true`.
    - Assertion without evidence (1 case):
        - Row 20: *"Do office hours require an appointment?"* — The FAQ mentions office hours times but does not state whether an appointment is required. The model matched the keyword "office hours" and confidently asserted `"Questions are welcome during office hours every Tuesday from 17:00 to 18:00."` with `"needs_review": false` instead of asking for confirmation.
    - Broken format: 0 cases. All 21 outputs complied with the format.
- Results of knowledge questions (section 6; classification before → after training) and observations:
    - *"When is the final assignment deadline for this course?"* (In FAQ: False): `asserted` → `asserted`. Output: `{"answer": "The assignment deadline is September 30 at 23:59 (Japan time).", "needs_review": false}`.
    - *"What is the instructor's name?"* (In FAQ: False): `needs_review` → `needs_review`. Output: `{"answer": "This FAQ does not contain that information. Confirmation from the course organizers is needed.", "needs_review": true}`.
    - *"What is the topic of the next lecture?"* (In FAQ: False): `needs_review` → `needs_review`. Output: `{"answer": "This FAQ does not contain that information. Confirmation from the course organizers is needed.", "needs_review": true}`.
    - *"When is the assignment deadline?"* (In FAQ: True, control): `asserted` → `asserted`. Output: `{"answer": "The assignment deadline is September 30 at 23:59 (Japan time).", "needs_review": false}`.
    - Observations: Training taught the model the correct phrase to use when an answer is not in the text. This worked well for unrelated questions like the instructor name. But when an unknown question shared words with the FAQ, like "final assignment deadline", the model got tricked by the matching words and gave the date with "needs_review": false anyway. Training fixed the wording, but not the judgment.
- What changed and what stayed the same with training:
    - What changed:
        - Format adherence (`format_rate`) reached 100 %, completely fixing missing keys (`missing_key`) and paragraph spills.
        - Content accuracy (`content_rate`) increased substantially from 38.1 % to 85.7 %.
        - The output phrasing adopted consistent, standardized phrasing across all responses.
    - What stayed the same:
        - If an unknown question shares words with the training data, the model easily gets confused. Fine-tuning teaches the model how to format its text and copy a specific style, but it does not teach the model to check whether facts are true. When asked about missing information that sounds familiar, the model still makes up believable answers instead of admitting it does not know.

## Homework

- Level attempted: Basic / Implementation / Advanced
- Settings or data changed:
- Results (comparison tables or numbers):
- Discussion of when fine-tuning is and is not suitable:
