# Aya Expanse: Combining Research Breakthroughs for a New Multilingual Frontier

## Introduction
* Goal: Develop multilingual models (8B and 32B parameters) that perform comparably to or better than monolingual models.
* Key innoations"
    * Data Arbitrage: Strategic sampling for synthetic multilingual data creation. ([paper](https://arxiv.org/pdf/2408.14960))
    * Iterative Preference Optimization: Tailoring models to human preferences in multiple languages. ([paper](https://www.semanticscholar.org/paper/RLHF-Can-Speak-Many-Languages%3A-Unlocking-Preference-Dang-Ahmadian/3da5f21144fef19dd88f7dcc11a5d9f2edbfe417))
    * Model Merging: Combining diverse models for better generalization. ([paper](https://arxiv.org/pdf/2410.10801))
* Performance Highlights
    - Aya Expanse models consistently outperform similar-sized models.
    - Larger parameter models like 32B demonstrate competitive performance against much larger models (e.g., Llama 3.1 70B).
    - Innovations in post-training processes (data arbitrage, preference optimization, model merging) are key drivers of success.

## Training Recipe

### Synthetic Data Generation via Data Arbitrage
* Challenge: Limited high-quality teacher models for low-resource languages.
* Solution: Use a pool of models and a reward-based system (Arbiter) to select the best completions.
* Result: Significant improvement in multilingual synthetic data quality, boosting model performance. Aya Expanse 8B had 9.1% improvement in win-rate compared to Gemma 9B (m-ArenaHard).

### Iterative Multilingual Preference Training
* Aligns models with user preferences across languages.
* Combines offline and online training using DPO:
    - Offline Preference Training:
        - The model is first trained on a curated dataset derived from the Multilingual Data Arbitrage phase:
            - Completions from diverse models in the arbitrage pool are scored by an internal Reward Model.
            - The highest-scoring outputs are labeled as "preferred," and the lowest-scoring outputs as "rejected."
        - This stage serves as the foundation for fine-tuning the model, using a stable and high-quality dataset.
    - Online Iterative Preference Training:
        - After offline training, the model undergoes online iterative training
            - Sampling: The current model generates multiple completions for each prompt.
            - Ranking: These completions are scored by the internal Reward Model, which ranks them from best to worst.
            - Training: The ranked completions are used to create new preference pairs, which are then used to fine-tune the model further.
    - Optimization Hyperparameters:
        - The iterative process is tuned with specific hyperparameters, such as the regularization coefficient β, to avoid overfitting or reward hacking. 
        - Aya Expanse found that three iterations of online preference training provided the optimal balance between gains in performance and computational cost.
* Achieves 7.1% win-rate improvement in multilingual benchmarks  for the Aya Expanse 8B model against Gemma 2 9B (m-ArenaHard).

### Model Merging
* Merges models trained on different *language families* to balance cross-lingual transfer and linguistic diversity.
* Techniques:
    - Weighted averaging (most effective).
    - SLERP, TIES-merging, and DARE-TIES (less consistent).
* Larger models benefit more from merging, with 3x gains observed in 32B models compared to 8B.

(Figure 3. Improvement with Aya Expanse models through Aya post-training recipe
 compared to their predecessor Aya 23 models) using  m-ArenaHard

## Model Architecture
* Model based on  Cohere's Command series.
    - SwishGLU
    - RoPE
    - Grouped-query attention
* Supports 23 languages, including: Arabic,
Chinese (Simplified & Traditional),
Czech,
Dutch,
English,
French,
German,
Greek,
Hebrew,
Hindi,
Indonesian,
Italian,
Japanese,
Korean,
Persian,
Polish,
Portuguese,
Romanian,
Russian,
Spanish,
Turkish,
Ukrainian,
Vietnamese.

## Evaluation Set up
The Aya Expanse models were evaluated using a comprehensive multilingual benchmark framework, comparing their performance against leading open-weight models across multiple tasks and datasets. The evaluation included:
* Generative Tasks:
    - Dataset
        - m-ArenaHard Dataset: Translations of Arena-Hard-Auto prompts into 23 languages to test multilingual capabilities.
        - Dolly Evaluation Set: Machine-translated and human-edited prompts in 23 languages.
    - Metrics: Win-rate comparisons against competitor models using GPT-4-based judges.
* Discriminative Tasks (zero-shot)
    - Evaluated models' understanding of causal reasoning, story completion, and coreference resolution.
    - Datasets: XCOPA, XStoryCloze, and XWinograd.
* Language Understanding
    - Global-MMLU: Tested knowledge across diverse fields and 23 languages.
    - Metrics: 5-shot accuracy.
* Mathematical Reasoning: MGSM Benchmark
* Machine Translation: 
    - FLORES-200 Dataset: Assessed translation quality in both directions (X↔English) for 22 language pairs.
    - Metrics: chrF++ and xCOMET scores.

(Figure 1. Table1:Comprehensive evaluation suite including 8 datasets, 6 task categories for up
 to 23 languages)

## Results and Insights
* Results:
    - m-ArenaHard: Aya Expanse 8B achieved up to 70.6\% win-rate against competitors, with the 32B variant reaching 76.6\% win-rate.
        - Outperformed larger models like Llama 3.1 70B despite fewer parameters.
        - Consistent improvement across languages (Figure 4:  Language-specific win-rates on m
ArenaHard)
    - Dolly: Aya Expanse 8B and 32B achieved up to 83.9% and 89.9% win-rates, respectively, showing superior performance. (Figure 2. Pairwise win-rates on Dolly evaluation set)
* Discriminative Tasks:
    - At the 8B scale:
        - Aya Expanse 8B showed competitive performance with a 70.3% accuracy, surpassing most competitors except for Gemma 2 9B.
    - At the 32B scale:
        - Aya Expanse 32B achieved 72.6% accuracy, close to top competitors Qwen 2.5 32B (73.4%) and Gemma 2 27B (72.9%).
* Language Understanding: Aya Expanse models significantly improved over Aya 23
    - 8B: 53.7% accuracy, a notable improvement over the previous 48.1%.
    - 32B: 66.9% accuracy, outperforming Aya 23's 57.9%.
* Mathematical Reasoning: Aya Expanse achieved the best results in its class
    - 8B: 67.0% accuracy, more than double Aya 23's 36.6%.
    - 32B: 73.4% accuracy, surpassing all competitors except Llama 3.1 70B (78.0%).
* Machine Translation: Aya Expanse models excelled in translation:
    - 8B: 57.2 chrF++ and 93.2 xCOMET, leading in its parameter class.
    - 32B: 58.8 chrF++ and 93.5 xCOMET, outperforming larger models like Llama 3.1 70B.

Results for outside Dolly and m-ArenaHard listed in:
    - (Table 2: Academicmultilingual benchmark results for 8-billionmodel class)
    - (Table 3: Academicmultilingual benchmarkresults for32-billionmodel class)

