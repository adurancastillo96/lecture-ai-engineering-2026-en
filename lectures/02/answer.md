# Lecture 02 Answers

## Exercises

- Model and settings used (from `config` in `hw_basic/result.json`): check out the file lectures/02/baseline_exercise/result.json
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

### Basic 

- Level attempted: Basic
- Settings or data changed: LoRA Rank (`r`), changed from `lora_r: 16` (`baseline`) to `lora_r: 4` (`hw_basic`), keeping `lora_alpha: 32` constant.
- Results (comparison tables or numbers):

    | Setting / Metric | Baseline Exercise (`r=16`) | Homework Basic (`r=4`) | Change / Note |
    |---|:---:|:---:|---|
    | **LoRA Rank ($r$)** | 16 | 4 | Reduced rank (75% fewer adapter parameters) |
    | **LoRA Alpha ($\alpha$)** | 32 | 32 | Kept constant |
    | **Effective Scaling Factor ($\alpha / r$)** | 2.0 | 8.0 | Adapter weights scaled $4\times$ more strongly |
    | **Before: Format Rate** | 95.2% (20/21) | 95.2% (20/21) | Identical starting checkpoint |
    | **Before: Content Rate** | 38.1% (8/21) | 38.1% (8/21) | Identical starting checkpoint |
    | **After: Format Rate** | **100.0% (21/21)** | **100.0% (21/21)** | Both achieved perfect JSON format adherence |
    | **After: Content Rate (Overall)** | **85.7% (18/21)** | **85.7% (18/21)** | Overall accuracy appears identical at first glance |
    | **After: Both Rate** | 85.7% (18/21) | 85.7% (18/21) | Matches content rate (format is 100%) |
    | **Grounded Questions ($n=15$)** | 86.7% (13/15) | **93.3% (14/15)** | **+6.6%**: $r=4$ recalled FAQ facts better |
    | **Ungrounded Questions ($n=6$)** | **83.3% (5/6)** | 66.7% (4/6) | **-16.6%**: $r=4$ had more false assertions |
    | **Regressions** | 0 | 0 | No sample that passed before failed after |
    | **Inference Time (Eval)** | 37.7 s | 39.5 s | Comparable evaluation latency |

    **Key Divergent Cases between $r=16$ and $r=4$:**
    - **Row 12: *"Tell me how submissions after the deadline are handled"*** (Grounded in FAQ, gold `needs_review: false`):
        - `r=16` failed by evading: replied `"This FAQ does not contain that information..."` (`needs_review: true`).
        - `r=4` succeeded: correctly recalled the FAQ rule: `"Late submissions are accepted within 48 hours of the deadline, with a grade penalty."` (`needs_review: false`).
    - **Row 18: *"Will I receive feedback on my submitted assignment?"*** (Ungrounded, gold `needs_review: true`):
        - `r=16` succeeded: correctly requested confirmation (`needs_review: true`).
        - `r=4` failed by hallucinating / over-asserting: `"Feedback is provided during office hours every Tuesday from 17:00 to 18:00."` (`needs_review: false`). It mistakenly associated "feedback" with the office hours slot.
    - **Shared Failure Modes:**
        - Row 6 (*"Can I submit a format other than PDF?"*): Both models evaded (`needs_review: true`) due to negative/exclusion query phrasing.
        - Row 20 (*"Do office hours require an appointment?"*): Both models asserted the general office hours statement (`needs_review: false`) due to strong keyword matching on "office hours".

    **Analysis of Hyperparameter Effect:**
    - **Lowering the rank gave the adapter more influence:** In LoRA, the adapter multiplier is alpha divided by rank. Keeping alpha at 32 while dropping rank from 16 to 4 increased this multiplier from 2 to 8. This gave the adapter four times more push over the base model.
    - **The model became more confident recalling FAQ facts:** With that stronger push, the model hesitated less when answering known course rules. On Row 12, which asked about late submissions, the baseline model acted too cautious and claimed the FAQ lacked the answer. The new model confidently stated the 48 hour penalty rule and passed.
    - **That extra confidence made the model guess on missing information:** When an answer was absent from the FAQ, that same push backfired. On Row 18, which asked about assignment feedback, the model saw familiar words and tried too hard to answer. Instead of asking for confirmation, it confidently made up a fake rule stating that feedback takes place during office hours.

- Discussion of when fine-tuning is and is not suitable:
    - **When Fine-Tuning is Suitable:**
        - **Locking in a strict output format:** Both $r=16$ and $r=4$ moved from 95.2% to 100% format compliance, eliminating missing JSON keys and unwanted explanatory preambles.
        - **Standardizing phrasing and communication style:** Fine-tuning reliably trained the model to follow a fixed script. When an answer was not in the course FAQ, the model stopped guessing or using random phrasing. Instead, it learned to recite the exact required refusal message: (`"This FAQ does not contain that information. Confirmation from the course organizers is needed."`).
        - **Learning efficiently with smaller settings:** For structured formatting tasks, a compact rank ($r=4$) is completely sufficient, achieving equivalent format rate and overall accuracy with ~75% fewer adapter parameters.
    - **When Fine-Tuning is NOT Suitable:**
        - **Teaching a model to recognize what it does not know:** Fine-tuning does not teach a model the boundaries of its knowledge. When an unanswered question shares familiar words with the training data, such as questions about deadlines, feedback, or office hours, the model gets confused. Instead of admitting it lacks the answer, it invents a response with full confidence and marks that no review is needed.
        - **Storing facts that change often:** Training model weights is a poor way to store everyday information. Retraining a model whenever details change takes too much time, creates hallucinations (made up facts), and risks catastrophic forgetting (losing previously learned skills). For accurate, up-to-date facts, Retrieval-Augmented Generation (searching documents before answering) or pasting information directly into the prompt works much better.
          Fine-tuning should only be used to teach format, tone, and behavior.

### Implementation

- Level attempted: Implementation
- Settings or data changed:
- Results (comparison tables or numbers):
- Discussion of when fine-tuning is and is not suitable:

### Advanced

- Level attempted: Advanced
- Settings or data changed:
- Results (comparison tables or numbers):
- Discussion of when fine-tuning is and is not suitable: