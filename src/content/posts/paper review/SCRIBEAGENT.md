---
title: SCRIBEAGENT:TOWARDS SPECIALIZED WEB AGENTS USING PRODUCTION-SCALE WORKFLOW DATA
published: 2025-04-07
description: Review of paper <SCRIBEAGENT>
tags: [Paper Review]
category: Paper
draft: false
---

# ScribeAgent: Towards Specialized Web Agents Using Production-Scale Workflow Data

## 📌 Problem Statement

Most web agents today:

- Use proprietary, general-purpose LLMs (e.g., GPT-4)
- Rely on prompt engineering for task completion
- Lack deep understanding of HTML/DOM structure
- Are inefficient and expensive to serve

**ScribeAgent** proposes a new paradigm:

1. Fine-tune open-source LLMs using real-world web workflow data (6B tokens)
2. Use direct, single-stage generation instead of multi-stage pipelines
3. Outperform GPT-4-based agents in both static and dynamic environments

---

## 🔧 Method Overview

### 🏗️ Data Collection

- Data collected from Scribe plugin, which captures:
  - Webpage URL
  - HTML-DOM
  - Action type: click, type, hotkey
  - Natural language description of the action
  - CSS selector of the target element

- Covers:
  - 250+ websites
  - 10,000+ subdomains
  - Avg. 11 steps per task
  - ~6B training tokens

### 🔍 Preprocessing

- Clean HTML using BeautifulSoup

- Remove irrelevant metadata/scripts

- Use heuristics (char-to-token ratio) to filter out noisy attributes

- Encode each action in a 5-line format:

  ```
  Description: Click the "Menu" button
  Action: mouse click action
  Node: 832
  Target: <svg class="..." node="832">
  ```

### 🧠 Model Training

- Fine-tune using LoRA (parameter-efficient)
- Models evaluated: Mistral 7B, Qwen2.5 32B, Codestral 22B
- Best performing: Qwen2 and Qwen2.5 series

---

## 📊 Experimental Results

### Proprietary Dataset

- ScribeAgent significantly outperforms GPT-4 and GPT-4o
- 6× improvement in exact match accuracy after fine-tuning

### Mind2Web Benchmark (Static)

- Zero-shot ScribeAgent beats all existing baselines
- Good generalization across unseen tasks and domains

### WebArena Benchmark (Dynamic)

- ScribeAgent + GPT-4o forms a multi-agent system for end-to-end task execution
- Outperforms GPT-4-based agents in all categories
- Improves task success rate by **+14.1%** over SOTA (WebPilot)

---

## ✅ Key Contributions

- First LLM agent fine-tuned on large-scale, real-world workflow data
- Outperforms prompt-based GPT-4 agents in navigation and planning
- Full ablation studies on model selection, context length, and dataset size
- Reduced inference cost using smaller open-source models

---

## 🚧 Limitations and Future Work

- DOM length still challenges context size; plan to use memory module
- No planning module included yet
- Aim to extend to multi-modal and multilingual settings

---

**GitHub:** [https://github.com/colonylabs/ScribeAgent](https://github.com/colonylabs/ScribeAgent)
