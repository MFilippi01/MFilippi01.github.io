---
title: "Multi-Agent Framework for Natural Language-Based Human-Robot Collaboration"
collection: projects
permalink: /projects/multi-agent-framework/
date: 2026-02-23
layout: single
author_profile: true
toc: true
toc_sticky: true
classes: wide
excerpt: "Master's thesis on a configuration-driven multi-agent framework for robotic task planning and execution."
---

<p>
<a href="/files/multi-agent-framework-thesis.pdf" class="btn btn--primary" target="_blank">Download Full Document (PDF)</a>
<a href="/files/multi-agent-framework-executive-summary.pdf" class="btn btn--info" target="_blank">Download Executive Summary (PDF)</a>
</p>

## Overview

Robotic systems increasingly require intuitive interfaces that allow users to specify tasks at a high level of abstraction. Natural language provides a flexible solution, but translating linguistic instructions into reliable robotic actions remains challenging, as language models are not inherently grounded in the physical world.
This project presents a **configuration-driven multi-agent framework** that bridges natural language instructions and deterministic robotic execution. The architecture decomposes the system into specialized agents responsible for scene representation, planning, parameter extraction, object grounding, and execution control, ensuring a clear separation between probabilistic reasoning and deterministic components.
High-level plans are generated through **coordinated interactions with locally deployed Large Language Models**, combined with a **proposal–validation–correction mechanism** that improves planning reliability. Object grounding is addressed through a hybrid strategy that translates linguistic references into structured representations and resolves them deterministically within the scene model.
The framework was implemented on a real robotic setup and evaluated through model comparison, scaling analysis across different LLM sizes, and a study of planning failure modes.

## System Pipeline

The proposed framework transforms high-level natural language instructions into executable robotic actions through a structured multi-stage pipeline.  
Rather than directly mapping language to control commands, the system decomposes the process into a sequence of coordinated modules responsible for perception, planning, validation, parameter extraction, and execution.
The overall architecture follows a multi-agent design in which each component operates on structured inputs and produces explicitly defined outputs. This separation ensures controlled information flow between probabilistic reasoning modules based on Large Language Models and deterministic components responsible for validation and execution.

![Complete System Pipeline](/images/framework_pipeline.jpg)

### 1. Configuration Activation

The execution flow begins with loading and parsing the configuration file that defines the operational parameters of the system.  
This configuration specifies:
- the available primitive actions
- the perception models used for object detection
- the attributes included in the scene description
- the parameters governing robot execution and reasoning modules

Based on these definitions, the framework dynamically initializes the required components, including the robotic interface, perception models, and the locally deployed language model used for reasoning.
At the same time, the robotic manipulator is moved to a predefined **home position**, ensuring a consistent reference state for the first scene acquisition.

### 2. Scene Perception and Representation

Once the system is initialized, an image of the workspace is acquired from the home position.  
Using the perception models defined in the configuration, the framework detects the objects present in the environment and constructs a structured **scene representation**.
This representation maintains:
- objects present in the scene
- object attributes
- spatial relationships
- task-relevant states

Alongside the structured representation used internally by the system, a simplified **textual scene description** is generated to serve as contextual input for the reasoning modules.

### 3. Plan Generation

The planning agent receives:
- the user instruction
- the textual scene description
- the action schema defined in the configuration
- the selected locally deployed language model

Based on these inputs, the planning module interprets the instruction as a **goal specification** rather than as a direct command.  
The language model generates an ordered sequence of elementary actions that is consistent with the current scene state and the admissible action set.
The resulting sequence forms the **operative plan**, which is then presented to the operator for validation before further processing.

### 4. Full-Plan Simulation Loop

After the plan is confirmed, the framework enters a simulation phase designed to verify the entire sequence before any physical execution occurs.
During this stage the system iterates through the actions in the plan sequentially:
1. The **Action Selection Agent** identifies the corresponding execution module for the current action.
2. The **Action Parameter Extraction Agent** extracts the parameters required for execution.
3. Instead of executing the action physically, the system **updates the internal scene representation as if the action had been performed**.

This simulated execution allows the system to detect inconsistencies such as:
- parameter extraction failures  
- invalid object references  
- constraint violations
If any issue is detected, the plan is aborted before the robot performs any real action.

### 5. Physical Execution

Only after the full simulation cycle completes successfully does the system proceed to physical execution.
The actions are executed sequentially using the parameters previously extracted during the simulation phase.  
Before each execution step, compatibility checks ensure that the selected action is consistent with the current robot configuration and the required end-effector.
After every action, the scene representation is updated to maintain consistency between the internal model and the physical environment.



Through this structured pipeline, natural language reasoning, environmental perception, and robotic execution remain tightly coordinated while preserving clear architectural boundaries between probabilistic reasoning components and deterministic control modules.


## Plan Generation Mechanism

The transformation of a natural language instruction into an executable robotic plan is handled by the **Planning Agent**.  
This component receives three inputs: the user instruction, the textual description of the current scene, and the set of admissible actions defined in the configuration file. Based on this information, it generates an **operative plan**, represented as an ordered sequence of textual action statements.
To improve robustness when using locally deployed language models, plan generation is not performed through a single inference step. Instead, the system relies on a **structured sequence of coordinated LLM calls**, where each interaction with the model is assigned a specific role within the reasoning process.

### Stage 1 — Instruction Explicitization

The first stage converts the user instruction into a **fully explicit sequence of actions**, expanding implicit operations and enumerating all required steps.
This process is implemented through a **proposal–validation–correction mechanism** based on coordinated interactions with the language model.

![First-Stage Planning Mechanism](/images/plan_generation_stage1.jpg)

First, the model produces a candidate decomposition of the task into a sequence of actions. A second LLM call then verifies the generated plan against the instruction, scene description, and available action set. If inconsistencies are detected, a third call produces a corrected version of the plan.  
This staged interaction significantly improves planning reliability by introducing explicit intermediate validation and reducing hallucination effects.

### Stage 2 — Primitive Action Normalization

In the second stage, the explicit plan is refined to enforce strict compliance with the action schema defined in the configuration file. Each action statement is normalized into its canonical format so that the resulting sequence can be directly interpreted by the downstream execution components of the framework.



## Object Grounding

For manipulation actions such as **pick, place, and pour**, the robot requires a target pose expressed in workspace coordinates.  
In the considered framework, however, these poses are not directly specified by the user. Instead, actions reference objects through natural language expressions (e.g., *“pick the cube on the right”*). As a consequence, parameter extraction reduces to the problem of **object grounding**, namely linking a linguistic description to a specific entity in the scene.
To address this challenge, the framework adopts a **hybrid grounding strategy** that combines language-based interpretation with deterministic resolution. The process is structured into three stages.

### 1. Isolation of the Object Description

The first step extracts the portion of the action statement that describes the target object.  
This operation is deterministic because the Planning Agent generates actions following predefined syntactic patterns, allowing the system to isolate the object description without probabilistic parsing.

### 2. Translation into a Structured Representation

The extracted description is then provided to a **dedicated LLM call**, whose role is not to identify the object directly but to translate the linguistic expression into a **structured symbolic representation**.  
The output consists of nested dictionaries that encode object classes, attributes, and relational constraints. This representation captures the semantic structure of the description while remaining compatible with deterministic processing.

The figure below shows an example of the structured representation generated for the object description:
*“the cube closest to the occupied slot farthest from the free cube on the right”*.

![Example of LLM structured grounding output](/images/grounding_example_output.jpg)

### 3. Deterministic Resolution

Once the structured representation is obtained, the framework resolves it deterministically against the internal scene representation.  
The resolver evaluates the nested constraints from the innermost object outward, progressively filtering candidate objects based on class, attributes, and spatial relations. This process ultimately identifies the **unique object ID** corresponding to the linguistic reference. The associated robot pose is then retrieved from the scene representation and used as the execution parameter for the corresponding manipulation action.


## Experimental Evaluation

The proposed framework was experimentally evaluated on a real robotic setup implementing the manipulation primitives **pick**, **place**, **mix**, and **pour**. The experiments were designed to assess the reliability and computational performance of the language-driven pipeline, including the **Planning Agent**, and the **Parameter Extraction Agent**.
A total of **500 test executions** were conducted, covering a wide range of natural language instructions and scene configurations. The evaluation focuses on three main aspects: comparative performance across different locally deployed language models, scaling behavior with increasing model size, and a detailed analysis of planning failure modes within the system.

### Model Comparison

To evaluate the influence of the language model on the reasoning components of the framework, a comparative analysis was conducted across four locally deployed instruction-tuned LLMs.  
The evaluation considers the estimated accuracy of three modules: the **Planning Agent**, the **Object Grounding module**, and the **Mixing Speed Extraction Agent**.

The results highlight that **Qwen2.5-7B-Instruct-AWQ achieves the best performance across the evaluated modules**, confirming its suitability for structured reasoning tasks within the proposed architecture.

| Model | Planning Agent Accuracy (p) | Object Grounding Accuracy (p) | Mixing Speed Extraction Accuracy (p) |
|------|-----------------------------|-------------------------------|--------------------------------------|
| Qwen2.5-7B-Instruct-AWQ | 0.870 | 0.929 | 1.000 |
| Mistral-7B-Instruct-v0.2-AWQ |  0.689 | 0.451 | 0.933 |
| Meta-Llama-3.1-8B-Instruct-AWQ-INT4 | 0.698 |  0.774 | 0.987 |
| DeepSeek-Coder-6.7B-Instruct-AWQ | 0.620 | 0.894 | 1.000 |

### Performance Scaling

To analyze the impact of model size on both reasoning accuracy and computational performance, an additional evaluation was conducted using four models from the **Qwen2.5 instruction-tuned family**.  
The objective of this experiment is to investigate the trade-off between **accuracy and inference throughput**, measured in terms of average token generation speed.

The analysis considers the three main reasoning modules of the framework: the **Planning Agent**, the **Object Grounding module**, and the **Mixing Speed Extraction Agent**.

#### Planning Agent

| Model | Planning Agent Accuracy (p) | Average Tokens per Second [s] |
|------|-----------------------------|--------------------------------|
| Qwen2.5-0.5B-Instruct | 0.068 | 175.41 |
| Qwen2.5-1.5B-Instruct | 0.326 | 77.27 |
| Qwen2.5-3B-Instruct | 0.542 | 54.53 |
| Qwen2.5-7B-Instruct-AWQ | 0.870 | 43.09 |

#### Object Grounding

| Model | Object Grounding Accuracy (p) | Average Tokens per Second [s] |
|------|--------------------------------|--------------------------------|
| Qwen2.5-0.5B-Instruct | 0.191 | 198.75 |
| Qwen2.5-1.5B-Instruct | 0.609 | 88.98 |
| Qwen2.5-3B-Instruct | 0.923 | 58.68 |
| Qwen2.5-7B-Instruct-AWQ | 0.929 | 43.63 |

#### Mixing Speed Extraction

| Model | Mixing Speed Accuracy (p) | Average Tokens per Second [s] |
|------|----------------------------|--------------------------------|
| Qwen2.5-0.5B-Instruct | 0.400 | 123.86 |
| Qwen2.5-1.5B-Instruct | 0.747 | 69.96 |
| Qwen2.5-3B-Instruct | 1.000 | 47.97 |
| Qwen2.5-7B-Instruct-AWQ | 1.000 | 41.14 |



The following figures illustrate the trade-off between **reasoning accuracy and inference throughput** across the Qwen model family.

<div style="display: flex; gap: 20px; justify-content: center; align-items: flex-start;">

<div style="text-align: center;">
<img src="/images/accuracy_vs_model_size.png" width="450">
</div>

<div style="text-align: center;">
<img src="/images/tokens_vs_model_size.png" width="450">
</div>

</div>

### Planning Agent Failure Mode Analysis

To better understand the internal limitations of the planning component, the behavior of the **Planning Agent** was analyzed separately across its two sequential stages: **Instruction Explicitization** and **Primitive Action Normalization**.  
This analysis makes it possible to identify where planning errors originate and how they propagate within the overall reasoning pipeline.

#### First Stage — Instruction Explicitization

The first stage is responsible for transforming the user instruction into a fully explicit action sequence through a coordinated proposal–validation–correction mechanism.  
Failure modes at this level are mainly associated with errors in task decomposition, missing or redundant actions, and inconsistencies between the generated plan and the scene constraints.

<div style="text-align: center; margin-bottom: 30px;">
<img src="/images/first_stage_failure_modes.PNG" width="500">
<br>
<em>Failure mode distribution for the first stage of the Planning Agent.</em>
</div>

The diagram highlights the dominant sources of error in the explicitization process, showing how the most critical failures arise before action normalization and parameter extraction.

#### Second Stage — Primitive Action Normalization

The second stage refines the explicit plan by converting each action sequence into a normalized representation compliant with the action schema defined in the configuration file.  
At this level, failure modes are primarily related to formatting inconsistencies, incorrect action canonicalization, and incompatibilities with the admissible primitive action set.

<div style="text-align: center; margin-bottom: 30px;">
<img src="/images/second_stage_failure_modes.PNG" width="500">
<br>
<em>Failure mode distribution for the second stage of the Planning Agent.</em>
</div>

The second-stage analysis complements the previous one by showing which errors are introduced during normalization, thus providing a clearer view of the internal robustness of the planning pipeline.

## Key Contributions

The main contributions of this work can be summarized as follows:
- **Configuration-driven multi-agent architecture** that integrates natural language reasoning with deterministic robotic execution through clearly separated system components.
- **Structured LLM planning pipeline** based on coordinated model calls and a proposal–validation–correction mechanism that improves robustness in task decomposition.
- **Hybrid object grounding approach** combining language-based schema extraction with deterministic constraint-based resolution on the structured scene model.
- **Full-plan simulation mechanism** that validates the entire action sequence before physical execution, ensuring consistency and preventing execution errors.
- **Experimental evaluation of locally deployed LLMs**, including model comparison, performance scaling analysis, and failure mode analysis of the planning process.
