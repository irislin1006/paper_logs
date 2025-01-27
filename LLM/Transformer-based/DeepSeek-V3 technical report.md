Below is a **detailed, section-by-section walkthrough** of the DeepSeek-V3 technical report. The goal is to clarify both the **high-level ideas** and the **technical nuances** found in the paper.

---

## 1. Introduction

**Core Idea**  
DeepSeek-V3 is a large-scale language model (LLM) following the Mixture-of-Experts (MoE) paradigm. It has a total of 671B parameters, with 37B parameters “activated” for each token (i.e., each token route goes through about 37B parameters). It pushes the boundary of open-source LLMs by reaching near state-of-the-art performance at a fraction of the usual training costs.

**Key Features**  
1. **Multi-Head Latent Attention (MLA)**: A specialized attention mechanism that significantly reduces key-value (KV) cache size during inference, thus enabling higher inference efficiency and supporting large context windows more cost-effectively.
2. **DeepSeekMoE**: The Mixture-of-Experts Feed-Forward Network architecture that partitions “experts” across different GPUs. This approach keeps training costs low because, for each token, only a subset of experts is activated.
3. **Auxiliary-Loss-Free Load Balancing**: Instead of the traditional auxiliary losses that penalize load imbalance, DeepSeek-V3 dynamically adjusts a bias term to achieve balanced routing without directly penalizing the model. This often improves final accuracy.
4. **Multi-Token Prediction (MTP)**: In pre-training, the model predicts multiple future tokens at once from each position, effectively increasing training signals and enabling a form of “look-ahead.” MTP can also be used to speed up decoding (speculative decoding).
5. **Efficient Infrastructures**: Use of FP8 training, a specialized pipeline (called DualPipe), and advanced communication schemes to drastically reduce both GPU hours and memory footprints.
6. **Stable Large-Scale Training**: Successfully trains on 14.8T tokens (in total) without major “loss spikes” or restarts. Final training costs are only about 2.788M H800 GPU hours (including pre-training + context extension + post-training).
7. **Open-Source**: Checkpoints publicly available, and the paper claims best or near-best performance among open-source models (especially in code and math domains).

---

## 2. Architecture

DeepSeek-V3 keeps a Transformer backbone but makes specific modifications in two main parts: **attention** and **Feed-Forward Networks (FFNs)** (the latter with an MoE design).

### 2.1. Basic Architecture

#### 2.1.1. Multi-Head Latent Attention (MLA)

- **Problem**: In standard self-attention, for each token, you have to store all the key-value (KV) states of past tokens. That leads to memory blow-up at large sequence lengths.  
- **Solution (MLA)**: 
  - Compressed hidden dimension for keys and values.  
  - Specifically, each token’s key-value states are first projected down to a smaller dimensional vector (called a “latent” vector), then re-expanded with separate up-projection weights.  
  - Only the smaller “latent” state plus a separate “decoupled key” gets cached during generation. This significantly shrinks the memory overhead for KV caching.  
  - In practice, each token has a “compressed key/value” plus a “decoupled key/query” to preserve positional embeddings (via rotary embeddings).  
- **Result**: MLA requires significantly smaller memory usage during inference, enabling longer context or more efficient serving, with near-equal quality to standard attention.

#### 2.1.2. DeepSeekMoE (Feed-Forward Networks)

- **Basic MoE Idea**: Instead of having one large FFN per layer, have multiple “experts” (each is essentially an FFN). Each token chooses which experts to use.  
- **Shared + Routed Experts**:  
  - A small number of **shared experts** that every token sees.  
  - A larger pool of **routed experts** that only selected tokens see.  
- **Token-Expert Affinity**: For each token, compute gating scores (e.g., via a dot product between the token’s hidden state and a learnable expert “centroid”). Then pick the top-K highest scores.  
- **Auxiliary-Loss-Free Strategy**:  
  - Typically, MoE training uses an auxiliary loss to push the model to distribute tokens evenly across experts. But this can harm final accuracy if that loss is too large.  
  - In DeepSeek-V3, they add a “bias” term to each expert’s gating score. After each batch, they adjust biases up/down for underloaded or overloaded experts, thereby keeping load balanced in a purely dynamic way.  
  - This yields good load balance while reducing the penalty on model performance.  
- **Sequence-Wise vs. Batch-Wise**:  
  - Traditional approach: put a load-balance term on each sequence so that every single sequence tries to balance expert usage.  
  - V3 approach: focuses on balancing “on average” across the entire batch, giving more freedom for domain specialization.

### 2.2. Multi-Token Prediction

- **Motivation**: Typically LLM training uses next-token prediction. MTP extends that to next-2 or next-3 tokens from each position, hopefully improving data efficiency and helping the model “plan” deeper.  
- **Implementation**:  
  - Add separate modules for each “extra step.” E.g., for a 1-depth MTP, predict the next token _and_ one more token.  
  - Each MTP module uses a short sub-Transformer block plus the main model’s embedding and output layers (physically shared to save memory).  
  - Loss: a sum of cross-entropies for each future token. Weighted by a factor \(\lambda\).  
- **Inference**:  
  - You can simply ignore the MTP modules if you want single-token decoding.  
  - Or you can use them to do “two-at-a-time” speculative decoding for speed.

---

## 3. Infrastructures

DeepSeek-V3 is trained on a cluster with 2,048 **NVIDIA H800** GPUs. Each node has 8 GPUs with NVLink/NVSwitch inside the node and InfiniBand (IB) across nodes.

### 3.1. Compute Clusters

- **Hardware**: 2,048 H800 GPUs, interconnected with IB (between nodes) and NVLink (intra-node).
- **Network**: Full bandwidth on IB so that all GPUs can communicate.

### 3.2. Training Framework

They develop an in-house system called **HAI-LLM** that supports:
- Pipeline parallelism (PP) across 16 stages,  
- Expert parallelism (EP) across 64 ranks (8 nodes),  
- ZeRO-1 data parallelism for everything else.

#### 3.2.1. DualPipe for Efficient Pipeline Parallelism

- **Problem**: Traditional pipeline parallel (like 1F1B) can lead to pipeline bubbles, or can’t overlap well the communication vs. computation.  
- **DualPipe**:  
  - The main trick is to feed micro-batches from both ends (bidirectional) and further rearrange forward/backward passes so that dispatch/communication are overlapped with other layers’ computations.  
  - Minimizes pipeline idle time and hides communication behind computation.  
- **Result**: Less pipeline bubble overhead, near-complete overlap of cross-node communication with attention/FFN computations.

#### 3.2.2. Efficient Cross-Node All-to-All

- For MoE, tokens might need to be “dispatched” to experts on other GPUs/nodes. That is an all-to-all or many-to-many communication.  
- They carefully fuse or overlap IB (inter-node) with NVLink (intra-node) communication: the token first goes to the GPU with the same local index on the target node, then is forwarded within that node to whichever GPU holds the target expert.  
- Use warp specialization to keep GPU usage for communication low (around 20 Streaming Multiprocessors).  
- Overlap dispatch with attention or MLP computations.

#### 3.2.3. Memory Savings

- Recompute RMSNorm or MLA up-projection on the fly during backprop.  
- Keep exponential moving average parameters in CPU, updated asynchronously.  
- Physical parameter sharing of embedding/output heads for multi-token-prediction modules.

### 3.3. FP8 Training

DeepSeek-V3 implements a fine-grained **FP8** training approach. The gist:

1. **FP8 for GEMM**: The main matrix multiplications (Linear layers) are done in 8-bit floating point.  
2. **Higher Precision for Sensitive Ops**: Embeddings, gating, normalization, etc., remain in BF16 or FP32.  
3. **High-Precision Accumulation**: They promote partial results to higher precision quickly so that large matrix multiplications (with big K dimension) do not degrade.  
4. **Fine-Grained Quantization**: Instead of a single scaling factor per entire matrix, they do tile-wise (1×128) or block-wise (128×128) scaling to mitigate outlier issues.  
5. **Low-Precision Storage**: Use BF16 or FP8 for caching activations and storing optimizer states, which drastically reduces memory consumption.

### 3.4. Inference and Deployment

They separate the pipeline into **Prefilling** (process the entire input context) and **Decoding** (generating tokens one-by-one).

- **Prefilling**:  
  - Use a smaller pipeline parallel size (TP4 + DP8 + EP32 per 4-node group).  
  - Possibly host “redundant experts” for balancing the load. That is, replicate certain experts so that no single GPU is overloaded.  
  - Pack two micro-batches simultaneously to hide the all-to-all overhead.

- **Decoding**:  
  - Larger scale: 40 nodes → 320 GPUs, with a bigger EP factor (EP320).  
  - To keep the load balanced, again replicate heavy-load experts or apply on-the-fly dynamic redundancy.  
  - The authors note they can also do partial SM usage for dispatch/combine since the batch size is smaller during decoding, so the attention pass can happen in parallel.

### 3.5. Suggestions on Hardware Design

The authors propose changes for future GPUs:

1. **Hardware-Assisted Communication**: A dedicated co-processor for IB+NVLink data forwarding, so that SMs (the main GPU compute unit) aren’t wasted on dispatch/collect.  
2. **Better FP8 Tensor Cores**: Full or partial FP32 accumulation, to avoid losing bits in large sums.  
3. **Support for Tile/Block-Wise Quantization**: So scaling factors can be applied inside the Tensor Cores.  
4. **Online Quantization**: Possibly do the conversion as data moves from HBM to shared memory.  
5. **Native Transposed GEMM**: So that the backward pass for certain operators can skip extra memory copies.

---

## 4. Pre-Training

### 4.1. Data Construction

- **14.8T tokens** from a high-quality multilingual corpus.  
- Large coverage of math and programming content.  
- Document packing (to fill sequences) but no cross-sample attention.  
- Some proportion (10%) uses Fill-In-Middle (FIM) for better code generation skill.  
- Tokenizer: Byte-level BPE with a vocabulary size of 128k, improved for multilingual.

### 4.2. Hyper-Parameters

- 61 Transformer layers, hidden size 7168, 128 attention heads.  
- MLA compression: KV dimension 512, query compression dimension 1536, etc.  
- MoE: from layer 4 onward, each FFN is replaced with 1 shared + 256 routed experts. Each token picks top-8 among them.  
- MTP depth = 1 (next-2 tokens predicted).  
- Learning rate schedule: starts at 2.2e-4, decays eventually to 7.3e-6.  
- Batch size eventually goes up to 15,360 (parallel across many GPUs).

### 4.3. Long Context Extension

- After the main pre-training with a 4k context, they do two short extension phases:  
  1. Extend from 4k to 32k context.  
  2. Extend from 32k to 128k context.  
- Use YaRN approach (a method for resizing rotary embeddings).  
- They only do ~1k steps at each extension stage with a specialized schedule, so the model can handle up to 128k tokens.

### 4.4. Evaluations of the Base Model

They test on standard language tasks (MMLU, C-Eval, reading comprehension, etc.), code benchmarks (HumanEval, MBPP), and math tasks (MATH, GSM8K).

**Findings**:
- DeepSeek-V3 base outperforms other large open-source base models like Qwen2.5-72B and LLaMA-3.1-405B in many tasks, especially in coding (HumanEval, LiveCodeBench) and math (MATH).  
- Also strong for multilingual tasks.  
- Achieves better or close performance to bigger or similarly sized dense models, at lower cost because it is MoE.

### 4.5. Discussion (Ablations)

1. **Multi-Token Prediction**: They tested smaller MoE baselines with/without MTP. MTP consistently improves performance on various tasks.  
2. **Auxiliary-Loss-Free vs. “Aux Loss Based”**: The dynamic bias approach yields better performance or at least no performance drop, while still balancing load well.  
3. **Batch-Wise vs. Sequence-Wise**: Encouraging load balance on a per-sequence level is too restrictive; letting it balance over the entire batch leads to better specialization of experts.

---

## 5. Post-Training

DeepSeek-V3 undergoes **two** further steps after base pre-training:

1. **Supervised Fine-Tuning (SFT)**
2. **Reinforcement Learning (RL)**

### 5.1. Supervised Fine-Tuning

- **Data**: 1.5M curated instructions from multiple domains.  
- **Particularly Strong in Reasoning**: They add new data generated by an internal long-chain-of-thought model (DeepSeek-R1). They filter or refine R1’s output so that V3 can “inherit” R1’s strong reasoning patterns but produce more concise responses.  
- They also add standard simpler instruction data for creative tasks, Q&A, role-play, etc.  
- 2-epoch fine-tuning, with standard LR scheduling.

### 5.2. Reinforcement Learning

- **Reward Model**:  
  - For tasks with deterministic solutions (like code or math), they rely on a rule-based checker (like running code or verifying numeric answers).  
  - For more open-ended tasks, they train a model-based RM from DeepSeek-V3-SFT.  
- **Group Relative Policy Optimization (GRPO)**:  
  - No separate value network; they compare N outputs for the same prompt to compute relative advantage.  
  - This approach was also used in DeepSeek-V2.  
- Combined with “constitutional AI” ideas, letting the model refine its own alignment with curated instructions.

### 5.3. Evaluations

**Key Observations**:
- Outperforms older DeepSeek versions, as well as Qwen2.5 or LLaMA-3.1, especially in code and math.  
- Performance is close to GPT-4o or Claude-3.5 on numerous tasks (though not always surpassing them).  
- Also tested on open-ended conversation tasks (like AlpacaEval 2.0, Arena-Hard) using GPT-4 as a judge. Achieves best or near-best results among open-source models, often close to closed-source.

### 5.4. Discussion

1. **Distillation from R1**: Distilling solutions from a specialized long-chain-of-thought model significantly boosts math/coding performance.  
2. **Self-Rewarding**: Using the model’s own (or a variant of it) evaluation for feedback in general tasks can be effective (though it has limitations).  
3. **Multi-Token Prediction**: The acceptance rate for the second predicted token is 85–90%, speeding up decoding.

---

## 6. Conclusion, Limitations, and Future Directions

1. **Conclusions**:  
   - DeepSeek-V3 is currently among the best open-source LLMs, bridging the gap with top proprietary models.  
   - Achieved strong code and math performance in particular, as well as broad knowledge tasks (MMLU, etc.).  
   - Trained in under ~2.8 million H800 GPU-hours, an impressive cost-effectiveness for a model of this scale.

2. **Limitations**:  
   - Inference deployment can still require large clusters due to MoE’s cross-node communication.  
   - Decoding could be even faster with more advanced parallelization or future hardware.  

3. **Future Directions**:  
   - Refine the architecture (beyond Transformers), possibly exploring infinite context or memory-based methods.  
   - Keep scaling data quality and quantity.  
   - Expand the “deep thinking” approach (especially for domain-specific reasoning).  
   - Develop broader, more robust evaluation frameworks to avoid “overfitting” to certain benchmarks.

---

## Key Takeaways

1. **MoE Efficiency**: By activating only a subset of experts per token, the model can effectively scale to hundreds of billions of parameters yet remain cost-effective in training.
2. **MLA**: A well-engineered attention scheme that drastically cuts memory usage for key-value caches, which is particularly valuable at context lengths of 4K–128K.
3. **Low-Precision (FP8) + DualPipe**: A synergy of new pipeline scheduling and advanced quantization that hides communication overhead and pushes throughput, reducing final training time and cost.
4. **Performance**: Consistently top-tier results among open-source models, extremely high in code (both coding competition and software engineering tasks) and math (MATH, AIME, etc.).
5. **Alignment**: A multi-stage post-training approach (SFT + RL, some distillation from R1) yields strong alignment without sacrificing core capabilities.

---

### Additional Notes

- **Why MoE for Large Scale**: A big theme in MoE is that it decouples “total parameters” from “activated parameters.” The model can have hundreds or thousands of specialized experts, but the runtime only “activates” a handful for each token. This preserves training quality (many parameters to learn specialized patterns) but avoids the full memory or compute cost if you were to run them all at once.
- **Bias Update Trick**: The auxiliary-loss-free load balancing is a noteworthy design. Instead of imposing a direct penalty for unbalanced experts, the model learns to shift a gating bias so that if an expert is “overloaded,” it becomes less likely to be selected. If an expert is underloaded, it becomes more likely. This typically yields balanced usage without harming the main next-token objective.
- **Speculative Decoding**: With MTP, the model can propose multiple next tokens in one step. A final check quickly verifies if the second token matches the base model distribution. Because acceptance rate is 85–90%, it significantly speeds up generation.

---

## Summary

DeepSeek-V3 is an **MoE-based** large language model with 671B parameters total (37B active). Through **MLA** attention, **auxiliary-loss-free balancing**, **FP8 training**, and a pipeline design that fully overlaps computation and communication, it significantly **reduces training costs**. After **14.8T tokens** of pre-training and a well-crafted **post-training** (SFT + RL + knowledge distillation from specialized reasoning models), DeepSeek-V3 achieves **top-tier performance** in code generation, mathematical reasoning, multilingual tasks, and more—rivaling or approaching the best closed-source models in many benchmarks. The open release of DeepSeek-V3 paves the way for further large-scale MoE research and more powerful open-source LLM applications.