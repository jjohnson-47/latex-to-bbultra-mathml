**Detailed Implementation Flow: Stage 1 - Specification & Foundation Refinement**

**Overall Stage Objective:** To rigorously analyze the project plan, extract all technical constraints and requirements, and produce finalized, human-approved specifications for core data structures (Pydantic), Jinja2 template organization, and XML validation logic. This solid foundation is critical for guiding the agent-assisted development in subsequent stages.

**Agentic Patterns Employed in Stage 1:**

* **Task Decomposition:** Stage 1 is broken into distinct tasks (1.1 - 1.4) targeting specific aspects of the specification.
* **Tool Use:** AI models will primarily use their built-in capabilities for document comprehension, analysis, summarization, and potentially light code analysis/suggestion. No external tools are strictly required by the agents *in this stage*.
* **Reasoning (Planning & Analysis):** The agent's primary role here is analytical reasoning – parsing the project plan, identifying relationships between sections, and extracting specified constraints.
* **Memory:** Short-term memory is used by the agent to process the project plan context. The outputs of each task serve as the persistent "memory" or input artifacts for subsequent tasks and stages, managed by the developer.

**Infrastructure/Setup for Stage 1:**

* Access to the full Project Plan document (text format preferred for easy input to models).
* An environment/interface to interact with the chosen AI models (e.g., API access via scripts, web UI like OpenAI Playground, Anthropic Console, Google AI Studio).
* A system for organizing the outputs (e.g., project documentation files in Markdown, dedicated spec documents).

---

**TASK 1.1: Review Project Plan for Key Constraints & Structures**

* **Task Objective:** Use an AI agent to perform a detailed pass over the Project Plan (specifically Sections 4-10 as requested, but consider referencing other sections like 3, 13, 14 for completeness) to extract and clearly summarize all technical requirements and constraints related to the core implementation areas.
* **Detailed Execution Steps:**
    1. **Prepare Input:** Copy the relevant text sections (or the entire document if context window allows) of the "Project Plan: IMS QTI 2.1 Assessment Package Generator" document.
    2. **Formulate Prompt:** Construct a detailed prompt for the AI agent.
    3. **Execute Prompt:** Send the prompt and project plan text to the chosen AI model.
    4. **Receive & Process Output:** Obtain the structured summary from the agent.
    5. **HITL Review (Critical):** Proceed to the HITL step described below.
* **Agent Role:** Requirements Analyst Assistant
* **Prompting Strategy:**
  * **Input:** Full text of Project Plan (Sections 4-10 minimally, potentially more).
  * **Instruction:** "Act as a meticulous Requirements Analyst. Analyze the provided Project Plan for the IMS QTI Generator. Extract and list *all specific technical requirements, constraints, design decisions, and specified technologies/libraries* related to the following areas. Structure your output clearly using Markdown headings for each category:
        1. **Pydantic Data Modeling (Input):** (Ref: Section 5, 4) Specify required base models, specific interaction models mentioned, handling of pedagogical intent vs. XML structure, required fields (identifier, prompt, etc.), feedback structures, scoring intent, LOM subset focus (mention BB Ultra mapping importance), asset reference structure.
        2. **Jinja2 Templating Strategy:** (Ref: Section 6, 4) Specify required modular organization (base, interactions, includes/macros), critical technical details (XML autoescaping setup, namespace handling strategy, custom filters/globals needed like `generate_uuid`, `render_mathml` including CDATA strategy), XHTML handling requirements (validation/sanitization), whitespace control importance.
        3. **Validation Strategy (`lxml` Focus):** (Ref: Section 7, 3) Specify the requirement for mandatory XSD validation, the tool (`lxml.etree`), which files to validate (items, test, manifest) against which schemas (QTI 2.1, CP 1.2), the schema caching strategy requirement, and the *required details* for error reporting (parsing `error_log`, linking to item identifier).
        4. **Packaging Strategy (`zipfile` Focus):** (Ref: Section 8, 4) Specify the standard (IMS CP 1.2), manifest file name (`imsmanifest.xml`) and location (root), the need for resource elements (`identifier`, `type`, `href`), structure of `<file>` elements, requirement for relative paths matching ZIP structure, use of `zipfile` module, and compression (`ZIP_DEFLATED`).
        5. **MathML Handling:** (Ref: Section 9, 6) Specify input format (Presentation MathML), optional use of Pandoc, storage requirement, and the precise embedding strategy (ensure root `<math>` tag with namespace, wrap in CDATA within XHTML contexts via `render_mathml`).
        6. **Asset Handling:** (Ref: Section 4) Specify the data model needs (list of metadata), storage approach (unique IDs), upload interface requirement (frontend), backend logic (association, retrieval, packaging), manifest entry requirements (`<file href="...">`), and relative path usage in QTI content (`src`/`data`).
        7. **Platform Compatibility (Blackboard Ultra Focus):** (Ref: Section 10, 3) Specify BB Ultra as the primary test target, the need to test known potential pain points (response processing, interactions, metadata, feedback, MathML, assets), and the strategy (prioritize standards, targeted testing, document quirks, avoid non-standard workarounds).
        *Summarize concisely but ensure all explicit technical directives and constraints mentioned in the plan are captured.*"
* **Recommended Models & Justification:**
  * **Claude 3.7 Sonnet/Pro:** Excellent for long-context comprehension, nuanced understanding of requirements documents, and generating structured textual output. Its constitutional training may also help in adhering closely to the provided text.
  * **Gemini 2.5 Pro:** Very large context window, strong reasoning capabilities suitable for parsing and synthesizing information from a dense technical document.
* **Tool Use:** Primarily the model's internal document analysis capability.
* **HITL Integration:**
  * **Trigger:** Agent produces the summary output.
  * **Purpose:** **Mandatory Validation & Verification.** The developer *must* meticulously compare the agent's extracted list against the *original* project plan (Sections 4-10 and others).
  * **Activities:** Check for accuracy (did the agent extract the correct requirements?), completeness (did it miss any critical constraints mentioned?), and interpretation (did it misunderstand any technical points?). Add annotations or corrections directly to the agent's output.
  * **Context:** Original Project Plan document, Agent's generated summary.
* **Guardrails & Quality Checks:**
  * **Cross-Referencing:** Every point in the agent's summary must be traceable back to a specific statement in the project plan.
  * **No Blind Trust:** Treat the agent's output as a high-quality draft checklist, not the final source of truth. The project plan document remains authoritative.
* **Anticipated Errors & Mitigation:**
  * **Omission:** Agent might miss subtle requirements or constraints embedded in descriptive text. **Mitigation:** Thorough HITL review and cross-referencing by the developer who has read the plan.
  * **Misinterpretation:** Agent might misunderstand technical jargon or the intent behind a requirement (e.g., the nuance between pedagogical intent and XML structure for Pydantic). **Mitigation:** Developer expertise applied during HITL review.
  * **Hallucination:** Agent might invent requirements not present in the text. **Mitigation:** Strict cross-referencing during HITL review.
* **Output / Artifact:** A developer-validated, annotated, and finalized list/document summarizing all key technical requirements and constraints extracted from the project plan. This artifact is critical input for the subsequent tasks in Stage 1.

---

**TASK 1.2: Define Detailed Pydantic Models**

* **Task Objective:** Based on the validated requirements (from TASK 1.1) and Project Plan Section 5, finalize the specific structure, fields, types, and basic validation rules for the core Pydantic input models.
* **Detailed Execution Steps:**
    1. **Developer Drafting:** Using the validated requirements list and Section 5, the developer drafts the initial Python code structure for the key Pydantic models (e.g., `BaseItem`, `ChoiceInteractionData`, `AssetReference`, `LomMetadata`, etc.). Focus on capturing the fields and basic types. This is a human-led architectural step.
    2. **Prepare Input for Agent (Optional Refinement/Review):** Select a specific draft model or group of related models.
    3. **Formulate Prompt (Optional):** Create a prompt asking the agent to review the draft against requirements or suggest refinements.
    4. **Execute Prompt (Optional):** Send the draft code and relevant requirements to the agent.
    5. **Receive Suggestions (Optional):** Get feedback from the agent.
    6. **HITL Finalization:** The developer reviews any agent suggestions, integrates necessary changes, adds detailed type hints, default values, and crucially, defines the specific Pydantic validation logic (`@field_validator`, constraints etc.) required by the plan.
* **Agent Role (Optional):** Code Review Assistant / Pydantic Advisor
* **Prompting Strategy (If using Agent for Review/Refinement):**
  * **Input:** Developer's draft Python code for Pydantic model(s), relevant snippet from the validated requirements list (TASK 1.1 output), relevant snippet from Project Plan Section 5.
  * **Instruction:** "Review the following draft Pydantic model(s) [Python Code Snippet]. Check its compliance against these specific requirements extracted from the project plan: [Relevant Requirements List Snippet].
    * Are all required fields mentioned in the requirements present?
    * Are the types appropriate based on the description?
    * Identify any potential conflicts or areas where the model doesn't fully capture the intent described in the requirements (e.g., pedagogical intent, LOM subset, asset structure).
    * Suggest specific Pydantic features (e.g., `field_validator`, `constr`, `conlist`, `RootModel`, `default_factory=uuid.uuid4`) that could be used to implement the specified validation rules or default behaviors mentioned in the requirements. Focus on correctness and clarity."
* **Recommended Models & Justification (If using Agent):**
  * **Gemini 2.5 Pro / GPT-4.1/4.5 Series:** Strong Python and Pydantic knowledge, good at code analysis and suggesting concrete code implementations for validation.
  * **Claude 3.7 Sonnet:** Also capable, potentially better if the focus is more on explaining *why* a suggestion is made based on the requirements text.
* **Tool Use:** Agent uses code analysis capabilities.
* **HITL Integration:**
  * **Trigger:** Developer completes the initial draft; Agent (optionally) provides feedback.
  * **Purpose:** **Mandatory Final Design & Specification.** The developer makes the final decisions on model structure, field names, types, validation logic, and inheritance. Even if an agent provides suggestions, the developer must understand and approve them, integrating them into the final design.
  * **Context:** Validated Requirements List, Project Plan Section 5, Developer's draft code, Agent's suggestions (if used).
* **Guardrails & Quality Checks:**
  * **Plan Alignment:** Final models must directly reflect the decisions and requirements outlined in Section 5 and the validated list from TASK 1.1.
  * **Pydantic Best Practices:** Use appropriate Pydantic features for validation and typing.
  * **Clarity:** Models should be understandable and represent the logical structure of the input data clearly.
* **Anticipated Errors & Mitigation:**
  * **Agent Misunderstanding (if used):** Agent might suggest incorrect Pydantic validators or misunderstand complex cross-field validation needs. **Mitigation:** Developer's Pydantic expertise and final review are essential.
  * **Developer Oversight:** Developer might miss a requirement during manual drafting. **Mitigation:** Use the validated list from TASK 1.1 as a checklist during finalization.
* **Output / Artifact:** Finalized Python Pydantic model definitions (potentially as `.py` file stubs or within a dedicated specification document). These definitions are the concrete specification for input data validation and structure.

---

**TASK 1.3: Define Jinja2 Template Structure**

* **Task Objective:** Based on the validated requirements (TASK 1.1) and Project Plan Section 6, decide and document the organizational structure for the Jinja2 templates.
* **Detailed Execution Steps:**
    1. **Developer Planning:** Using the validated requirements list and Section 6, the developer outlines a proposed directory structure (e.g., `/templates`, `/templates/base`, `/templates/interactions`, `/templates/includes`). Identify the key base files (`base_item.xml`, `base_manifest.xml`), potential includes (`_lom_metadata.xml`), and macros (`render_mathml`).
    2. **Prepare Input for Agent (Optional Suggestion):** Relevant requirements related to Jinja2 structure from TASK 1.1 output and Section 6.
    3. **Formulate Prompt (Optional):** Ask agent to suggest a structure based on requirements and best practices.
    4. **Execute Prompt (Optional):** Send requirements to the agent.
    5. **Receive Suggestion (Optional):** Get proposed structure/component list.
    6. **HITL Finalization:** Developer reviews any agent suggestions, compares with Section 6 guidelines and personal preference/experience, and makes the *final decision* on the directory layout, naming conventions, and the list of primary template files, includes, and macros to be created.
* **Agent Role (Optional):** Architectural Suggestion Assistant
* **Prompting Strategy (If using Agent for Suggestion):**
  * **Input:** Relevant Jinja2 requirements from TASK 1.1 output, Project Plan Section 6 text.
  * **Instruction:** "Based on the Jinja2 templating requirements outlined in the project plan [Requirements Snippet] (emphasizing modularity, base templates, includes/macros for interactions, metadata, MathML, response processing, XML autoescaping context) and general best practices for generating complex, structured XML with Jinja2, propose a concrete directory structure and list the key template files, includes, and macros likely needed. Explain the rationale for the structure."
* **Recommended Models & Justification (If using Agent):**
  * **Claude 3.7 Sonnet/Pro:** Good at understanding architectural requirements and explaining design rationale.
  * **Gemini 2.5 Pro:** Strong reasoning, can infer logical groupings for template components.
* **Tool Use:** Agent uses conceptual design based on text analysis.
* **HITL Integration:**
  * **Trigger:** Developer outlines initial plan; Agent (optionally) provides suggestions.
  * **Purpose:** **Mandatory Final Architectural Decision.** The developer establishes the definitive template organization plan, considering maintainability and clarity.
  * **Context:** Validated Requirements List, Project Plan Section 6, Agent's suggestion (if used), developer's own design preferences.
* **Guardrails & Quality Checks:**
  * **Plan Alignment:** Structure must support the modularity, escaping, and namespace requirements from Section 6.
  * **Practicality:** Structure should be logical and easy for the solo developer to navigate and maintain.
* **Anticipated Errors & Mitigation:**
  * **Agent Over-complication (if used):** Agent might suggest an unnecessarily deep or complex structure. **Mitigation:** Developer applies judgment based on project scope.
  * **Developer Oversight:** Might initially forget a necessary reusable component. **Mitigation:** Reviewing the list against the interaction types and features planned.
* **Output / Artifact:** A documented plan outlining the Jinja2 template directory structure, key files, and planned reusable components (includes/macros). This could be a simple text file, markdown document, or comments in the project's README.

---

**TASK 1.4: Define Core Validation Logic Requirements**

* **Task Objective:** Solidify the precise requirements for the `lxml` validation function, including schema locations, caching mechanism, and error reporting details, based on the validated requirements (TASK 1.1) and Project Plan Section 7.
* **Detailed Execution Steps:**
    1. **Developer Specification:** Review the validated requirements list (TASK 1.1) focusing on validation, and re-read Project Plan Section 7. Draft the specific requirements for the validation function:
        * How will XSD file paths be provided/configured? (e.g., config file, constants).
        * Confirm the explicit requirement for schema caching (likely a dictionary passed into or managed by the validation utility).
        * Define the exact information needed from `lxml`'s `error_log` upon failure (e.g., line number, column, message, failing element if possible) and the desired format (e.g., custom exception class attributes).
    2. **Prepare Input for Agent (Optional Review/Clarification):** Developer's draft specification points, relevant sections from TASK 1.1 output and Section 7.
    3. **Formulate Prompt (Optional):** Ask agent to review the drafted spec for clarity or completeness against the plan.
    4. **Execute Prompt (Optional):** Send draft spec and source requirements to the agent.
    5. **Receive Feedback (Optional):** Get agent's review comments.
    6. **HITL Finalization:** Developer reviews any agent feedback and finalizes the detailed specification for the validation logic. This spec will guide the implementation in TASK 2.4.1.
* **Agent Role (Optional):** Specification Review Assistant
* **Prompting Strategy (If using Agent for Review):**
  * **Input:** Developer's draft specification points for the validation function, relevant requirements from TASK 1.1 output and Section 7.
  * **Instruction:** "Review these draft specification points for the `lxml` validation logic: [Draft Spec Points]. Check if they fully and clearly address the requirements outlined in the project plan regarding XSD paths, schema caching, and error log parsing/reporting: [Relevant Requirements Snippet]. Are there any ambiguities? Does the specification capture all necessary details needed to implement the function correctly?"
* **Recommended Models & Justification (If using Agent):**
  * **Claude 3.7 Sonnet/Pro / Gemini 2.5 Pro:** Good at comparing a specification against source requirements for completeness and clarity.
* **Tool Use:** Agent uses text analysis and comparison.
* **HITL Integration:**
  * **Trigger:** Developer drafts the validation spec; Agent (optionally) reviews.
  * **Purpose:** **Mandatory Final Specification.** The developer ensures the validation logic requirements are precise, unambiguous, and complete before implementation begins.
  * **Context:** Validated Requirements List, Project Plan Section 7, Developer's draft spec, Agent's feedback (if used).
* **Guardrails & Quality Checks:**
  * **Plan Alignment:** Spec must cover XSD paths, caching, and error reporting details as mandated by the plan.
  * **Implementability:** Spec should be clear enough for direct implementation in Stage 2.
* **Anticipated Errors & Mitigation:**
  * **Vague Specification:** Initial draft might lack detail on error parsing. **Mitigation:** Focused review during HITL, potentially aided by agent feedback prompting for more specifics.
  * **Missing Config:** Forgetting to specify *how* XSD paths are managed. **Mitigation:** Checklist review against plan requirements.
* **Output / Artifact:** A finalized, detailed specification document or section describing precisely how the `lxml` validation should be implemented (inputs, outputs, behavior, error handling, configuration).

---

**Conclusion of Stage 1:**

Upon successful completion of these tasks, you will have:

1. A **developer-verified summary** of all critical technical requirements from the project plan.
2. **Finalized Pydantic model definitions** specifying the exact structure and validation of input data.
3. A **clear plan for Jinja2 template organization**, including directory structure and key components.
4. A **precise specification for the XML validation logic**, detailing schema handling, caching, and error reporting.

These artifacts provide the essential, human-validated blueprint needed to effectively guide the AI agents in the code and template generation tasks of Stage 2, significantly reducing the risk of misaligned or incorrect implementations.
