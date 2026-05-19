---
layout: post
title: "Lessons From Building an Industrial RAG-SFT QA System"
date: 2026-05-13 10:00:00 +0800
tags: [RAG, SFT, LoRA, Industry, LLM]
---

When I started our industrial knowledge QA project, I expected model tuning to be the hardest part. In practice, the real bottleneck was data and retrieval quality. We built a full RAG-SFT pipeline for device manuals and deployed it to a factory setting, and the biggest lesson was simple: if retrieval is noisy, generation cannot save you.

The first challenge came from document structure. Most manuals were long PDFs with dense tables, page headers, and inconsistent layouts. A naive chunking pipeline mixed unrelated content and often broke answerable evidence across boundaries. We moved to a page-aware hybrid indexing strategy. Instead of only text chunks, we preserved page-level context and metadata, then combined lexical and dense retrieval. This reduced many failure cases where the model hallucinated because it never saw the right evidence in one place.

Our corpus finally reached 1,502 manuals and 39,599 QA pairs. At that scale, another issue appeared: hard negatives were too easy. The reranker quickly learned shortcuts and stopped improving. We introduced a three-tier negative sampling strategy. The first tier used random negatives for stability. The second tier used semantic near-misses from similar manuals. The third tier used fine-grained, page-neighbor distractors that were lexically close but semantically wrong. This forced the reranker to learn actual discrimination instead of keyword matching.

For reranking, we trained on bge-reranker-v2-m3 and added knowledge distillation. Distillation gave us a stronger target signal, especially on borderline examples where binary labels were not expressive enough. Training became more stable, and recall at top positions improved in cases with long-tail terminology from electrical equipment and industrial control docs.

Evaluation was another turning point. Early on, we only tracked retrieval metrics and a basic exact-match style QA score. These numbers looked acceptable, but users still reported unreliable answers. We then added an LLM-as-Judge framework focused on faithfulness, evidence alignment, and answer safety. It was not perfect, but it exposed hidden errors much earlier. After integrating this judge feedback into iteration loops, faithfulness improved by about 10%, and the overall business pass rate increased by 55%.

On the generation side, we used LoRA SFT with DeepSpeed ZeRO-2 to keep training efficient. One practical lesson was that instruction format consistency mattered almost as much as hyperparameters. We standardized prompts to explicitly separate task, constraints, and evidence snippets. We also filtered low-confidence pairs before SFT. This reduced noisy supervision and improved final response quality; JudgeLM score moved from 4.65 to 5.40 in our internal benchmark.

Deployment taught me that latency and controllability are part of model quality. In a factory environment, users prefer slightly shorter but verifiable answers over long speculative ones. We added citation-style evidence snippets in responses and tightened fallback behavior when confidence was low. Trust improved quickly because users could see where answers came from.

Another lesson was that product feedback loops should start early. Engineers and operators asked different questions from what our offline dataset emphasized. Some asked troubleshooting sequences, others asked parameter lookup under time pressure. We added these patterns into data curation and prompt templates, and performance gains were larger than another round of pure model tuning.

Finally, I learned that industrial RAG is a systems problem, not just a model problem. Data parsing, retrieval design, training objectives, evaluation protocols, and UX constraints all interact. The success of this project was not one breakthrough trick, but disciplined iteration across the stack. That mindset also helped our dataset get adopted by CCKS2025 Task 3 and supported our patent filing. If I start a new RAG project tomorrow, I will invest in evidence quality and evaluation design first, then tune the model.
