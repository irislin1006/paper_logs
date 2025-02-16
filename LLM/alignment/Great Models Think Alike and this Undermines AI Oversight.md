# Great Models Think Alike and this Undermines AI Oversight

---


## **1. Problem Statement & Motivation**
- As **language models (LMs)** improve, it is becoming **harder for humans** to evaluate and oversee them.
- There is **hope** that other LMs can automate oversight (**"AI oversight"**), but this **depends on the diversity of models** used for oversight.
- The **key concern**: If models become more similar, their **mistakes become correlated**, which is problematic for AI safety and oversight.

### **Key Hypothesis:**
- AI oversight is only effective if the oversight model is **different enough** from the models it is judging.  
- The paper introduces **Chance Adjusted Probabilistic Alignment (CAPA)** to measure **LM similarity**.

---

## **2. Key Contributions**
1. **New Metric (CAPA):**  
   - **Measures LM similarity** based on how often they make similar mistakes.
   - **Adjusts for chance agreement** (avoids falsely high similarity for accurate models).
   - **Uses probability distributions** instead of just discrete correctness labels.

2. **Findings:**
   - **AI Judges Favor Similar Models**: LLMs used as judges tend to favor models that are similar to them.
   - **Weak-to-Strong Training Gains Depend on Model Dissimilarity**:  
     - Training a strong model on annotations from a weaker model works **better when the models are functionally different**.
   - **Error Correlation Increases with Model Capability**:  
     - As models become more advanced, their mistakes become **more similar**, raising concerns about systemic blind spots.

---

## **3. Key Sections & Results**
### **Methodology: CAPA Metric (Section 2)**
- Traditional similarity measures (e.g., Cohen’s kappa, error consistency) fail to capture key aspects of LM similarity.
- **CAPA** improves over them by:
  - **Using probability distributions** rather than binary correctness.
  - **Accounting for accuracy differences** so high-accuracy models are not unfairly judged as similar.
- Formula:
  \[
  \kappa_p = \frac{c_{\text{pobs}} - c_{\text{pexp}}}{1 - c_{\text{pexp}}}
  \]
  where:
  - \(c_{\text{pobs}}\) is the **probabilistic observed agreement**.
  - \(c_{\text{pexp}}\) is the **expected agreement by chance**.

### **LLM-as-a-Judge Bias (Section 3)**
- **Experiment:** Evaluated how different models act as judges in tasks like MMLU-Pro.
- **Key Finding:** LLM judges systematically rate **more similar models higher**, even if they are less capable than dissimilar models.
- **Implication:** AI judges are biased towards models with which they share **common failure patterns**.

### **Weak-to-Strong Generalization (Section 4)**
- **Experiment:** Large models were trained on annotations from smaller models.
- **Key Finding:**  
  - Performance improvement **increases when the weak model is functionally different** from the strong model.
  - If the weak and strong models are **too similar**, less knowledge is gained from training.

### **Rising Error Correlation in Advanced LMs (Section 5)**
- **Experiment:** 130 LMs were evaluated for similarity trends.
- **Key Finding:**  
  - As LMs become **more capable**, their mistakes **become more similar**.
  - **Risk:** AI oversight relies on diversity, but models are converging in their failure modes.

---

## **4. Broader Implications**
- **Bias in AI Judges**: Using LLMs as evaluators needs **bias correction**.
- **Training Strategies**: Selecting a diverse set of weak models may **enhance training outcomes**.
- **AI Safety Risk**: Model convergence could lead to **systematic blind spots**, affecting **trustworthy AI**.

---

## **5. Limitations & Future Work**
- **Causal Confirmation Needed**:  
  - Future work should explore whether **changing model architecture** can **reduce similarity without reducing capability**.
- **Measuring Similarity for Free-Text Generation**:  
  - Current methods focus on MCQ and classification; **new techniques are needed for generative models**.
- **Addressing Safety Risks**:  
  - More research is needed to mitigate correlated failures in **real-world AI deployments**.

---

## **Final Takeaways**
- AI oversight is **compromised** when models are too **similar**.
- **CAPA metric** offers a **better way** to measure LM similarity.
- As LMs advance, **they make more similar mistakes**, which **threatens independent AI oversight**.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++

### **In-Depth Summary of the Paper: "Great Models Think Alike and this Undermines AI Oversight"**

---

## **1. Core Concept**
### **Main Idea and Problem Addressed**
The paper investigates the role of model similarity in AI oversight, which involves using language models (LMs) to evaluate and supervise other models. It introduces **Chance Adjusted Probabilistic Agreement (CAPA)** as a novel metric to measure functional similarity between LMs based on shared errors, rather than just accuracy.

As AI models become increasingly capable, their evaluation and supervision become more complex and expensive. The common approach is to rely on other LMs for judgment and annotation tasks, a paradigm referred to as **AI oversight**. However, this raises a fundamental question:  
*"Can we trust AI oversight when models become too similar?"*

### **Importance of the Problem**
- **Bias in AI evaluations:** When models act as judges, they may favor outputs from similar models, introducing affinity bias.
- **Training risks:** When models are trained on annotations from other LMs, gains are higher when the models are different, leveraging complementary knowledge. However, if all models become too similar, this benefit diminishes.
- **Correlated failure modes:** As LMs grow more capable, their errors become increasingly similar, leading to systemic risks in AI oversight.

---

## **2. Key Contributions**
The paper presents three major findings:

1. **Bias in LLM-as-a-Judge evaluations:**  
   - LMs acting as judges systematically favor models that are similar to themselves.  
   - CAPA quantifies this effect and shows that similarity biases judgment beyond just capability-based evaluations.

2. **Impact of similarity on LM-based training (Weak-to-Strong Generalization):**  
   - Training a strong model using annotations from a weak supervisor is most effective when the two models have **low similarity** (i.e., they make different mistakes).  
   - Complementary knowledge plays a crucial role in performance improvements.

3. **The correlation of model errors increases with capability:**  
   - As LMs become more powerful, their mistakes become more aligned, reducing the diversity of perspectives in AI oversight and increasing risks of correlated failures.

---

## **3. Methodology & Experiment Setup**
### **Novel Similarity Metric: CAPA (Chance Adjusted Probabilistic Agreement)**
The authors refine traditional **error consistency** metrics by addressing two key limitations:
1. **Distinguishing different mistakes:** Traditional error consistency counts any wrong predictions as similar, even if they differ in content.
2. **Incorporating probability distributions:** Unlike binary correctness measures, CAPA uses output probabilities to assess the degree of similarity in model errors.

The CAPA metric is defined as:

\[
\kappa_p = \frac{c_{pobs} - c_{pexp}}{1 - c_{pexp}}
\]

Where:  
- \( c_{pobs} \) is the observed agreement probability based on output likelihoods.
    - (Observed Agreement): Measures how often the two models assign high probability to the same answers.
- \( c_{pexp} \) is the expected chance agreement, adjusted for model accuracy.
    - (Expected Agreement): Measures how often we expect them to agree just by chance, based on their accuracy.

### **Experiments Conducted**
1. **LLM-as-a-Judge Bias Analysis**
   - **Task:** Models judged free-text responses from other models on **MMLU-Pro** (a benchmark for multi-task understanding).
   - **Judgment Process:** Judges labeled responses as correct/incorrect without seeing reference answers.
   - **Key Metric:** CAPA similarity between the evaluated model and the judge was computed.
   - **Result:** Judges consistently rated more similar models as better, even controlling for accuracy.

2. **Weak-to-Strong Generalization**
   - **Task:** A strong model was fine-tuned using annotations from a weak supervisor across **15 NLP tasks**.
   - **Key Hypothesis:** Gains should be higher when the weak and strong models are functionally dissimilar.
   - **Key Metric:** CAPA similarity between weak and strong models.
   - **Result:** Performance improvements were inversely correlated with similarity (r = -0.85).

3. **Error Correlation in Increasingly Capable LMs**
   - **Dataset:** **130 LMs** from Hugging Face's OpenLLM Leaderboard.
   - **Task:** Models' errors were analyzed across **MMLU-Pro** and **BBH**.
   - **Key Finding:** Higher capability LMs exhibit increasingly similar mistakes (positive correlation between CAPA and model capability).

---

## **4. Results & Performance**
### **Findings on LLM-as-a-Judge Bias**
- LMs acting as judges favor models similar to themselves.  
- A strong positive correlation exists between CAPA similarity and judgment scores (r = 0.84, p < 0.01).  
- Controlling for accuracy, similarity still significantly influences judgment.

### **Findings on Weak-to-Strong Generalization**
- Gains in **weak-to-strong training** are maximized when models are dissimilar.  
- Complementary knowledge transfer enables higher accuracy improvements than previously estimated.  
- The weak-to-strong trained model surpassed both the weak and strong baseline models.

### **Findings on Correlated Model Errors**
- As model capability increases, their errors become **more similar** (positive CAPA correlation).  
- This trend weakens AI oversight by reducing the diversity of errors that supervision mechanisms rely on.  
- The authors caution that relying on AI oversight for safety may introduce risks from **common blind spots**.

---

## **5. Limitations & Missing Elements**
### **Limitations Identified**
1. **Causality Uncertainty:**  
   - The study establishes correlation but does not directly prove **causation**.  
   - More controlled interventions are needed to manipulate model similarity without affecting capability.

2. **Lack of Free-Text Evaluation Metrics:**  
   - The study focuses on **multiple-choice questions (MCQ)** because free-text evaluation remains an open problem.  
   - A future **free-response CAPA metric** could better capture LLM similarity in real-world use.

3. **Need for Diverse Architectures:**  
   - Current findings are based on transformer models.  
   - Investigating **non-transformer models (e.g., Mamba)** could provide deeper insights into similarity effects.

4. **Risk of Overfitting to Existing Benchmarks:**  
   - The study relies on popular benchmarks like **MMLU-Pro and BBH**.  
   - Results might not generalize to **real-world open-ended tasks**.

---

## **6. Potential Extensions or Future Work**
### **Proposed Future Directions**
1. **Developing Causal Interventions:**  
   - Methods to **control** model similarity while preserving capability.
   - Examining interventions like **architectural modifications** or **diverse training paradigms**.

2. **Extending CAPA to Free-Text Generation:**  
   - The study is limited to **MCQ tasks**.
   - A future CAPA-like metric tailored for **open-ended text responses** could be valuable.

3. **Investigating Oversight in Multi-Agent Systems:**  
   - If multiple LMs are used for oversight, how can we ensure diversity in their evaluation?
   - Would AI juries composed of diverse models mitigate similarity biases?

4. **Studying Effects Across Different Architectures:**  
   - The study focuses on transformer-based models.
   - Exploring whether **state-space models (e.g., Mamba)** exhibit similar error correlation trends.

5. **Addressing the Generator-Verifier Gap:**  
   - AI oversight is often used when verifying correctness is easier than generating solutions.
   - Studying how model similarity affects this **generator-verifier dynamic** is an open research problem.

---

## **Final Thoughts**
This paper provides an **important critique** of the increasing reliance on AI oversight for model evaluation and supervision. By introducing the **CAPA metric**, it uncovers hidden biases in **LLM-based judgment**, demonstrates the risks of **correlated failure modes**, and highlights the necessity of **diverse model perspectives** in AI supervision.

### **Key Takeaways**
- AI models **prefer similar models** when acting as judges.
- Training benefits from **dissimilar** weak and strong model pairs.
- Increasing model capability leads to **more correlated mistakes**, creating **systemic risks**.

The research underscores the **urgency of maintaining diversity** in model development and evaluation to prevent **blind spots in AI oversight**.

### **Understanding CAPA (Chance Adjusted Probabilistic Agreement)**  
## **1. Why Do We Need CAPA?**
Traditional similarity metrics have limitations:
- **Accuracy Bias:** Two highly accurate models may appear more similar than they really are just because they have fewer opportunities to make mistakes.
- **Ignoring Different Mistakes:** If two models make different wrong predictions, traditional error consistency still counts them as similar, which is misleading.
- **No Probabilities Considered:** If two models assign different probabilities to their predictions, that should matter, but older metrics don’t capture this.

### **Example of the Problem**
Imagine two models predicting the answer to a multiple-choice question with options A, B, and C:
- **Model 1:** Predicts A with 99% confidence and B with 1%.
- **Model 2:** Predicts B with 60% confidence and A with 40%.

If the correct answer is A:
- **Model 1 is correct**, but **Model 2 is incorrect**.
- A traditional metric would see **only correctness** (1 vs. 0) and mark them as different.
- But Model 2 was actually closer to Model 1 in terms of probability assignment.
- CAPA captures this finer-grained similarity.

---

## **2. How is CAPA Computed?**
CAPA is based on **how much two models agree beyond what we would expect by chance**, considering their accuracy.

The formula for CAPA is:

\[
\kappa_p = \frac{c_{pobs} - c_{pexp}}{1 - c_{pexp}}
\]

Where:
- **\( c_{pobs} \) (Observed Agreement):** Measures how often the two models assign high probability to the same answers.
- **\( c_{pexp} \) (Expected Agreement):** Measures how often we expect them to agree **just by chance**, based on their accuracy.

### **Step-by-Step Explanation**
1. **Compute \( c_{pobs} \) (Observed Agreement)**  
   - Instead of just counting correct/incorrect answers, we check how much probability mass both models assign to the same options.
   - If both models predict **A with high probability**, that counts as strong agreement.
   - If one predicts **A with high confidence** while the other spreads its probability between **B and C**, that counts as lower agreement.

2. **Compute \( c_{pexp} \) (Expected Agreement by Chance)**
   - If two models were independent, they would agree **just because of random chance**.
   - We estimate this using their accuracy levels and how they distribute probability over different answers.

3. **Compute CAPA**
   - CAPA tells us how much of the agreement is **real** (i.e., beyond what would happen just by chance).

---

## **3. What Do CAPA Values Mean?**
CAPA is always between **-1 and 1**, where:
- **+1** → The two models make the **exact same predictions all the time**.
- **0** → The models agree **only as much as expected by chance**.
- **-1** → The models always make **opposite predictions**.

### **Interpreting CAPA in Practice**
- **High CAPA (near +1):** The two models make **very similar mistakes**, meaning they process information in a similar way.
- **Medium CAPA (around 0):** The two models are **functionally different**, meaning they don't make the same mistakes.
- **Low CAPA (negative values):** The two models often **make opposite mistakes**, meaning they have very different perspectives.

---

## **4. Why is CAPA Important?**
- **For Evaluating AI Judges:**  
  - If a judge model has a **high CAPA** with an evaluated model, it may **overrate its performance** because it finds its responses familiar.
  - This introduces **bias in AI oversight**.
  
- **For Training AI Models (Weak-to-Strong Generalization):**  
  - If a strong model is trained using annotations from a weak model that has **high CAPA** with it, the strong model won’t learn much new.
  - If the weak model has **low CAPA** (i.e., it makes different kinds of mistakes), the strong model can learn **more complementary knowledge** and improve significantly.

---

## **5. CAPA vs. Traditional Metrics**
| **Metric** | **Adjusts for Accuracy?** | **Distinguishes Different Mistakes?** | **Uses Probabilities?** |
|------------|----------------|-----------------------------|----------------|
| Error Consistency | ✅ Yes | ❌ No | ❌ No |
| Cohen’s Kappa | ❌ No | ✅ Yes | ❌ No |
| Pearson Correlation (Errors) | ✅ Yes | ❌ No | ❌ No |
| KL Divergence | ❌ No | ✅ Yes | ✅ Yes |
| **CAPA (New Metric)** | ✅ Yes | ✅ Yes | ✅ Yes |

CAPA is the **only metric that satisfies all three** criteria.

---

## **6. A Simple Example**
| **Question** | **Correct Answer** | **Model 1 Prediction** | **Model 2 Prediction** |
|-------------|----------------|-----------------|-----------------|
| Q1 | A | **A (90%)** | **A (85%)** |
| Q2 | B | **C (60%)** | **B (70%)** |
| Q3 | C | **C (95%)** | **C (80%)** |

**Traditional metrics** would say **Model 1 and Model 2 are only similar on Q1 and Q3**.  
**CAPA, however, will recognize that on Q2, Model 2 was "closer" to the right answer than Model 1, even though it was still wrong.**  

This makes CAPA a **better similarity measure** than just comparing correctness.

---

## **7. Final Takeaways**
- CAPA **measures model similarity based on mistakes, not just correctness**.
- It **accounts for probability distributions**, capturing subtle similarities in how models think.
- Higher CAPA **means models fail in the same way**, which is a problem for **AI oversight**.
- AI systems should aim for **lower CAPA between training models** to maximize learning from diverse mistakes.

This metric is **critical for ensuring diversity in AI models** and preventing **systemic risks from correlated failures**.