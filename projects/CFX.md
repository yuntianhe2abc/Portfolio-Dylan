---
layout: default
title: CFX Engineering Rule Validation System
---

# CFX Engineering Rule Validation System

## Table of Contents

- [Overview](#overview)
- [Project Index](#project-index)
- [Problem](#problem)
- [Core Workflow](#core-workflow)
- [System Design](#system-design)
- [Reliability Design](#reliability-design)
- [My Role](#my-role)
- [Business Value](#business-value)
- [Tradeoffs and Limitations](#tradeoffs-and-limitations)
- [Project Summary](#project-summary)
- [Back to Portfolio](#back-to-portfolio)

## Overview

CFX is an engineering rule validation system for a manufacturing scenario where important validation rules were not directly available as executable logic, even when the relevant specifications existed.

The system turns CFX specification documents, message definitions, field descriptions, cross-message relationships, and reviewed engineering rules into reusable knowledge assets, executable Python validators, and automated event-flow validation results.

The core value was not simple document retrieval. The main challenge was turning reviewed rules plus document-derived context into executable validator scripts through a controlled multi-step workflow.

## Project Index

| Area | Description |
| --- | --- |
| Domain | Manufacturing event-flow validation |
| Main users | Engineers validating CFX event flows and rule compliance |
| Main technologies | RAG, agentic workflows, Python validators, structured testing, API-based runtime validation |
| Core output | Executable validator assets with checks, tests, diagnosis, and gated save |
| Main engineering theme | Turning ambiguous engineering knowledge into executable and reusable validation logic |

## Problem

Many engineering rules were distributed across specifications, message definitions, field descriptions, and cross-event relationships. Some rules were implicit rather than written as ready-to-run validation logic.

That created several gaps:

- engineers had to reconstruct rules from scattered specification context
- some rules only became clear after reading multiple messages together
- flow-level semantics mattered more than isolated message definitions
- rule understanding still had to be translated into executable validation logic
- generated code needed to be checked, tested, and promoted carefully before reuse

For this reason, I treated CFX as a rule operationalization system rather than a document QA system.

## Core Workflow

The validator-generation workflow includes:

1. Rule and context grounding
2. Control-type inference when required by rule preconditions
3. Validator generation
4. Syntax and import checks
5. Repair when basic checks fail
6. Rule-specific test-pack generation
7. Schema validation for generated test packs
8. Integration testing between the validator and the test pack
9. Failure diagnosis
10. Targeted retry, repair, or regeneration
11. Gated save only after defined checks pass

This structure made the workflow slower than direct prompt-to-code generation, but it created clearer control points and reduced the risk of saving low-confidence validator assets.

## System Design

The system can be understood as four connected layers.

### Knowledge Extraction and Research

This layer extracts usable engineering context from specification documents and related materials.

Message-level research focuses on:

- message meaning
- field definitions
- required fields
- local constraints

Cross-message and cross-event research focuses on:

- prerequisites
- ordering relationships
- flow dependencies
- process conditions

### Rule Understanding and Consolidation

This layer turns extracted findings into reusable rule assets. Instead of stopping at document search, it consolidates implicit or scattered rule knowledge into reviewed rules and structured intermediate assets that downstream workflows can use.

### Agentic Validator Generation

This is the execution-oriented part of the system. It takes reviewed rules and specification-derived context and turns them into executable Python validators through bounded generation, checking, repair, testing, diagnosis, and save-promotion steps.

### Runtime Validation

The runtime layer exposes validation capability through APIs and flow-checking endpoints. It can generate validators for individual rules, validate a single event flow, or validate a larger production flow across selected events or rules.

## Reliability Design

The reliability design focused on avoiding a common failure mode in LLM systems: producing output that looks plausible but is not operationally trustworthy.

Key choices included:

- separating generation, checking, testing, diagnosis, and saving into distinct stages
- using deterministic syntax and import checks before deeper validation
- generating rule-specific test packs instead of relying only on generic tests
- validating generated test-pack schemas before integration testing
- diagnosing whether failures likely came from the script, test pack, context, or rule ambiguity
- treating save as a promotion step rather than a simple file write

The most important principle was that generated code is not the product. Verified validator assets are the product.

## My Role

I led the project end to end, including user communication, problem framing, solution design, workflow design, and implementation.

My main technical contribution was designing and implementing the execution-oriented workflow that turns reviewed engineering rules and document-derived context into executable Python validators.

Specific contributions included:

- designed the layered agent-tool architecture
- defined bounded generation and repair workflows for validators and test packs
- introduced deterministic validation layers such as syntax checks, import checks, schema validation, and integration testing
- added failure diagnosis to distinguish between script issues, test-pack issues, and insufficient context
- implemented gated save so reusable validator assets were only promoted after passing required checks
- connected upstream rule and context assets to downstream executable validation

## Business Value

The project reduces the gap between reading specifications and validating real production event flows.

Main value areas:

- reduced manual effort in rule lookup and reconstruction
- reusable engineering knowledge and reviewed rule assets
- a path from reviewed rules to executable validation logic
- improved traceability in validation workflows
- stronger support for automation in engineering rule checking

## Tradeoffs and Limitations

The design intentionally favored controllability over maximum autonomy.

Key tradeoffs:

- **Reliability vs throughput:** more checks and gates reduced output speed but improved artifact quality
- **Bounded control vs open exploration:** explicit stop conditions made the workflow easier to debug but less flexible
- **Structured intermediates vs simpler architecture:** reviewed rules and context bundles added maintenance cost but improved downstream inputs
- **Behavioral testing vs absolute certainty:** generated tests improved confidence but did not prove full correctness
- **Asset quality vs convenience:** gated save prevented low-confidence validators from polluting the reusable asset pool

Known limitations:

- some rules remain too ambiguous to compile into a reliable validator automatically
- generated tests can share the same misunderstanding as generated validators
- validators that pass generated tests may still miss real-world edge cases
- upstream rule extraction quality directly affects downstream validator quality
- grounding reduces semantic drift but does not eliminate it completely

## Project Summary

CFX is an engineering rule validation system for manufacturing event flows. The hardest part was not retrieval, but converting scattered specification knowledge and reviewed rules into executable, testable, reusable validator assets.

I designed a layered agent-tool workflow that grounds rules, generates validators, checks imports and syntax, creates rule-specific tests, validates test schemas, runs integration tests, diagnoses failures, and only saves validators after defined checks pass. The project demonstrates engineering judgment around reliability, failure handling, and asset promotion rather than only model prompting.

## Back to Portfolio

[Return to the portfolio home page](../)
