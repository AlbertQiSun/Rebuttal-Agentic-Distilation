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

The key implementation details are included in Appendix B.2 (lines 478-493) due to the strict space constraints of the short paper.

- SFT:
    - We apply dual filtering: (1) String-F1 > 0.65, and (2) strict search tool checking that validates models consistently invoke search rather than relying on parametric knowledge
    - Training: 18K trajectories before filtering, 3 epochs, AdamW optimizer, lr=1e-5, batch size=4
    - For rejection finetuning, the same dual-filtering is applied, and we train on 15K trajectories before filtering for 3 epochs, Adam optimizer, 
- OPD:
    - System prompt instructs the model to always use search tools (see Appendix B.4, lines 515-518), and teacher model's log-probability distribution regularizes student behavior
    - Training: 3K samples, 8 trajectories/sample, 4 epochs, lr=2e-6. We used a batch size of 4 with gradient accumulation, resulting in an effective global batch size of 32.
- Rejection Finetuning:
    - We apply the same dual filtering as SFT.
    - Training: 10K trajectories before filtering, 2 epochs, AdamW optimizer, lr=5e-6, batch size=4

Regluar distillation
### **For W2 (Search Tool Call Statistics)**

We report search tool call numbers in Section 4.2 (lines 222-225) on HotpotQA benchmark, but it's only for ASP-trained models. 
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

ASP not only increases search frequency but enforces consistent, reiable search behavior. By resolving the problems of insufficient bad retrieval and hallucination, general performance improves as demonstrated by 9.7% of improvement in F1 score over standard distillation. 

### **For W3 (Efficiency and Latency Analysis)**

We appreciate this concern. In this paper, we concentrate on improving the performance of SLMs on agentic search tasks, but we are also glad to provide some latency-related analysis. 

Performance-Matched Latency Comparison (HotpotQA, H20 GPU, vllm, faiss-gpu):
|Model|String-F1|Avg. Latency|Search Calls|
|---|---|---|---|
|Vanilla-Qwen3-1.7B|42.3|~1.8s|1.72|
|SFT-Qwen3-1.7B (ASP)|57.6|~3.1s|2.47|
|Vanilla-Qwen3-8B|58.2|~5.6s|2.31|
|Vanilla-Qwen3-32B|60.3|~8.3s|3.02|

ASP increases search call numbers from 1.72 to 2.47 calls per query in order to reach comparable performance to LLMs. However, while ASP may add some search overhead, ASP-trained SLMs still remain significantly more efficient than performance-matched larger models. For instance, SFT-1.7B achieves 8B-level performance with ~44% lower latency. Although our contribution is **enabling SLMs to achieve LLM-level performance through better search behavior**, not optimizing latency, ASP-trained SLMs naturally maintain their efficiency advantage over larger models while closing the performance gap.

### **Reproducibility Concern:**
Regarding reproducibility (score: 2):
- We will release all code, model checkpoints, and training scripts upon acceptance
- We will refine implementation details in both section 4 and Appendix B 
- All datasets used for benchmark are publicly available, including training samples

We hope that these additions will further enhance the clarity and transparency of our work, thereby contributing positively to its reproducibility assessment.

Thank you again for your valuable feedback.