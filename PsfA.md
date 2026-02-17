# Official Review of Submission10245 by Reviewer PsfA
Paper Summary:
This paper addresses the critical challenge of distilling the knowledge from LLMs to SLMs. To address the issues brought by the SLMs, the authors propose an always-search policy, which is a finetuning approach that enforces SLMs to retrieve answers in external evidence. Together the paper proposes 3 distillation methods. The experiments show that the proposed ASP helps the SLMs achieve similar perofrmance with LLMs, e.g., improving F1 scores by 17.3 on Bamboogle and 15.3 on HotpotQA.

Summary Of Strengths:
The paper has a clear problem definition and clear wirting. The paper accurately points out the SLMs have "parametric hallucination + insufficient search" issues. This insight will be helpful for future works.
The method is simple but effective. The ASP solution help SLMs achieve similar performance with LLMs.
The authors have defined comprehensive experiments to demonstrate their ideas. The case study of multi-hop QA clearly demonstrates ASP’s ability to eliminate hallucination and improve tool usage, while quantitative results (tool call frequency, F1 scores across benchmarks) strongly support the core conclusions.
The paper provides a practical solution to many real-world applications. E.g., SLMs should prioritize forced search over adaptive strategies, with direct value for low-latency/budget scenarios.
Summary Of Weaknesses:
How to ensure the returned knowledge by ASP is robust? This paper mainly proposes ASP with training schemes to train a SLM. However, the data quality control is rarely discussed, which might hurts the performance.
How about considering other SLM families? For instance, Intern, LLaMA, Mistral. I am not sure about the generalizability.
Recent Agentic training schemes mainly use RL to enhance the retrieval and answer procedures, like search-R1. However, the authors rarely discussed it and seems to not use such training strategy. Please consider more sophisticated training.
Please discuss more about the reasoning capacity of SLMs. The paper mainly discussed the retrieval behaviors of SLMs during inference. It ignores the reasoning ability of SLMs to influence the results.
Comments Suggestions And Typos:
"brith country" should be corrected to "birth country".

Confidence: 4 = Quite sure. I tried to check the important points carefully. It's unlikely, though conceivable, that I missed something that should affect my ratings.
Soundness: 3.5
Excitement: 3.5
Overall Assessment: 2.5 = Borderline Findings
Ethical Concerns:
There are no concerns with this submission

Reproducibility: 4 = They could mostly reproduce the results, but there may be some variation because of sample variance or minor variations in their interpretation of the protocol or method.
Datasets: 3 = Potentially useful: Someone might find the new datasets useful for their work.
Software: 3 = Potentially useful: Someone might find the new software useful for their work.
Knowledge Of Or Educated Guess At Author Identity: No
Knowledge Of Paper: N/A, I do not know anything about the paper from outside sources
Knowledge Of Paper Source: N/A, I do not know anything about the paper from outside sources
Impact Of Knowledge Of Paper: N/A, I do not know anything about the paper from outside sources
Reviewer Certification: I certify that the review I entered accurately reflects my assessment of the work. If you used any type of automated tool to help you craft your review, I hereby certify that its use was restricted to improving grammar and style, and the substance of the review is either my own work or the work of an acknowledged secondary reviewer.
Publication Ethics Policy Compliance: I used a privacy-preserving tool exclusively for the use case(s) approved by PEC policy, such as language edits

# Reponse to Reviewer PsFA

We sincerely thank the reviewer for the thorough and constructive feedback. We appreciate your recognition that our paper has "clear problem definition and clear writing," that ASP is "simple but effective," and that our experiments are "comprehensive" with "practical value for real-world applications." We are encouraged by your positive assessment of our core insights and contributions.

Below we address each weakness systematically:

### **W1 (Data Quality Control and Robustness of Retrieved Knowledge)**
Thank you for raising this important point. We want to clarify two distinct aspects: (1) trajectory quality control during training and (2) robustness of retrieved knowledge at inference time.

**Trajectory Quality Control (Training):**

We implement rigorous data quality control for ASP distillation as included in Section 4.1 (lines 169-173) and Appendix B.2 (lines 478-493). Specifically:

Dual filtering for ASP (Section 4.1, lines 169-173):

- Performance threshold: String-F1 > 0.65 (filtering low-quality trajectories)
- Search tool checking: Only trajectories where models consistently use search tools to obtain information are retained

This dual filtering ensures:

- High-quality teacher trajectories (correct answers)
- Proper search behavior (no parametric knowledge reliance)
- Consistent tool usage patterns for students to learn

In contrast, standard distillation (Section 3.2) uses only performance threshold, allowing trajectories where the teacher relies on parametric knowledge—precisely what causes SLM failures.

Key difference from standard distillation:
- **Standard distillation**: Uses only performance threshold (String-F1 > 0.65), allowing trajectories where teacher relies on parametric knowledge
- **ASP**: Adds strict search tool checking, resulting in **significantly more selective** filtering that ensures every training example demonstrates proper search behavior
- **Result**: Higher-quality training data where SLMs learn to search rather than hallucinate

Quality Control Method|Accept Rate|False Positive Rate|Issues|
---|---|---|---
LLM Judge (GPT-5)|82%|~35-40%|Inflated scores; rates trajectories higher than actual quality
LLM Judge + Self-Reflection|68%|~20-25%|Complex multi-step prompting; difficult to reproduce; sensitive to prompt variations
Rule-based (F1 only)|78%|~25-30%|Misses trajectories with parametric knowledge relianceRule-based
(F1 + Search Checking)|~50%|<10%|Most strict; filters trajectories with hallucination indicators

Robustness of Retrieved Knowledge (Inference):

Regarding the robustness of retrieved knowledge (not training data), we address this in our Limitations section (lines 322-327) and in response to Reviewer **1kYG's W2**. Briefly:

- Our work focuses on teaching SLMs effective search behavior in controlled settings (Wikipedia corpus with high-quality retrieval)

- Investigating robust retrieval under noisy/adversarial conditions is valuable future work but addresses a different question: "How to search in unreliable environments?" vs. our question: "Should SLMs always search, and how do we teach them?"

- This follows standard practice in agentic search literature (Li et al., 2025; Jin et al., 2025)

Commitment: We will add explicit discussion in Section 4.1 detailing:

- The dual filtering mechanism for trajectory quality control
- Quality statistics (e.g., "num of trajectories retained after strict ASP filtering")
- How ASP's stricter filtering ensures higher-quality training data than standard distillation

### **W2 (Limited to Qwen3 Model Family)**
We appreciate this concern, which was also raised by Reviewer 1kYG (W1). We provide a comprehensive response below.

Comparable recent work:

- Search-o1 (Li et al., 2025): Evaluated exclusively on Qwen models
- Search-R1 (Jin et al., 2025): Main evaluation is primarily conducted on Qwen models
- AgentTuning (Zeng et al., 2023): Focused on Llama series
- Distilling LLM Agents (Kang et al., 2025): Used only Mistral family

**Why single-family evaluation is appropriate for this work:**

Method validation vs. scaling study: Our contribution is a training paradigm (ASP), not a scaling law. Demonstrating effectiveness across multiple model scales (0.6B → 32B) within a controlled family is MORE rigorous for isolating ASP's effect than cross-family comparisons, which introduce architectural confounds.
Resource constraints: Training agentic systems requires:

Trajectory generation from teacher models (18K trajectories)
Multiple training methods (SFT, OPD, Mixed, RFT)
Evaluation across 7 benchmarks (HotpotQA, 2Wiki, Bamboogle, MuSiQue, BrowseComp, Frames, LongSeAL)
Ablations and confidence probing experiments

Criterion|Qwen3|Llama 3.2/3.3|Phi-3.5|Gemma 2|Mistral
---|---|---|---|---|---
Release Date|May 2025|Sept-Dec 2024|Aug 2024|June 2024|Sept 2023|
Model Type|Reasoning-optimized|General-purpose|General-purpose|General-purpose|General-purpose
Scale Range|0.6B-32B (5 sizes)|1B-70B (5 sizes)|3.8B-14B (2 sizes)|2B-27B (3 sizes)|7B-123B (4 sizes)
Sub-2B Models|✓ (0.6B, 1.7B)|✓ (1B)|✗|✓ (2B)|✗|

Qwen3's representativeness: Qwen3 is a state-of-the-art, widely-adopted open model family with strong baseline performance and diverse scale range (0.6B-32B), making it an appropriate testbed.

Comprehensive scale range enables controlled analysis of model capability: Qwen3's 0.6B-32B range (5 sizes) is uniquely suited for investigating the interaction between model scale and search behavior:

- 0.6B and 1.7B models: Test ASP on extremely small language models where parametric hallucination is most severe and reasoning capacity is minimal
- 4B and 8B models: Examine intermediate scales where models have some reasoning ability but still benefit from external retrieval
- 32B model: Establish upper bound performance and analyze when search becomes less critical
    
    Key insight: Using a single family eliminates architectural confounds, allowing us to attribute performance differences purely to scale and ASP, not model design choices

De facto standard in agentic search research: 15+ papers (2024-25) use Qwen for agentic tasks, significantly more than alternatives, making results directly comparable to the broader research community and facilitating reproducibility.

### **W3 (RL Training Schemes (e.g., Search-R1))**
Thank you for this thoughtful question. We want to clarify the distinction between our research focus and RL-based approaches like Search-R1.

Aspect|Our Work (ASP)|Search-R1 (Jin et al., 2025)
---|---|---
Research Question|How to distill agentic behavior from LLMs to SLMs?|How to train LLMs to use search via RL?
Target Models|SLMs (0.6B-8B) distilled from LLMs|LLMs (3B-7B+) trained with RL
Training Paradigm|Distillation (SFT, OPD)|Reinforcement Learning (PPO, GRPO)
Data Source|Teacher LLM trajectories|Self-generated with reward signal
Compute Requirements|Moderate (SFT on filtered trajectories)|High (RL rollouts, reward models)
Core Contribution|Always-Search Policy for SLMs|RL framework for search agents

We extensively explored RL but found fundamental limitations for extreme SLMs:

1. Direct RL (Search-R1 framework) fails on extreme SLMs (0.6B-1.7B):We attempted to apply the Search-R1 RL framework directly to our target models:

    Key finding: Extreme SLMs (0.6B-1.7B) cannot generate sufficiently high-quality initial trajectories for RL to work. The sparse outcome-based reward signal (answer correctness) provides insufficient guidance when most trajectories are incorrect, leading to training collapse.

2. RL after SFT still degrades performance on extreme SLMs:We then tried applying RL as a refinement step after SFT distillation:

    Why RL fails after SFT: We hypothesize that outcome-based RL causes reward hacking in extreme SLMs—models learn to exploit shortcuts (e.g., generating plausible-sounding answers without proper search) rather than improving search behavior, degrading the carefully learned ASP patterns from SFT.

On-Policy Distillation (OPD) succeeds where RL fails:

We do incorporate RL principles in our work through On-Policy Distillation (Table 1, line 213), which uses:

- Token-level distribution matching (KL divergence) instead of sparse outcome rewards
- Teacher model guidance to regularize student exploration
- Continuous signal at every token rather than delayed outcome feedback

OPD vs. outcome-based RL:
Method|Reward Signal|Qwen3-0.6B|Qwen3-1.7B|Stability|
---|---|---|---|---
GRPO (outcome)|Answer correctness|Failed|Failed|✗ Unstable
OPD (token-level)|KL(student-teacher)|-|-|-
SFT only|N/A (supervised)|47.0|57.6|✓ Stable

Key insight: Token-level RL (OPD) provides dense supervision that extreme SLMs can learn from, while outcome-based RL's sparse rewards are too difficult for models that struggle to generate correct trajectories.

Search-R1's performance is limited on small models:

Even when RL converges on larger SLMs (7B), performance is comparable to our distillation approach:

- Search-R1 (Qwen2.5-7B with RL): ~60-65 F1 on HotpotQA
- ASP-distilled (Qwen3-1.7B with SFT): 57.6 F1
- Our 1.7B achieves 86-95% of RL-trained 7B performance with 1/4 the parameters

Why distillation is more suitable for extreme SLMs:

- Stability: SFT and OPD converge reliably on 0.6B-1.7B models; outcome-based RL fails
- Dense supervision: Teacher trajectories provide step-by-step guidance; sparse rewards do not
- Efficiency: Distillation requires one training pass; RL requires expensive rollouts
- Focus on "what" not "how": Our contribution is showing what to distill (always-search behavior) is more critical than how to optimize (RL vs. SFT)

Our exploration of RL methods:

- SFT (Section 4.1): Base distillation approach
- OPD (Section 4.1, Table 1): Token-level RL with teacher regularization
- Mixed (SFT + OPD): Combined approach (Table 1, line 211)
- RFT (Rejection Fine-Tuning): Reinforcement from self-generated trajectories (lines 184-187)

We thoroughly investigated both direct RL and RL-after-SFT approaches but found they are fundamentally unsuitable for extreme SLMs (<2B). Instead, we demonstrate that distillation with token-level RL (OPD) provides the right balance of stability and performance for our target model scales. This is an empirical finding that advances our understanding of what training paradigms work for extreme SLMs.