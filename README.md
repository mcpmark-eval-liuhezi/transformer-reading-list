# Transformer Reading List

A curated, verified reading list on the Transformer architecture for our lab reading group.

**How this list was built.** Every entry is grounded in the paper itself. The abstract and section outline for each paper are taken from the paper's **arXiv LaTeX source**, so what you see below is the paper's real structure rather than a paraphrase. The category, preprint version and venue notes are quoted from the official arXiv abstract page.

The core paper is *Attention Is All You Need*. The remaining papers trace three threads that grew directly out of it: **efficient attention** (FlashAttention, Native Sparse Attention), **long-context modeling** (LongRoPE2), and **attention alternatives** — state-space models and linear-recurrent hybrids (Mamba, Mamba-2, Gated DeltaNet).

## Papers

| # | Title | arXiv ID | Primary Category |
|---|-------|----------|------------------|
| 1 | Attention Is All You Need | [1706.03762](https://arxiv.org/abs/1706.03762) | Computation and Language (cs.CL) |
| 2 | FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness | [2205.14135](https://arxiv.org/abs/2205.14135) | Machine Learning (cs.LG) |
| 3 | Mamba: Linear-Time Sequence Modeling with Selective State Spaces | [2312.00752](https://arxiv.org/abs/2312.00752) | Machine Learning (cs.LG) |
| 4 | Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality | [2405.21060](https://arxiv.org/abs/2405.21060) | Machine Learning (cs.LG) |
| 5 | Gated Delta Networks: Improving Mamba2 with Delta Rule | [2412.06464](https://arxiv.org/abs/2412.06464) | Computation and Language (cs.CL) |
| 6 | Native Sparse Attention: Hardware-Aligned and Natively Trainable Sparse Attention | [2502.11089](https://arxiv.org/abs/2502.11089) | Computation and Language (cs.CL) |
| 7 | LongRoPE2: Near-Lossless LLM Context Window Scaling | [2502.20082](https://arxiv.org/abs/2502.20082) | Computation and Language (cs.CL) |

---

## Per-paper notes

### 1. Attention Is All You Need

- **arXiv ID:** [1706.03762](https://arxiv.org/abs/1706.03762)
- **Primary category:** Computation and Language (cs.CL)
- **Current version:** v7 — last revised 2 Aug 2023
- **Venue note (arXiv Comments field):** "15 pages, 5 figures" — no venue is listed on the arXiv page.

**Abstract** (from the arXiv LaTeX source):

> The dominant sequence transduction models are based on complex recurrent or convolutional neural networks that include an encoder and a decoder. The best performing models also connect the encoder and decoder through an attention mechanism. We propose a new simple network architecture, the Transformer, based solely on attention mechanisms, dispensing with recurrence and convolutions entirely. Experiments on two machine translation tasks show these models to be superior in quality while being more parallelizable and requiring significantly less time to train. Our model achieves 28.4 BLEU on the WMT 2014 English-to-German translation task, improving over the existing best results, including ensembles, by over 2 BLEU. On the WMT 2014 English-to-French translation task, our model establishes a new single-model state-of-the-art BLEU score of 41.8 after training for 3.5 days on eight GPUs, a small fraction of the training costs of the best models from the literature. We show that the Transformer generalizes well to other tasks by applying it successfully to English constituency parsing both with large and limited training data.

**Section outline** (from the arXiv LaTeX source):

1. Introduction
2. Background
3. Model Architecture
4. Why Self-Attention
5. Training
6. Results
7. Conclusion
8. Attention Visualizations

**Annotation:** This is the founding paper of the architecture: it replaces recurrence and convolution entirely with attention, introducing scaled dot-product attention, multi-head attention, and the encoder-decoder stack that everything else in this list builds on. Its results on WMT 2014 machine translation (28.4 BLEU En-De, 41.8 BLEU En-Fr) established the quality-per-training-dollar argument that made the Transformer the default. Read it first — the "Why Self-Attention" section in particular is the cleanest statement of why the design trades path length for parallelism.

---

### 2. FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness

- **arXiv ID:** [2205.14135](https://arxiv.org/abs/2205.14135)
- **Primary category:** Machine Learning (cs.LG)
- **Current version:** v2 — last revised 23 Jun 2022
- **Venue note:** the arXiv page lists no Comments field, so no venue note is shown there.

**Abstract** (from the arXiv LaTeX source):

> Transformers are slow and memory-hungry on long sequences, since the time and memory complexity of self-attention are quadratic in sequence length. Approximate attention methods have attempted to address this problem by trading off model quality to reduce the compute complexity, but often do not achieve wall-clock speedup. We argue that a missing principle is making attention algorithms IO-aware—accounting for reads and writes between levels of GPU memory. We propose FlashAttention, an IO-aware exact attention algorithm that uses tiling to reduce the number of memory reads/writes between GPU high bandwidth memory (HBM) and GPU on-chip SRAM. We analyze the IO complexity of FlashAttention, showing that it requires fewer HBM accesses than standard attention, and is optimal for a range of SRAM sizes. We also extend FlashAttention to block-sparse attention, yielding an approximate attention algorithm that is faster than any existing approximate attention method. FlashAttention trains Transformers faster than existing baselines: 15% end-to-end wall-clock speedup on BERT-large (seq. length 512) compared to the MLPerf 1.1 training speed record, 3× speedup on GPT-2 (seq. length 1K), and 2.4× speedup on long-range arena (seq. length 1K-4K). FlashAttention and block-sparse FlashAttention enable longer context in Transformers, yielding higher quality models (0.7 better perplexity on GPT-2 and 6.4 points of lift on long-document classification) and entirely new capabilities: the first Transformers to achieve better-than-chance performance on the Path-X challenge (seq. length 16K, 61.4% accuracy) and Path-256 (seq. length 64K, 63.1% accuracy).

**Section outline** (from the arXiv LaTeX source; the source defines a `\sysname` macro that expands to *FlashAttention*, resolved below):

1. Introduction
2. Background
3. FlashAttention: Efficient Attention with Tiling & Recomputation
4. FlashAttention: Algorithm, Analysis, and Extensions
5. Analysis: IO Complexity of FlashAttention
6. Experiments
7. Limitations and Future Directions
8. Checklist
9. Related Work
10. Algorithm Details
11. Proofs
12. Extension Details
13. Full Experimental Results

**Annotation:** FlashAttention is the key systems-level answer to the Transformer's quadratic cost: rather than approximating attention, it makes *exact* attention fast by tiling the computation and keeping intermediate results in on-chip SRAM, so the HBM↔SRAM traffic — not the FLOPs — is what gets minimized. The paper proves the IO complexity is optimal for a range of SRAM sizes and extends the idea to block-sparse attention. This is the work that made long-context training practical on GPUs, and it is now the default attention implementation in most modern stacks.

---

### 3. Mamba: Linear-Time Sequence Modeling with Selective State Spaces

- **arXiv ID:** [2312.00752](https://arxiv.org/abs/2312.00752)
- **Primary category:** Machine Learning (cs.LG)
- **Current version:** v2 — last revised 31 May 2024
- **Venue note:** the arXiv page lists no Comments field, so no venue note is shown there.

**Abstract** (from the arXiv LaTeX source):

> Foundation models, now powering most of the exciting applications in deep learning, are almost universally based on the Transformer architecture and its core attention module. Many subquadratic-time architectures such as linear attention, gated convolution and recurrent models, and structured state space models (SSMs) have been developed to address Transformers' computational inefficiency on long sequences, but they have not performed as well as attention on important modalities such as language. We identify that a key weakness of such models is their inability to perform content-based reasoning, and make several improvements. First, simply letting the SSM parameters be functions of the input addresses their weakness with discrete modalities, allowing the model to selectively propagate or forget information along the sequence length dimension depending on the current token. Second, even though this change prevents the use of efficient convolutions, we design a hardware-aware parallel algorithm in recurrent mode. We integrate these selective SSMs into a simplified end-to-end neural network architecture without attention or even MLP blocks (Mamba). Mamba enjoys fast inference (5× higher throughput than Transformers) and linear scaling in sequence length, and its performance improves on real data up to million-length sequences. As a general sequence model backbone, Mamba achieves state-of-the-art performance across several modalities such as language, audio, and genomics. On language modeling, our Mamba-3B model outperforms Transformers of the same size and matches Transformers twice its size, both in pretraining and downstream evaluation.

**Section outline** (from the arXiv LaTeX source):

1. Introduction
2. State Space Models
3. Selective State Space Models
4. Empirical Evaluation
5. Discussion
6. Conclusion
7. Discussion: Selection Mechanism
8. Related Work
9. Mechanics of Selective SSMs
10. Hardware-aware Algorithm For Selective SSMs
11. Full Experiments
12. Experimental Details and Additional Results

**Annotation:** Mamba is the most influential "attention alternative" to date: it makes the SSM parameters input-dependent (the *selection* mechanism), which is what finally lets a subquadratic model do content-based reasoning on language. The trade-off is that selection breaks the convolutional form, so the paper contributes a hardware-aware parallel scan algorithm to keep training efficient. Read this to understand the strongest current case that attention is not the only viable sequence mixer.

---

### 4. Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality

- **arXiv ID:** [2405.21060](https://arxiv.org/abs/2405.21060)
- **Primary category:** Machine Learning (cs.LG)
- **Current version:** v1 — submitted 31 May 2024 (only version)
- **Venue note (arXiv Comments field):** "ICML 2024"

**Abstract** (from the arXiv LaTeX source):

> While Transformers have been the main architecture behind deep learning's success in language modeling, state-space models (SSMs) such as Mamba have recently been shown to match or outperform Transformers at small to medium scale. We show that these families of models are actually quite closely related, and develop a rich framework of theoretical connections between SSMs and variants of attention, connected through various decompositions of a well-studied class of structured semiseparable matrices. Our state space duality (SSD) framework allows us to design a new architecture (Mamba-2) whose core layer is an a refinement of Mamba's selective SSM that is 2-8× faster, while continuing to be competitive with Transformers on language modeling.

**Section outline** (from the arXiv LaTeX source):

1. Introduction
2. Background and Overview
3. State Space Models are Structured Matrices
4. Structured Masked Attention: Generalizing Linear Attention
5. State Space Duality
6. A Hardware-Efficient Algorithm for SSD Models
7. The Mamba-2 Architecture
8. Systems Optimization for SSMs
9. Empirical Validation
10. Related Work and Discussion
11. Conclusion
12. Glossary
13. Efficient Algorithms for the Scalar SSM Scan (1-SS Multiplication)
14. Theory Details
15. Experimental Details

**Annotation:** This paper unifies the two camps rather than opposing them: it shows that structured state-space models and several attention variants are both special cases of structured *semiseparable* matrices, each reachable via a different decomposition. The resulting "state space duality" theory yields Mamba-2, whose SSD layer admits both a linear recurrent form and a dual quadratic form that can reuse mature attention/FlashAttention-style kernels. Conceptually this is the best bridge paper on the list — it explains *why* the SSM-versus-attention debate is less binary than it looks.

---

### 5. Gated Delta Networks: Improving Mamba2 with Delta Rule

- **arXiv ID:** [2412.06464](https://arxiv.org/abs/2412.06464)
- **Primary category:** Computation and Language (cs.CL)
- **Current version:** v3 — last revised 6 Mar 2025
- **Venue note (arXiv Comments field):** "ICLR 2025 camera ready"

**Abstract** (from the arXiv LaTeX source):

> Linear Transformers have gained attention as efficient alternatives to standard Transformers, but their performance in retrieval and long-context tasks has been limited. To address these limitations, recent work has explored two distinct mechanisms: gating for adaptive memory control and the delta update rule for precise memory modifications. We observe that these mechanisms are complementary—gating enables rapid memory erasure while the delta rule facilitates targeted updates. Building on this insight, we introduce the gated delta rule and develop a parallel training algorithm optimized for modern hardware. Our proposed architecture, Gated DeltaNet, consistently surpasses existing models like Mamba2 and DeltaNet across multiple benchmarks, including language modeling, common-sense reasoning, in-context retrieval, length extrapolation, and long-context understanding. We further enhance performance by developing hybrid architectures that combine Gated DeltaNet layers with sliding window attention or Mamba2 layers, achieving both improved training efficiency and superior task performance.
>
> Code: https://github.com/NVlabs/GatedDeltaNet

**Section outline** (from the arXiv LaTeX source; the appendix headings are reproduced as they appear in the source, including the typo "Contunued"):

1. Introduction
2. Preliminary
3. Gated Delta Networks
4. Experiments
5. Related Work
6. Conclusion
7. Acknowledgment
8. Appendix
9. Extended WY representation for gated delta rule
10. Experiment Contunued

**Annotation:** Gated DeltaNet combines two ideas that had previously been pursued separately in linear attention — gating (fast, coarse memory control) and the delta rule (precise, targeted memory updates) — into a single gated-delta rule with a chunk-parallel training algorithm that runs well on modern hardware. It beats both Mamba-2 and DeltaNet on retrieval, length extrapolation and long-context understanding, which is exactly where linear models had historically been weakest. The hybrid variants that mix in sliding-window attention or Mamba-2 layers are a good snapshot of where the field has converged in practice.

---

### 6. Native Sparse Attention: Hardware-Aligned and Natively Trainable Sparse Attention

- **arXiv ID:** [2502.11089](https://arxiv.org/abs/2502.11089)
- **Primary category:** Computation and Language (cs.CL)
- **Current version:** v2 — last revised 27 Feb 2025
- **Venue note:** the arXiv page lists no Comments field, so no venue note is shown there.

**Abstract** (from the arXiv LaTeX source; the source defines a `\method` macro that expands to *NSA*, resolved below):

> Long-context modeling is crucial for next-generation language models, yet the high computational cost of standard attention mechanisms poses significant computational challenges. Sparse attention offers a promising direction for improving efficiency while maintaining model capabilities. We present NSA, a Natively trainable Sparse Attention mechanism that integrates algorithmic innovations with hardware-aligned optimizations to achieve efficient long-context modeling. NSA employs a dynamic hierarchical sparse strategy, combining coarse-grained token compression with fine-grained token selection to preserve both global context awareness and local precision. Our approach advances sparse attention design with two key innovations: (1) We achieve substantial speedups through arithmetic intensity-balanced algorithm design, with implementation optimizations for modern hardware. (2) We enable end-to-end training, reducing pretraining computation without sacrificing model performance. As shown in Figure 1, experiments show the model pretrained with NSA maintains or exceeds Full Attention models across general benchmarks, long-context tasks, and instruction-based reasoning. Meanwhile, NSA achieves substantial speedups over Full Attention on 64k-length sequences across decoding, forward propagation, and backward propagation, validating its efficiency throughout the model lifecycle.

**Section outline** (from the arXiv LaTeX source):

1. Introduction
2. Rethinking Sparse Attention Methods
3. Methodology
4. Experiments
5. Efficiency Analysis
6. Discussion
7. Related Works
8. Conclusion
9. Example of AIME Results

**Annotation:** NSA attacks long-context cost from inside attention rather than by replacing it: a hierarchical sparse strategy that fuses compressed (coarse) tokens with selected (fine-grained) tokens, so global coverage and local precision are both preserved. Two things distinguish it from most prior sparse-attention work — it is designed to be *arithmetic-intensity balanced* for real hardware, and it is *natively trainable*, so the model learns the sparsity during pretraining instead of having it bolted on afterwards. On 64k-token sequences it reports substantial speedups over full attention across decoding, forward and backward passes while matching or exceeding full-attention quality.

---

### 7. LongRoPE2: Near-Lossless LLM Context Window Scaling

- **arXiv ID:** [2502.20082](https://arxiv.org/abs/2502.20082)
- **Primary category:** Computation and Language (cs.CL)
- **Current version:** v1 — submitted 27 Feb 2025 (only version)
- **Venue note:** the arXiv page lists no Comments field, so no venue note is shown there.

**Abstract** (from the arXiv LaTeX source; the source defines a `\sysname` macro that expands to *LongRoPE2*, resolved below):

> LongRoPE2 is a novel approach that extends the effective context window of pre-trained large language models (LLMs) to the target length, while preserving the performance on the original shorter context window. This is achieved by three contributions: (1) a hypothesis that insufficient training in higher RoPE dimensions contributes to the persistent out-of-distribution (OOD) issues observed in existing methods; (2) an effective RoPE rescaling algorithm that adopts evolutionary search guided by "needle-driven" perplexity to address the insufficient training problem; (3) a mixed context window training approach that fine-tunes model weights to adopt rescaled RoPE for long-context sequences while preserving the short-context performance with the original RoPE. Extensive experiments on LLaMA3-8B and Phi3-mini-3.8B across various benchmarks validate the hypothesis and demonstrate the effectiveness of LongRoPE2. Remarkably, LongRoPE2 extends LLaMA3-8B to achieve a 128K effective context length while retaining over 98.5% of short-context performance, using only 10B tokens -- 80x fewer than Meta's approach, which fails to reach the target effective context length. Code will be available at https://github.com/microsoft/LongRoPE.

**Section outline** (from the arXiv LaTeX source):

1. Introduction
2. Context Window Extension and Challenges
3. LongRoPE2 Methodology
4. Experiments
5. Conclusion
6. Acknowledgement
7. Impact Statement
8. Related Works
9. Additional Experiments and Analysis
10. Synthetic data sample

**Annotation:** LongRoPE2 is the position-encoding side of long-context modeling: it diagnoses why RoPE-based context extension keeps hitting out-of-distribution failures (insufficient training in the higher RoPE dimensions, whose rotation periods exceed the pretrained window) and fixes the diagnosis rather than just the symptom. Its three-part recipe — an evolutionary search over per-dimension rescaling factors guided by "needle-driven" perplexity, plus mixed context-window training — pushes LLaMA3-8B to a 128K *effective* context while retaining over 98.5% of short-context performance on 80× fewer tokens than the original recipe. Read this together with FlashAttention: one paper removes the compute barrier to long context, this one removes the positional barrier.

---

## Suggested reading order

1. **Attention Is All You Need** — the architecture, on its own terms.
2. **FlashAttention** — why the quadratic attention cost is a memory problem, and how exact attention got fast.
3. **LongRoPE2** — extending context via position encoding.
4. **Native Sparse Attention** — cutting attention cost from inside the mechanism.
5. **Mamba** — the strongest attention alternative.
6. **Transformers are SSMs** — the theory that unifies the two families.
7. **Gated Delta Networks** — where the hybrid line of work currently stands.

## Notes on sourcing

- Abstracts and section outlines were extracted from each paper's arXiv LaTeX source (`/src/<id>` on arXiv). Where a source defines a title macro (e.g. `\sysname`, `\method`), it has been resolved to the paper's name in the text above; the wording is otherwise unchanged.
- Categories, versions and venue notes were taken from the official arXiv abstract pages (`https://arxiv.org/abs/<id>`) and are reported as shown there. "Primary category" is the first subject listed in the Subjects field.
- Where a paper's arXiv page shows no Comments field, no venue note is reported, since the page displays none.
