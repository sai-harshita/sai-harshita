# 👋 Hi There, I'm Pentamsetty Sai Harshita

Artificial Intelligence Engineer with hands-on experience building **production-ready AI systems**,  
spanning **Generative AI, Machine Learning, and Data Engineering**.

Currently pursuing **MSc in Artificial Intelligence @ NTU Singapore**, with industry experience across  
GenAI, cloud data platforms, and applied ML research.

---

## 🧠 Technical Focus
- Large Language Models & Generative AI systems
- Diffusion models, ControlNet, LoRA fine-tuning
- End-to-end ML pipelines (training → evaluation → deployment)
- Data platforms, analytics engineering, and automation
- AI systems that scale beyond notebooks

---

## 🛠 Core Tech Stack

**Languages**  
Python, Java, R, C  

**AI / ML**  
PyTorch, TensorFlow, Hugging Face, Transformers, LangChain  
LLMs, RAG pipelines, Diffusion Models, ControlNet, Stable Diffusion  

**Computer Vision & NLP**  
CNNs, OCR, Seq2Seq, Neural Machine Translation, Style Transfer  

**Data & Cloud**  
Snowflake, Databricks, SQL, Power BI, Linux Automation  

**MLOps / Tooling**  
Git, Linux, Experiment Tracking, Model Evaluation Metrics (FID, LPIPS, ArtFID)

---

## 💼 Industry Experience

**DWH / BI Developer — Amdocs**  
Cloud data warehousing on Snowflake, analytics pipelines, and Linux automation.

**Generative AI Intern — Atos**  
Built LLM-powered applications, text-to-video pipelines, document analysis chatbots,  
and GenAI content generation systems.

**Data Scientist Intern — GeakMinds**  
Time-series forecasting on large-scale US Census datasets, dashboarding, and stakeholder reporting.


---

## 📫 Connect
- LinkedIn: https://www.linkedin.com/in/pentamsetty-sai-harshita
- Email: p.harshita2002@gmail.com


Hi [Name],

My understanding of your question is: you are asking about the “stickiness” of AWS Strands not just at a vendor level, but at the actual code and architecture level — meaning, once we do `pip install strands-agents` and start building agents, how hard would it be to move away later?

My finding is that Strands itself is not a hard lock-in. The core SDK is open-source and relatively loosely coupled. If we only use `from strands import Agent`, define tools cleanly, and keep the model provider configurable, we can still switch between Bedrock, OpenAI, Anthropic, Gemini, LiteLLM, Ollama, or even another framework later with moderate effort.

The stickiness increases progressively based on what we add around it:

* Low stickiness: using only the core Strands SDK.
* Medium stickiness: using `BedrockModel`, because the code now depends on AWS credentials, AWS regions, Bedrock model IDs, and Bedrock’s API behavior.
* Higher stickiness: using AWS-specific tools, S3-backed session storage, Bedrock Knowledge Bases, IAM permissions, or CloudWatch monitoring.
* Highest stickiness / near hard lock: using Bedrock AgentCore services such as AgentCore Runtime, Memory, Gateway, Identity, Code Interpreter, Browser, and Observability. At this point, migration is not just changing a model provider — we would need to rebuild hosting, memory, identity, tool access, telemetry, and possibly session/state migration.

So I would separate it into two decisions:

1. Strands SDK core: low to medium stickiness.
2. Strands + Bedrock + AgentCore stack: high stickiness.

A hard lock, in this context, means the code or runtime depends on proprietary AWS services that do not have a direct drop-in replacement elsewhere. For example, imports from `bedrock_agentcore.*`, S3 session storage, IAM-based identity, AgentCore Gateway policies, and CloudWatch-specific observability would all create stronger migration friction.

To manage this, we can keep the architecture loosely coupled from the start:

* Keep the model provider behind configuration or a factory layer.
* Do not hardcode Bedrock model IDs throughout the code.
* Keep business logic separate from Strands `@tool` wrappers.
* Prefer MCP-based tools where possible, because MCP reduces framework lock-in.
* Use OpenTelemetry for observability rather than tying everything directly to CloudWatch.
* Use a portable session store where migration flexibility matters.
* Treat AgentCore as an optional production deployment choice, not the default assumption.

Migration is still possible irrespective of stickiness, but the cost depends on how deep we go into AWS. If we stay close to Strands core, migration mainly means rewriting agent orchestration and tool wrappers. If we adopt full AgentCore, migration also includes moving state, replacing identity, rebuilding tool gateways, remapping telemetry, and revalidating prompts after moving away from Bedrock.

Stickiness can also be an advantage if the client is already AWS-first. In that case, using Bedrock and AgentCore can reduce engineering effort because AWS gives managed hosting, scaling, identity, memory, tool gateway, monitoring, and security controls out of the box. The key is not to avoid stickiness completely, but to make it an explicit architecture decision rather than something that happens accidentally through implementation drift.

Compared with other frameworks, LangGraph is better for durable, checkpointed workflows; Azure AI Foundry is stronger for Azure-native clients but has its own Azure lock-in; Semantic Kernel is better for Microsoft/.NET-heavy enterprises; and Strands is strongest when the priority is fast AWS-aligned agent development with production support.

My bottom-line view: Strands is loosely coupled at the SDK level, but can become tightly coupled at the AWS platform level depending on Bedrock, AgentCore, S3, IAM, Gateway, and CloudWatch usage. We should choose the level of coupling consciously based on the client’s cloud strategy and migration-risk appetite.

Reference links:

* Strands model provider abstraction: https://strandsagents.com/docs/user-guide/concepts/model-providers/
* Amazon Bedrock AgentCore overview: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html
* LangGraph durable execution comparison point: https://docs.langchain.com/oss/python/langgraph/durable-execution

