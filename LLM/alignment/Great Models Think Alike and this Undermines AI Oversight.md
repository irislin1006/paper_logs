# Great Models Think Alike and this Undermines AI Oversight

---

### **Paper Overview: "Great Models Think Alike and this Undermines AI Oversight"**
#### **Authors & Affiliations**  
The paper is authored by researchers from various institutions, including the ELLIS Institute Tübingen, Max Planck Institute, University of Tübingen, IIIT Hyderabad, Contextual AI, and Stanford University.

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
