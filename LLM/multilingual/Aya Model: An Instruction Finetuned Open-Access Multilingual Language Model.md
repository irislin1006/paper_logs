# Aya Model: An Instruction Finetuned Open-Access Multilingual Language Model

## Introduction
Most LLMs cater to high-resource languages and cause disparities in multilingual NLP performance. The paper seeks to reduce these inequalities by: 
1. Expanding language coverage.
2. Developing a fine-tuned multilingual model capable of instruction-following in underrepresented languages.
3. Addressing challenges such as dataset quality, task diversity, and multilingual safety.

The main contribution of this paper is:
1. Multilingual Model - Aya
   - Aya is built on top of mT5 13B by finetuning with instruction mixture of dataset.
   - It covers 101 languages (>50% of these are low-resourced)
2. Multilingual Dataset
   - Training corpus contains 203 million examples (21.5% in English)
   - The dataset is built on top of xP3x, Aya collections, and machine-translated datasets.
   - Synthetic data generation was also ised to augment multilingual training.
3. Evaluation
   - Extensive multilingual evaluation suites covering 99 languages were introduced.
   - Evaluation tasks included discriminative (e.g., natural language inference) and generative tasks (e.g., translation and summarization).
4. Safety & Bias Analysis
   - Multilingual safety concerns were addressed by applying safety context distillation, reducing harmful generations by 78–89%.

## Experiments and Results
* Setup
    * Pretrained Base Model: mT5 13B was chosen for its wide language coverage.
    * Training Configuration: Used the Adafactor optimizer, cross-entropy loss, and data packing to train the model efficiently over 30,000 update steps.
* Aya outperforms mT0, BLOOMZ, and other models across tasks, particularly in:
    * Language Coverage: Doubles the coverage of languages compared to similar models.
    * Accuracy: Gains of 13.1% in discriminative tasks and 11.7% in generative tasks over mT0.
    * Translation Quality: Aya's spBLEU scores for FLORES-200 showed significant improvements in lower-resource languages.