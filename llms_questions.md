### Training: Pre-training vs. Fine-tuning

**Question:** Explain the difference between **Pre-training** and **Fine-tuning** in terms of the _objective_ and the _data_ used.

**Answer:**

*   **Pre-training (The Generalist):**
    
    *   **Goal:** Learn the statistical structure of language and general world knowledge.
        
    *   **Data:** Massive, unlabeled datasets (The Internet, books, Wikipedia).
        
    *   **Objective:** "Next Token Prediction" (predicting the next word given the previous words). It is self-supervised (the data acts as its own label).
        
*   **Fine-tuning (The Specialist):**
    
    *   **Goal:** Adapt the model to a specific task (e.g., following instructions, coding, medical diagnosis) or style.
        
    *   **Data:** Smaller, high-quality, labeled datasets (Instruction-Response pairs).
        
    *   **Objective:** Still often Next Token Prediction, but calculated only on the _response_ part of the data to guide the model toward specific _behaviors_ rather than just general knowledge.
        

**Follow-up:**

> "Does Fine-tuning add new knowledge to the model?"
> 
> **Answer:** Generally, no. Fine-tuning is better at teaching the model a _format_ or a _style_ rather than new facts. If you fine-tune a model on a new physics theory it hasn't seen, it might hallucinate. New knowledge is best added via RAG (Retrieval Augmented Generation) or Pre-training.

----------------------------


### 5\. Inference: Temperature & Sampling

**Question:** When generating text, we often adjust a parameter called **Temperature**. **How does Temperature mathematically affect the output, and why does a higher temperature make the model more 'creative'?**

**Answer:** LLMs output a probability distribution over the entire vocabulary (e.g., "apple": 10%, "banana": 5%, "car": 0.01%).

-   **The Mechanism:** Temperature is a divisor applied to the "logits" (raw scores) before they enter the Softmax function.

-   **Low Temperature (<1.0):** Exaggerates the differences. High probabilities get even higher; low ones get lower. The model becomes conservative and deterministic, almost always picking the #1 most likely word.

-   **High Temperature (>1.0):** Flattens the curve. The difference between the #1 word and the #10 word shrinks. This gives the less likely (but potentially more interesting) words a fighting chance to be picked, resulting in "creativity" or randomness.

**Follow-up:**

> "What is the difference between Temperature and 'Top-K' sampling?"
>
> **Answer:** Temperature changes the *shape* of the probability curve. Top-K simply *cuts off* the tail. Top-K says, "Only consider the top K most likely words, and zero out the rest." Top-K prevents the model from choosing complete nonsense (very low probability tokens), whereas high temperature simply makes everything equally likely.

----------------------------

### 6\. Challenges: Hallucinations

**Question:** LLMs are known to "hallucinate" (make things up). **Based on how they are trained, why is hallucination a feature, not a bug?**

**Answer:** LLMs are probabilistic engines, not truth databases.

-   They are trained to generate the most **plausible** next token, not the **truthful** one.

-   If you ask for a citation for a fake physics paper, the model looks at the pattern of academic citations. It knows citations usually contain names, a year, and a title. It generates a sequence that *looks* exactly like a citation (plausible syntax) but refers to a paper that doesn't exist (factually void).

-   The model prioritizes **fluency and coherence** over factual accuracy because that is what the "Next Token Prediction" objective optimizes for.

**Follow-up:**

> "How can we reduce hallucinations without retraining the model?"
>
> **Answer:** The most common method is **RAG (Retrieval-Augmented Generation)**. Instead of asking the model to rely on its internal memory, you provide the relevant facts in the prompt context and ask the model to "answer only using the provided context."
