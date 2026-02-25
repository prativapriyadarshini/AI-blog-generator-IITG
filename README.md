# AI-blog-generator-IITG
This report walks through the full development lifecycle of an AI-powered blog
generation system. At its core is Meta's Llama-3.2-3B-Instruct, a compact but
capable language model that we fine-tuned using QLoRA — a technique that lets
us train on consumer-grade hardware without sacrificing output quality. The
model learns from real blog posts, and when you give it a title, it produces a
coherent, well-structured markdown blog post complete with introduction, body
sections, and a conclusion


##  Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  TRAINING PIPELINE                       │
│                                                          │
│  HuggingFace Hub  →  Data Cleaning  →  Prompt Format    │
│  nepalprabin/           (drop URLs,      (Llama-3.1      │
│  blog_dataset           strip WS,         chat template) │
│                         skip nulls)                      │
│                          ↓                               │
│       SFTTrainer · cosine LR · packing · 200 steps       │
│                          ↓                               │
│  Llama-3.2-3B (frozen)  +  QLoRA Adapter (r=32, α=64)   │
│                          ↓                               │
│              blog_writer_lora/  (~50MB)                  │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                  INFERENCE PIPELINE                      │
│                                                          │
│   Blog Title  →  Llama-3.1 Chat Template                 │
│                          ↓                               │
│                       Model          
│                          ↓                               │
│   Markdown Blog Post (Introduction + Sections + Conclusion+
      _Based on selected word limits_ )│
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                 INTEGRATION LAYER                        │
│                                                          │
│   Gradio Blocks · Soft Theme · Rendered Preview          │
│   share=True  →  Public gradio.live URL (72hr)           │
└─────────────────────────────────────────────────────────┘
```


