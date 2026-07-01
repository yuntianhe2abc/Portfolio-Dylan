---
layout: default
title: RMA Intelligent Analysis and FA Report Automation System
---

# RMA Intelligent Analysis and FA Report Automation System

## Table of Contents

- [Overview](#overview)
- [Project Index](#project-index)
- [Problem](#problem)
- [Main Features](#main-features)
- [System Design](#system-design)
- [Core Technical Choices](#core-technical-choices)
- [My Role](#my-role)
- [Business Value](#business-value)
- [Tradeoffs and Limitations](#tradeoffs-and-limitations)
- [Project Summary](#project-summary)
- [Back to Portfolio](#back-to-portfolio)

## Overview

RMA is an intelligent workflow system for return material authorization and failure-analysis scenarios. It helps engineers move from incomplete customer complaint information to structured case intake, guided precheck, historical-case retrieval, fault confirmation, and final FA report generation.

The project is not a simple chatbot. It turns an experience-heavy engineering process into an orchestrated, traceable, and reusable workflow.

## Project Index

| Area | Description |
| --- | --- |
| Domain | RMA and failure analysis |
| Main users | FA and RMA engineers |
| Main technologies | FastAPI, LangGraph, RAG, structured outputs, tool calling, streaming responses |
| Core output | Guided case workflow and automated FA report generation |
| Main engineering theme | Turning fragmented troubleshooting into a controlled AI workflow |

## Problem

In real RMA workflows, engineers often face incomplete and inconsistent starting information. Customer complaints may be missing fields such as model type, serial number, return date, part number, or fault description.

The broader workflow also has several pain points:

- precheck steps depend heavily on individual engineering experience
- different engineers may follow different troubleshooting depth and order
- historical FA cases are hard to search quickly and reuse effectively
- fault confirmation requires both case context and supporting evidence
- final FA reports and image insertion create repetitive manual work

The design goal was to avoid collapsing this entire workflow into one unstructured conversation. The system needed clear stages, persistent case state, structured decisions, and explicit tool boundaries.

## Main Features

### Case Information Extraction

The system continuously extracts and completes structured case information from multi-turn conversations, including fields such as:

- `model_type`
- `client_model`
- `pn`
- `sn`
- `fault_description`

When required information is missing, the system asks follow-up questions instead of moving prematurely into deeper analysis.

### Guided Precheck

The precheck stage guides engineers through diagnostic steps based on case characteristics such as plug type, model type, and whether the issue is related to no-power behavior.

The system returns structured precheck results, including whether the precheck is complete and whether a fault cause has already been confirmed.

### Deep Search and Fault Confirmation

When precheck is not enough, the workflow enters a deeper analysis stage. It calls retrieval tools to search historical FA cases and uses explicit fault-confirmation tools to support the final decision.

The model is responsible for deciding when to call tools, while tools return bounded and structured outputs that drive downstream workflow state.

### FA Report Automation

After fault confirmation, the workflow supports FA report generation, including PPT creation, cloud upload, and image insertion into selected slides based on user instructions.

This closes the loop from intake and analysis to deliverable report output.

## System Design

The system uses FastAPI as the API layer to receive chat requests and stream responses. The core business workflow is orchestrated with LangGraph as a staged state machine.

The main workflow includes four nodes:

1. `routing`
2. `precheck`
3. `deep_search`
4. `generate_report`

The shared workflow state is stored in `RMAState`, which contains conversation history, case information, current stage, runtime context, and UI events.

For each request, the system restores the previous session state, extracts additional case fields from the latest user input, and then routes the case based on completeness and current workflow stage.

If information is incomplete, the system asks for missing fields. If the required context is ready, it enters precheck. If precheck confirms the fault, the system can move to report generation. Otherwise, it escalates to deep search and tool-supported fault confirmation.

## Core Technical Choices

### Staged Graph Instead of One Free-Form Agent

I used a staged graph because process order matters in RMA workflows. A single agent with all tools attached would be easier to demo, but harder to make predictable. It could jump into deep search too early, ask inconsistent follow-up questions, or produce fault conclusions before minimum case context is ready.

The graph structure gives each phase a clear responsibility and makes transitions explicit.

### Structured Outputs

The workflow uses structured outputs for information extraction and precheck results. This makes downstream routing more reliable because the system can act on machine-readable state instead of parsing free-form text.

For example, precheck output can include fields such as `reply`, `precheck_done`, and `fault_cause`.

### Explicit Tool Calls

Historical-case retrieval, fault confirmation, PPT generation, and image insertion are treated as explicit tools rather than hidden behavior inside a single model response.

This separates reasoning from execution:

- the model decides what action is needed
- the tool performs a bounded operation
- the workflow uses structured tool results to update state

### Session Persistence

The system persists state by `chat_id`, which is important because real RMA cases are usually advanced over multiple turns. Persistence makes the product behave like a case workflow rather than a set of isolated chat requests.

## My Role

I designed and implemented the system workflow, including problem framing, architecture, stage design, tool integration, and practical engineering concerns.

My main contributions included:

- abstracted the RMA process into a graph-based state machine
- separated case extraction, precheck, deep search, and report generation into modular workflow nodes
- designed a shared state object to connect multi-turn interaction and workflow decisions
- integrated RAG retrieval, fault confirmation, PPT generation, and image insertion as workflow tools
- supported streaming responses, exception fallback, database-backed session persistence, and multiple runtime modes

## Business Value

The project turns a fragmented and experience-heavy RMA process into a structured AI workflow.

Main value areas:

- improves case-handling efficiency
- reduces missed precheck steps
- increases reuse of historical FA knowledge
- lowers repetitive FA report-generation work
- helps the engineering team standardize troubleshooting knowledge
- creates a more traceable case workflow across multiple turns

## Tradeoffs and Limitations

Key tradeoffs:

- **Control vs flexibility:** a staged graph is less free-form than a general agent, but it makes workflow order and debugging clearer
- **Precheck discipline vs speed:** requiring precheck before deep search can add steps, but reduces premature conclusions
- **Structured outputs vs generation freedom:** schemas make routing more reliable, but introduce stricter failure modes
- **Tool boundaries vs integration complexity:** explicit tools are easier to observe and test, but require more orchestration
- **Persistence vs stateless simplicity:** saved workflow state improves continuity, but introduces versioning and recovery concerns

Known limitations:

- initial case extraction can miss or mis-normalize noisy customer input
- retrieval quality depends on case context completeness and historical-data quality
- deep search can still be influenced by strong but only partially relevant historical cases
- the current system would benefit from stronger formal evaluation and node-level observability
- confidence and escalation mechanisms could be made more explicit

## Project Summary

RMA is an intelligent workflow system for RMA and FA engineers. It helps move from incomplete customer complaints to structured case intake, guided precheck, historical-case retrieval, fault confirmation, and report generation.

I used FastAPI for the API layer and LangGraph for staged workflow orchestration. The key design choice was to model the process as controlled stages rather than a single open-ended agent. That allowed the system to enforce required information collection, use structured outputs for workflow decisions, call tools explicitly for retrieval and fault confirmation, persist case state across turns, and generate FA reports as the final deliverable.

The project demonstrates practical AI engineering because it focuses on workflow control, state management, tool boundaries, and production-oriented concerns rather than only generating smart-looking responses.

## Back to Portfolio

[Return to the portfolio home page](../)
