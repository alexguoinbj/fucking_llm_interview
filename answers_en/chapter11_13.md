# 11.13 Inference Economics and Compute Resource Scheduling — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 11: Cutting-Edge Topics and Comprehensive Design · 15 questions

**1. Why Is LLM Inference Evolving from a Pure Technical Problem into an Economics Problem Similar to the "Electricity Market"?**

<details>
<summary>💡 Show Answer</summary>

The core resource consumed by LLM inference (GPU/accelerator compute) exhibits several economic characteristics similar to electricity:

- **Supply is relatively rigid and difficult to expand rapidly in the short term**: Newly added compute is constrained by chip manufacturing cycles and data center construction cycles, making total supply relatively fixed in the short term and difficult to scale elastically and rapidly like internet bandwidth. This resembles the "short-term power generation capacity rigidity" characteristic of electricity systems;
- **Demand exhibits distinct temporal fluctuations**: Model call demands across different regions and user groups display cyclical diurnal and weekday/holiday fluctuations, creating a "peak-valley" demand structure highly similar to intra-day peak-valley load characteristics in power systems;
- **Inability to be stored at scale and low cost**: Compute resources cannot be stored at low cost like commodity inventory for "peak shaving and valley filling" (an inference request must consume current compute resources in real-time to complete, and unlike electricity, cannot be time-shifted via energy storage devices; although requests themselves can be queued for deferred processing, this essentially sacrifices timeliness rather than truly "storing compute").
**Economic Implications:** These characteristics make a single fixed price incapable of simultaneously balancing the dual goals of "improving resource utilization efficiency" and "guaranteeing service quality during peak periods." Consequently, an increasing number of vendors are drawing inspiration from mature dynamic pricing mechanisms in electricity markets (such as peak-valley electricity rates) to guide demand distribution. This serves as the economic foundation for the emergence of billing models such as "peak-valley Tokens."

</details>

**2. What Is the "Peak-Valley Token" Billing Model? What Is Its Pricing Logic?**

<details>
<summary>💡 Show Answer</summary>

Peak-valley Token billing refers to cloud vendors adopting differential per-token pricing for model calls based on compute supply and demand conditions at different times of the day, using price leverage to guide users into scheduling non-urgent, deferrable tasks to off-peak hours (e.g., some vendors have launched nighttime prices as low as 80% off daytime rates).

**Pricing Logic:**
- **Price as a demand-side adjustment signal**: During peak hours (typically working hours), compute demand is concentrated and supply is relatively tight. Maintaining higher prices both reflects the true scarcity of resources and suppresses the concentration of non-urgent demand during peak periods. During off-peak hours (such as late night), compute experiences significant idleness. Substantial price cuts incentivize users to migrate deferrable batch-processing tasks (such as offline data processing, model evaluation, and non-real-time batch content generation) to off-peak hours, improving overall resource utilization;
- **Differential adaptability across task types**: This pricing mechanism is naturally more suitable for latency-insensitive batch tasks, whereas its value for online service tasks requiring real-time response (such as interactive user dialogue) is limited—because such tasks cannot simply be "deferred to nighttime execution";
- **Limitations of the analogy with traditional electricity peak-valley pricing**: User-side adjustment capabilities in electricity peak-valley pricing (such as factories adjusting production shifts) are relatively mature, whereas the user-side behavioral adjustment capability of "scheduling AI inference tasks to off-peak hours" is still in its infancy. It heavily relies on developers proactively designing task scheduling logic to align with price signals, rather than end-users directly perceiving and responding to them. Therefore, this mechanism currently impacts enterprise-level, high-volume developer clients far more than directly affecting the pricing of consumer-facing real-time dialogue products.

</details>

**3. What Fundamental Change Has Test-Time Compute Scaling Brought to the Inference Cost Structure?**

<details>
<summary>💡 Show Answer</summary>

Test-Time Compute Scaling—which improves response quality during the inference stage through multiple sampling, longer chains of thought, and multi-round verification (as previously touched upon in inference-related discussions)—fundamentally alters the cost structure of LLM services from an economic perspective.

- **From "fixed-cost dominated" to "adjustable variable cost"**: Investments in the traditional pre-training stage represent one-off fixed costs (once training is completed, regardless of how many users are served, the training cost of the model itself is sunk). In contrast, Test-Time Compute renders **the cost of each specific call adjustable on demand**—users/developers can proactively choose to invest more or less compute at inference time based on the quality requirements of the task. This provides a direct technical foundation for differential pricing (where different "thinking depth" tiers correspond to different prices), underpinning the economic logic behind several vendors introducing tiered pricing products such as "fast tier" and "deep thinking tier";
- **Rebalancing marginal cost and marginal benefit**: For enterprise users, this means that "whether to spend extra compute on a task to gain higher accuracy" becomes a quantifiable trade-off decision based directly on specific task value (if the cost of task failure is high, the additional inference cost is easily justified; if the task itself has high fault tolerance, selecting a cheaper, faster tier is more economical). Procurement decisions for inference services are becoming much closer to traditional enterprise procurement decisions for other resources, requiring dedicated cost-benefit analysis rather than simply "pursuing the strongest model."



![与预训练缩放限制相比，推理期扩展计算下的性能缩放曲线 (Performance scaling curves under expanded test-time computation compared to pre-training scaling limits.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig188_test_time_compute.png)

</details>

**4. What Is the Economic Essence of Model Routing as a Cost Optimization Means?**

<details>
<summary>💡 Show Answer</summary>

The "reasoning sandwich" layered scheduling discussed earlier in the Agent Harness chapter is essentially a specific application of model routing technology. From an economic perspective, it can be understood as a **resource allocation mechanism differentiated by price and quality**.

- **Core logic**: Tasks of varying complexity have significantly different demands on model capabilities. If the strongest (and most expensive) model is uniformly used for all requests, a vast number of simple tasks will cause excessive consumption and waste of compute resources. Model routing automatically assesses the complexity of each request and allocates it to the lowest-cost model tier capable of handling it, routing only truly complex, high-value requests to the highest-cost flagship model. This significantly reduces average service costs while maintaining overall service quality;
- **Cost trade-off of the routing decision itself**: The routing judgment itself consumes compute resources (whether using a lightweight classifier or the main model itself for preliminary judgment). Therefore, the routing mechanism design must ensure that the "cost of routing decision" is significantly lower than the "resource savings brought by routing decisions"; otherwise, the mechanism itself becomes uneconomical;
- **Impact on service providers' pricing strategies**: The maturity of model routing technology enables service providers to maintain low public pricing (due to low marginal costs for simple tasks) while preserving high-quality service capabilities for complex tasks. This forms the economic basis for many current AI products offering both "free/low-cost versions" and "premium versions" sharing the same backend routing infrastructure, rather than training completely independent model systems for different price tiers.

</details>

**5. How Does "Compute Interconnect" (e.g., High-Bandwidth Memory, Inter-Chip Interconnects) Becoming an Industry Focus Impact the Cost Structure of LLMs?**

<details>
<summary>💡 Show Answer</summary>

The recent sharp rise in the market values of memory chip vendors (such as High Bandwidth Memory, HBM) and the intense focus on compute interconnect technologies reflect that the cost bottleneck of LLM training and inference is extending beyond pure "compute capability" to "data movement efficiency."

- **Amplification effect of the memory wall on cost**: The core issue targeted by IO-aware optimization techniques like FlashAttention discussed earlier in the architecture chapter is that in many scenarios, the real bottleneck is not the raw compute power of the chip, but the bandwidth constraints of moving data between memory and compute units. This means that even if the compute power of the chip itself continues to increase, if memory bandwidth and interconnect bandwidth fail to keep pace, the growth of actually usable effective compute will be severely compromised, indirectly driving up the actual comprehensive cost per unit of effective compute;
- **High-bandwidth memory becoming a new cost and capacity bottleneck**: As architectures like MoE impose higher demands on VRAM capacity and bandwidth (expert parameters require rapid loading and switching) and long-context inference requires maintaining larger KV Caches, the capacity and pricing of High Bandwidth Memory are becoming new variables constraining the overall cost of LLM services. This is the direct reason behind memory chip vendors' significantly enhanced bargaining power and market capitalization in the current AI infrastructure wave;
- **Impact on enterprise procurement decisions**: This trend reminds enterprises that when evaluating the total cost of ownership (TCO) for self-built or rented compute infrastructure, they cannot focus solely on chip compute specifications (such as peak FLOPS). They must also comprehensively evaluate easily overlooked yet critical metrics like memory bandwidth and inter-chip interconnect bandwidth that equally dictate actual usable performance and cost-effectiveness. This aligns with the practical constraints discussed earlier regarding efficient training techniques (such as parallel strategies like ZeRO that place high demands on communication bandwidth) needing joint trade-offs during cluster hardware selection.

</details>

**6. What Economic Concerns Are Reflected in Policy Discussions Regarding "Public Ownership of Shares in Leading AI Companies"?**

<details>
<summary>💡 Show Answer</summary>

Reports indicate that the United States is studying schemes for public ownership of shares in leading AI companies to address concerns over unfair distribution of AI profits. The economic background of this discussion is worth outlining:

- **Concerns over misalignment between value creation and value distribution brought by AI**: If AI technology massively boosts productive efficiency and creates immense economic value, yet this value becomes highly concentrated in the hands of a few leading companies and their shareholders who control core models and infrastructure, while the broader workforce facing potential job disruptions due to AI automation fails to share in corresponding benefits—this concern over "the disconnect between productivity gains and income distribution" serves as the core economic logic driving such policy discussions. It is also a reemergence in the AI era of distributional fairness issues that repeatedly surfaced during previous major technological revolutions (such as the Industrial Revolution);
- **The concept of public shareholding as a revenue redistribution mechanism**: Allowing the public (through forms such as sovereign wealth funds or public pension fund holdings) to directly hold equity in leading AI companies is a policy tool concept aimed at enabling capital gains from AI progress to benefit the public more broadly without directly interfering with market competition mechanisms. However, such schemes still involve numerous details requiring thorough deliberation regarding specific shareholding structure design and avoiding distortions to normal corporate governance and innovation incentives;
- **Connection to previously discussed AI governance topics**: This policy discussion also reflects that AI governance is gradually expanding from an earlier focus primarily on "technical safety" (preventing models from generating harmful content or being abused) to broader issues of "economic and social distributional fairness." Compared to regulatory frameworks like the EU AI Act discussed earlier that focus on technical compliance, this touches upon deeper socio-economic structural issues and constitutes an essential dimension when grasping the complete picture of AI governance.

</details>

**7. What Economic Trade-Offs Are Typically Involved in the SLA (Service Level Agreement) Design of Inference Services?**

<details>
<summary>💡 Show Answer</summary>

Enterprise AI inference services usually need to explicitly guarantee service levels (such as response latency caps and availability percentages) through SLAs. Designing such agreements is essentially an economic trade-off between **service quality guarantees** and **resource costs**.

- **Trade-off between latency guarantees and resource redundancy**: To ensure promised latency metrics are met even during peak demand periods, service providers must reserve a certain amount of resource redundancy (they cannot push resource utilization to 100%). Higher redundancy yields more reliable SLA guarantees but lowers ordinary resource utilization efficiency. This redundancy cost is ultimately passed on to customers through service pricing, creating the economic logic that "higher service certainty corresponds to higher prices";
- **Tiered SLAs and differential pricing**: Similar to model routing and peak-valley pricing discussed earlier, many service providers design differential pricing for different SLA tiers (such as a low-cost "best-effort" channel vs. a high-cost "strict latency guarantee" channel). This allows customers to make cost choices based on their business's actual sensitivity to latency and reliability, rather than requiring all customers to pay uniformly for top-tier guarantees;
- **Penalty costs and insurance mechanisms**: The design of compensation clauses when a service fails to meet SLA commitments is essentially a risk pricing mechanism—service providers must assess the probability of SLA breach and compensation costs, internalizing them into the overall pricing strategy. This is highly analogous to the SLA economics of traditional infrastructure services (such as cloud computing and telecommunications). As an emerging yet maturing infrastructure category, AI inference services are rapidly borrowing and adapting these established commercial practices.

</details>

**8. What Impact Has the Popularization of Inference Cost Optimization Techniques (Quantization, Speculative Decoding, etc.) Had on the Pricing Strategies of AI Products?**

<details>
<summary>💡 Show Answer</summary>

The widespread adoption of quantization techniques (discussed earlier in the "Efficient Training and Model Compression" section) and inference acceleration techniques such as speculative decoding (discussed in the harness section) is profoundly shaping the pricing landscape of AI products.

- **Driving a continuous decline in overall price levels**: As technologies like quantization, speculative decoding, and more efficient attention mechanisms (such as the aforementioned NSA and MLA) mature and proliferate, the compute cost required to produce outputs of equivalent quality continually decreases. In a fully competitive market environment, this efficiency gain is typically passed on quickly as lower end-service prices, forming the techno-economic foundation behind the overall downward trend in LLM API pricing in recent years;
- **Accelerating the process of "democratizing capabilities"**: Lower inference costs make high-quality model services—previously affordable only to leading vendors with deep pockets—accessible to a broader array of developers and enterprises at lower costs. It also makes deploying capable models locally on edge devices (smartphones, PCs) far more feasible. This "democratization of capabilities" trend echoes the "efficient training route for small models" discussed earlier on a market level;
- **Impact on the competitive landscape**: When inference cost efficiency becomes a primary variable influencing pricing competitiveness, vendors with superior inference optimization technology can offer more competitively priced services while maintaining equal (or even higher) profit margins. This turns "inference system engineering capability" (rather than merely model algorithmic capability itself) into a crucial dimension of differentiated competition among AI companies, echoing the central theme running throughout this book that "system engineering surrounding models is as important as the models themselves."

</details>

**9. Where Does the Increased Complexity in Cost Accounting under Agent Scenarios Lie Compared to Traditional Single-Turn Q&A Calls?**

<details>
<summary>💡 Show Answer</summary>

The cost optimization strategies for Agents were discussed earlier in the Agent product engineering architecture section. Here, we further expand on the sources of complexity from an economic accounting perspective:

- **Cost uncertainty and long-tail distribution**: Token consumption and costs for single-turn Q&A calls are relatively predictable (input and output lengths are bounded). In contrast, an Agent task may involve dozens to hundreds of tool call steps. The total actual token consumption depends heavily on the inherent complexity of the task and specific scenarios encountered during execution (such as triggering retries or requiring multi-round corrections), causing significant variance in actual costs even among similar tasks and posing difficulties for cost estimation and product pricing;
- **Risk of sunk costs from failed/looping tasks**: In failure modes such as infinite loops and repeated retries discussed earlier, without effective circuit-breaking and detection mechanisms, these failed tasks that "fail to yield valid results" will nonetheless consume massive compute costs. From a commercial standpoint, these costs represent pure sunk losses. How to identify and terminate such tasks early via previously mentioned loop detection and progress metrics directly impacts the overall cost-effectiveness level of Agent products;
- **Model choice between outcome-based billing vs. resource-consumption billing**: Facing such uncertainties, some Agent products have begun exploring a pricing model of "billing based on final completed task outcomes" (rather than billing by consumed tokens). While easier for users to understand and budget, this model requires service providers to absorb the risk of cost fluctuations during task execution. This demands that the aforementioned cost optimization techniques (model routing, loop detection, context compression, etc.) be sufficiently mature and reliable to ensure stable and reasonable profit margins for providers, explaining why Agent product pricing models are currently more diverse and exploratory than traditional API call billing.



![自主智能体循环中代表多重成本变量的成本核算流程 (Cost accounting flow representing multiple cost variables in autonomous agent loops.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig194_agent_cost_accounting.png)

</details>

**10. How Do the Two Trends of "Compute Democratization" and "Compute Centralization" Coexist in the Current AI Industry?**

<details>
<summary>💡 Show Answer</summary>

On one hand, the aforementioned declining inference costs and the growth of edge small models are driving the "democratization" of compute usage (allowing more entities to access AI capabilities at lower thresholds). On the other hand, the compute scale required to train frontier models continues to surge, exhibiting a contrasting trend of "centralization." While seemingly contradictory, these two trends actually coexist across different layers.

- **Centralization on the training side**: The compute investment required to train foundation models with frontier capabilities (clusters of tens or hundreds of thousands of high-end accelerator cards) carries an extremely high threshold that continues to escalate alongside growing model sizes and training data volumes. This concentrates participation in frontier model training within a handful of well-funded tech giants and state-level projects, forming a clear trend of "compute capital centralization";
- **Democratization on the usage/inference side**: Concurrently, trained model capabilities can be widely reused through APIs, open-source weights, edge deployments, and other formats. Coupled with the techno-economic trend of declining inference costs, the barrier to using these pre-trained capabilities is dropping rapidly, enabling an increasing number of SMEs, individual developers, and general consumers to low-costly "consume" near-frontier AI capabilities, giving rise to a trend of "capability usage democratization";
- **Long-term impact on the industry landscape**: This dual pattern of "centralized training and decentralized usage" implies that value distribution in the AI industry may display structural features akin to traditional infrastructure industries (such as power and telecommunications): "highly concentrated upstream capital alongside flourishing downstream applications." This provides leading companies possessing foundation model training capabilities with sustained strategic bargaining power while reserving vast spaces for value creation and commercialization for downstream startups and developers building specific applications and services around these foundational capabilities. The evolution of this pattern is also key to understanding the underlying industrial logic behind policy discussions on "public shareholding" and "fairness in profit distribution" discussed earlier.

</details>

**11. What Impact Has the Proliferation of Open-Source Models Had on Inference Economics?**

<details>
<summary>💡 Show Answer</summary>

With the continuous emergence of high-performance open-source models (especially open-weight models capable of benchmarking against closed-source models in specific evaluations), their impact on the overall inference economics landscape is multifaceted:

- **Competitive pressure on closed-source model pricing**: When open-source models with comparable capabilities can be self-deployed by enterprises (incurring only infrastructure and operational costs without paying per-call API licensing fees), the pricing power of closed-source model providers is noticeably squeezed. This serves as another major driver pushing the aforementioned trend of "continuous decline in overall price levels";
- **Cost trade-off between self-hosted deployment and cloud API calls**: Enterprises must evaluate the economic trade-off between "procuring hardware, deploying open-source models, and bearing self-operated maintenance and optimization costs" versus "calling cloud APIs directly, paying as you go, but lacking full control over models and data." The optimal resolution depends on the enterprise's call volume (larger volume increases the marginal cost advantage of self-hosting, yielding a "break-even point"), requirements for data privacy and customization, and whether internal engineering teams possess sufficient capability to handle deployment and operational complexity;
- **Accelerated diffusion of inference optimization techniques in the open-source ecosystem**: Because weights and inference frameworks of open-source models are relatively transparent, the community can explore and validate various inference optimization techniques (quantization, speculative decoding, etc.) for open-source models much faster. This collaborative innovation pace within open-source communities sometimes surpasses the internal optimization cadence of single vendors, further accelerating the drop in overall inference costs—a frequently underestimated yet genuine accelerating effect of the open-source ecosystem on industry-wide inference economics.

</details>

**12. What Is the Economic Value Proposition of On-Device Small Models? Is Their Relationship with Cloud LLM Services Competitive or Complementary?**

<details>
<summary>💡 Show Answer</summary>

On-device small models refer to relatively small-scale models capable of running locally on end-user devices such as smartphones and personal computers without relying on cloud network requests.

**Economic Value Proposition:**
- **Shift of inference cost from service providers to user devices**: Running on-device means the compute required for inference is borne by the hardware of the user's device. Service providers no longer need to pay cloud compute costs for every call, fundamentally altering the traditional cost model of "paying providers per call volume." For scenarios with extremely high call frequencies but low task complexity (such as predictive text input or simple local data processing), on-device deployment offers a significant marginal cost advantage;
- **Latency and privacy as additional value dimensions**: On-device inference eliminates round-trip network latency to deliver faster response speeds while keeping data locally processed without cloud uploads, naturally satisfying high data privacy requirements. Beyond pure compute cost considerations, these two aspects constitute independent value propositions of on-device small models.
**Relationship with cloud LLMs:** A more accurate positioning is **complementary rather than purely competitive**—on-device small models excel at high-frequency, relatively simple tasks sensitive to latency and privacy, whereas complex tasks demanding deep reasoning or up-to-date knowledge still rely on more powerful cloud models. In practice, an increasing number of products adopt a hybrid architecture of "on-device models handling routine requests, cloud models handling complex requests," which is essentially an extension and application of the model routing concept at a macro "device-cloud collaboration" architectural level.



![本地端侧小模型与云端大语言模型之间的路由逻辑与权衡 (Routing logic and trade-offs between local on-device small models and cloud-based LLMs.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig197_on_device_vs_cloud.png)

</details>

**13. What New Possibilities Does the Continuous Evolution of Inference Economics Bring to the Business Model Design of AI Products?**

<details>
<summary>💡 Show Answer</summary>

- **Exploration from "subscription models" to "outcome/value-based billing"**: As previously discussed, with the proliferation of Agent products and the rising complexity of cost accounting, purely fixed subscription models or simple per-token billing may no longer be the sole or optimal business models. Billing models tied directly to value creation—such as "paying based on final completed task outcomes"—are being trialed by more products, requiring service providers to exert finer control over their cost structures and risk exposures;
- **Granular design of tiered product matrices**: Leveraging technologies and business tools like model routing, Test-Time Compute tiering, and peak-valley pricing, service providers can design tiered product matrices far more sophisticated than early "single model, single price" setups (combining multiple speed/quality/cost tiers). This delivers precisely tailored choices for different customer segments and task types, turning granular tiering itself into a distinct dimension of product competitiveness;
- **New business paradigms spawned by shifting infrastructure cost structures**: As inference costs continuously decline and on-device capabilities improve, application scenarios previously deemed commercially unviable due to prohibitive costs (such as real-time interactive applications requiring extremely high call frequencies) are becoming economically feasible. This creates space for spawning entirely new, hitherto non-existent business models and product forms—representing one of the most imaginative yet hardest-to-predict long-term impacts of evolving inference economics on the AI ecosystem.

</details>

**14. Carbon Emissions and Energy Consumption Are Becoming New Variables in Inference Economics—How Is This Manifested?**

<details>
<summary>💡 Show Answer</summary>

As the scale of AI data centers continues to expand, their energy consumption and carbon emissions are evolving from a previously marginal topic into active economic variables influencing inference cost structures and corporate decision-making.

- **Energy costs occupying a significant proportion of data center operating expenses**: Electricity consumption of large-scale GPU clusters represents one of the fastest-growing components of data center operating costs. This leads data center site selection decisions to increasingly prioritize local electricity rates, power supply stability, and proximity to low-cost clean energy sources (such as hydroelectric or nuclear power bases), directly affecting the relative cost competitiveness of compute services across regions;
- **Rise of regulatory and disclosure requirements**: Some regions have already begun requiring operators of large data centers to disclose energy consumption and carbon emission data. The future possibility of carbon taxes or carbon trading mechanisms being applied to the AI compute industry cannot be ruled out. Such potential regulatory costs represent risk factors that enterprises must factor into long-term compute infrastructure investment decisions;
- **Dual value of inference efficiency optimization**: Inference efficiency optimization techniques discussed earlier—such as quantization, sparse attention, and speculative decoding—not only directly lower compute costs but objectively reduce energy consumption per unit task. Consequently, "efficiency optimization" technologies are assuming environmental sustainability value alongside their commercial value. Some vendors have begun highlighting "energy consumption per call" of their models or services in marketing as a differentiating selling point, reflecting how energy efficiency is evolving from a pure technical metric into a commercial metric with market communication value.

</details>

**15. What Economic Risks Lie Behind the "Arms Race" Phenomenon in AI Infrastructure Investments?**

<details>
<summary>💡 Show Answer</summary>

In recent years, capital expenditure by major tech companies on AI infrastructure (data centers, chip procurement, compute cluster construction) has shown a continuous upward trend. Behind this large-scale investment race lie several noteworthy economic risks:

- **Mismatch risk between overcapacity and demand uncertainty**: Construction cycles for compute infrastructure typically span one to two years or even longer, whereas the actual growth speed and structure of AI application demand (which tasks genuinely require massive compute, whether users are willing to pay for corresponding service levels) carry significant uncertainty. If actual demand growth falls short of expectations, risks of relative compute overcapacity and prolonged return-on-investment (ROI) cycles may emerge—drawing parallels to "overcapacity" cycles historically experienced by other infrastructure industries (such as early telecommunications fiber-optic network expansion);
- **Asset depreciation risks stemming from rapid technology iteration**: If next-generation chip or model architectures (such as innovations in attention mechanisms and evolution of quantization formats discussed earlier) yield dramatic efficiency improvements, compute assets procured at scale based on legacy architectures may face accelerated economic obsolescence. This requires enterprises to remain sensitive to technology evolution cadences when making long-term infrastructure investment decisions, avoiding over-betting on a single technological path or allowing procurement cycles to fall severely out of sync with tech iteration cycles;
- **Potential systemic risks in financing structures and debt leverage**: A portion of large-scale compute infrastructure investments relies on debt financing or special financing structures (such as circular investment arrangements between chipmakers and cloud providers). If underlying commercial return expectations fail to materialize as planned, such highly leveraged financing structures could amplify financial risks across the entire industry value chain. This forms the primary concern driving recent focus among market observers and regulators regarding whether AI infrastructure investments harbor structural bubble risks. While currently inconclusive, this serves as a valuable perspective for comprehending the macroeconomic context of the AI industry today.

</details>

