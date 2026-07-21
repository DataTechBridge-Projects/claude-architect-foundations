---
title: Test Dataset Generation and Model-Based Grading
parent: The Prompt Layer
nav_order: 20
duration: "12:00"
---

# Test Dataset Generation and Model-Based Grading
{: .no_toc }

*Video duration: 12:00*

The previous lecture established that a prompt evaluation workflow needs two things before it can run at all: a representative set of inputs, and a way to score outputs against success criteria. This lecture goes deeper on both halves of that requirement — how to actually build a good test dataset, and how to score outputs at a scale where manual human review of every single case is no longer practical.

## Building a Test Dataset Deliberately

A test dataset built casually — grabbed from whatever inputs happened to be lying around, or invented from the author's mental model of what a "typical" input looks like — tends to systematically under-represent exactly the cases that matter most for finding real weaknesses in a prompt. A dataset built deliberately covers three distinct categories of input, each doing different diagnostic work.

**Representative cases** are inputs that reflect the actual, realistic distribution of what the system will see in production — not a curated set of the cleanest, easiest examples, but a genuine cross-section including the ordinary messiness of real input: varying length, varying phrasing, varying levels of completeness. This category tells you whether the prompt handles the *common* case reliably, which matters because even a small failure rate on the common case compounds into a large absolute number of failures at production volume.

**Edge cases** are inputs that are rare but real — the empty input, the input that technically satisfies the task's category but only barely, the input containing conflicting or unusual signals. These matter disproportionately because edge cases are exactly where prompts that look solid on typical inputs tend to break down first, and because a failure on an edge case is often more damaging than a failure on a typical case, precisely because edge cases are more likely to be genuinely unusual situations where a wrong answer has outsized consequences.

**Adversarial cases** are inputs deliberately constructed to probe for a specific known weakness or attempt to defeat the prompt's intended behavior — text that looks like an embedded instruction, an input crafted to trigger a known incidental correlation from a few-shot example set, or a case specifically designed to test whether a stated constraint actually holds under pressure rather than just under polite conditions. Unlike representative and edge cases, which are drawn from or modeled on real traffic, adversarial cases are deliberately authored to attack the prompt, which makes them the closest analog in this discipline to a security test rather than a functionality test — and they matter most for prompts that process untrusted input, echoing the sanitization concerns raised elsewhere in this module.

A mature test dataset isn't static, either. As production traffic surfaces a new failure mode — a category of input nobody anticipated when the dataset was first built — that specific case should be added to the dataset going forward, the same way a specific failure pattern earns a targeted few-shot example. This keeps the dataset an accurate, growing reflection of what the system actually has to handle, rather than a fixed snapshot that quietly goes stale as production traffic evolves past it.

## Model-Based Grading: The Basic Idea

Scoring outputs by hand against a rubric is reliable but doesn't scale — a test dataset large enough to meaningfully cover representative, edge, and adversarial cases can easily run into the hundreds or thousands of examples, and re-grading that many outputs by hand every time a prompt changes is slow enough that it discourages the very iteration the evaluation workflow depends on. **Model-based grading** — sometimes called **LLM-as-judge** — addresses this by using a separate model call to score each output against the rubric automatically, at a speed and cost that makes frequent re-evaluation practical.

The mechanics mirror independent review, and deliberately so: a grading prompt is given the input, the output being evaluated, and an explicit rubric to check the output against, and it returns a structured verdict — a score, a pass/fail per criterion, or a category label — rather than free-form commentary a human would have to interpret before the result could be aggregated into a metric. This lets an evaluation run that would take a human grader days to complete run in the time it takes to make a batch of model calls, which is what makes running the full evaluation suite after every meaningful prompt revision, rather than only occasionally, actually feasible.

## Tradeoffs and Failure Modes

Model-based grading is not a free substitute for human judgment, and treating it as one introduces its own class of risk. **Grader bias** is the most direct failure mode: a grading model can share systematic blind spots with the model being graded, particularly if both are the same underlying model family, in much the same way inline self-critique shares blind spots with the generation it's checking. A grader that consistently misjudges a certain kind of borderline case in the same direction the generator does will produce evaluation scores that look reassuring while missing exactly the failure the evaluation was supposed to catch — which is why some teams deliberately use a different model, or a differently framed grading prompt, as the judge, applying the same independence principle from the multi-pass-review lecture to the grading step itself.

**Rubric ambiguity** is the second major failure mode, and it's really the grading-prompt version of the ambiguity problem the prompting-best-practices lecture warned about for generation prompts. A rubric criterion like "the response should be helpful" gives a grading model just as much room for inconsistent interpretation as it would give a generation model — and an inconsistent grader produces noisy scores that make it hard to tell whether a prompt revision actually helped or the grader simply scored this run differently than the last one by chance. The fix is the same discipline applied one layer up: rubric criteria for a model-based grader need to be as explicit and checkable as success criteria for the generation prompt itself — "does the response answer the specific question asked, without introducing information not present in the source document" rather than "is this a good response."

A third, subtler risk is **overfitting to the grader** — if a prompt is iterated purely against a model-based grader's score, with no periodic human spot-check, it's possible to converge on a version of the prompt that scores well specifically because it exploits some quirk of how the grading model evaluates responses, rather than because it's genuinely better for the actual downstream use case. Guarding against this means treating model-based grading as a scaling mechanism for evaluation, not a total replacement for human judgment — a sound practice is running the full evaluation suite through model-based grading for speed and frequency, while periodically sampling a subset of graded outputs for human review, specifically to confirm the grader's scores still track what a human would actually conclude.

## How This Fits Alongside the Broader Evaluation Workflow

Test dataset generation and model-based grading are the machinery that make the evaluation workflow from the previous lecture practical at scale — the dataset supplies the representative, edge, and adversarial inputs to run a prompt against, and the grader turns each resulting output into a scored, comparable data point without requiring a human to read every single one. Neither piece works well in isolation: an excellent dataset graded inconsistently produces noisy, untrustworthy comparisons between prompt versions, and a reliable grader run against a thin or unrepresentative dataset will confidently certify a prompt that still fails badly on the inputs the dataset never included. Treated together, with periodic human spot-checks keeping the grader itself honest, they turn prompt evaluation from an occasional manual exercise into a routine, low-friction part of every prompt revision.

**Why it matters for the exam:** Expect questions asking you to distinguish representative, edge, and adversarial test cases by function, and to recognize when a described test dataset is missing one of those categories. Also expect scenario questions testing model-based grading's specific failure modes — a grader sharing the same blind spot as the model it's grading, or a vague rubric producing inconsistent scores — and questions on why periodic human review of a sample of graded outputs remains necessary even after model-based grading is in place.
