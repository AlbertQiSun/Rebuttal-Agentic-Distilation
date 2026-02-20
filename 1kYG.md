# Official Review of Submission10245 by Reviewer 1kYG
Paper Summary:
This paper addresses the challenge of enabling Small Language Models to act as effective search agents. Through evaluations on multi-hop QA benchmarks like HotpotQA and Bamboogle, the authors find that SLMs suffer from underutilizing search tools and parametric hallucination—over-relying on limited internal knowledge to generate speculative answers—with standard LLM-to-SLM agent distillation yielding only marginal improvements. To solve this, they propose the Always-Search Policy (ASP), a lightweight fine-tuning approach that enforces SLMs to retrieve external evidence via search tools before answering, prioritizing evidence-grounded reasoning over internal knowledge. Experimental results show ASP boosts SLM performance by 15.3–17.3 points on key benchmarks, allowing 1.7B models to match 8B LLMs.

Summary Of Strengths:
The proposed Always-Search Policy (ASP) fills a key void in distilling LLM search capabilities into SLMs. Unlike standard distillation (which mimics LLM trajectories and fails to account for SLMs’ limited parametric knowledge), ASP enforces evidence-grounded reasoning via mandatory search.

The paper’s systematic analysis identifies that SLMs’ primary limitation is poor retrieval, not answer generation. This granular diagnosis resolves ambiguities in prior work and directs future research toward retrieval optimization for SLMs, rather than redundant reasoning-focused fine-tuning.

The work delivers compelling, well-calibrated results across 7 diverse benchmarks, demonstrating that ASP-enabled 1.7B SLMs match or exceed 8B LLMs’ performance (e.g., 58.2 String-F1 on HotpotQA) and generalize to out-of-distribution tasks.

Summary Of Weaknesses:
The paper’s evaluations are limited exclusively to the Qwen3 model series, with no validation across other SLM architectures. Tt is unclear whether ASP’s effectiveness stems from Qwen3-specific characteristics or is a broadly applicable paradigm for SLMs, weakening the claim of its universal utility.

ASP’s design implicitly assumes that retrieved external information is accurate and noise-free, which contradicts real-world search environments. The paper does not address how ASP-trained SLMs would perform when faced with low-quality or incorrect retrieved evidence, limiting its practical relevance for deployment.

To strengthen the paper’s qualitative validation of ASP, adding 2–3 additional case studies across diverse multi-hop QA scenarios will better demonstrate ASP’s ability.

Comments Suggestions And Typos:
Section D (Case Study): “brith country”

Section 3.1: “small models usually has”

Section 5: “frequently fail to annswer”

Confidence: 3 =  Pretty sure, but there's a chance I missed something. Although I have a good feel for this area in general, I did not carefully check the paper's details, e.g., the math or experimental design.
Soundness: 3 = Acceptable: This study provides sufficient support for its main claims. Some minor points may need extra support or details.
Excitement: 3 = Interesting: I might mention some points of this paper to others and/or attend its presentation in a conference if there's time.
Overall Assessment: 3 = Findings: I think this paper could be accepted to the Findings of the ACL.
Ethical Concerns:
There are no concerns with this submission

Needs Ethics Review: No
Reproducibility: 4 = They could mostly reproduce the results, but there may be some variation because of sample variance or minor variations in their interpretation of the protocol or method.
Datasets: 4 = Useful: I would recommend the new datasets to other researchers or developers for their ongoing work.
Software: 4 = Useful: I would recommend the new software to other researchers or developers for their ongoing work.
Knowledge Of Or Educated Guess At Author Identity: No
Knowledge Of Paper: N/A, I do not know anything about the paper from outside sources
Knowledge Of Paper Source: N/A, I do not know anything about the paper from outside sources
Impact Of Knowledge Of Paper: N/A, I do not know anything about the paper from outside sources
Reviewer Certification: I certify that the review I entered accurately reflects my assessment of the work. If you used any type of automated tool to help you craft your review, I hereby certify that its use was restricted to improving grammar and style, and the substance of the review is either my own work or the work of an acknowledged second

# Reponse to Reviewer 1kYG

Dear reviewer 1kYG,
We sincerely appreciate your recognition of our ASP methodology as well as the thoroughness of systematic analysis and experiment results. Below, we respond to each identified weakness respectively. 

### **W1 (Limited to Qwen3 Model Family)** 
TODO: add Llama exp results
We appreciate this observation. However, we respectfully note that evaluation on a single model family is **standard practice in the agentic search and agent distillation literature**, particularly for resource-intensive training paradigms:

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

### **W2 (Noisy/Incorrect Retrieval)**
We address this in our Limitations section (lines 322-327):

"In addition, the effectiveness of Always-Search Policy implicitly assumes that retrieved information is always accurate and reliable, whereas real-world search environments often contain noisy or misleading content. Developing mechanisms to robustly handle such noise is another crucial aspect."

We acknowledge this is a critical gap for real-world deployment. However, we made a deliberate methodological choice to use closed-domain retrieval (Wikipedia corpus) rather than web search. In general, this setup follows standard practice in recent agentic search literature (e.g., Li et al., 2025; Jin et al., 2025) and benchmark settings (HotpotQA, 2WikiMultiHopQA), which facilitates direct comparison with prior work. There are also two specific reasons. First, it better evaluates genuine multi-hop reasoning. When applying web search APIs, web search allows model to hack multihop questions by collapsing sub-queries into a single search that retrieves the final answer directly. Second, a fixed corpus ensures a controlled and reproducible evaluation setting by removing variability from external search engines. This enables researcher to attribute performance gap more directly.

### **W3 (Additional Case Studies)**
TODO: I think it is better to have the case study before writing what we are going to add

We appreciate this suggestion and fully agree that more qualitative examples would strengthen the paper. We will add 2-3 additional case studies to Appendix D demonstrating:

1. Multi-hop reasoning with entity disambiguation: A case where vanilla SLM conflates similar entities but ASP correctly retrieves and distinguishes them (e.g., "Who directed the 2019 film 'The Great Silence'?" vs. the 1968 film)
2. Temporal reasoning: a case where SLM's outdated parametric knowledge fails but ASP retrieves current information (e.g., "What is the current population of the capital of the country that won the 2022 World Cup?")
3/ Comparative reasoning: A case requiring synthesizing information from multiple searches (e.g., "Which film won more Oscars: the debut film of director X or the highest-grossing film of 2015?")

These will illustrate ASP's advantages across diverse reasoning patterns, not just the single entity-linking example currently shown.

### **Typos and Minor Correction**
Thank you pointing out! We will fix:

- Section D: "brith country" → "birth country"
- Section 3.1: "usually has" → "usually have"
- Section 5: "annswer" → "answer"

## Commitments
We commit to the following revisions for the camera-ready version:

- Clarify scope and limitations (W1)
- Expand robustness discussion (W2)
- Enhance qualitative validation (W3)
- Fix all identified typos