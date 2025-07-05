# **Definitive Development: The Agent-Driven Methodology**

Welcome to the DefinitiveSpec hub. This repository contains the core documents defining our structured, AI-augmented approach to software development, known as the Definitive Development Methodology (DDM).

The central idea is to transform complex software requirements into verified, high-quality code by creating precise, interconnected specifications (`.dspec` files) that act as a single source of truth. These specifications are used to direct a powerful, stateful AI agent (`DSAC`) that assists in strategic analysis, simulation, and implementation.

This document provides an index and guide to the essential documents.

---

## **Core Documents**

This methodology is defined by three key documents, each serving a distinct purpose and audience.

### 1. 📄 `methodology_guide.md` - The User Manual & Onboarding Guide

*   **Purpose:** This is the primary guide for understanding and using DDM on the PrimeCart project. It explains the "why" behind the v4.1 methodology, details the core specification artifacts (`requirement`, `model`, `code`, etc.) with rich examples, and walks through the modern, interactive development lifecycle.
*   **Audience:** **Everyone.** This is the **"START HERE"** document for all new team members, regardless of role.
*   **When to Read:** Read this first to get a comprehensive understanding of the DDM process, how to write effective specs, and how to collaborate with the v4.1 agent.
*   **Key Contents:**
    *   **Chapter 1-2:** The core principles of DDM v4.1 (Verifiable Certainty, AI as a Strategic Partner) and the tooling ecosystem.
    *   **Chapter 3:** A detailed breakdown of all DSpec artifact types with complete examples from the PrimeCart application.
    *   **Chapter 4:** The 5-phase DDM Lifecycle and a practical workflow illustrating modern commands like `Explore Idea` and `Implement Code Spec` (with its Test-First protocol).
    *   **Appendices:** Common pitfalls and best practices for interacting with the interactive v4.1 AI agent.

### 2. ⚙️ `dspec_agent_context_new.md` - The Agent's Identity & Operational Contract

*   **Purpose:** This is the complete, normative, and machine-readable contract for the `DSAC v4.1 Autonomous AI Agent`. It is not prose; it is a specification that defines the agent's core identity, its mandatory operational lifecycle, its capabilities, and its knowledge base.
*   **Audience:**
    *   **Primary:** The AI Agent itself. This file is the primary input for every task.
    *   **Secondary:** Architects and Tech Leads responsible for governing the project's technical patterns via the `Architectural Profile`.
    *   **Tertiary:** Developers who need to look up the exact template for a specific `pattern`.
*   **When to Read:** Read this when you need the absolute ground truth about the agent's behavior. Architects will read this when defining or evolving the project's `Architectural Profile`.
*   **Key Contents:**
    *   **Agent Identity:** The core **Principle of Verifiable Certainty** and the `[Fact]`/`[Assumption]` communication protocol.
    *   **Part 1: The DDM Operational Lifecycle:** The mandatory 5-phase lifecycle (`Sync`, `Focus`, `Analyze`, `Execute`, `Verify`) that governs all agent actions.
    *   **Part 2: Knowledge Base:** The schemas for all DSpec artifacts.
    *   **Part 3: Implementation Library:** The "Cookbook" of `pattern`s and `directive`s.
    *   **Part 4: Foundational Grammar (EBNF):** The required syntax for all `.dspec` files.

### 3. 🗺️ `agent_guide.md` - Tactical Field Manual & SOPs

*   **Purpose:** A collection of role-specific, tactical documents outlining Standard Operating Procedures (SOPs) for specific, advanced tasks within the v4.1 framework.
*   **Audience:** Role-specific (Developers, Architects, Team Leadership).
*   **When to Read:** Read the relevant section before performing one of the specific tasks described. It's a field manual, not a book to be read cover-to-cover.
*   **Key Contents:**
    *   **Operator's Field Manual:** Step-by-step instructions for developers on using the v4.1 agent's core commands (`Explore Idea`, `Analyze Impact`, `Implement Code Spec`, `Reconcile Spec with Code`, etc.).
    *   **Architectural Profile Governance Process:** An SOP for architects on how to propose, review, and approve changes to the project's core patterns.
    *   **v4.1 Agent Compliance Test Suite:** A set of `test` specs to verify that the agent correctly adheres to its operational contract.

---

## **The Methodology in a Nutshell**

This DDM version is built on these core principles:

*   **Verifiable Certainty is Paramount:** The agent operates only on facts derived from specs and will `[PAUSE]` to ask for clarification when faced with ambiguity.
*   **AI as a Strategic Partner:** The agent is stateful and maintains a `Project Index`, enabling it to perform deep impact analysis and assist with strategic planning.
*   **The Specification is the Single Source of Truth:** The `.dspec` files define everything.
*   **Interactive Refinement & The Feedback Loop:** The process is a collaborative conversation. The agent `[PAUSE]`s with a `[RECOMMENDATION]` when it finds an issue, empowering the operator to make the final decision.

### The DDM Lifecycle
```
+---------------------------------------------------------------------------------+
|   (Operator Command: "Explore Idea", "Implement Code Spec", etc.)               |
|                                       |                                         |
|                                       v                                         |
|  [ Phase 0: Sync ] -> Agent loads/updates Project Index from all .dspec files   |
|                                       |                                         |
|                                       v                                         |
|  [ Phase 1: Focus ] -> Identifies relevant specs for the task                   |
|                                       |                                         |
|                                       v                                         |
|  [ Phase 2: Pre-Generation Analysis ] -> Runs guardrails (Security, N+1, etc.)  |
|      ^                                |                                         |
|      | (If PAUSE, await Operator)     v                                         |
|      +--------------------------------+         (Interactive Feedback Loop)     |
|                                                                                 |
|  [ Phase 3: Core Task Execution ] -> Runs command (generates code, runs analysis)|
|                                       |                                         |
|                                       v                                         |
|  [ Phase 4: Post-Generation Verification ] -> IntegrityVerifier self-checks output |
|                                       |                                         |
|                                       v                                         |
|  [ Phase 5: System Refinement ] -> Operator-driven learning via "Distill Pattern"|
|                                       |                                         |
|                                       v                                         |
|   (Final Response: Generated artifacts, analysis reports, etc.)                 |
+---------------------------------------------------------------------------------+
```

---

## **Recommended Reading Path**

To get started with the DDM and the DSAC agent, follow this path based on your role:

#### **For All Roles (Developers, QAs, PMs, Architects):**
1.  **Start Here:** Read the `methodology_guide.md` to understand the core philosophy, the artifacts, and the interactive workflow.

#### **For Developers:**
1.  **After the Guide:** Read the **"Operator's Field Manual"** in `agent_guide.md`. This is your tactical guide for using commands like `Implement Code Spec` and understanding the `[PAUSE]` protocol.
2.  **Reference as Needed:** When writing `detailed_behavior`, refer to **Part 3 (Implementation Library)** of `dspec_agent_context_new.md` to see the available `pattern` keywords and how they work.

#### **For Architects & Tech Leads:**
1.  **Read Everything:** You are the custodians of the methodology and its technical implementation.
2.  **Master:** The `dspec_agent_context_new.md` is your primary domain. You own the patterns that will go into the project's `Architectural Profile`.
3.  **Implement:** The **"Architectural Profile Governance Process"** in `agent_guide.md` is your SOP for managing architectural patterns, often informed by the `Distill Pattern from...` command.

#### **For Product Owners & QA Engineers:**
1.  **Primary Document:** The `methodology_guide.md` is your main resource. Focus on how to write clear `requirement`, `kpi`, and `test` specifications.
2.  **Reference for Tasks:** Refer to the **"Operator's Field Manual"** in `agent_guide.md` to understand how to use strategic commands like `Analyze What-If` and `Run Simulation`.