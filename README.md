# Transformer Reading List

A curated, verified reading list on the **Transformer architecture** for our lab reading group.

**How this list was built — no second-hand summaries.** For every entry below, the **abstract** and the **section outline** are taken from the paper's own **arXiv LaTeX source**, so what you see is the paper's real structure rather than a paraphrase. The **primary category**, **current preprint version**, and any **venue note** are quoted from the official **arXiv abstract page**, exactly as that page shows them. The only editorial content is the short "What it contributes" note under each paper. Where a LaTeX source writes a shorthand macro (`\sysname`, `\method`, `\mq`) or a cross-reference, it has been resolved to the term it denotes; abstracts are otherwise verbatim from the source (so an abstract can differ by a word or two from the rendered arXiv page).

**Scope.** The mandatory core paper, plus papers on three threads that grew directly out of it: **efficient attention variants** (IO-aware exact attention, shared KV heads), **long-context modeling** (sparse attention, block-level routing, positional scaling), and **attention alternatives** such as state-space models and gated delta-rule linear attention. Recent papers are preferred.

## Papers

| # | Title | arXiv ID | Primary category |
|---|-------|----------|------------------|
| 1 | Attention Is All You Need *(core)* | [1706.03762](https://arxiv.org/abs/1706.03762) | Computation and Language (cs.CL) |
| 2 | FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness | [2205.14135](https://arxiv.org/abs/2205.14135) | Machine Learning (cs.LG) |
| 3 | GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints | [2305.13245](https://arxiv.org/abs/2305.13245) | Computation and Language (cs.CL) |
| 4 | Native Sparse Attention: Hardware-Aligned and Natively Trainable Sparse Attention | [2502.11089](https://arxiv.org/abs/2502.11089) | Computation and Language (cs.CL) |
| 5 | MoBA: Mixture of Block Attention for Long-Context LLMs | [2502.13189](https://arxiv.org/abs/2502.13189) | Machine Learning (cs.LG) |
| 6 | Optimizing Mixture of Block Attention | [2511.11571](https://arxiv.org/abs/2511.11571) | Machine Learning (cs.LG) |
| 7 | LongRoPE2: Near-Lossless LLM Context Window Scaling | [2502.20082](https://arxiv.org/abs/2502.20082) | Computation and Language (cs.CL) |
| 8 | Mamba: Linear-Time Sequence Modeling with Selective State Spaces | [2312.00752](https://arxiv.org/abs/2312.00752) | Machine Learning (cs.LG) |
| 9 | Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality | [2405.21060](https://arxiv.org/abs/2405.21060) | Machine Learning (cs.LG) |
| 10 | Gated Delta Networks: Improving Mamba2 with Delta Rule | [2412.06464](https://arxiv.org/abs/2412.06464) | Computation and Language (cs.CL) |

**Suggested reading order.** Start with 1 (the architecture itself), then 2-3 (making attention cheaper: an IO-aware exact kernel, then shared KV heads for decoding), then 4-7 (long context: natively trainable sparsity, block-level routing and its optimization, and positional scaling), then 8-10 (attention alternatives: selective state-space models, the SSM-attention duality, and gated delta-rule linear attention).

---

## 1. Attention Is All You Need — *core paper*

Vaswani, Shazeer, Parmar, Uszkoreit, Jones, Gomez, Kaiser, Polosukhin · [arXiv:1706.03762](https://arxiv.org/abs/1706.03762) · [PDF](https://arxiv.org/pdf/1706.03762)

- **Primary category (arXiv page):** Computation and Language (cs.CL) — the page's Subjects line reads: *Computation and Language (cs.CL); Machine Learning (cs.LG)*
- **Current version (arXiv page):** v7 — *"[Submitted on 12 Jun 2017 (v1), last revised 2 Aug 2023 (this version, v7)]"*
- **Venue note (arXiv page):** the Comments field reads *"15 pages, 5 figures"*; no venue is listed on the page

**Abstract** *(from the arXiv LaTeX source)*

> The dominant sequence transduction models are based on complex recurrent or convolutional neural networks that include an encoder and a decoder. The best performing models also connect the encoder and decoder through an attention mechanism. We propose a new simple network architecture, the Transformer, based solely on attention mechanisms, dispensing with recurrence and convolutions entirely. Experiments on two machine translation tasks show these models to be superior in quality while being more parallelizable and requiring significantly less time to train. Our model achieves 28.4 BLEU on the WMT 2014 English-to-German translation task, improving over the existing best results, including ensembles, by over 2 BLEU. On the WMT 2014 English-to-French translation task, our model establishes a new single-model state-of-the-art BLEU score of 41.8 after training for 3.5 days on eight GPUs, a small fraction of the training costs of the best models from the literature. We show that the Transformer generalizes well to other tasks by applying it successfully to English constituency parsing both with large and limited training data.

**Section outline** *(from the arXiv LaTeX source, in order)*

1. Introduction
2. Background
3. Model Architecture
4. Why Self-Attention
5. Training
6. Results
7. Conclusion
8. Attention Visualizations

**What it contributes.** The founding paper of the architecture: it dispenses with recurrence and convolutions entirely, building the encoder-decoder stack out of scaled dot-product attention, multi-head attention, and position-wise feed-forward layers with positional encodings. It set a new state of the art on WMT 2014 machine translation (28.4 BLEU En-De, 41.8 BLEU En-Fr) at a small fraction of previous training cost, and Section 3 ("Model Architecture") defines the component vocabulary that every other paper in this list optimizes, sparsifies, or replaces. Section 4 ("Why Self-Attention") is the cleanest statement of the design's core trade-off: constant path length between any two positions in exchange for massively more parallelizable computation.

---

## 2. FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness

Dao, Fu, Ermon, Rudra, Ré · [arXiv:2205.14135](https://arxiv.org/abs/2205.14135) · [PDF](https://arxiv.org/pdf/2205.14135)

- **Primary category (arXiv page):** Machine Learning (cs.LG) — the page's Subjects line reads: *Machine Learning (cs.LG)*
- **Current version (arXiv page):** v2 — *"[Submitted on 27 May 2022 (v1), last revised 23 Jun 2022 (this version, v2)]"*
- **Venue note (arXiv page):** none — the page shows no Comments field and no journal reference

**Abstract** *(from the arXiv LaTeX source; the source's `\sysname` macro denotes FlashAttention)*

> Transformers are slow and memory-hungry on long sequences, since the time and memory complexity of self-attention are quadratic in sequence length. Approximate attention methods have attempted to address this problem by trading off model quality to reduce the compute complexity, but often do not achieve wall-clock speedup. We argue that a missing principle is making attention algorithms *IO-aware*—accounting for reads and writes between levels of GPU memory. We propose FlashAttention, an IO-aware exact attention algorithm that uses tiling to reduce the number of memory reads/writes between GPU high bandwidth memory (HBM) and GPU on-chip SRAM. We analyze the IO complexity of FlashAttention, showing that it requires fewer HBM accesses than standard attention, and is optimal for a range of SRAM sizes. We also extend FlashAttention to block-sparse attention, yielding an approximate attention algorithm that is faster than any existing approximate attention method. FlashAttention trains Transformers faster than existing baselines: 15% end-to-end wall-clock speedup on BERT-large (seq. length 512) compared to the MLPerf 1.1 training speed record, 3× speedup on GPT-2 (seq. length 1K), and 2.4× speedup on long-range arena (seq. length 1K-4K). FlashAttention and block-sparse FlashAttention enable longer context in Transformers, yielding higher quality models (0.7 better perplexity on GPT-2 and 6.4 points of lift on long-document classification) and entirely new capabilities: the first Transformers to achieve better-than-chance performance on the Path-X challenge (seq. length 16K, 61.4% accuracy) and Path-256 (seq. length 64K, 63.1% accuracy).

**Section outline** *(from the arXiv LaTeX source, in order; `\sysname` = FlashAttention)*

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

**What it contributes.** The key systems-level answer to the Transformer's quadratic cost: instead of approximating attention, it makes *exact* attention fast by tiling the computation so the N×N attention matrix is never materialized in GPU HBM, keeping intermediate results in on-chip SRAM with online softmax updates and backward-pass recomputation. It proves the resulting IO complexity is optimal for a range of SRAM sizes and extends the idea to block-sparse attention, beating prior approximate methods. This is the work that made long-context training practical on GPUs, and it is now the default attention implementation in most modern stacks.

---

## 3. GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints

Ainslie, Lee-Thorp, de Jong, Zemlyanskiy, Lebrón, Sanghai · [arXiv:2305.13245](https://arxiv.org/abs/2305.13245) · [PDF](https://arxiv.org/pdf/2305.13245)

- **Primary category (arXiv page):** Computation and Language (cs.CL) — the page's Subjects line reads: *Computation and Language (cs.CL); Machine Learning (cs.LG)*
- **Current version (arXiv page):** v3 — *"[Submitted on 22 May 2023 (v1), last revised 23 Dec 2023 (this version, v3)]"*
- **Venue note (arXiv page):** Comments: *"Accepted at EMNLP 2023. Added to related work"*

**Abstract** *(from the arXiv LaTeX source; the source's `\mq` macro denotes MQA and `\gmq`/`\fullgmq` denote grouped-query attention)*

> Multi-query attention (MQA), which only uses a single key-value head, drastically speeds up decoder inference. However, MQA can lead to quality degradation, and moreover it may not be desirable to train a separate model just for faster inference. We (1) propose a recipe for uptraining existing multi-head language model checkpoints into models with MQA using 5% of original pre-training compute, and (2) introduce grouped-query attention (GQA), a generalization of multi-query attention which uses an intermediate (more than one, less than number of query heads) number of key-value heads. We show that uptrained GQA achieves quality close to multi-head attention with comparable speed to MQA.

**Section outline** *(from the arXiv LaTeX source, in order; headings transcribed verbatim, including the source's spelling of "Acknowlegements")*

1. Introduction
2. Method
3. Experiments
4. Related Work
5. Conclusion
6. Limitations
7. Acknowlegements
8. Training Stability

**What it contributes.** Targets the KV-cache memory and bandwidth bottleneck in autoregressive decoding: grouped-query attention shares key-value heads across groups of query heads, interpolating between multi-head attention (quality) and multi-query attention (speed). The companion contribution is a practical uptraining recipe that converts an existing multi-head checkpoint into a GQA/MQA model using about 5% of the original pretraining compute, so fast inference no longer requires training a separate model. GQA is now a standard component of production LLMs.

---

## 4. Native Sparse Attention: Hardware-Aligned and Natively Trainable Sparse Attention

Yuan, Gao, Dai, Luo, Zhao, Zhang, Xie, Wei, Wang, Xiao, Wang, Ruan, Zhang, Liang, Zeng · [arXiv:2502.11089](https://arxiv.org/abs/2502.11089) · [PDF](https://arxiv.org/pdf/2502.11089)

- **Primary category (arXiv page):** Computation and Language (cs.CL) — the page's Subjects line reads: *Computation and Language (cs.CL); Artificial Intelligence (cs.AI); Machine Learning (cs.LG)*
- **Current version (arXiv page):** v2 — *"[Submitted on 16 Feb 2025 (v1), last revised 27 Feb 2025 (this version, v2)]"*
- **Venue note (arXiv page):** none — the page shows no Comments field and no journal reference

**Abstract** *(from the arXiv LaTeX source; the source's `\method` macro denotes NSA, and its cross-reference to Figure 1 is resolved)*

> Long-context modeling is crucial for next-generation language models, yet the high computational cost of standard attention mechanisms poses significant computational challenges. Sparse attention offers a promising direction for improving efficiency while maintaining model capabilities. We present NSA, a Natively trainable Sparse Attention mechanism that integrates algorithmic innovations with hardware-aligned optimizations to achieve efficient long-context modeling. NSA employs a dynamic hierarchical sparse strategy, combining coarse-grained token compression with fine-grained token selection to preserve both global context awareness and local precision. Our approach advances sparse attention design with two key innovations: (1) We achieve substantial speedups through arithmetic intensity-balanced algorithm design, with implementation optimizations for modern hardware. (2) We enable end-to-end training, reducing pretraining computation without sacrificing model performance. As shown in Figure 1, experiments show the model pretrained with NSA maintains or exceeds Full Attention models across general benchmarks, long-context tasks, and instruction-based reasoning. Meanwhile, NSA achieves substantial speedups over Full Attention on 64k-length sequences across decoding, forward propagation, and backward propagation, validating its efficiency throughout the model lifecycle.

**Section outline** *(from the arXiv LaTeX source, in order)*

1. Introduction
2. Rethinking Sparse Attention Methods
3. Methodology
4. Experiments
5. Efficiency Analysis
6. Discussion
7. Related Works
8. Conclusion
9. Example of AIME Results

**What it contributes.** A sparse attention mechanism that is *natively trainable* — trained end-to-end from scratch rather than retrofitted onto a full-attention model — using a hierarchical strategy that combines coarse-grained token compression with fine-grained token selection. Its distinguishing contribution is hardware alignment: an arithmetic-intensity-balanced algorithm design turns theoretical sparsity into measured speedups on 64k-token sequences across decoding, forward, and backward passes, while matching or exceeding full-attention baselines on general, long-context, and instruction-following benchmarks.

---

## 5. MoBA: Mixture of Block Attention for Long-Context LLMs

Lu, Jiang, Liu, Du, Jiang, Hong, Liu, He, Yuan, Wang, Huang, Yuan, Xu, Xu, Lai, Chen, Zheng, Yan, Su, Wu, Zhang, Yang, Zhou, Zhang, Qiu · [arXiv:2502.13189](https://arxiv.org/abs/2502.13189) · [PDF](https://arxiv.org/pdf/2502.13189)

- **Primary category (arXiv page):** Machine Learning (cs.LG) — the page's Subjects line reads: *Machine Learning (cs.LG); Artificial Intelligence (cs.AI); Computation and Language (cs.CL)*
- **Current version (arXiv page):** v1 — *"[Submitted on 18 Feb 2025]"*
- **Venue note (arXiv page):** Comments: *"15 pages"* — no venue is listed

**Abstract** *(from the arXiv LaTeX source; the source's trailing `\url` is resolved to the repository address)*

> Scaling the effective context length is essential for advancing large language models (LLMs) toward artificial general intelligence (AGI). However, the quadratic increase in computational complexity inherent in traditional attention mechanisms presents a prohibitive overhead. Existing approaches either impose strongly biased structures, such as sink or window attention which are task-specific, or radically modify the attention mechanism into linear approximations, whose performance in complex reasoning tasks remains inadequately explored.
>
> In this work, we propose a solution that adheres to the "less structure" principle, allowing the model to determine where to attend autonomously, rather than introducing predefined biases. We introduce Mixture of Block Attention (MoBA), an innovative approach that applies the principles of Mixture of Experts (MoE) to the attention mechanism. This novel architecture demonstrates superior performance on long-context tasks while offering a key advantage: the ability to seamlessly transition between full and sparse attention, enhancing efficiency without the risk of compromising performance. MoBA has already been deployed to support Kimi's long-context requests and demonstrates significant advancements in efficient attention computation for LLMs. Our code is available at https://github.com/MoonshotAI/MoBA.

**Section outline** *(from the arXiv LaTeX source, in order)*

1. Introduction
2. Method
3. Experiments
4. Related Work
5. Conclusion
6. Appendix

**What it contributes.** Imports mixture-of-experts routing into attention: each query attends to a top-k subset of key-value blocks selected by a lightweight gate, following a "less structure" principle in which the model itself decides where to attend instead of hard-coding sink or window patterns. A practically important property is that MoBA can switch seamlessly between sparse and full attention, and the paper reports that it is already deployed in production to serve Kimi's long-context requests.

---

## 6. Optimizing Mixture of Block Attention

Xiao, Guo, Mazaheri, Han · [arXiv:2511.11571](https://arxiv.org/abs/2511.11571) · [PDF](https://arxiv.org/pdf/2511.11571)

- **Primary category (arXiv page):** Machine Learning (cs.LG) — the page's Subjects line reads: *Machine Learning (cs.LG); Computation and Language (cs.CL)*
- **Current version (arXiv page):** v2 — *"[Submitted on 14 Nov 2025 (v1), last revised 19 Dec 2025 (this version, v2)]"*
- **Venue note (arXiv page):** Comments: *"The first two authors contributed equally to this work"* — no venue is listed

**Abstract** *(from the arXiv LaTeX source; the source's citation macro is resolved to "(Lu et al., 2025)")*

> Mixture of Block Attention (MoBA) (Lu et al., 2025) is a promising building block for efficiently processing long contexts in LLMs by enabling queries to sparsely attend to a small subset of key-value blocks, drastically reducing computational cost. However, the design principles governing MoBA's performance are poorly understood, and it lacks an efficient GPU implementation, hindering its practical adoption. In this paper, we first develop a statistical model to analyze MoBA's underlying mechanics. Our model reveals that performance critically depends on the router's ability to accurately distinguish relevant from irrelevant blocks based on query-key affinities. We derive a signal-to-noise ratio that formally connects architectural parameters to this retrieval accuracy. Guided by our analysis, we identify two key pathways for improvement: using smaller block sizes, and applying a short convolution on keys to cluster relevant signals, which enhances routing accuracy. While theoretically better, small block sizes are inefficient on GPUs. To bridge this gap, we introduce FlashMoBA, a hardware-aware CUDA kernel that enables efficient MoBA execution even with the small block sizes our theory recommends. We validate our insights by training LLMs from scratch, showing that our improved MoBA models match the performance of dense attention baselines. FlashMoBA achieves up to 14.7× speedup over FlashAttention-2 for small blocks, making our theoretically-grounded improvements practical. Code is available at: https://github.com/mit-han-lab/flash-moba.

**Section outline** *(from the arXiv LaTeX source, in order)*

1. Introduction
2. Preliminaries
3. A Statistical Model of MoBA
4. FlashMoBA: An Optimized Kernel for Small-Block MoBA
5. Experiments
6. Related Work
7. Conclusion
8. Reproducibility Statement.
9. LLM Usage Statement
10. Detailed Derivation of Signal-to-Noise Ratio
11. Key Convolution Design
12. Kernel Implementation Details

**What it contributes.** The follow-up that explains *why* MoBA works: a statistical model of block routing and a derived signal-to-noise ratio that formally connect block size and routing accuracy, pointing to two concrete improvements — smaller blocks and a short key-side convolution. Because small blocks are GPU-inefficient, the authors build FlashMoBA, a custom CUDA kernel that makes the theoretically preferred regime practical, matching dense-attention baselines at up to 14.7× speedup over FlashAttention-2.

---

## 7. LongRoPE2: Near-Lossless LLM Context Window Scaling

Shang, Zhang, Wang, Zhang, Lopez, Yang, Chen, Yang · [arXiv:2502.20082](https://arxiv.org/abs/2502.20082) · [PDF](https://arxiv.org/pdf/2502.20082)

- **Primary category (arXiv page):** Computation and Language (cs.CL) — the page's Subjects line reads: *Computation and Language (cs.CL)*
- **Current version (arXiv page):** v1 — *"[Submitted on 27 Feb 2025]"*
- **Venue note (arXiv page):** none — the page shows no Comments field and no journal reference

**Abstract** *(from the arXiv LaTeX source; the source's `\sysname` macro denotes LongRoPE2)*

> LongRoPE2 is a novel approach that extends the effective context window of pre-trained large language models (LLMs) to the target length, while preserving the performance on the original shorter context window. This is achieved by three contributions: (1) a hypothesis that insufficient training in higher RoPE dimensions contributes to the persistent out-of-distribution (OOD) issues observed in existing methods; (2) an effective RoPE rescaling algorithm that adopts evolutionary search guided by "needle-driven" perplexity to address the insufficient training problem; (3) a mixed context window training approach that fine-tunes model weights to adopt rescaled RoPE for long-context sequences while preserving the short-context performance with the original RoPE. Extensive experiments on LLaMA3-8B and Phi3-mini-3.8B across various benchmarks validate the hypothesis and demonstrate the effectiveness of LongRoPE2. Remarkably, LongRoPE2 extends LLaMA3-8B to achieve a 128K effective context length while retaining over 98.5% of short-context performance, using only 10B tokens -- 80x fewer than Meta's approach, which fails to reach the target effective context length. Code will be available at https://github.com/microsoft/LongRoPE.

**Section outline** *(from the arXiv LaTeX source, in order; the source's `\sysname` macro in the third heading is resolved)*

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

**What it contributes.** A complementary, non-architectural route to long context: better positional-embedding scaling. It hypothesizes that persistent out-of-distribution behavior comes from undertraining in the higher RoPE dimensions, then finds rescaling factors via evolutionary search guided by "needle-driven" perplexity, and preserves short-context performance with mixed context window training. The result is a 128K effective context for LLaMA3-8B while retaining over 98.5% of short-context performance, using 10B tokens — 80× fewer than Meta's own approach.

---

## 8. Mamba: Linear-Time Sequence Modeling with Selective State Spaces

Gu, Dao · [arXiv:2312.00752](https://arxiv.org/abs/2312.00752) · [PDF](https://arxiv.org/pdf/2312.00752)

- **Primary category (arXiv page):** Machine Learning (cs.LG) — the page's Subjects line reads: *Machine Learning (cs.LG); Artificial Intelligence (cs.AI)*
- **Current version (arXiv page):** v2 — *"[Submitted on 1 Dec 2023 (v1), last revised 31 May 2024 (this version, v2)]"*
- **Venue note (arXiv page):** none — the page shows no Comments field and no journal reference

**Abstract** *(from the arXiv LaTeX source)*

> Foundation models, now powering most of the exciting applications in deep learning, are almost universally based on the Transformer architecture and its core attention module. Many subquadratic-time architectures such as linear attention, gated convolution and recurrent models, and structured state space models (SSMs) have been developed to address Transformers' computational inefficiency on long sequences, but they have not performed as well as attention on important modalities such as language. We identify that a key weakness of such models is their inability to perform content-based reasoning, and make several improvements. First, simply letting the SSM parameters be functions of the input addresses their weakness with discrete modalities, allowing the model to *selectively* propagate or forget information along the sequence length dimension depending on the current token. Second, even though this change prevents the use of efficient convolutions, we design a hardware-aware parallel algorithm in recurrent mode. We integrate these selective SSMs into a simplified end-to-end neural network architecture without attention or even MLP blocks (**Mamba**). Mamba enjoys fast inference (5× higher throughput than Transformers) and linear scaling in sequence length, and its performance improves on real data up to million-length sequences. As a general sequence model backbone, Mamba achieves state-of-the-art performance across several modalities such as language, audio, and genomics. On language modeling, our Mamba-3B model outperforms Transformers of the same size and matches Transformers twice its size, both in pretraining and downstream evaluation.

**Section outline** *(from the arXiv LaTeX source, in order; items 7-12 are appendix sections)*

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

**What it contributes.** The flagship attention alternative of the modern literature: it makes structured state-space models *input-dependent* ("selective"), restoring the content-based reasoning that earlier subquadratic models lacked, and designs a hardware-aware parallel-scan algorithm so training stays efficient despite losing convolutions. The resulting architecture — with no attention or even MLP blocks — reaches 5× Transformer inference throughput, scales linearly to million-length sequences, and matches Transformers twice its size on language modeling. Reading it directly against paper 1 makes the attention-versus-recurrent-state trade-off concrete.

---

## 9. Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality

Dao, Gu · [arXiv:2405.21060](https://arxiv.org/abs/2405.21060) · [PDF](https://arxiv.org/pdf/2405.21060)

- **Primary category (arXiv page):** Machine Learning (cs.LG) — the page's Subjects line reads: *Machine Learning (cs.LG)*
- **Current version (arXiv page):** v1 — *"[Submitted on 31 May 2024]"*
- **Venue note (arXiv page):** Comments: *"ICML 2024"*

**Abstract** *(from the arXiv LaTeX source; the phrase "is an a refinement" is transcribed verbatim from the source)*

> While Transformers have been the main architecture behind deep learning's success in language modeling, state-space models (SSMs) such as Mamba have recently been shown to match or outperform Transformers at small to medium scale. We show that these families of models are actually quite closely related, and develop a rich framework of theoretical connections between SSMs and variants of attention, connected through various decompositions of a well-studied class of structured semiseparable matrices. Our state space duality (SSD) framework allows us to design a new architecture (Mamba-2) whose core layer is an a refinement of Mamba's selective SSM that is 2-8× faster, while continuing to be competitive with Transformers on language modeling.

**Section outline** *(from the arXiv LaTeX source, in order; items 12-15 are appendix sections. The fourth heading is written in the source with a `\texorpdfstring` construction whose rendered form is given here)*

1. Introduction
2. Background and Overview
3. State Space Models are Structured Matrices
4. Structured Masked Attention: Generalizing Linear Attention with Structured Matrices
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

**What it contributes.** Shows that state-space models and variants of attention are two faces of the same object — structured *semiseparable matrices* — and formalizes the connection as state space duality (SSD): the linear-time SSM recurrence and the quadratic-time masked kernel-attention form are dual algorithms for the same computation. This yields Mamba-2, a refinement of Mamba's selective SSM that is 2-8× faster and can import Transformer-style systems optimizations such as tensor parallelism. It is the best single bridge between the attention and SSM literatures, and pairs naturally with paper 8.

---

## 10. Gated Delta Networks: Improving Mamba2 with Delta Rule

Yang, Kautz, Hatamizadeh · [arXiv:2412.06464](https://arxiv.org/abs/2412.06464) · [PDF](https://arxiv.org/pdf/2412.06464)

- **Primary category (arXiv page):** Computation and Language (cs.CL) — the page's Subjects line reads: *Computation and Language (cs.CL); Machine Learning (cs.LG)*
- **Current version (arXiv page):** v3 — *"[Submitted on 9 Dec 2024 (v1), last revised 6 Mar 2025 (this version, v3)]"*
- **Venue note (arXiv page):** Comments: *"ICLR 2025 camera ready"*

**Abstract** *(from the arXiv LaTeX source; the source's trailing `\url` is resolved to the repository address)*

> Linear Transformers have gained attention as efficient alternatives to standard Transformers, but their performance in retrieval and long-context tasks has been limited. To address these limitations, recent work has explored two distinct mechanisms: gating for adaptive memory control and the delta update rule for precise memory modifications. We observe that these mechanisms are complementary—gating enables rapid memory erasure while the delta rule facilitates targeted updates. Building on this insight, we introduce the gated delta rule and develop a parallel training algorithm optimized for modern hardware. Our proposed architecture, Gated DeltaNet, consistently surpasses existing models like Mamba2 and DeltaNet across multiple benchmarks, including language modeling, common-sense reasoning, in-context retrieval, length extrapolation, and long-context understanding. We further enhance performance by developing hybrid architectures that combine Gated DeltaNet layers with sliding window attention or Mamba2 layers, achieving both improved training efficiency and superior task performance. Code: https://github.com/NVlabs/GatedDeltaNet

**Section outline** *(from the arXiv LaTeX source, in order; headings transcribed verbatim, including the source's spelling of "Experiment Contunued")*

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

**What it contributes.** The linear-attention branch of the attention-alternative thread: it unifies two previously separate mechanisms — gating for rapid memory erasure and the delta rule for precise, targeted memory modification — into a single gated delta rule, with a parallel training algorithm that runs efficiently on modern hardware. Gated DeltaNet outperforms Mamba2 and DeltaNet on language modeling, in-context retrieval, and length extrapolation, and the paper shows that hybrid architectures mixing these layers with sliding-window attention or Mamba2 layers perform best of all. It is a good lens on where the field has converged: fast recurrent state plus a little attention.

---

*Verification note: every abstract and section outline above was transcribed from the paper's arXiv LaTeX source; every category, version and venue note was read off the paper's official arXiv abstract page at the time of writing.*
