# Stable Diffusion Ethics Analysis

This repository contains the code for our ethical analysis of the stable diffusion family of models especially: `stabilityai/stable-diffusion-xl-base-1.0`, `stable-diffusion-v1-5/stable-diffusion-v1-5` and `stabilityai/stable-diffusion-2-1`, an open source image generation model published by Stability AI.

The main focus of our analysis is a single design decision: the safety checker in Stable Diffusion can be turned off by any developer with one line of code. We use formal logic and fuzzy logic to show what this means in practice, and we use the Hugging Face API to show what it looks like in the real world.

---

## What Each Notebook Does

### Script 1 — Safety Constraint (`script1_safety_constraint.ipynb`)

This is the starting point. It asks a simple question: can the system generate every image requested and also never generate harmful content? The answer is no, and this notebook proves it formally using Z3.

We set up two goals. The functional goal is that the system should generate an image for every prompt. The ethical goal is that it should never generate harmful content when the safety checker is off. We then run Z3 to check if both can be satisfied at the same time.

When the safety checker is on, Z3 returns SAT. No conflict. When the safety checker is off and a harmful prompt arrives, Z3 returns UNSAT. The two goals cannot coexist. The notebook also confirms this against the real model by loading the pipeline with and without `safety_checker=None`.

---

### Script 2 — Prompt Moderation (`script2_prompt_moderation.ipynb`)

This notebook takes a different approach. Instead of binary yes or no values, it models harm on a spectrum using fuzzy logic. The idea was to test whether making harm continuous rather than binary removes the conflict.

It does not. The notebook builds two separate moderation systems, one representing public safety and one representing developer freedom, and runs both on the same prompts. Even with continuous scores, the moment one system says block and the other says generate, the conflict is detected. The contradiction survives fuzzy encoding.

This notebook directly responds to the argument that binary modelling is too rigid. Spectrum-based modelling changes how harm is measured. It does not change the fact that a decision still has to be made.

---

### Script 3 — Ecosystem Audit (`script3_ecosystem_audit.ipynb`)

This notebook moves from theory to the real world. It uses the Hugging Face API to check two things: whether the official model has any safety documentation, and whether there are applications built on Stable Diffusion that advertise NSFW or uncensored content.

The results are straightforward. The official model has no safety tags. Across 300 spaces audited, 21 were found advertising NSFW or uncensored content. Z3 then formally encodes the licence rule: if an application is built on Stable Diffusion and is NSFW, it violates the licence. Z3 returns UNSAT, meaning every such application is a formal licence violation under the model's own terms.

---

### Script 4 — Developer Conflict (`script4_developer_conflict.ipynb`)

This notebook focuses on the developer specifically. It models the conflict between a developer's goal of unrestricted generation and their ethical obligation not to cause harm.

Importantly, this notebook does not include a safety checker variable at all. The conflict is encoded purely between three things: whether the prompt is harmful, whether consent is given, and whether the image is generated. The point is that the conflict exists independently of the pipeline configuration.

When a harmful prompt arrives with no consent, Z3 returns UNSAT. When consent is present, Z3 returns SAT. This shows the constraint is targeted, not a blanket block. A developer working on legitimate content with consent is not caught by it at all.

---

### Script 5 — Platform Conflict (`script5_platform_conflict.ipynb`)

This notebook looks at the responsibility of platforms like Hugging Face. It encodes the obligation that platforms should not host models with safety features removed without review, and checks this against real examples found in the ecosystem audit.

Z3 returns UNSAT, showing that hosting unscreened derivative models with safety removed conflicts with the platform's obligation to protect the public from harmful content.

---

## How to Run

Install the dependencies:

```
pip install z3-solver scikit-fuzzy numpy requests diffusers torch transformers
```

Run each notebook in order. Scripts 1, 2, 4, and 5 run entirely on their own. Script 3 makes live calls to the Hugging Face API so you will need an internet connection.

Script 1 also loads the actual Stable Diffusion pipeline in Part 4. This requires a Hugging Face account and enough disk space to download the model weights. The Z3 parts in Parts 1 to 3 run without this.

---

## Repository Structure

```
├── script1_safety_constraint.ipynb
├── script2_prompt_moderation.ipynb
├── script3_ecosystem_audit.ipynb
├── script4_developer_conflict.ipynb
├── script5_platform_conflict.ipynb
└── README.md
```

---

## Context

This code is written as part of a reflective portfolio for COMP41820 AI and Ethics, at University College Dublin. The full analysis including stakeholder identification, ethical framework, and debate with an LLM is in the accompanying report.
