**Detailed Implementation Flow: Stage 4 - Iteration & Refinement**

**Overall Stage Objective:** To systematically address bugs, refine implementations based on testing feedback (especially Blackboard Ultra compatibility findings), improve overall robustness, and incrementally add new features or support for additional QTI interaction types, ensuring changes are validated and don't introduce regressions.

**Agentic Patterns Employed in Stage 4:**

* **Task Decomposition:** Stage 4 activities are broken into debugging (4.1), targeted refinement (4.2), and new feature implementation (4.3), which itself involves decomposition similar to Stage 2.
* **Tool Use:**
  * AI models act as **Debugging Assistants** (analyzing errors, suggesting fixes), **Refactoring Assistants** (implementing targeted changes based on refinement goals), and **Targeted Code/Template Generators** (for fixes or new features).
  * Developer uses IDE debugger, Git (branches, diffs), `pytest` (for regression testing), the application itself, the Blackboard Ultra test instance, and Stage 3 MAT results/bug reports.
* **Reasoning (Planning & Analysis):**
  * Developer analyzes bug reports and MAT findings to determine root causes and plan corrections or refinements.
  * AI assists by reasoning about code to suggest fixes or refactoring approaches based on provided context (error messages, code snippets, desired changes).
  * Developer plans the implementation steps for new features.
* **Memory:**
  * **Short-Term:** Context provided in prompts (specific code snippets, error messages, refinement goals, MAT findings).
  * **Long-Term (Developer Managed):** The entire project history in Git, Stage 1-3 artifacts (specs, code, tests), Stage 3 MAT results log, and any formal bug tracking system are crucial memory components informing this stage.

**Infrastructure/Setup for Stage 4:**

* All infrastructure from Stages 1-3.
* **Bug Tracking / Issue Management:** A system (even a simple text file or spreadsheet, but ideally Git issues or a dedicated tool) to track bugs and refinement tasks identified in Stage 3.
* **Branching Strategy (Git):** Use feature branches or bugfix branches consistently to isolate changes during this stage.

---

**TASK 4.1: Debug and Fix Issues**

* **Task Objective:** Address specific bugs identified during unit testing, integration testing (Stage 3.2, 3.3), or Manual Acceptance Testing (Stage 3.4).
* **Task ID:** `TASK 4.1.N` (where N = 1, 2, 3... for each bug/issue)
* **Detailed Execution Steps:**
    1. **Select Issue:** Pick a bug report or documented issue from the tracker. Ensure it has clear steps to reproduce or refers to a specific failing test.
    2. **Reproduce Issue:** Confirm you can reliably reproduce the bug in your local environment or see the failing test.
    3. **Analyze Root Cause (Human-Led, AI-Assisted):**
        * Use standard debugging techniques (breakpoints, logging, code inspection).
        * **AI Assistance (Optional):** If stuck, formulate a prompt for an AI Debugging Assistant. Provide:
            * The relevant code snippet(s).
            * The full error message and traceback (if applicable).
            * Steps to reproduce the bug.
            * Ask the AI to "analyze this error and code, suggest potential root causes, and propose specific code changes to fix the issue."
    4. **Formulate Fix Strategy:** Based on the analysis, decide on the necessary code or template changes.
    5. **Implement Fix (Human-Led or AI-Assisted):**
        * **Human-Led:** Implement the fix directly.
        * **AI-Assisted:** If the fix involves boilerplate or a well-defined change suggested by the AI, provide the relevant code and the desired change in a prompt (e.g., "Modify this function [code snippet] to add a null check for the 'item.metadata' attribute before accessing its 'title' field.").
    6. **HITL Review (Mandatory for AI-Generated Fixes):** If AI generated the fix code, review it meticulously for correctness, unintended side effects, and adherence to project style/architecture.
    7. **Test Fix:**
        * Verify the original bug is resolved (re-run the steps to reproduce or the failing test).
        * **Regression Testing:** Run the relevant subset (or ideally, the full suite) of unit and integration tests (`pytest`) to ensure the fix hasn't broken other functionality.
    8. **Commit:** Commit the *reviewed and tested* fix to a dedicated branch, referencing the issue ID (e.g., "fix: Resolve issue #42 - Null pointer when processing items with no metadata (Agent-Assisted)").
    9. **Merge:** Merge the fix branch back into the main development branch after review (if applicable for workflow).
* **Agent Role:** Debugging Assistant, Targeted Code Generator
* **Example Prompt (AI Debugging Assistance):**

    ```
    "I'm encountering an error in my Python QTI generator application.

    Error Message & Traceback:
    [Paste full traceback here, e.g., TypeError: 'NoneType' object is not subscriptable in create_ims_package during asset handling]

    Relevant Code Snippet(s):
    [Paste relevant sections of the create_ims_package function, especially around asset handling loops and dictionary access]

    Steps to Reproduce:
    The error occurs when generating a package if the input data includes an item associated with an asset, but the main 'asset_files' dictionary passed to 'create_ims_package' is None instead of an empty dictionary {}.

    Request:
    Analyze the traceback and code. What is the likely root cause of this 'TypeError'? Suggest a specific, robust code change within the 'create_ims_package' function to prevent this error, ensuring it handles cases where 'asset_files' might be None gracefully (e.g., by treating None as empty).
    "
    ```

* **Recommended Models & Justification:**
  * **Gemini 2.5 Pro / GPT-4.1/4.5 Series:** Strong analytical and coding capabilities, good at interpreting tracebacks and suggesting targeted fixes.
  * **Claude 3.7 Series:** Also strong, potentially better at explaining the reasoning behind a suggested fix.
* **HITL Integration:**
  * **Trigger:** Developer needs help diagnosing a bug; AI suggests a fix; Fix is implemented.
  * **Purpose:** Validate AI's analysis (if used); **Mandatory Review** of any AI-generated fix code; Confirm the fix works; Ensure no regressions.
  * **Activities:** Debugging, code review, running original failing test, running regression test suite.
  * **Context:** Bug report/issue details, Codebase, Error messages, Agent's analysis/fix suggestion, Test suite (`pytest`).
* **Guardrails & Quality Checks:**
  * **Issue Resolution:** The original reported bug must be verifiably fixed.
  * **No Regressions:** The fix must not introduce new failures in the existing test suite.
  * **Code Quality:** Fix should maintain or improve code clarity and adhere to project standards.
  * **Version Control:** Use branches and clear commit messages linking to the issue.
* **Anticipated Errors & Mitigation:**
  * **Incorrect AI Analysis:** AI might misinterpret the error or suggest a flawed fix. **Mitigation:** Developer's own debugging skills and critical review of AI suggestions are paramount.
  * **Fix Causes Regression:** The fix inadvertently breaks something else. **Mitigation:** Comprehensive regression testing (`pytest`) is essential before committing/merging.
  * **Fix is a Patch, Not Root Cause:** The fix might mask a deeper underlying issue. **Mitigation:** Thorough root cause analysis during the debugging phase.
* **Output / Artifact:** Fixed code committed to version control. Updated issue tracker status. Passing test suite.

---

**TASK 4.2: Refine Jinja2 Templates/Logic Based on BB Ultra Testing**

* **Task Objective:** Make targeted adjustments to Jinja2 templates or the data/logic feeding them, specifically to address rendering issues, compatibility quirks, or functional discrepancies observed during MAT in Blackboard Ultra (TASK 3.4), while staying within the QTI 2.1 standard.
* **Task ID:** `TASK 4.2.N` (where N = 1, 2, 3... for each refinement)
* **Detailed Execution Steps:**
    1. **Select Refinement Task:** Choose a specific issue documented in the MAT results log (e.g., "MathML rendering garbled for complex fractions in BB Ultra," "Image alt text not appearing," "Specific LOM metadata field not displayed").
    2. **Analyze Issue:** Review the MAT notes, screenshots, and the generated XML for the failing case. Compare the generated XML structure against the QTI standard and any available BB Ultra best practice documentation (or known working examples). Hypothesize the cause (e.g., incorrect MathML CDATA wrapping, missing attribute, unsupported QTI element usage by BB Ultra, incorrect relative path).
    3. **Formulate Refinement Strategy:** Decide on the specific change needed in the Jinja2 template or potentially the Pydantic model/orchestration logic providing context. *Prioritize standard-compliant solutions.*
    4. **Implement Refinement (Human-Led or AI-Assisted):**
        * **Human-Led:** Make the precise change to the Jinja2 template (e.g., adjusting a filter call, modifying XML structure slightly, ensuring an attribute is always present).
        * **AI-Assisted:** Provide the AI Refactoring Assistant with:
            * The relevant Jinja2 template snippet.
            * The corresponding Pydantic context model(s).
            * A clear description of the problem observed in BB Ultra (from MAT log).
            * The *specific, desired change* (e.g., "Modify this template to always wrap the output of the `render_mathml` filter in `<![CDATA[...]]>`," or "Ensure the `<img>` tag generated by this macro always includes an `alt` attribute, using the value from `asset.alt_text` or defaulting to an empty string if null.").
    5. **HITL Review (Mandatory for AI-Generated Refinements):** Review any AI-generated template changes for correctness, ensuring they implement the intended refinement without breaking XML validity or Jinja2 syntax.
    6. **Test Refinement:**
        * **Render Test:** Re-render the template with the original problematic sample data. Inspect the updated XML output.
        * **Validation:** Validate the newly rendered XML with `lxml` (TASK 2.4.1).
        * **Re-Test in BB Ultra:** Generate a new QTI package specifically for this test case. Import it into Blackboard Ultra and meticulously verify that the original issue (rendering, functionality) is resolved.
        * **Regression Testing:** Run relevant automated tests (`pytest`) to ensure the change didn't break other parts of the template or related functionality.
    7. **Commit:** Commit the *reviewed and re-tested* refinement, linking to the MAT finding or issue ID.
* **Agent Role:** Refactoring Assistant, Targeted Template Generator
* **Example Prompt (AI Refinement Assistance):**

    ```
    "Refine the following Jinja2 template snippet used for generating QTI assessmentItem XML.

    Problem Description (Observed in Blackboard Ultra MAT):
    When MathML content generated by the 'render_mathml' filter includes specific characters like '<' or '&', Blackboard Ultra's MathJax fails to render it correctly, showing raw markup instead of the equation. The generated XML validates against the QTI XSD.

    Relevant Jinja2 Snippet (within an itemBody element):
    ...
    <p>Consider the equation: {{ render_mathml(item.equation_mathml) }}</p>
    ...

    Pydantic Context:
    Assume 'item.equation_mathml' contains a valid Presentation MathML string like '<math xmlns="http://www.w3.org/1998/Math/MathML"><mi>a</mi><mo>&lt;</mo><mi>b</mi></math>'.
    Assume 'render_mathml' filter currently just returns this string.

    Required Refinement:
    Modify the way 'render_mathml(item.equation_mathml)' is used or modify the filter itself (conceptual) so that its output, when embedded within the XHTML context (<p>), is wrapped in '<![CDATA[...]]>' to prevent the LMS/browser from misinterpreting characters within the MathML before MathJax processes it. The final output in the XML should look like: '<p>Consider the equation: <![CDATA[<math xmlns="http://www.w3.org/1998/Math/MathML">...</math>]]></p>'.

    Generate the updated Jinja2 snippet demonstrating how to apply this CDATA wrapping consistently when calling the filter in this context. You can assume a modified filter or show how to add the CDATA tags directly in the template around the filter call. Prioritize clarity and XML validity.
    "
    ```

* **Recommended Models & Justification:**
  * **Claude 3.7 Series:** Often excels at nuanced text/markup generation and adhering to complex formatting instructions like CDATA wrapping, making it well-suited for template refinement.
  * **Gemini 2.5 Pro / GPT-4.1/4.5 Series:** Also capable, especially if the refinement involves more logical changes than pure structural markup adjustments.
* **HITL Integration:**
  * **Trigger:** Need to address a BB Ultra compatibility issue from MAT; AI suggests a template modification.
  * **Purpose:** Validate the proposed refinement addresses the observed issue; **Mandatory Review** of template changes; Verify XML validity; **Crucially: Re-test the specific scenario in Blackboard Ultra**; Check for regressions.
  * **Activities:** Code/template review, XML validation, targeted manual testing in BB Ultra, running automated regression tests.
  * **Context:** MAT results log/issue description, Original template code, Generated XML (before/after), AI's suggested refinement, Blackboard Ultra test instance, QTI standard docs.
* **Guardrails & Quality Checks:**
  * **BB Ultra Improvement:** The refinement MUST demonstrably fix or improve the specific issue observed in Blackboard Ultra.
  * **Standard Compliance:** The change should ideally remain fully compliant with QTI 2.1 / IMS CP 1.2 standards. Avoid platform-specific hacks that break standards.
  * **XML Validity:** The refined template must still produce XSD-valid XML.
  * **No Regressions:** Automated tests should continue to pass. Re-test related functionality manually in BB Ultra if necessary.
* **Anticipated Errors & Mitigation:**
  * **Refinement Doesn't Work:** The change might not fix the BB Ultra issue (e.g., the root cause was misunderstood). **Mitigation:** Re-analyze the problem; try alternative standard-compliant approaches; consult BB documentation or communities.
  * **Refinement Breaks Standards:** The change introduces non-standard markup. **Mitigation:** Prioritize standard compliance during HITL review; use XSD validation.
  * **Refinement Causes Regressions:** The change negatively impacts other interaction types or scenarios. **Mitigation:** Thorough regression testing (automated and potentially targeted manual).
* **Output / Artifact:** Updated and committed Jinja2 templates or Python logic. Updated MAT log showing the issue is resolved. Passing automated tests.

---

**TASK 4.3: Implement Support for Additional QTI Interaction Types / Features**

* **Task Objective:** Extend the generator's capabilities by adding support for new QTI interaction types (e.g., `orderInteraction`, `matchInteraction`, `textEntryInteraction`) or other features requested post-MVP.
* **Task ID:** `TASK 4.3.N` (where N = 1, 2, 3... for each new feature/interaction)
* **Detailed Execution Steps:** This task essentially requires cycling back through the relevant parts of Stages 1, 2, and 3 for the new feature:
    1. **Define Requirements (Stage 1 Mindset):**
        * Analyze the requirements for the new interaction type/feature based on the QTI specification and project goals.
        * Define the necessary additions/changes to Pydantic models (TASK 1.2 adaptation).
        * Plan the required Jinja2 template snippets/macros (TASK 1.3 adaptation).
        * Identify any new validation logic needed (TASK 1.4 adaptation).
    2. **Implement Components (Stage 2 Process):**
        * Implement required Pydantic models (TASK 2.1.N - Agent-assisted + HITL).
        * Implement required Jinja2 templates/macros (TASK 2.3.N - Agent-assisted + **Critical HITL**). Pay close attention to the QTI spec for the new interaction.
        * Implement any new validation or utility functions (TASK 2.4.N / 2.5.N - Agent-assisted + HITL).
        * Update FastAPI endpoints or orchestration logic if needed to handle the new type/feature (TASK 2.2.N / TASK 3.1 adaptation).
    3. **Integrate & Test (Stage 3 Process):**
        * Integrate the new components into the main application flow (TASK 3.1 adaptation).
        * Write new unit tests for the new models and logic (TASK 3.2.N - Agent-assisted + HITL).
        * Write new integration tests for API flows involving the new feature (TASK 3.3.N - Agent-assisted + HITL).
        * Perform **Manual Acceptance Testing in Blackboard Ultra** specifically for the new interaction type/feature (TASK 3.4 adaptation). Document findings rigorously.
    4. **Commit & Merge:** Follow branching/commit strategy for the new feature.
* **Agent Role:** Code Generator, Template Generator, Test Generator Assistant (as per Stage 2 & 3 roles).
* **HITL Integration:** Mandatory review points mirror those in Stages 2 and 3 for code generation, template generation, test generation, and MAT validation.
* **Guardrails & Quality Checks:** Same as Stages 2 & 3, applied to the new feature components. Ensure the new feature doesn't break existing functionality (regression testing).
* **Anticipated Errors & Mitigation:** Similar risks as Stages 2 & 3, plus the risk of misunderstanding the nuances of less common QTI interaction types. **Mitigation:** Careful study of QTI specs, meticulous HITL for templates, thorough MAT in BB Ultra for the new interaction type.
* **Output / Artifact:** Application extended with the new feature/interaction type. Updated code, templates, tests, and potentially updated MAT logs reflecting the new capabilities.

---

**Conclusion of Stage 4:**

Stage 4 is often ongoing or cyclical. After completing a cycle of Stage 4 activities (addressing a batch of bugs/refinements or adding a significant feature), the application should be:

1. More **robust**, with known bugs fixed.
2. More **compatible** with Blackboard Ultra, based on specific refinements from MAT feedback.
3. Potentially **more feature-rich**, supporting additional interaction types or capabilities.
4. Covered by **updated and passing test suites**, including regression checks.

The key output is an improved version of the QTI Generator, ready for further testing, deployment, or another cycle of refinement based on new feedback or requirements. The process emphasizes continuous improvement driven by testing and feedback, facilitated by strategic AI assistance while maintaining developer control and quality oversight.
