**Report: Agentic Workflow Strategy for IMS QTI 2.1 Generator Development**

**To:** Solutions Team Implementation (Solo Developer)
**From:** AI Workflow Architect & Agentic Systems Designer
**Date:** April 28, 2025
**Subject:** Strategy and Detailed Plan for Agent-Assisted Development of the IMS QTI 2.1 Assessment Package Generator

**1. Executive Summary**

This report outlines a strategy for developing the IMS QTI 2.1 Assessment Package Generator, leveraging state-of-the-art AI models (as of April 2025) as powerful assistants within a structured development workflow. The goal is **not** autonomous code generation but rather **agent-assisted development**, where AI significantly accelerates implementation while the developer maintains full control over architecture, quality, and final code via strategic Human-in-the-Loop (HITL) checkpoints. We will focus on using agents for tasks like code scaffolding, implementation of specific functions based on detailed specifications, Jinja2 template generation, test case creation, and documentation drafting, all while adhering strictly to the project plan's requirements (QTI 2.1, IMS CP 1.2, Pydantic models, Jinja2 templating, `lxml` validation, Blackboard Ultra compatibility).

**2. High-Level Strategy: Phased Agentic Assistance**

We will structure the development process into logical phases, integrating AI assistance strategically within each. The developer remains the architect and final arbiter.

1. **Phase 1: Specification & Foundation Refinement (Human-Led, AI-Assisted Review):** Solidify Pydantic models, Jinja2 template structure, and core logic definitions. Use AI for analysis and suggestion.
2. **Phase 2: Core Component Implementation (Agent-Assisted Coding & HITL Review):** Iteratively build core components (API endpoints, Pydantic models, Jinja2 templates, validation logic, packaging logic) using AI agents for code generation based on *precise* specifications, followed by mandatory developer review and refinement.
3. **Phase 3: Integration & Testing (Human-Led, AI-Assisted Test Generation):** Integrate components, write tests (potentially using AI to generate boilerplate or initial test cases), and perform rigorous manual testing (especially Blackboard Ultra validation).
4. **Phase 4: Iteration & Refinement (Feedback-Driven Agent Assistance):** Address bugs, refine templates based on testing (especially BB Ultra feedback), and implement additional features, using agents for targeted modifications.

**3. Detailed Agentic Workflow Plan**

This plan breaks down the development process into stages, outlining tasks, AI agent roles, recommended models, HITL integration points, and guardrails.

**Stage 1: Specification & Foundation Refinement**

* **Objective:** Finalize core data structures, template organization, and validation requirements based on the project plan.
* **Tasks:**
  * `TASK 1.1`: Review Project Plan for Key Constraints & Structures.
  * `TASK 1.2`: Define detailed Pydantic Models (Python code structure, validation rules).
  * `TASK 1.3`: Define Jinja2 Template Structure (base templates, interaction snippets, macros, includes).
  * `TASK 1.4`: Define Core Validation Logic requirements (XSD paths, error reporting needs).
* **Agentic Integration:**
  * **Agent Role:** Requirements Analyst Assistant. Use an agent to parse the project plan (Sections 4-10) and extract/summarize key technical requirements for Pydantic models, Jinja2 strategy, and validation logic. It can also suggest potential ambiguities or areas needing further definition.
  * **Example Prompt (TASK 1.1):** "Analyze the provided project plan (Sections 4-10) for the IMS QTI Generator. Extract and list all specific technical requirements and constraints related to: 1. Pydantic data models (input structure, validation). 2. Jinja2 templating (structure, namespaces, escaping, MathML handling, asset paths). 3. `lxml` validation (schemas, caching, error reporting). 4. `zipfile` packaging. Summarize these concisely."
  * **Recommended Model:** Claude 3.7 Sonnet/Pro (strong comprehension, summarization, handling complex documents), Gemini 2.5 Pro (large context window, strong reasoning).
* **HITL Integration:**
  * **Trigger:** Agent output (summary/suggestions).
  * **Purpose:** **Mandatory Review & Final Decision.** The developer reviews the agent's extraction for accuracy and completeness, then *finalizes* the detailed specifications for Pydantic models, Jinja2 structure, and validation logic based on the project plan and agent feedback.
  * **Context:** Project Plan, Agent's analysis output.
* **Guardrails:** AI output is purely advisory; final specs are human-authored based on the project plan.

**Stage 2: Core Component Implementation (Iterative)**

* **Objective:** Build the core functional components of the application using AI agents for code generation, guided by the finalized specifications from Stage 1. Implement iteratively, component by component or feature by feature (e.g., start with `choiceInteraction`).
* **Sub-Stage 2.1: Pydantic Model Implementation**
  * **Task (`TASK 2.1.N`):** Implement Pydantic model `N` in Python code based on the finalized spec.
  * **Agent Role:** Code Generator.
  * **Example Prompt:** "Generate a Python Pydantic model named `ChoiceInteractionData` inheriting from `BaseItem`. It should include fields: `identifier` (UUID, default factory), `prompt` (str), `choices` (List[ChoiceOption]), `shuffle` (bool, default=False), `max_choices` (int, default=1). Define the nested `ChoiceOption` model with fields: `identifier` (UUID, default factory), `text` (str), `is_correct` (bool). Include appropriate type hints and basic validation (e.g., `choices` list must not be empty)."
  * **Recommended Model:** Gemini 2.5 Pro (strong Python/Pydantic knowledge), GPT-4.1/4.5 series. O4-mini/Gemini Flash for very simple models.
  * **HITL:** **Mandatory Code Review.** Check generated code against spec, verify validation logic, types, default values, inheritance.
  * **Guardrails:** Valid Python syntax, correct Pydantic usage, matches finalized spec, passes linter (e.g., Ruff/Black).
  * **Error Mitigation:** Agent might miss complex cross-field validation or subtle Pydantic features. Developer review catches this.

* **Sub-Stage 2.2: FastAPI Endpoint Implementation**
  * **Task (`TASK 2.2.N`):** Implement FastAPI endpoint `N` (e.g., `/generate/package`).
  * **Agent Role:** Code Generator.
  * **Example Prompt:** "Generate a Python FastAPI POST endpoint at `/generate/qti_package`. It should accept a JSON body validated by the `AssessmentPackageRequest` Pydantic model. On success, it should trigger the QTI generation logic (placeholder function `create_qti_package(data: AssessmentPackageRequest)`) and return a JSON response `{'status': 'success', 'package_url': '...'}` with status 200. Handle Pydantic validation errors automatically (FastAPI default) and return a 400 error. Handle potential errors during `create_qti_package` by catching a specific exception (e.g., `GenerationError`) and returning a JSON response `{'status': 'error', 'message': '...'}` with status 500. Use async/await."
  * **Recommended Model:** Gemini 2.5 Pro, GPT-4.1/4.5 series.
  * **HITL:** **Mandatory Code Review.** Check routing, request/response models, status codes, error handling, async usage, integration with generation logic.
  * **Guardrails:** Valid FastAPI code, correct Pydantic integration, proper error handling, follows REST principles.

* **Sub-Stage 2.3: Jinja2 Template Generation**
  * **Task (`TASK 2.3.N`):** Generate Jinja2 template snippet/macro `N` (e.g., `_choiceInteraction.xml`). **This is high-risk for agent errors.**
  * **Agent Role:** Template Generator (requires careful prompting).
  * **Example Prompt (Highly Specific):** "Generate a Jinja2 template snippet for a QTI 2.1 `choiceInteraction` block. Assume the context variable `item` is an instance of the `ChoiceInteractionData` Pydantic model (defined previously). The `responseIdentifier` should be 'RESPONSE'. MaxChoices should be set from `item.max_choices`. Shuffle should be set from `item.shuffle`. Iterate through `item.choices` (list of `ChoiceOption` models) to create `simpleChoice` elements. Each `simpleChoice` identifier must be `item.choices[loop.index0].identifier`. The choice text (`item.choices[loop.index0].text`) should be placed inside the `simpleChoice` tags. Ensure the QTI namespace (`xmlns="http://www.imsglobal.org/xsd/imsqti_v2p1"`) is assumed to be declared on a parent element. Use Jinja2 XML autoescaping context. Handle potential XHTML/MathML in `item.choices[loop.index0].text` appropriately (assume it might need wrapping - use a placeholder filter `render_content` for now)."
  * **Recommended Model:** Claude 3.7 Sonnet/Pro (excels at complex instructions, structured formats), Gemini 2.5 Pro (strong reasoning). *Expect iteration.*
  * **HITL:** ***Critical Review.*** Check:
    * XML well-formedness and QTI 2.1 structure.
    * Correct variable usage matching Pydantic models.
    * Namespace handling (implicitly or explicitly).
    * Correct logic (loops, conditionals).
    * Placeholders for complex rendering (MathML CDATA, asset paths) addressed correctly later.
    * Test rendering with sample data and inspect output meticulously.
  * **Guardrails:** Adherence to QTI structure specified, use of provided variable names, conceptual XML correctness. *Manual validation is key.*
  * **Error Mitigation:** Agents struggle with precise XML structure, namespaces, and context-dependent escaping. Provide examples, break down complex templates, review outputs carefully. Use placeholders for complex parts (like MathML CDATA wrapping) and implement those filters/macros separately.

* **Sub-Stage 2.4: Validation Logic Implementation (`lxml`)**
  * **Task (`TASK 2.4.1`):** Implement the XML validation function using `lxml`.
  * **Agent Role:** Code Generator.
  * **Example Prompt:** "Generate a Python function `validate_xml(xml_string: str, xsd_path: str, schema_cache: dict)` using `lxml.etree`. It should: 1. Check if the `xsd_path` is already in the `schema_cache` dict. If so, retrieve the `lxml.etree.XMLSchema` object. 2. If not, parse the XSD file using `lxml.etree.XMLSchema(file=xsd_path)` and store the result in `schema_cache` with `xsd_path` as the key. 3. Parse the input `xml_string`. 4. Validate the parsed XML using the cached schema object's `validate()` method. 5. If validation fails, raise a custom exception `XMLValidationError` containing the detailed `error_log` from the validator. 6. If validation succeeds, return True. Handle potential `lxml.etree.XMLSyntaxError` during XML parsing and `IOError` during XSD reading, raising appropriate custom exceptions."
  * **Recommended Model:** Gemini 2.5 Pro, GPT-4.1/4.5 series.
  * **HITL:** **Mandatory Code Review.** Check correct `lxml` API usage, schema caching logic, error handling, exception details (`error_log` extraction).
  * **Guardrails:** Correct `lxml` usage, efficient caching, robust error handling, returns expected outputs/exceptions.

* **Sub-Stage 2.5: Packaging Logic Implementation (`zipfile`)**
  * **Task (`TASK 2.5.1`):** Implement the ZIP packaging function.
  * **Agent Role:** Code Generator.
  * **Example Prompt:** "Generate a Python function `create_ims_package(manifest_xml: str, item_files: Dict[str, str], asset_files: Dict[str, str], output_zip_path: str)` using the `zipfile` module. `item_files` is a dict mapping internal resource identifier to item XML content string. `asset_files` is a dict mapping the relative path within the ZIP (e.g., 'assets/image1.png') to the source file path on the filesystem. The function must: 1. Create a ZIP archive at `output_zip_path` using `ZIP_DEFLATED` compression. 2. Write the `manifest_xml` string as `imsmanifest.xml` at the root of the ZIP. 3. Write each item XML string from `item_files` to a file named `<identifier>.xml` (using the key from `item_files`) at the root of the ZIP. 4. Add each asset file from the source path (value in `asset_files`) into the ZIP at the relative path specified by the key in `asset_files` (maintaining directory structure within the ZIP)."
  * **Recommended Model:** Gemini 2.5 Pro, GPT-4.1/4.5 series.
  * **HITL:** **Mandatory Code Review.** Check correct `zipfile` usage, manifest placement (root), asset path handling (relative paths inside ZIP), file naming, compression.
  * **Guardrails:** `imsmanifest.xml` at root, correct internal paths based on manifest `href`s, all required files included.

**Stage 3: Integration & Testing**

* **Objective:** Combine implemented components, write tests, and perform validation, especially against Blackboard Ultra.
* **Tasks:**
  * `TASK 3.1`: Integrate components (API calls generation logic, validation, packaging).
  * `TASK 3.2`: Write Unit Tests (`pytest`) for individual functions/modules.
  * `TASK 3.3`: Write Integration Tests (`pytest` with `TestClient`) for API endpoints and end-to-end flow (simple cases).
  * `TASK 3.4`: Perform Manual Acceptance Testing (MAT) focusing on Blackboard Ultra import and functionality.
* **Agentic Integration:**
  * **Agent Role:** Test Generator Assistant, Documentation Assistant.
  * **Example Prompt (Test Generation - TASK 3.2):** "Given the Python function `validate_xml` (provide code), generate `pytest` unit tests covering: 1. Successful validation with valid XML/XSD. 2. Failed validation with invalid XML, checking for `XMLValidationError` and inspecting the error log content. 3. Handling of `XMLSyntaxError` for malformed XML input. 4. Handling of `IOError` for non-existent XSD file. 5. Correct usage of the schema cache (mock the cache dictionary)."
  * **Example Prompt (Documentation - TASK 3.2):** "Generate a Python docstring (Google style) for the function `validate_xml` (provide code), explaining its purpose, arguments, return value, and raised exceptions."
  * **Recommended Model:** Gemini 2.5 Pro, Claude 3.7 Sonnet (good for explanation/documentation), GPT-4.1/4.5 series.
* **HITL Integration:**
  * **Trigger:** Agent-generated tests, documentation drafts, completion of integration steps, MAT results.
  * **Purpose:** **Mandatory Review & Validation.** Review generated tests for correctness and coverage. Review documentation for clarity and accuracy. *Crucially*, perform MAT against Blackboard Ultra, meticulously documenting successes, failures, rendering issues, and scoring discrepancies.
  * **Context:** Component code, generated tests/docs, test data, Blackboard Ultra instance, Project Plan (especially Section 10).
* **Guardrails:** Unit/integration tests must pass. MAT results in Blackboard Ultra are the primary acceptance criteria. Agent-generated tests/docs are starting points, requiring human validation.

**Stage 4: Iteration & Refinement**

* **Objective:** Address issues found during testing, refine implementations based on feedback (especially BB Ultra quirks), and add further features.
* **Tasks:**
  * `TASK 4.1`: Debug and fix issues identified in Stages 2 & 3.
  * `TASK 4.2`: Refine Jinja2 templates based on BB Ultra testing results (e.g., adjust MathML embedding, confirm asset path strategies).
  * `TASK 4.3`: Implement support for additional QTI interaction types (repeat Stage 2 processes).
* **Agentic Integration:**
  * **Agent Role:** Refactoring Assistant, Targeted Code Generator.
  * **Example Prompt (Refinement - TASK 4.2):** "Modify the Jinja2 macro `render_mathml(mathml_string)` (provide code). Ensure it wraps the entire input `mathml_string` within `<![CDATA[...]]>` tags *only if* the rendering context requires it (add a context flag check, e.g., `if embedding_in_xhtml`). Also ensure the root `<math>` tag within the string always has the `xmlns='http://www.w3.org/1998/Math/MathML'` attribute, adding it if missing."
  * **Recommended Model:** Use models appropriate for the specific task (coding, templating) as in Stage 2.
* **HITL Integration:**
  * **Trigger:** Bug reports, BB Ultra test feedback, requirement to add new features.
  * **Purpose:** Define the required changes, review agent-generated modifications, ensure regressions are not introduced.
  * **Context:** Existing code, bug report/feedback, specific change requirement.
* **Guardrails:** All changes must pass existing tests. Changes must align with architectural principles and project plan constraints. Re-test affected areas in Blackboard Ultra.

**4. API & Model Recommendations (April 2025 Summary)**

* **Strong Coding (Python, FastAPI, Libs):** Gemini 2.5 Pro, GPT-4.1/4.5 series.
* **Complex Instructions & Structured Output (Jinja2/XML):** Claude 3.7 Sonnet/Pro, Gemini 2.5 Pro.
* **Requirements Analysis / Documentation:** Claude 3.7 Sonnet/Pro, Gemini 2.5 Pro.
* **Test Generation:** Gemini 2.5 Pro, Claude 3.7 Sonnet.
* **Cost/Speed Optimization (Simple Tasks):** O4-mini, Gemini Flash, Claude 3.7 Haiku (use judiciously for well-defined, simple code snippets or formatting).
* **Justification:** These models represent the state-of-the-art in reasoning, coding, instruction following, and handling large contexts, crucial for tackling the complexities of this project. Claude's constitutional AI principles might offer safety benefits, while Gemini's integration potential could be useful if deploying on Google Cloud. Choose based on observed performance for specific task types during development.

**5. Guardrails for Agent-Assisted Development Workflow**

* **HITL is Non-Negotiable:** ALL agent-generated code, tests, templates, and documentation MUST be reviewed by the developer before integration.
* **Specification Driven:** Agents are given *specific tasks* based on *pre-defined, human-approved specifications* (Pydantic models, template structure, function signatures). Avoid vague prompts.
* **Linting & Formatting:** Enforce automated code quality checks (e.g., Ruff, Black) on all committed code, including reviewed agent output.
* **Incremental Integration:** Integrate agent-generated components one by one, testing each integration point.
* **Version Control Discipline:** Commit reviewed code frequently. Use descriptive commit messages, potentially noting agent assistance. Never commit unreviewed code.
* **Test Coverage:** Maintain adequate unit and integration test coverage. Agent-generated code must be covered by tests.
* **Prompt & Output Logging:** Consider logging agent prompts and their corresponding outputs (especially for complex tasks like template generation) to aid debugging and reproducibility.

**6. Anticipated Errors & Mitigation in Agent Output**

* **Incorrect Logic/API Usage:** Agents may misunderstand nuances of libraries (FastAPI, `lxml`, Jinja2) or QTI/IMS CP specs. *Mitigation:* Detailed prompts, mandatory HITL code review, unit/integration testing.
* **Subtle XML/Jinja2 Errors:** Incorrect nesting, missing namespaces, improper escaping (especially with CDATA), flawed template logic. *Mitigation:* Meticulous HITL review of templates and *rendered output*, specific prompts addressing known issues (e.g., MathML CDATA), iterative refinement based on `lxml` validation failures.
* **Ignoring Constraints:** Agent might generate generic code ignoring specific BB Ultra compatibility notes or project plan constraints. *Mitigation:* Include constraints explicitly in prompts, focused HITL review against requirements.
* **Incomplete Error Handling:** Generating code that works for the happy path but fails on edge cases or errors. *Mitigation:* Explicitly prompt for error handling, HITL review focuses on robustness.
* **Hallucinated Code:** Generating code that looks plausible but is functionally incorrect or uses non-existent features. *Mitigation:* HITL review, testing.

**7. Strategic HITL Integration Summary**

| HITL Trigger                                     | Purpose                                                                 | Required Context                                                    | Feedback Mechanism                                                                   |
| :----------------------------------------------- | :---------------------------------------------------------------------- | :------------------------------------------------------------------ | :----------------------------------------------------------------------------------- |
| Agent Code/Template Generation Complete        | **Validation, Quality Control,** Architectural Alignment, Debugging     | Spec, Prompt, Agent Output (Code/Template), Linter/Test Results     | Correct/Refine code; Update prompt library with successful/failed patterns.        |
| Agent Test/Documentation Generation Complete   | **Validation,** Meaningfulness Check, Accuracy Check                    | Function Code, Prompt, Agent Output (Test/Doc)                      | Correct/Refine tests/docs; Refine test generation prompts.                           |
| Agent Requirements Analysis Complete           | **Validation,** Completeness Check, Decision Making                     | Project Plan, Prompt, Agent Output (Summary)                        | Finalize specs based on validated/refined analysis.                                |
| Integration Failure / Test Failure             | Debugging, Root Cause Analysis                                          | Failing Code, Test Output, Agent Prompts/Outputs (if relevant)      | Identify faulty component (agent-generated or manual); Plan correction.            |
| Manual BB Ultra Test Result                    | **Primary Acceptance,** Identify Quirks, Refine Generation Strategy     | Generated QTI Package, BB Ultra Instance, Test Protocol/Checklist | Document BB behavior; Refine templates/logic; Update internal knowledge base.      |
| Ambiguity in Spec / Complex Design Decision    | Clarification, Architectural Decision                                   | Project Plan, Current Code/Specs, Problem Description             | Update specs/design documents; Formulate specific prompts for agent if needed. |
| Need for Refactoring / New Feature Implementation | Planning, Specification, **Validation** of Agent-assisted modification | Existing Code, Requirement, Refactoring Goal                      | Define changes; Review agent modifications; Ensure tests pass.                        |

**8. Self-Critique & Refinement of this Plan**

* **Potential Bottleneck:** The HITL review, especially for complex Jinja2/XML templates, could be time-consuming for a solo developer. *Refinement:* Emphasize breaking template generation into smaller, manageable macro/snippet tasks. Start with simpler interaction types first. Leverage `lxml` validation heavily immediately after rendering.
* **Risk:** Over-reliance on agents for tasks requiring deep domain knowledge (QTI subtleties, BB Ultra quirks). *Refinement:* Position agents strictly as implementers of *human-defined* logic for these areas. The developer must embed the domain knowledge into the specifications and prompts, and verify it in the HITL step.
* **Complexity Management:** The workflow involves multiple stages and tools. *Refinement:* Maintain clear task tracking (referencing TASK IDs). Keep prompts and specifications well-organized. Start with the core MVP features before expanding.
* **Manageability:** Ensure the prompt engineering effort doesn't outweigh the time saved by code generation. *Refinement:* Develop reusable prompt templates for common tasks (Pydantic models, FastAPI endpoints). Focus agent use on boilerplate and well-defined functions first.

**9. Conclusion & Next Steps**

This agent-assisted development workflow provides a structured approach to leverage AI capabilities effectively while maintaining rigorous quality control, essential for this project's success. The solo developer acts as the architect, domain expert, and quality gatekeeper, using AI agents as highly capable coding assistants.

**Immediate Next Steps for the Developer:**

1. **Setup:** Establish the development environment (Python, FastAPI, DB, Jinja2, `lxml`, `pytest`, Pandoc if needed). Acquire official XSD schemas. Set up version control (Git).
2. **Stage 1 Execution:** Perform `TASK 1.1` (use an agent or do manually). Finalize detailed Pydantic models (`TASK 1.2`), Jinja2 structure (`TASK 1.3`), and validation requirements (`TASK 1.4`).
3. **Stage 2 (MVP Start):** Begin implementing the MVP components (e.g., `choiceInteraction`, basic manifest, validation, packaging) following the agent-assisted workflow:
    * Write precise specs/prompts.
    * Use agents for generation (`TASK 2.X.N`).
    * **Perform mandatory HITL review.**
    * Integrate and unit test (`TASK 3.2`).
4. **Establish BB Ultra Testing:** Gain access to a Blackboard Ultra test instance and prepare initial test packages/protocol (`TASK 3.4`).

By following this plan, prioritizing HITL reviews, and iteratively building upon validated components, the development team can efficiently produce a high-quality, robust, and compliant IMS QTI 2.1 package generator.
