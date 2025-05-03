# IMS QTI 2.1 Assessment Package Generator

[![Status: Planning](https://img.shields.io/badge/status-planning-blue)](./docs/Agentic_Workflow_Strategy.md)

**Current Status (April 28, 2025):** This project is currently in the **pre-implementation planning phase**. The core architecture, technology stack, and a detailed **Agent-Assisted Development Strategy** have been defined, but code implementation (Stage 1 execution onwards) has not yet begun.

## Overview

This project aims to develop a robust web application capable of generating IMS QTI 2.1 compliant assessment packages (ZIP files containing `assessmentItem`, `assessmentTest`, and `imsmanifest.xml`). The system will utilize Python, FastAPI, Pydantic, and Jinja2 to create standards-compliant packages based on structured input data, with a strong focus on validation and compatibility, particularly with **Blackboard Ultra**.

A key aspect of this project is the **agent-assisted development workflow**, leveraging state-of-the-art AI models (as of April 2025) as powerful assistants for tasks like code scaffolding, template generation, test creation, and analysis, while maintaining strict developer oversight, quality control, and adherence to specifications through mandatory Human-in-the-Loop (HITL) reviews and stage-gate verification processes.

## Problem Statement

Creating valid, interoperable, and feature-rich IMS QTI 2.1 assessment packages manually is complex and error-prone due to:

* Intricate XML schemas (QTI 2.1, IMS CP 1.2, LOM, MathML, XHTML).
* Strict validation requirements.
* Need for accurate content representation (especially MathML and assets).
* Platform-specific compatibility quirks (e.g., Blackboard Ultra).
* Challenges in managing unique identifiers and resource packaging.

This tool aims to automate the generation process, ensuring validity, consistency, and compatibility, thereby reducing errors and saving significant authoring time.

## Core Objectives

* **Strict QTI 2.1 & IMS CP 1.2 Compliance:** Generate packages validating against official XSD schemas.
* **Blackboard Ultra Compatibility:** Primary acceptance criterion is reliable import and function in BB Ultra.
* **Content Fidelity:** Accurately render prompts, choices, feedback, MathML, and assets.
* **Interaction Support:** Support a configurable range of QTI 2.1 interaction types (starting with core types).
* **Robustness:** Implement comprehensive input (Pydantic) and output (lxml) validation.
* **Maintainability & Extensibility:** Design a modular system for future updates.
* **Testability:** Ensure components are designed for unit and integration testing.

## Planned Technology Stack

* **Backend:** Python 3.x
* **Web Framework:** FastAPI (for built-in validation via Pydantic, async support, auto-docs)
* **Data Validation (Input):** Pydantic
* **Templating:** Jinja2 (with XML autoescaping)
* **XML Validation:** `lxml`
* **Packaging:** Standard Python `zipfile` module
* **Database:** PostgreSQL (Recommended)
* **Testing:** `pytest` (with plugins like `pytest-cov`, `pytest-asyncio`, `TestClient`)
* **MathML Conversion (Optional):** Pandoc
* **Async Task Queue (Optional):** Celery with Redis/RabbitMQ

## Development Approach: Agent-Assisted Workflow

This project will **not** be built via traditional manual coding alone, nor via fully autonomous AI generation. It will utilize a structured **4-Stage Agent-Assisted Development Workflow**:

1. **Stage 1: Specification & Foundation Refinement:** AI assists in analyzing requirements; Developer finalizes specs for Pydantic models, Jinja2 structure, validation logic.
2. **Stage 2: Core Component Implementation:** AI generates code/templates based on *precise* Stage 1 specs; Developer performs **mandatory HITL review** for correctness, quality, and security before integration.
3. **Stage 3: Integration & Testing:** Components are integrated; AI assists in generating unit/integration test boilerplate; Developer performs **mandatory HITL review** of tests and executes crucial **Manual Acceptance Testing (MAT)**, especially in Blackboard Ultra.
4. **Stage 4: Iteration & Refinement:** AI assists in debugging, refactoring, and implementing fixes/features based on Stage 3 testing feedback; Developer performs **mandatory HITL review** and re-testing.

**Key Principles:**

* **Developer Control:** The developer remains the architect, final reviewer, and quality gatekeeper.
* **HITL is Non-Negotiable:** All AI-generated artifacts (code, templates, tests, docs) undergo mandatory human review before use.
* **Specification Driven:** AI agents operate based on detailed, human-approved specifications derived from the project plan.
* **Stage-Gate Verification:** Strict verification checklists must be passed before proceeding from one stage to the next, ensuring foundational integrity.

*(See the detailed [Agentic Workflow Strategy](./docs/Agentic_Workflow_Strategy.md) document for the full plan - Link assumes strategy doc is saved here)*

## Project Stages (Planned)

The development is planned across these structured stages with verification gates:

1. **Stage 1:** Specification & Foundation Refinement
    * `=> Stage 1 Verification`
2. **Stage 2:** Core Component Implementation (Iterative)
    * `=> Stage 2 Verification`
3. **Stage 3:** Integration & Testing
    * `=> Stage 3 Verification`
4. **Stage 4:** Iteration & Refinement (Cyclical)
    * `=> Stage 4 Cycle Verification`

## Getting Started

*(Placeholder)*

Instructions for setting up the development environment, configuring the application, running tests, and generating packages will be added here once implementation begins (post-Stage 1 execution and verification).

Key setup steps will likely include:

* Cloning the repository.
* Setting up a Python virtual environment.
* Installing dependencies (`pip install -r requirements.txt`).
* Configuring environment variables (database, API keys if needed, paths).
* Obtaining QTI/CP XSD schemas.
* Running database migrations (if applicable).

## Contributing

*(Placeholder)*

This project is initially planned as a solo developer effort utilizing the defined agent-assisted workflow. Contribution guidelines may be established later if the project scope or team structure evolves.

## License

*(Placeholder - Choose one)*

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

*(Alternatively: License TBD)*
