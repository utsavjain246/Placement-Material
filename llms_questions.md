### Training: Pre-training vs. Fine-tuning

**Question:** Explain the difference between **Pre-training** and **Fine-tuning** in terms of the _objective_ and the _data_ used.

**Answer:**

- **Pre-training (The Generalist)**  
  - **Goal:** Learn broad statistical structure of language and general world knowledge so the model can produce coherent text in many contexts.  
  - **Data:** Very large, mostly unlabeled corpora (web pages, books, Wikipedia, etc.).  
  - **Objective:** Self-supervised objectives such as Next Token Prediction (autoregressive) or Masked Token Prediction (masked language models). The model learns to predict tokens from surrounding context; the data itself provides the supervision.

- **Fine-tuning (The Specialist)**  
  - **Goal:** Adapt the pretrained model to a specific task, behavior, or domain (e.g., instruction following, classification, code generation, or a specialized domain like medicine).  
  - **Data:** Smaller, curated, labeled datasets or instruction–response pairs. Data quality and alignment with the target task are more important than sheer size.  
  - **Objective:** Often still framed as token-level prediction (e.g., maximize likelihood of desired responses), but computed on task-specific examples so the model learns the required behavior, format, or constraints.

Follow-up — "Does fine-tuning add new knowledge to the model?"  
- Short answer: Not usually in the same way as human learning of new facts. Fine-tuning primarily changes the model's behavior, style, and how it weighs existing internal representations. If the fine-tuning data contains factual information the base model never saw, the model can incorporate those facts into its outputs, but this is often limited by the model's capacity and may be less robust than learning via large-scale pretraining. Fine-tuning can also cause the model to overfit to the new data or forget (partially) some previously learned behaviors if not done carefully (techniques such as regularization or adapters help mitigate that).

---

### Inference: Temperature & Sampling

**Question:** When generating text, we often adjust a parameter called **Temperature**. How does Temperature mathematically affect the output, and why does a higher temperature make the model more "creative" or random?

**Answer:**

- LLMs produce a vector of raw scores (logits) for each token; a Softmax converts logits into probabilities:
  - `p_i = exp(z_i) / sum_j exp(z_j)` where `z_i` is the logit for token i.
- Temperature `T` rescales logits before Softmax:
  - `p_i = exp(z_i / T) / sum_j exp(z_j / T)`.
  - If `T < 1`, logits are scaled up, making the distribution peakier (more concentrated on top tokens) → more deterministic outputs.
  - If `T > 1`, logits are scaled down, flattening the distribution → lower-probability tokens get relatively more probability mass → more diverse/random outputs.
  - `T = 1` is the default (no scaling).

- Intuition: Temperature affects certainty. Low T amplifies confident choices; high T increases uncertainty so rarer tokens are sampled more often, which can produce novel or surprising continuations.

Follow-up — "What is the difference between Temperature and 'Top-K' (and 'Top-p') sampling?"  
- **Temperature** changes the shape of the whole probability distribution (soft adjustment).  
- **Top-K** sampling truncates the distribution to only the K most probable tokens, renormalizes their probabilities, and samples from that set (hard cutoff). This prevents very unlikely tokens from ever being chosen.  
- **Top-p (nucleus)** sampling chooses the smallest set of top tokens whose cumulative probability ≥ p, then samples from that set. This adapts the cutoff dynamically based on the distribution's entropy.  
- Common practice: combine temperature with Top-K or Top-p for better control (temperature for global randomness; Top-K/Top-p to remove extremely unlikely tokens).

---

### Challenges: Hallucinations

**Question:** LLMs are known to "hallucinate" (make things up). Based on how they are trained, why is hallucination a feature, not a bug?

**Answer:**

- Objective mismatch: Models are trained to maximize the likelihood of the next token (produce plausible continuations) rather than to maximize factual accuracy. The training objective rewards fluency and plausibility, not truth.
- Pattern completion: Given a prompt, the model completes patterns seen during training. If asked for a fact it doesn't know, it will generate a plausible-sounding answer based on correlated patterns (names, dates, citation formats), not a validated fact.
- No built-in verification: Standard LLM inference is a single-pass generation without an external truth-check; the model has no direct access to a canonical source to confirm facts.
- Data noise and bias: Training data contains errors, fictional text, and biases; models can reproduce those.

Follow-up — "How can we reduce hallucinations without retraining the model?"

- Retrieval-Augmented Generation (RAG): Retrieve relevant documents from an external corpus and provide them in the prompt context. Grounding outputs in retrieved facts greatly reduces hallucinations.  
- Tooling / external APIs: Use search engines, knowledge bases, or fact-checking APIs at runtime to verify or supply facts.  
- Prompting strategies: Ask the model to cite sources, be explicit about uncertainty, or return "I don't know" when unsure. Use chain-of-thought or step-by-step reasoning carefully together with verification.  
- Post-processing: Validate model outputs with rule-based checks, SQL/schema validation, or external verification steps; redact or flag unsupported claims.  
- Constrained decoding: Force outputs into templates or limited vocabularies where possible (e.g., canonical identifiers).  
- Ensembles and calibration: Compare multiple independent generations or use calibration techniques to detect low-confidence outputs.  
- Human-in-the-loop: Have humans verify critical outputs before use, especially in high-stakes domains.

Notes: These mitigations reduce but do not eliminate hallucinations. For mission-critical or safety-sensitive applications, always combine grounding, automatic checks, and human oversight.

---
