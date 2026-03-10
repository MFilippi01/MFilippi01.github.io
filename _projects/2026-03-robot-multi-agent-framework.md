---
title: "Multi-Agent Framework for Natural Language-Based Human-Robot Collaboration"
collection: projects
permalink: /projects/multi-agent-framework/
date: 2026-03-01
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

Robotic systems increasingly require intuitive interaction mechanisms that allow human users to specify tasks at a high level of abstraction. Natural language provides a flexible interface for this purpose, but translating linguistic instructions into reliable robotic actions remains a challenging problem. Language models can interpret complex instructions, yet their outputs are not inherently grounded in the physical world and may produce plans that are syntactically correct but operationally infeasible.

This project presents a **configuration-driven multi-agent framework** designed to bridge the gap between natural language instructions and deterministic robotic execution. The proposed architecture decomposes the overall system into specialized agents responsible for scene representation, high-level planning, action parameter extraction, object grounding, and execution control. By separating probabilistic reasoning modules from deterministic execution components, the framework ensures structured information flow and improves system robustness.

A continuously maintained scene representation stores objects, attributes, spatial relations, and task states, providing a consistent reference for reasoning and decision-making. High-level plans are generated through coordinated interactions with **locally deployed Large Language Models (LLMs)**, enabling on-device reasoning without reliance on external cloud services, which progressively constructs structured action sequences. To ensure reliability, the planning process incorporates a **proposal–validation–correction mechanism** that verifies generated plans before execution.

Object grounding is addressed through a hybrid strategy that combines language-based interpretation with deterministic matching against the maintained scene model. Linguistic object references are first transformed into structured schemas by the language model and then resolved through a constraint-based matching process.

The framework has been implemented and evaluated on a real robotic setup. Experimental results include a comparative analysis of different locally deployed language models, an investigation of performance scaling across model sizes, and a detailed analysis of planning failure modes within the system. The results demonstrate that structuring language-driven reasoning within a modular architecture significantly improves reliability and interpretability in robotic task planning.

## System Pipeline

The proposed framework transforms high-level natural language instructions into executable robotic actions through a structured multi-stage pipeline.  
Rather than directly mapping language to control commands, the system decomposes the process into a sequence of coordinated modules responsible for perception, planning, validation, parameter extraction, and execution.

The overall architecture follows a multi-agent design in which each component operates on structured inputs and produces explicitly defined outputs. This separation ensures controlled information flow between probabilistic reasoning modules based on Large Language Models and deterministic components responsible for validation and execution.

![Complete System Pipeline](/images/complete_pipeline.jpg)

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

---

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

![Example of LLM structured grounding output](/images/grounding_example_output.jpg)

The figure above shows an example of the structured representation generated for the object description:

*“the cube closest to the occupied slot farthest from the free cube on the right”*.

### 3. Deterministic Resolution

Once the structured representation is obtained, the framework resolves it deterministically against the internal scene representation.  

The resolver evaluates the nested constraints from the innermost object outward, progressively filtering candidate objects based on class, attributes, and spatial relations. This process ultimately identifies the **unique object ID** corresponding to the linguistic reference. The associated robot pose is then retrieved from the scene representation and used as the execution parameter for the corresponding manipulation action.
