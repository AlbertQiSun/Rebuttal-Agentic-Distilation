# Official Review of Submission10245 by Reviewer mVAr

Paper Summary:
This work analyzes small language models (SLMs) for search agent. The authors observe that SLMs tend to over-rely on internal knowledge and underutilize search tool calls, and that distillation from larger teacher models fails to adequately address this issue. To overcome this limitation, they propose a novel distillation method based on an Always-Search Policy (ASP). As the name suggests, ASP constrains the SLM to invoke the search tool whenever external information is required, rather than relying on its internal knowledge.

The effectiveness of ASP distillation is comprehensively validated on multi-hop question-answering benchmarks (e.g., HotpotQA, 2WikiMultiHopQA, Bamboogle, and MuSiQue) as well as challenging information-seeking tasks (e.g., BrowseComp-Plus, FRAMES, and LongSeAL). As further evidence supporting ASP, the authors compare it with an adaptive search policy and observe performance degradation under the adaptive setting, reinforcing the effectiveness of ASP for SLMs.

Summary Of Strengths:
S1. The studied problem is timely and important. The motivation is well-written and easy-to-understand.
S2. The proposed methods is practical.
S3. The experimental results and discussions are insightful.
Summary Of Weaknesses:
W1. Lack of implementation details: Fine-tuning SLM for ASP is one of the core contributions; however, critical details necessary for understanding and reproducing the method are missing in Section 4.1, Fine-tuning Techniques.
W2. Incomplete analysis of under-searching behavior: To substantiate the claim regarding the “under-searching” tendency of SLMs, it would be informative to report how many search tool calls are made by Distilled-Qwen3-1.7B.
W3. Incomplete comparison of efficiency and latency: Since ASP increases the number of search tool calls, a discussion or analysis of latency would be helpful. In particular, it would be interesting to understand how the end-to-end latency of an SLM trained with ASP distillation compares to that of a larger teacher model that adaptively balances search and internal knowledge usage.
Comments Suggestions And Typos:
None

Confidence: 5 = Positive that my evaluation is correct. I read the paper very carefully and am familiar with related work.
Soundness: 3.5
Excitement: 3.5
Overall Assessment: 3.5 = Borderline Conference
Ethical Concerns:
There are no concerns with this submission

Needs Ethics Review: No
Reproducibility: 2 = They would be hard pressed to reproduce the results: The contribution depends on data that are simply not available outside the author's institution or consortium and/or not enough details are provided.
Datasets: 1 = No usable datasets submitted.
Software: 1 = No usable software released.
Knowledge Of Or Educated Guess At Author Identity: No
Knowledge Of Paper: N/A, I do not know anything about the paper from outside sources
Knowledge Of Paper Source: N/A, I do not know anything about the paper from outside sources
Impact Of Knowledge Of Paper: N/A, I do not know anything about the paper from outside sources
Reviewer Certification: I certify that the review I entered accurately reflects my assessment of the work. If you used any type of automated tool to help you craft your review, I hereby certify that its use was restricted to improving grammar and style, and the substance of the review is either my own work or the work of an acknowledged secondary reviewer.
Publication Ethics Policy Compliance: I used a privacy-preserving tool exclusively for the use case(s) approved by PEC policy, such as language edits


# Response to the comment from mVAr
Dear reviewer mVAr,

Thank you for your thorough review and insightful feedback. We appreciate your recognition of our work's timeliness and practical value. 

Below we address each concern in detail:

### **For W1 (Implementation details):**

We appreciate this important point about reproducibility. The key implementation details are included in Appendix B.2 (lines 478-493) due to the strict space constraints of the short paper track. We acknowledge that Section 4.1 should better highlight ASP's implementation mechanics. 

- For SFT with ASP (Trajectory-based Offline Distillation):
    - We apply dual filtering: (1) String-F1 > 0.65, and (2) strict search tool checking that validates models consistently invoke search rather than relying on parametric knowledge
    - Training: 18K filtered trajectories, 3 epochs, AdamW optimizer, lr=1e-5, batch size=4
    - The filtering ensures only trajectories where the model searches for all required information are retained
- For OPD with ASP:
    - System prompt explicitly instructs the model to always use search tools (see Appendix B.4, lines 515-518)
    - Teacher model's log-probability distribution regularizes student behavior
    - Training: 3K samples, 8 trajectories/sample, 4 epochs, lr=2e-6. We used a batch size of 4 with gradient accumulation, resulting in an effective global batch size of 32.

We will highlight these details in the main text in the final version.

### **For W2 (Search Tool Call Statistics)**

We report this in Section 4.2 (lines 222-225), but it's only for ASP-trained models. 
ModelAvg. |Tool CallsPerformance |(F1)
|---|---|---|
|Vanilla-Qwen3-1.7B|1.72|42.3|
|**Distilled-Qwen3-1.7B**|**1.89**|**47.9**|
|SFT-Qwen3-1.7B (ASP)|2.47|57.6|
|OPD-Qwen3-1.7B (ASP)|2.84|56.2|
|Vanilla-Qwen3-32B|3.02|60.3|

However, the "under-searching" issue is not merely quantitative but also qualitative. Our error analysis (Appendix C) reveals that among the 66 failure cases of Distilled-Qwen3-1.7B:
- 23 cases (35%): Insufficient/bad retrieval - the model either searches too few times OR formulates poor-quality queries that fail to retrieve relevant information
- 33 cases (50%): Hallucination - the model ignores retrieved evidence or fills gaps with parametric knowledge
- 16 cases (24%): show both issues co-occurring - suggesting the model's search behavior is fundamentally unreliable

    (**Note:** categories are not mutually exclusive; total annotations exceed 66 cases)

**Key insight:** Simply increasing search frequency from 1.72→1.89 doesn't address the root problem. The distilled model exhibits:
- Syntactic failures: Struggles with tool-calling format (Section 3.1, line 117)
- Semantic failures: Even when it searches, queries are often poorly formulated or the model disregards results

This is why we emphasize that ASP not only increases search frequency (**2.47**-**2.84** calls) but enforces consistent, reliable search behavior - every search is meaningful and results are properly utilized, as demonstrated by the 9.7 F1 improvement over standard distillation.

### **For W3 (Efficiency and Latency Analysis)**

We appreciate this concern, but respectfully suggest that latency **is not the primary bottleneck** when comparing models of equivalent performance. The key question should be: "For comparable accuracy, which model offers better efficiency?"

Performance-Matched Latency Comparison (HotpotQA, H20 GPU, vllm, faiss-gpu):
|Model|String-F1|Avg. Latency|Search Calls|
|---|---|---|---|
|Vanilla-Qwen3-1.7B|42.3|~1.8s|1.72|
|SFT-Qwen3-1.7B (ASP)|57.6|~3.1s|2.47|
|Vanilla-Qwen3-8B|58.2|~5.6s|2.31|
|Vanilla-Qwen3-32B|60.3|~8.3s|3.02|

Key observations:
- ASP-trained 1.7B achieves 8B-level performance (57.6 vs 58.2 F1) with ~44% lower latency (3.1s vs 5.5s)
- The latency-performance tradeoff is favorable: While ASP adds search overhead compared to vanilla-1.7B, it achieves substantially better performance while remaining far more efficient than performance-matched larger models
- This is the expected and desired outcome - smaller models with more tool usage outperforming larger models is precisely the motivation for our work

Regarding the increased search calls: Yes, ASP increases searches from 1.72→2.47 calls per query. However, this is the mechanism of improvement, not a limitation. The alternative to achieve similar performance would be deploying an 8B or 32B model, which incurs significantly higher computational cost.

The performance-latency tradeoff is straightforward and expected: smaller models are faster, larger models are slower. Our contribution is **enabling SLMs to achieve LLM-level performance through better search behavior**, not optimizing latency. The fact that ASP-trained SLMs naturally maintain their efficiency advantage over larger models while closing the performance gap is implicit in the model size choice.

Optional addition: If desired, we can add a brief latency discussion to Section 4.2 in our final version, though we believe the performance-matched comparison above makes the tradeoff clear.

### **Reproducibility Concern:**
Regarding reproducibility (score: 2):
- We will release all code, model checkpoints, and training scripts upon acceptance
- We will refine Appendix and section 4.2 for implementation details
- All datasets used for benchmark are publicly available

We hope these additions will improve the reproducibility score.

We sincerely appreciate your thorough review and constructive suggestions. We believe addressing these points will significantly strengthen the paper's clarity and reproducibility. We are committed to releasing all code, models, and detailed documentation upon acceptance to facilitate community adoption of ASP.

Thank you again for your valuable feedback.