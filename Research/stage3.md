**Detailed Implementation Flow: Stage 3 - Integration & Testing**

**Overall Stage Objective:** To integrate the Pydantic models, FastAPI endpoints, Jinja2 rendering logic, `lxml` validation, and `zipfile` packaging components; develop comprehensive unit and integration tests (leveraging AI assistance); and perform rigorous Manual Acceptance Testing (MAT), especially validating import and functionality within Blackboard Ultra.

**Agentic Patterns Employed in Stage 3:**

* **Task Decomposition:** Stage 3 is broken into integration, unit testing, integration testing, and manual testing tasks (3.1 - 3.4).
* **Tool Use:**
  * AI models act as **Test Generation Assistants** and potentially **Documentation Assistants**.
  * Developer uses testing frameworks (`pytest`), HTTP clients for testing (`pytest-httpx` or FastAPI's `TestClient`), the `lxml` validation function (TASK 2.4.1), the packaging function (TASK 2.5.1), and crucially, a **Blackboard Ultra instance**.
* **Reasoning (Planning & Analysis):**
  * Developer plans the integration logic, ensuring components are called correctly.
  * AI assists in reasoning about test cases (coverage, edge cases) based on provided code.
  * Developer performs analytical reasoning during MAT to diagnose issues in Blackboard Ultra.
* **Memory:**
  * **Short-Term:** Context provided in prompts (code snippets for test generation).
  * **Long-Term (Developer Managed):** The integrated codebase in Git, the growing test suite, Stage 1 specifications, and crucially, the feedback/results log from MAT (especially BB Ultra findings).

**Infrastructure/Setup for Stage 3:**

* All infrastructure from Stage 2 (Python env, libs, Git, AI API access).
* **Testing Framework:** `pytest` installed and configured. Consider plugins like `pytest-cov` (for coverage), `pytest-asyncio` (for async code), `pytest-httpx` or FastAPI's `TestClient`.
* **Blackboard Ultra Access:** Confirmed access to a reliable Blackboard Ultra test instance (Sandbox or dedicated test environment).
* **Sample Data:** A growing set of diverse input data (e.g., JSON files corresponding to Pydantic models) covering different interaction types, edge cases, MathML, assets, etc. (partially developed during Stage 2 template testing).
* **MAT Protocol/Checklist:** A basic plan or checklist outlining the steps for testing package import and item functionality in Blackboard Ultra.

---

**TASK 3.1: Integrate Components**

* **Task Objective:** Connect the individual components developed in Stage 2. This involves modifying placeholder function calls in FastAPI endpoints (TASK 2.2.N) to invoke the actual Jinja2 rendering, `lxml` validation (TASK 2.4.1), and `zipfile` packaging (TASK 2.5.1) logic.
* **Detailed Execution Steps (Primarily Human-Led):**
    1. **Identify Integration Points:** Locate the placeholder function calls within your FastAPI endpoint(s) (e.g., the `create_qti_package` placeholder from TASK 2.2).
    2. **Implement Orchestration Logic:** Write the Python code within the endpoint (or ideally, refactor into a dedicated service/orchestration function called by the endpoint) that performs the following sequence:
        * Takes the validated Pydantic input data.
        * Prepares context data for Jinja2.
        * Renders the necessary `assessmentItem` XML strings using the Jinja2 templates (TASK 2.3.N) and appropriate context. Handle potential Jinja2 rendering errors.
        * **Validate each rendered item XML** using the `validate_xml` function (TASK 2.4.1) against the QTI 2.1 XSD. Aggregate errors.
        * Render the `imsmanifest.xml` string using its Jinja2 template, providing context about the items and assets.
        * **Validate the rendered manifest XML** using `validate_xml` (TASK 2.4.1) against the IMS CP 1.2 XSD. Aggregate errors.
        * If validation passes: Call the `create_ims_package` function (TASK 2.5.1) with the manifest string, validated item XML strings, and asset file information.
        * Handle exceptions from validation or packaging, translating them into appropriate FastAPI error responses (e.g., raising `GenerationError` as designed in TASK 2.2).
        * Return the success response (e.g., path to the ZIP file) or an aggregated error response.
    3. **Refactor (Optional but Recommended):** Consider moving the core orchestration logic out of the FastAPI endpoint function itself into a separate, testable service/utility module to keep the endpoint clean.
    4. **Initial Smoke Test:** Run the application locally and make a test request using `curl` or an HTTP client with simple sample data to ensure the basic flow executes without crashing. Check logs for errors.
* **Agent Role:** Minimal. Could potentially be used to *suggest* refactoring patterns for the orchestration logic if provided with the endpoint code and descriptions of the validation/packaging functions.
* **HITL Integration:**
  * **Trigger:** Developer completes the integration coding.
  * **Purpose:** **Mandatory Code Review.** Ensure components are called correctly, data flows appropriately, errors are handled and propagated, and the sequence matches the project plan diagram (Section 12).
  * **Context:** FastAPI endpoint code (TASK 2.2), Jinja2 rendering logic, Validation function (TASK 2.4.1), Packaging function (TASK 2.5.1), Project Plan diagram.
* **Guardrails & Quality Checks:**
  * **Correct Sequence:** Rendering -> Validation -> Manifest -> Validation -> Packaging.
  * **Error Propagation:** Validation or packaging errors must prevent package creation and result in informative API error responses.
  * **Data Flow:** Ensure the correct data (Pydantic models, XML strings, asset paths) is passed between components.
  * **Cleanliness:** Endpoint logic should be clear; consider refactoring complex orchestration.
* **Anticipated Errors & Mitigation:**
  * **Incorrect Data Passing:** Passing wrong variables or formats between functions. **Mitigation:** Careful HITL review, type hinting, debugging during smoke tests.
  * **Unhandled Exceptions:** Forgetting to catch errors from rendering, validation, or packaging. **Mitigation:** HITL review focusing on `try...except` blocks and error propagation logic.
  * **Configuration Issues:** Incorrect paths to XSDs or asset storage. **Mitigation:** Ensure configuration is loaded correctly; add logging.
* **Output / Artifact:** Updated Python code with integrated components, runnable locally via FastAPI server.

---

**TASK 3.2: Write Unit Tests (`pytest`)**

* **Task Objective:** Develop unit tests for individual functions and modules created in Stage 2 (and potentially helper functions created during TASK 3.1 integration) to verify their correctness in isolation. Leverage AI to generate test boilerplate and suggest test cases.
* **Task ID:** `TASK 3.2.N` (where N = 1, 2, 3... for each function/module tested)
* **Detailed Execution Steps:**
    1. **Select Unit:** Choose a function or method to test (e.g., `validate_xml`, a specific Pydantic model's validator, a Jinja2 filter, the core logic of `create_ims_package`).
    2. **Prepare Context:** Get the Python code for the unit under test. Identify its inputs, outputs, dependencies (which might need mocking), and potential edge cases.
    3. **Formulate Prompt (AI Assistance):** Create a prompt for the AI Test Generator Assistant. Include the code snippet, specify the testing framework (`pytest`), and ask for test cases covering:
        * Happy paths (expected inputs/outputs).
        * Edge cases (e.g., empty inputs, zero values, specific boundary conditions).
        * Error conditions (e.g., invalid inputs, expected exceptions).
        * Mocking requirements (if the function has external dependencies like file system access or database calls, although Stage 2 components are mostly self-contained or use basic types).
    4. **Execute Prompt:** Send to the AI model.
    5. **Receive & Pre-Check Tests:** Get the generated `pytest` code. Visually check for basic structure and relevance.
    6. **HITL Review (Mandatory):** Proceed to detailed HITL review of the generated tests.
    7. **Refine & Augment:** Correct any errors in the generated tests. Crucially, *add any missing test cases* identified during review based on your understanding of the code and requirements. Ensure tests have clear assertions (`assert`).
    8. **Integrate & Run:** Add the reviewed and refined test code to the appropriate test module (e.g., `tests/unit/test_validation.py`). Run `pytest` and ensure the new tests pass. Aim for increasing code coverage (`pytest --cov`).
    9. **Commit:** Commit the reviewed and passing unit tests.
    10. **(Optional) Generate Docstrings:** Use an AI assistant to generate docstrings for the function that was just tested, improving code documentation.
* **Agent Role:** Test Generator Assistant, Documentation Assistant
* **Example Prompt (Test Generation - TASK 3.2 for `validate_xml`):**

    ```
    "Generate Python unit tests using the 'pytest' framework for the following function and custom exception.

    Code Under Test:
    [Paste the Python code for validate_xml function and XMLValidationError exception from TASK 2.4.1 here]

    Requirements:
    - Use pytest fixtures where appropriate (e.g., for sample XML/XSD content, schema cache).
    - Mock file system operations (e.g., reading XSDs) using 'unittest.mock' or pytest fixtures like 'tmp_path' if needed to simulate IOError or schema parsing errors.
    - Mock the 'schema_cache' dictionary behavior.
    - Test Cases to Generate:
        1. Successful validation: Provide valid XML string and valid XSD path, assert function returns True.
        2. Failed validation (invalid XML structure): Provide XML violating the XSD, assert 'XMLValidationError' is raised. Check that the exception instance contains the 'error_log' attribute.
        3. Failed parsing (malformed XML): Provide a non-well-formed XML string, assert 'XMLValidationError' is raised (due to XMLSyntaxError).
        4. Failed XSD loading (IOError): Mock the XSD file read to raise IOError, assert a relevant standard exception (e.g., ValueError as specified in the function's implementation) is raised.
        5. Failed XSD parsing (Schema Parse Error): Mock the XSD content to be invalid, assert a relevant standard exception is raised.
        6. Cache Hit: Call the function twice with the same XSD path, assert the XSD is parsed only once (requires mocking/spying on 'lxml.etree.XMLSchema').
        7. Cache Miss: Call with different XSD paths, assert schemas are parsed individually.

    Ensure tests clean up any created temporary files if using 'tmp_path'. Include necessary imports ('pytest', 'lxml.etree', 'your_module.validate_xml', 'your_module.XMLValidationError', 'unittest.mock').
    "
    ```

* **Example Prompt (Documentation - Optional):**

    ```
    "Generate a Python docstring (Google style) for the following function:

    [Paste the Python code for validate_xml function here]

    Explain its purpose, arguments (including types and purpose of 'schema_cache'), return value, and the custom 'XMLValidationError' it raises, including the context of the 'error_log'. Also mention other standard exceptions it might raise (e.g., for XSD loading issues)."
    ```

* **Recommended Models & Justification:**
  * **Gemini 2.5 Pro / GPT-4.1/4.5 Series:** Strong coding abilities, good understanding of `pytest` patterns, mocking, and generating relevant test cases from code.
  * **Claude 3.7 Sonnet:** Also capable, particularly good at generating clear explanations (useful for docstrings) and potentially more structured test code.
* **HITL Integration:**
  * **Trigger:** Agent generates unit test code or docstring.
  * **Purpose:** **Mandatory Review & Validation.** Ensure test correctness, adequacy, and coverage. Ensure documentation accuracy and clarity.
  * **Activities (Tests):**
    * Verify tests accurately reflect the function's logic.
    * Check assertions (`assert`) are meaningful and correct.
    * Ensure edge cases and error conditions identified are actually covered.
    * **Crucially: Identify missing test cases** the agent didn't think of.
    * Verify mocks are set up and used correctly.
    * Confirm tests pass when run.
  * **Activities (Docs):**
    * Verify docstring accurately describes parameters, return values, and exceptions.
    * Check for clarity and adherence to style guide (e.g., Google style).
  * **Context:** Function code under test, Agent-generated test code/docstring, `pytest` documentation, Project requirements.
* **Guardrails & Quality Checks:**
  * **Test Coverage:** Aim for high unit test coverage of core logic modules (use `pytest --cov`).
  * **Passing Tests:** All committed unit tests must pass.
  * **Meaningful Assertions:** Tests should validate specific, expected outcomes.
  * **Isolation:** Unit tests should ideally not have external dependencies (use mocks).
  * **Accurate Documentation:** Docstrings must match code behavior.
* **Anticipated Errors & Mitigation:**
  * **Incomplete Tests:** Agent might miss important edge cases or error conditions. **Mitigation:** Thorough HITL review by the developer who understands the requirements and potential pitfalls; augment generated tests.
  * **Incorrect Mocks:** Agent might set up mocks improperly. **Mitigation:** HITL review of mock setup and usage; debugging failing tests.
  * **Brittle Tests:** Tests that break easily with minor code refactoring. **Mitigation:** Focus tests on public API/behavior rather than internal implementation details where possible.
  * **Inaccurate Docstrings:** Agent hallucinates behavior or misinterprets parameters. **Mitigation:** Careful HITL review against the actual code.
* **Output / Artifact:** Reviewed, comprehensive, and passing suite of unit tests covering core functions/modules. Updated code with accurate docstrings. Increased confidence in individual component correctness.

---

**TASK 3.3: Write Integration Tests (`pytest` with `TestClient`)**

* **Task Objective:** Develop integration tests that verify the interaction between components, typically by testing the FastAPI endpoints and the end-to-end flow from API request to generated output (or error response).
* **Task ID:** `TASK 3.3.N` (where N = 1, 2, 3... for each endpoint/flow tested)
* **Detailed Execution Steps:**
    1. **Select Flow:** Choose an API endpoint and a specific scenario to test (e.g., successful generation of a ChoiceInteraction package, generation resulting in a validation error, request with invalid input data).
    2. **Prepare Context:**
        * Identify the API endpoint path and method.
        * Prepare sample input data (JSON payload) for the chosen scenario.
        * Determine the expected outcome: success status code (e.g., 200) and aspects of the response body (e.g., `'status': 'success'`), or specific error status code (e.g., 422, 500) and error message structure.
        * If testing successful generation, identify expectations about the generated ZIP file (e.g., presence of `imsmanifest.xml`, specific item file, maybe even basic content checks within the XML if feasible).
    3. **Formulate Prompt (AI Assistance):** Create a prompt for the AI Test Generator Assistant. Include:
        * The API endpoint details (path, method).
        * The sample input JSON data.
        * The expected HTTP status code and response body structure/content.
        * Instructions to use FastAPI's `TestClient` (or `pytest-httpx`).
        * If testing success, describe checks needed on the result (e.g., "assert the response status is 200", "assert 'package_url' exists in the JSON response"). If a ZIP is created locally, maybe add steps to "use the 'zipfile' module to open the generated zip and assert 'imsmanifest.xml' exists within it".
        * If testing errors, specify the expected status code and error message structure.
    4. **Execute Prompt:** Send to the AI model.
    5. **Receive & Pre-Check Tests:** Get the generated `pytest` integration test code.
    6. **HITL Review (Mandatory):** Proceed to detailed HITL review.
    7. **Refine & Augment:** Correct generated test code. Add more detailed assertions, especially if checking the contents of generated files. Ensure setup/teardown is handled correctly (e.g., cleaning up generated ZIP files).
    8. **Integrate & Run:** Add the test to the appropriate integration test module (e.g., `tests/integration/test_api.py`). Run `pytest` (potentially marking unit vs integration tests). Ensure tests pass.
    9. **Commit:** Commit the reviewed and passing integration tests.
* **Agent Role:** Test Generator Assistant
* **Example Prompt (Integration Test - Success Case):**

    ```
    "Generate a Python integration test using 'pytest' and FastAPI's 'TestClient' for our QTI Generator API.

    API Endpoint: POST /generate/qti_package
    Testing Scenario: Successful generation of a simple Choice Interaction package.

    Sample Input Data (JSON):
    {
      'package_metadata': { 'title': 'Simple Choice Test' },
      'items': [
        {
          'identifier': 'item_choice_1',
          'title': 'Capital of France?',
          'interaction_type': 'ChoiceInteractionData', # Assuming model mapping happens
          'prompt': '<p>What is the capital of France?</p>',
          'choices': [
            {'identifier': 'choice_a', 'text': 'Berlin', 'is_correct': False},
            {'identifier': 'choice_b', 'text': 'Paris', 'is_correct': True},
            {'identifier': 'choice_c', 'text': 'London', 'is_correct': False}
          ],
          'shuffle': False,
          'max_choices': 1
        }
      ],
      'assets': []
    }

    Requirements:
    - Use FastAPI's 'TestClient' initialized with the main FastAPI app instance.
    - Send a POST request to '/generate/qti_package' with the above JSON payload.
    - Assertions:
        1. Assert the HTTP response status code is 200.
        2. Assert the response JSON contains a key 'status' with the value 'success'.
        3. Assert the response JSON contains a key 'package_url'.
    - (Optional Advanced Check - If feasible and function returns local path):
        # 4. Get the file path from 'package_url' (assuming it's a local path for testing).
        # 5. Use the 'zipfile' module to open the generated ZIP file.
        # 6. Assert that 'imsmanifest.xml' exists within the ZIP archive.
        # 7. Assert that 'item_choice_1.xml' exists within the ZIP archive.
        # 8. Ensure the generated ZIP file is cleaned up after the test.

    Include necessary imports ('pytest', 'fastapi.testclient', 'your_main_app', 'zipfile', 'os'). Structure the test within a test function or class.
    "
    ```

* **Recommended Models & Justification:**
  * **Gemini 2.5 Pro / GPT-4.1/4.5 Series:** Good at generating tests involving API interactions, handling JSON data, and using testing clients like `TestClient`.
* **HITL Integration:**
  * **Trigger:** Agent generates integration test code.
  * **Purpose:** **Mandatory Review & Validation.** Ensure tests accurately simulate user interaction, check relevant outcomes, and validate component integration.
  * **Activities:**
    * Verify the correct endpoint, method, and input data are used.
    * Check assertions match the expected success or error outcome (status code, response body).
    * If checking generated files: Ensure the checks are robust (file existence, potentially basic content checks).
    * Verify test setup and teardown (e.g., cleaning up files).
    * Identify any missing integration scenarios (e.g., different error types, more complex inputs).
    * Ensure tests pass when run against the integrated application.
  * **Context:** Integrated application code, API endpoint definitions, Sample input data, Agent-generated test code, `TestClient` documentation.
* **Guardrails & Quality Checks:**
  * **End-to-End Flow:** Tests should cover the main paths from request to response (success/error).
  * **Component Interaction:** Implicitly verifies that rendering, validation, and packaging work together.
  * **Passing Tests:** All committed integration tests must pass.
  * **Realistic Scenarios:** Test cases should reflect likely usage patterns and potential error conditions.
* **Anticipated Errors & Mitigation:**
  * **Complex Assertions:** Agents might struggle to generate code for deep inspection of generated ZIP/XML files. **Mitigation:** Developer implements these more complex checks during HITL, keeping generated file checks pragmatic.
  * **Environment Issues:** Tests might fail due to configuration differences (e.g., hardcoded paths). **Mitigation:** Use relative paths, configuration files, environment variables consistently in both app and tests.
  * **Integration Bugs:** Tests might reveal bugs where components don't work together as expected. **Mitigation:** This is the *purpose* of integration tests! Use the failures to debug the integration logic (TASK 3.1).
* **Output / Artifact:** Reviewed, comprehensive, and passing suite of integration tests verifying API endpoints and core generation flows.

---

**TASK 3.4: Perform Manual Acceptance Testing (MAT) - Blackboard Ultra Focus**

* **Task Objective:** Manually test the end-to-end process by generating QTI packages using the application (via its API or a simple UI if built) and importing them into Blackboard Ultra to verify successful import, rendering, and functionality according to the project plan's primary acceptance criteria.
* **Task ID:** `TASK 3.4` (Ongoing activity, performed iteratively)
* **Detailed Execution Steps (Human-Led):**
    1. **Prepare Test Cases:** Based on the MAT protocol/checklist and the sample data suite, select specific test cases covering:
        * All implemented QTI interaction types.
        * Content with MathML.
        * Content with various asset types (images).
        * Different feedback configurations.
        * Metadata variations.
        * Known BB Ultra sensitivity points (from Project Plan Section 10).
    2. **Generate Packages:** Use the application (run locally or deployed) to generate QTI ZIP packages for each test case using the prepared sample input data.
    3. **Access Blackboard Ultra:** Log in to the designated BB Ultra test instance.
    4. **Import Packages:** Use BB Ultra's content import tools (e.g., Content Market -> IMS Content Package, or direct Test/Pool import if applicable) to upload the generated ZIP packages.
    5. **Verify Import:** Check for any import errors reported by Blackboard Ultra. Document any failures immediately with package details and error messages.
    6. **Verify Functionality (CRITICAL):** For successfully imported packages:
        * Locate the imported content (Tests, Pools, Question Banks).
        * **Instructor View:** Preview items. Check rendering of text, prompts, choices, MathML (is MathJax rendering it correctly?), images/assets. Check item settings (points, metadata visibility).
        * **Student View (Attempt):** Attempt the test/questions. Verify interaction works as expected (selecting choices, entering text). Check feedback display (timing, content, formatting). Verify scoring is calculated correctly based on responses. Check asset rendering during attempt.
    7. **Document Findings:** Meticulously record the results for each test case in a structured format (e.g., spreadsheet, test management tool, shared document). Note:
        * Test Case ID/Description.
        * Input Data Used.
        * Generated Package Filename.
        * Import Status (Success/Failure + Error Message).
        * Rendering Checks (Prompt, Choices, MathML, Assets - Pass/Fail + Notes/Screenshots).
        * Functionality Checks (Interaction, Scoring, Feedback - Pass/Fail + Notes/Screenshots).
        * Any unexpected behavior or visual glitches.
    8. **Feedback Loop:** Use the documented findings (especially failures or rendering issues) to inform Stage 4 (Iteration & Refinement). Create specific bug reports or refinement tasks.
* **Agent Role:** Minimal. Potentially used to help *format* the MAT results documentation based on structured notes provided by the developer.
* **HITL Integration:**
  * **Trigger:** Need to validate against the primary acceptance criteria (BB Ultra compatibility). Performed after significant features are integrated and pass automated tests.
  * **Purpose:** **Primary Acceptance Testing.** Verify real-world usability and compatibility in the target LMS, catching issues that automated tests (especially XSD validation) cannot. Identify platform-specific quirks.
  * **Activities:** Executing the import and verification steps within Blackboard Ultra, carefully observing and documenting behavior. Analyzing failures to determine if they are bugs in the generator or known BB Ultra limitations.
  * **Context:** Generated QTI ZIP packages, Blackboard Ultra test instance, MAT protocol/checklist, Project Plan requirements (esp. Section 10).
* **Guardrails & Quality Checks:**
  * **Systematic Testing:** Follow the MAT protocol to ensure consistent testing across features.
  * **Thorough Documentation:** Record results clearly, including screenshots for rendering issues.
  * **Focus on Acceptance Criteria:** Directly test against the requirement for successful BB Ultra import and function.
* **Anticipated Errors & Mitigation:**
  * **BB Ultra Import Errors:** Package rejected despite passing XSD validation. **Mitigation:** Analyze BB Ultra error logs (if available); compare generated XML against known-working examples; consult BB documentation/forums; potentially refine generator output *within the bounds of the standard* based on findings (Stage 4).
  * **Rendering Issues:** MathML not displaying, images broken, formatting lost. **Mitigation:** Investigate the generated XML/XHTML, check asset paths, verify the CDATA strategy for MathML, compare against BB Ultra documentation/examples; potentially refine templates (Stage 4).
  * **Functional/Scoring Errors:** Interactions behaving unexpectedly, scores calculated incorrectly. **Mitigation:** Double-check the generated QTI response processing templates and declarations against the standard and BB Ultra's likely supported subset; simplify scoring logic if needed; refine templates (Stage 4).
  * **Undocumented Quirks:** Encountering undocumented behavior specific to the BB Ultra version. **Mitigation:** Document thoroughly, search for known issues, potentially develop workarounds *if essential and standard-compliant*.
* **Output / Artifact:** Detailed Manual Acceptance Testing results document/log, highlighting successes, failures, rendering issues, and functional discrepancies in Blackboard Ultra. Bug reports or refinement tasks entered into project tracking for Stage 4.

---

**Conclusion of Stage 3:**

Upon completion of Stage 3, you will have:

1. An **integrated application** where components work together.
2. A robust suite of **passing unit tests** verifying individual component logic.
3. A suite of **passing integration tests** verifying API endpoints and core end-to-end flows.
4. Crucially, documented results from **Manual Acceptance Testing in Blackboard Ultra**, providing real-world validation and identifying necessary refinements.

The application is now functionally complete (at least for the implemented features) and has undergone significant automated and manual testing. The findings from MAT (especially TASK 3.4) directly feed into **Stage 4: Iteration & Refinement**, where bugs will be fixed, and templates/logic adjusted based on the observed behavior in the target platform.
