Below is the plain text conversion of the core components, layout, and content of the paper **arXiv:2310.08560**, titled **"MemGPT: Towards LLMs as Operating Systems"**.

---

# MemGPT: Towards LLMs as Operating Systems

**Authors:** Charles Packer, Sarah Wooders, Kevin Lin, Vivian Fang, Shishir G. Patil, Ion Stoica, Joseph E. Gonzalez

**Affiliation:** University of California, Berkeley

**Date / Version:** Originally published October 2023; updated February 2024 (arXiv:2310.08560v2)

---

## Abstract

Large language models (LLMs) have revolutionized AI but are severely constrained by limited context windows, hindering their utility in tasks like extended conversations and document analysis. To enable using context beyond limited context windows, we propose **virtual context management**, a technique drawing inspiration from hierarchical memory systems in traditional operating systems (OSes) that provide the appearance of large memory resources through data movement between fast and slow memory.

Using this technique, we introduce **MemGPT (Memory-GPT)**, a system that intelligently manages different memory tiers in order to effectively provide extended context within the LLM's limited context window, and utilizes interrupts to manage control flow between itself and the user. We evaluate our OS-inspired design in two domains where the limited context windows of modern LLMs severely handicaps their performance:

1. **Document analysis**, where MemGPT is able to analyze large documents that far exceed the underlying LLM's context window.
2. **Multi-session chat**, where MemGPT can create conversational agents that remember, reflect, and evolve over long-term interactions.

---

## 1. Introduction

In recent years, large language models (LLMs) and their underlying transformer architecture have become the cornerstone of conversational AI, leading to a wide array of consumer and enterprise applications. However, directly extending the context length of transformers incurs a quadratic increase in computational time and memory cost due to the self-attention mechanism, making the design of new long-context architectures a pressing research challenge.

In this paper, we study how to provide the illusion of an infinite context while continuing to use fixed-context models. Our approach borrows from the idea of **virtual memory paging**, developed to enable applications to work on datasets that far exceed the available memory by paging data between main memory and disk. We leverage the recent progress in function calling abilities of LLM agents to design **MemGPT**, an OS-inspired LLM system for virtual context management.

---

## 2. MemGPT Architecture

In MemGPT, we treat context windows as a constrained memory resource and design a memory hierarchy for LLMs analogous to memory tiers used in traditional OSes.

### 2.1 Memory Hierarchy

MemGPT's OS-inspired multi-level memory architecture delineates between two primary memory types:

* **Main Context (Analogous to RAM / Physical Memory):** Consists of the immediate LLM prompt tokens. This is the active context window available to the LLM during an inference cycle.
* **External Context (Analogous to Disk Storage):** Holds information outside the LLM’s immediate context window. This acts as a larger, slower database containing deep history or massive document data that must be intentionally brought into main context to be read.

Main Context is further sub-divided into:

1. **System Instructions (Read-Only):** Static prompts containing instructions on MemGPT control flow, memory tier definitions, and directions for invoking functions.
2. **Core Memory:** A fixed-size writable slot containing persistent state variables (e.g., information about the user, persona definitions, or scratchpads for long-term tracking).
3. **FIFO Queue (Working Context):** Stores chronological message sequences (user messages, system events, and model outputs).

### 2.2 Mechanism and Control Flow

Instead of returning control to the user immediately after a single text completion, MemGPT utilizes an **event-driven loop managed by system interrupts**.

1. **Inference Trigger:** When an incoming message arrives, the queue manager appends it to the FIFO queue, concatenates the prompt tokens, and triggers the LLM inference.
2. **Function Calling & Memory Management:** The LLM executes reasoning and determines if it needs to access external storage or alter its core memory. It uses structured function calls to page data in and out of the context window.
3. **Memory Pressure Warnings:** When prompt tokens exceed a "warning token count" (e.g., 70% of the maximum window size), the queue manager injects a system message warning the LLM of an impending queue eviction. This memory pressure signal prompts the LLM to yield or compress its current information into external storage using MemGPT functions before data is lost.

---

## 3. Evaluated Domains

To demonstrate the utility of an OS-inspired system, MemGPT was evaluated across two key tasks constrained by finite context limits:

### 3.1 Document Analysis

Standard long-form text files quickly exceed the input capacities of modern LLMs. MemGPT bypasses this restriction by maintaining the document database inside its **External Context**. It runs iterative search, pagination, and retrieval functions to scan vast documents, pull chunks into the main context queue as needed, synthesize responses, and successfully execute deep-text queries over massive semantic records.

### 3.2 Multi-Session Conversational Agents

In traditional multi-session chat, conversations eventually forget older interactions due to truncation limitations. MemGPT addresses this by allowing conversational agents to continually write important insights to their core memory slots or page historical logs out to archival disk arrays. This architecture enables the model to actively remember, reflect, and maintain a consistent relationship with a user across an unbounded number of interaction cycles.

---

## 4. Key Takeaways

* **Virtual Context Window:** Decouples the physical constraints of an LLM's architecture from its functional memory capacity.
* **Autonomous Operation:** Shifts memory management responsibility to the LLM itself via automated tool/function execution.
* **Operating System Analogy:** Models prompt engineering and contextual data streaming exactly like CPU/RAM architectures routing operations to external storage volumes.

---

### References

* Packer, C., Wooders, S., Lin, K., Fang, V., Patil, S. G., Stoica, I., & Gonzalez, J. E. (2023). MemGPT: Towards LLMs as Operating Systems. *arXiv preprint arXiv:2310.08560*.
* Cited by: 786
