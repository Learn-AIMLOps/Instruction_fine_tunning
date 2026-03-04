                ┌──────────────────────────────────────────────┐
                │                PDF (Domain Book)             │
                │     - Text                                   │
                │     - Diagrams / Figures / Tables            │
                └──────────────────────────────────────────────┘
                                   │
                                   ▼
        ┌────────────────────────────────────────────────────────────┐
        │                Model C — Qwen2‑VL‑2B‑Instruct              │
        │      (ONLY used for image captioning / diagram extraction) │
        │                                                            │
        │  Purpose:                                                  │
        │   - Convert diagrams → semantic text                       │
        │   - Describe workflows, arrows, pipelines                  │
        │   - Produce captions like:                                 │
        │       “This diagram shows the LLMOps workflow…”            │
        └────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
        ┌────────────────────────────────────────────────────────────┐
        │      Unified Domain Dataset (Text + Captions + OCR)        │
        │      - Clean text                                          │
        │      - Diagram captions                                    │
        │      - Structured chunks                                   │
        └────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
        ┌────────────────────────────────────────────────────────────┐
        │        Model A — Qwen/Qwen2‑1.5B (Base Model)              │
        │        Continued Pretraining on Domain Data                │
        │                                                            │
        │  Purpose:                                                  │
        │   - Learn domain knowledge                                 │
        │   - Learn terminology, workflows, definitions              │
        │   - Become “domain‑aware”                                  │
        │                                                            │
        │  IMPORTANT:                                                │
        │   - NOT instruction‑tuned                                  │
        │   - NOT chat‑tuned                                         │
        │   - ONLY learns domain knowledge                           │
        └────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
        ┌────────────────────────────────────────────────────────────┐
        │      Model B — Qwen/Qwen2‑1.5B‑Instruct (Teacher)          │
        │      Generates IFT Dataset                                 │
        │                                                            │
        │  Purpose:                                                  │
        │   - Create instruction–input–response samples              │
        │   - Evaluate quality                                       │
        │   - Rewrite prompts if needed                              │
        │   - Produce 100% instruction‑following behavior            │
        └────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
        ┌────────────────────────────────────────────────────────────┐
        │      Final SFT Training (QLoRA)                            │
        │      Model A + IFT Dataset → Domain‑Aware Instruct Model   │
        │                                                            │
        │  Result:                                                   │
        │   - Knows the domain deeply                                │
        │   - Follows instructions perfectly                         │
        │   - Can generate domain‑specific answers                   │
        │   - Can reason about workflows, diagrams, pipelines        │
        └────────────────────────────────────────────────────────────┘




Stage 0: Raw / Official Base Model
(Qwen/Qwen2-1.5B  or  Qwen/Qwen2.5-1.5B — pure next-token prediction, strong general language but no instruction/chat awareness)
          │
          ▼
Stage 1: Optional Continued / Mid-Training / Domain Pretraining
   └─ on large unstructured or domain-heavy text chunks (math, code, long-context, your custom corpus…)
   └─ Goal: Strengthen base capabilities before alignment → becomes "your enhanced base"
          │
          ▼
Stage 2: IFT/SFT Dataset Creation
   └─ Use strong instruct model to synthesize data
   └─ Examples: Qwen/Qwen2-1.5B-Instruct, Qwen/Qwen2.5-7B-Instruct, or closed models (Claude, GPT-4o-mini…)
   └─ Preferred output format: **plain-text / Alpaca-style** (### Instruction: … ### Response: …)
          │           (cleanest way to teach instruction following from a pure base)
          ▼
Stage 3: Stage 1 Alignment — Supervised Fine-Tuning (SFT / IFT) on plain-text data
   └─ Teach basic single-turn instruction following
   └─ Training & inference format: ### Instruction: … ### Input: … ### Response: …
          │
          ▼
Stage 4: Result — Instruction-tuned model v1 (single-turn, plain-text style)
   └─ Base + adapters (LoRA/QLoRA)  or  merged full weights
   └─ Good for RAG, single-query tasks, API-style usage
          │
          │     ┌────────────── optional but very common for production-grade chat ──────────────┐
          ▼     ▼                                                            │
   [Many pipelines stop here — sufficient for lots of use-cases]     Stage 5: Stage 2 Alignment — Chat / Multi-turn / Role-aware tuning
          │                                                     └─ Convert existing dataset OR synthesize new data in special-token format
          │                                                        (ChatML: <|im_start|>user … <|im_end|>, <|im_start|>assistant … etc.)
          ▼                                                           │
Strong general instruction model (v1)                                 ▼
(single-turn, clean markdown-like prompts)                 Stage 6: Result — Chat-specialized model v2
          │                                             └─ Now natively handles roles, multi-turn, system prompts, tool calls
          ▼                                                       │
Deploy / inference as-is                                               ▼
   or ──────────────────────────────────────►  Final production chat / agent model
                                                     (preferred for conversational apps, long contexts, complex agents)









        ┌────────────────────────────────────────────────────────────┐
        │ Stage 0 ─ Raw Base Model                                   │
        │ Qwen/Qwen2-1.5B  or  Qwen/Qwen2.5-1.5B                     │
        │ (pure next-token prediction, no instruction/chat awareness)│
        └────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
        ┌────────────────────────────────────────────────────────────┐
        │ Stage 1 ─ Optional Continued / Domain Pretraining          │
        │ (on large unstructured/domain text chunks: math, code, etc.)│
        │ Goal: Build stronger general capabilities before alignment  │
        └────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
        ┌────────────────────────────────────────────────────────────┐
        │ Stage 2 ─ IFT/SFT Dataset Creation                         │
        │ Teacher: Qwen/Qwen2-1.5B-Instruct (or stronger Instruct)   │
        │ Generates high-quality instruction–input–response samples  │
        │ Preferred format: Plain-text / Alpaca-style                │
        │   ### Instruction: … ### Input: … ### Response: …          │
        │ Purpose: Clean, stable data for base-model-friendly SFT    │
        └────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
        ┌────────────────────────────────────────────────────────────┐
        │ Stage 3 ─ Stage 1 Alignment: Supervised Fine-Tuning (SFT)  │
        │ QLoRA / LoRA on Base Model + plain-text IFT Dataset        │
        │ Result: Instruction-tuned model v1                         │
        │   - Follows single-turn instructions reliably              │
        │   - Uses simple markdown-like format                       │
        │   - Good for RAG, API-style, single-query tasks            │
        └────────────────────────────────────────────────────────────┘
                                   │
                 ┌─────────────────┴─────────────────┐
                 ▼                                   ▼
┌───────────────────────────────┐       ┌───────────────────────────────────────┐
│ Many stop here                │       │ Stage 5 ─ Stage 2 Alignment:          │
│ (sufficient for many uses)    │       │ Chat / Multi-turn / Role-aware tuning │
└───────────────────────────────┘       │ Convert data OR generate new samples  │
                 │                      │ in special-token format               │
                 │                      │ (ChatML: 3user … 3assistant)          │
                 ▼                      └───────────────────────────────────────┘
        ┌────────────────────────────────────────────────────────────┐          │
        │ Stage 4 ─ Result: Strong single-turn instruct model        │          ▼
        │ (plain-text style, deployable as-is)                       │ ┌───────────────────────────────────────┐
        └────────────────────────────────────────────────────────────┘ │ Stage 6 ─ Final Result:               │
                                   │                                   │ Chat-specialized model v2             │
                                   └───────────────────────────────►   │ - Natively handles roles & multi-turn │
                                                                       │ - System prompts, tool use, agents    │
                                                                       │ - Production-grade conversational use │
                                                                       └───────────────────────────────────────┘                                                    
