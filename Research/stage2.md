**Detailed Implementation Flow: Stage 2 - Core Component Implementation (Iterative)**

**Overall Stage Objective:** To iteratively build and validate the core functional components of the QTI Generator (Pydantic models, FastAPI endpoints, Jinja2 templates, validation logic, packaging logic) using AI agents for code/template generation, guided by the finalized specifications from Stage 1, and ensuring quality through mandatory HITL reviews.

**Iterative Approach:** This stage is explicitly *iterative*. Do not attempt to generate everything at once.

* **Recommendation:** Start with a Minimum Viable Product (MVP) slice. For example:
    1. Implement the core Pydantic models needed for a single, simple interaction type (e.g., `ChoiceInteractionData`, `BaseItem`, `AssetReference`, minimal `LomMetadata`). (TASK 2.1.1 - 2.1.N)
    2. Implement the corresponding Jinja2 template/snippets for that interaction type and the base item/manifest structure. (TASK 2.3.1 - 2.3.N)
    3. Implement the `lxml` validation function. (TASK 2.4.1)
    4. Implement the `zipfile` packaging function. (TASK 2.5.1)
    5. Implement a basic FastAPI endpoint to tie these together for the chosen interaction type. (TASK 2.2.1)
    6. Perform initial unit/integration tests (Stage 3 preview) for this slice *before* moving to the next interaction type or feature.

**Agentic Patterns Employed in Stage 2:**

* **Task Decomposition:** Stage 2 is broken into sub-stages (2.1-2.5) focusing on specific component types. Each sub-stage involves discrete generation tasks (`TASK 2.X.N`).
* **Tool Use:** AI models act as **Code Generation Tools** based on specifications. The primary tools are the AI APIs themselves. Secondary tools used *by the generated code* (specified in prompts) include Pydantic validation, FastAPI routing, Jinja2 rendering, `lxml` validation, and `zipfile` packaging.
* **Reasoning (Planning & Execution):** Agent reasoning is primarily applied to translating specifications (natural language, code definitions) into functional code/templates. Planning occurs implicitly based on the prompt structure.
* **Memory:**
  * **Short-Term:** Context provided within each prompt (specs, related code definitions, constraints).
  * **Long-Term (Managed by Developer):** The validated outputs of Stage 1 (specs) and the accumulating, validated code/templates from previous iterations within Stage 2 serve as the persistent knowledge base guiding subsequent tasks. Version control (Git) is crucial here.

**Infrastructure/Setup for Stage 2:**

* Established Python Development Environment (IDE, Python version, virtual environment).
* Installed Libraries: FastAPI, Uvicorn, Pydantic, Jinja2, `lxml`, `python-dotenv` (for config), `pytest` (for early testing).
* Access to chosen AI Model APIs (OpenAI, Anthropic, Google) via SDKs or helper libraries.
* Stage 1 Artifacts: Finalized Pydantic specs, Jinja2 structure plan, Validation logic spec.
* Version Control System (Git) initialized for the project.
* Code Linters/Formatters configured (e.g., Ruff, Black).
* Official QTI 2.1 and IMS CP 1.2 XSD Schema files downloaded and accessible.

---

**Sub-Stage 2.1: Pydantic Model Implementation**

* **Task Objective:** Translate the finalized Pydantic model *definitions* (from TASK 1.2) into runnable, validated Python code.
* **Task ID:** `TASK 2.1.N` (where N = 1, 2, 3... for each model)
* **Detailed Execution Steps:**
    1. **Select Model:** Choose the next Pydantic model to implement based on the Stage 1 finalized spec (TASK 1.2 output).
    2. **Prepare Context:** Gather the specific definition for this model (fields, types, validation rules, inheritance) from the spec.
    3. **Formulate Prompt:** Create a precise prompt for the AI agent. Include the model name, inheritance, fields (with types, defaults), nested models (if any), and specific validation requirements mentioned in the spec. *Crucially, provide the finalized specification details in the prompt.*
    4. **Execute Prompt:** Send the prompt to the chosen AI model.
    5. **Receive & Pre-Check Code:** Get the generated Python code. Perform a quick visual scan for obvious errors.
    6. **HITL Review (Mandatory):** Proceed to the detailed HITL step.
    7. **Integrate & Format:** Once approved, add the code to the appropriate Python module (e.g., `models.py` or `schemas.py`), ensure imports are correct, and run the configured code formatter/linter (Ruff/Black).
    8. **Commit:** Commit the *reviewed and formatted* code to Git with a descriptive message (e.g., "feat: Implement Pydantic model ChoiceInteractionData (Agent-Assisted)").
* **Agent Role:** Code Generator (Python/Pydantic)
* **Example Prompt:** (Refined from strategy, assuming TASK 1.2 produced detailed specs)

    ```
    "Generate a Python Pydantic model class based on the following specification:

    Model Name: ChoiceInteractionData
    Inherits From: BaseItem (Assume BaseItem is defined elsewhere with fields like 'identifier', 'title')
    Fields:
    - prompt: str (Required, description: Main question text, may contain XHTML/MathML handled later)
    - choices: List[ChoiceOption] (Required, min_items=1, description: List of choices)
    - shuffle: bool (Default: False, description: Whether choices should be shuffled)
    - max_choices: int (Default: 1, description: Max number of choices allowed, must be >= 0)

    Nested Model Specification:
    Model Name: ChoiceOption
    Inherits From: BaseModel
    Fields:
    - identifier: UUID (Default factory: uuid.uuid4, description: Unique ID for this choice)
    - text: str (Required, description: Text content of the choice, may contain XHTML/MathML)
    - is_correct: bool (Required, description: True if this is a correct choice)
    - feedback: Optional[str] (Default: None, description: Specific feedback for selecting this option) # Added detail

    Use Pydantic v2 syntax. Include appropriate type hints. Add a field validator to ensure 'max_choices' is not negative. Generate necessary imports (List, Optional, UUID, uuid, BaseModel, Field, validator from pydantic).
    "
    ```

* **Recommended Models & Justification:**
  * **Gemini 2.5 Pro:** Excellent Python/Pydantic knowledge, handles structured instructions well. Potentially better integration if other Google Cloud services are used.
  * **GPT-4.1/4.5 Series (via OpenAI API):** State-of-the-art coding capabilities, strong adherence to Pydantic syntax and common patterns.
  * **O4-mini / Gemini Flash / Claude 3.7 Haiku (Use Judiciously):** Consider only for *very simple, small* models with minimal validation logic to optimize cost/speed, but expect higher need for HITL correction.
* **HITL Integration:**
  * **Trigger:** Agent generates the Python code for the model.
  * **Purpose:** **Mandatory Code Review & Validation.** Ensure correctness, completeness, and adherence to spec.
  * **Activities:**
    * Verify all specified fields, types, defaults, and inheritance are correctly implemented.
    * Check if nested models are defined correctly.
    * Verify generated validation logic (`@field_validator`, `constr`, etc.) matches the spec and works as intended.
    * Ensure valid Python syntax and Pydantic usage.
    * Check imports.
    * Assess code clarity and style (before auto-formatting).
  * **Context:** Finalized Pydantic spec (TASK 1.2 output), Agent's generated code, Pydantic documentation (if needed).
* **Guardrails & Quality Checks:**
  * **Spec Compliance:** Generated code MUST match the finalized spec from TASK 1.2.
  * **Valid Syntax:** Code must be syntactically correct Python.
  * **Pydantic Correctness:** Uses Pydantic features appropriately.
  * **Linter/Formatter Pass:** Code must pass checks (e.g., Ruff/Black) after review and before commit.
* **Anticipated Errors & Mitigation:**
  * **Incorrect Validation:** Agent might implement validation logic incorrectly or miss edge cases (especially complex cross-field validation). **Mitigation:** Meticulous HITL review focusing on validation logic; add unit tests early (Stage 3).
  * **Subtle Type Issues:** Might misuse `Optional`, `Union`, or complex generic types. **Mitigation:** Careful type checking during HITL.
  * **Import Errors:** Missing or incorrect imports. **Mitigation:** Check during HITL; IDE assistance.
* **Output / Artifact:** Reviewed, validated, formatted, and committed Python code file(s) containing the implemented Pydantic model(s).

---

**Sub-Stage 2.2: FastAPI Endpoint Implementation**

* **Task Objective:** Implement the FastAPI endpoints that will receive requests, use Pydantic models for validation, orchestrate the generation process, and return responses.
* **Task ID:** `TASK 2.2.N` (where N = 1, 2, 3... for each endpoint)
* **Detailed Execution Steps:**
    1. **Select Endpoint:** Choose the next FastAPI endpoint to implement (e.g., the main `/generate/qti_package`). Define its purpose, path, method, request body model, and expected success/error responses based on the overall architecture (Project Plan Section 4/12).
    2. **Prepare Context:** Gather the relevant Pydantic model (from TASK 2.1.N), required path/method, success/error response structures, and placeholder names for core logic functions (e.g., `create_qti_package`).
    3. **Formulate Prompt:** Create a specific prompt detailing the endpoint requirements. Include the path, method, request Pydantic model, success status code/body, error handling logic (Pydantic 4xx, internal server 5xx with specific exception capture), use of `async/await`, and function signatures for dependencies (even if placeholders).
    4. **Execute Prompt:** Send prompt to the chosen AI model.
    5. **Receive & Pre-Check Code:** Get the generated Python/FastAPI code.
    6. **HITL Review (Mandatory):** Proceed to detailed HITL.
    7. **Integrate & Format:** Add the code to the appropriate API router/module (e.g., `routers/generation.py`), ensure imports and dependencies (like placeholder functions or actual logic) are handled, and run formatter/linter.
    8. **Commit:** Commit the *reviewed and formatted* endpoint code.
* **Agent Role:** Code Generator (Python/FastAPI)
* **Example Prompt:** (Refined from strategy)

    ```
    "Generate a Python FastAPI endpoint using async/await based on this specification:

    Path: /generate/qti_package
    Method: POST
    Request Body Model: AssessmentPackageRequest (Assume this Pydantic model is defined, representing the full input)
    Dependencies (Placeholder Functions):
    - async def create_qti_package(data: AssessmentPackageRequest) -> str: # Returns path to generated ZIP
        # Placeholder for core generation logic
        pass
    - class GenerationError(Exception): pass # Custom exception

    Logic:
    1. Accept a POST request with a JSON body conforming to 'AssessmentPackageRequest'.
    2. FastAPI should automatically handle Pydantic validation errors, returning a 422 response.
    3. If validation passes, call the 'create_qti_package' function with the validated data.
    4. If 'create_qti_package' executes successfully:
       - Return a JSON response: {'status': 'success', 'package_url': '/path/to/generated/package.zip'} # Use the path returned by the function
       - Set HTTP status code to 200 (OK).
    5. If 'create_qti_package' raises a 'GenerationError':
       - Catch the specific 'GenerationError'.
       - Return a JSON response: {'status': 'error', 'message': str(e)} # Include the exception message
       - Set HTTP status code to 500 (Internal Server Error).
    6. If any other unexpected exception occurs during processing:
       - Return a generic JSON response: {'status': 'error', 'message': 'An unexpected error occurred.'}
       - Set HTTP status code to 500 (Internal Server Error).

    Include necessary FastAPI imports (APIRouter, Depends, HTTPException, status) and Python imports (e.g., Exception). Structure using an APIRouter.
    "
    ```

* **Recommended Models & Justification:**
  * **Gemini 2.5 Pro / GPT-4.1/4.5 Series:** Strong understanding of FastAPI patterns, async/await, dependency injection, and error handling.
* **HITL Integration:**
  * **Trigger:** Agent generates the Python code for the endpoint.
  * **Purpose:** **Mandatory Code Review & Validation.** Ensure correct routing, data handling, logic flow, error handling, and integration points.
  * **Activities:**
    * Verify path, method, request model integration (`Body(...)` or model directly).
    * Check success response structure and status code (200/201/202 as appropriate).
    * Meticulously review error handling: Correct Pydantic validation response (often automatic 422), specific exception catching (e.g., `GenerationError`), appropriate status codes (500), generic fallback catch.
    * Verify `async/await` usage is correct.
    * Check calls to placeholder/dependency functions.
    * Ensure necessary imports and router setup.
  * **Context:** Endpoint specification, relevant Pydantic model (TASK 2.1 output), Agent's generated code, FastAPI documentation.
* **Guardrails & Quality Checks:**
  * **Valid FastAPI Code:** Adheres to framework conventions.
  * **Correct Pydantic Integration:** Uses models correctly for request validation.
  * **Robust Error Handling:** Handles expected and unexpected errors gracefully with appropriate responses/codes.
  * **RESTful Principles:** Follows standard HTTP methods and status codes where applicable.
  * **Linter/Formatter Pass.**
* **Anticipated Errors & Mitigation:**
  * **Naive Error Handling:** Agent might only include a generic `except Exception` or miss specific error types. **Mitigation:** HITL focus on explicit error catching and status codes per the spec.
  * **Incorrect Async/Await:** Improper use leading to blocking or race conditions (less likely in simple endpoints, but possible). **Mitigation:** HITL review of async patterns.
  * **Incorrect Response Models:** Returning data that doesn't match the intended response schema. **Mitigation:** HITL check against spec; consider defining response Pydantic models too.
* **Output / Artifact:** Reviewed, validated, formatted, and committed Python code file(s) containing the implemented FastAPI endpoint(s) and router configuration.

---

**Sub-Stage 2.3: Jinja2 Template Generation**

* **Task Objective:** Generate the Jinja2 template files/snippets (`.xml`) based on the finalized structure (TASK 1.3) and Pydantic models (TASK 2.1), adhering strictly to QTI 2.1 and IMS CP 1.2 specs. **This remains a high-risk area requiring extreme diligence.**
* **Task ID:** `TASK 2.3.N` (where N = 1, 2, 3... for each template/snippet/macro)
* **Detailed Execution Steps:**
    1. **Select Template:** Choose the next Jinja2 file/snippet/macro to create (e.g., `_choiceInteraction.xml`, `base_item.xml`, `render_mathml` macro) based on the TASK 1.3 plan and the current implementation focus (e.g., choice interaction).
    2. **Prepare Context:** Gather:
        * The corresponding Pydantic model definition(s) that will provide context data (e.g., `ChoiceInteractionData`).
        * The specific QTI 2.1 / IMS CP 1.2 structure required for this component. Reference the official specifications!
        * Requirements from TASK 1.1/Plan Section 6 (namespaces, CDATA for MathML, autoescaping assumption).
    3. **Formulate Prompt (Highly Specific):** Construct a very detailed prompt. Include:
        * The target XML structure (maybe provide a simplified XML example).
        * The *exact* Pydantic context variable name(s) and their structure (reference the model definitions).
        * Explicit instructions for loops, conditionals, attribute values (referencing context variables).
        * Namespace requirements (e.g., "Assume parent declares necessary namespaces" or "Ensure this fragment uses prefix X for namespace Y").
        * Instructions for complex content (e.g., "Use the `render_content` filter for the `prompt` field", "Call the `render_mathml` macro for MathML strings").
        * Reminders about XML validity and Jinja2 syntax.
        * *Provide the Pydantic model definitions directly in the prompt context.*
    4. **Execute Prompt:** Send to the chosen AI model. *Expect potential failures or imperfect output.*
    5. **Receive & Pre-Check Template:** Get the generated `.xml` template content. Visually inspect for basic structure and Jinja2 syntax.
    6. **HITL Review (CRITICAL & MANDATORY):** Proceed to the most intensive HITL review.
    7. **Integrate & Test Render:** Place the *reviewed* template in the correct location (defined in TASK 1.3). **Crucially, write a small test script or unit test to render this template with sample Pydantic data and inspect the raw XML output.**
    8. **Validate Rendered Output (Early Check):** If possible, run the rendered XML output (from step 7) through the `lxml` validator (TASK 2.4.1) even before full integration. This provides rapid feedback.
    9. **Commit:** Commit the *reviewed and minimally validated* template file.
* **Agent Role:** Template Generator (Jinja2/XML)
* **Example Prompt:** (Using the highly specific example for `_choiceInteraction.xml` from the strategy document, assuming `ChoiceInteractionData` and `ChoiceOption` Pydantic models provided in context)

    ```
    "Generate a Jinja2 template snippet suitable for inclusion in a QTI 2.1 assessmentItem XML file. This snippet represents a 'choiceInteraction'.

    Context Variable: Assume a variable named 'item' is available, which is an instance of the 'ChoiceInteractionData' Pydantic model (definition provided below).
    Pydantic Context Models:
    [Paste ChoiceInteractionData and ChoiceOption Pydantic model code here]

    Target XML Structure (QTI 2.1):
    <choiceInteraction responseIdentifier="RESPONSE" shuffle="{{ item.shuffle }}" maxChoices="{{ item.max_choices }}">
      <prompt>{{ render_content(item.prompt) }}</prompt> {# Use render_content filter #}
      {% for choice in item.choices %}
      <simpleChoice identifier="{{ choice.identifier | string }}"> {# Ensure identifier is string #}
         {{ render_content(choice.text) }} {# Use render_content filter #}
      </simpleChoice>
      {% endfor %}
    </choiceInteraction>

    Requirements:
    1. The root element MUST be '<choiceInteraction>'.
    2. The 'responseIdentifier' attribute MUST be the literal string 'RESPONSE'.
    3. The 'shuffle' attribute MUST be dynamically set from 'item.shuffle' (boolean converted to 'true'/'false' string).
    4. The 'maxChoices' attribute MUST be dynamically set from 'item.max_choices' (integer converted to string, handle 0 value correctly for QTI).
    5. Include a '<prompt>' element containing the content of 'item.prompt'. Apply a filter named 'render_content' to this variable.
    6. Iterate through the list 'item.choices'. For each 'choice' object in the list:
       - Generate a '<simpleChoice>' element.
       - The 'identifier' attribute MUST be the UUID from 'choice.identifier', converted to a string.
       - The content of '<simpleChoice>' MUST be the text from 'choice.text'. Apply the 'render_content' filter to this variable.
    7. Assume the Jinja2 environment is configured for XML autoescaping.
    8. Assume required QTI namespaces are declared on a parent element; do not declare them here.
    9. Ensure correct Jinja2 syntax for variables, loops, and filters.
    "
    ```

* **Recommended Models & Justification:**
  * **Claude 3.7 Sonnet/Pro:** Excels at following complex, structured instructions and generating formatted text (like XML). The constitutional approach might help stick closer to the detailed constraints. Often performs better on XML/structured text tasks.
  * **Gemini 2.5 Pro:** Strong reasoning and instruction following, large context window helpful for including Pydantic definitions.
  * **GPT-4.1/4.5 Series:** Also highly capable, but may sometimes require more prompt iteration for precise XML structures compared to Claude.
  * ***Expect Iteration:*** Regardless of the model, generating perfect, complex XML/Jinja2 first-time is unlikely. Budget time for refinement based on HITL review and validation.
* **HITL Integration:**
  * **Trigger:** Agent generates the Jinja2 template content.
  * **Purpose:** ***CRITICAL Review & Validation.*** Ensure XML correctness, QTI compliance, correct data binding, and robustness.
  * **Activities:**
    * **XML Well-formedness:** Check basic XML syntax (tags closed, attributes quoted).
    * **QTI 2.1 Structure:** Compare generated structure against the official QTI spec for this element/interaction type.
    * **Variable Usage:** Meticulously check that *every* Jinja2 variable (`{{ ... }}`) matches the Pydantic context structure *exactly*.
    * **Logic:** Verify loops (`{% for ... %}`) and conditionals (`{% if ... %}`) implement the intended logic.
    * **Attributes:** Ensure all required attributes are present and their values are derived correctly from context.
    * **Namespaces:** Confirm namespace handling aligns with the strategy (implicit or explicit).
    * **Filter/Macro Calls:** Check that placeholders or calls to custom filters/macros (`render_content`, `render_mathml`) are correct.
    * ***Render Test:*** Render the template with diverse sample Pydantic data (including edge cases like empty lists where allowed, null values, content with special XML characters).
    * ***Inspect Rendered Output:*** Examine the raw XML generated from the render test. Is it valid? Does it look correct?
    * ***Validate Rendered Output:*** Run the sample rendered XML through the `lxml` validator (TASK 2.4.1).
  * **Context:** Finalized Pydantic model(s), QTI/IMS CP specifications, Jinja2 Structure Plan (TASK 1.3), Agent's generated template, Sample Pydantic data, `lxml` validation tool/script.
* **Guardrails & Quality Checks:**
  * **QTI/CP Spec Adherence:** Template must generate XML conforming to the standard.
  * **Variable Name Accuracy:** Jinja2 variables MUST match Pydantic attributes.
  * **XML Validity:** Rendered output must be well-formed XML and pass XSD validation.
  * **Escaping Handled:** Rely on Jinja2 autoescaping, ensure CDATA is used where specified (MathML).
* **Anticipated Errors & Mitigation:**
  * **Subtle XML Errors:** Incorrect nesting, missing attributes, invalid attribute values, namespace confusion. **Mitigation:** Meticulous HITL, rendering with sample data, *mandatory* `lxml` validation of rendered output.
  * **Incorrect Variable Access:** Typo in variable name, accessing non-existent attribute. **Mitigation:** HITL comparison against Pydantic models; render testing will often fail.
  * **Flawed Logic:** Incorrect loop bounds, faulty conditional checks. **Mitigation:** HITL review of logic; render testing with diverse data.
  * **Escaping Issues:** Double-escaping, failing to use CDATA for MathML. **Mitigation:** HITL focus on complex content handling; configure Jinja2 environment correctly; test rendering.
  * **Strategy:** Break complex templates into smaller includes/macros. Generate, review, and validate these smaller pieces individually.
* **Output / Artifact:** Reviewed, validated (via rendered sample + lxml), and committed Jinja2 template file(s) (`.xml`).

---

**Sub-Stage 2.4: Validation Logic Implementation (`lxml`)**

* **Task Objective:** Implement the core XML validation function using `lxml`, based on the precise specification defined in TASK 1.4.
* **Task ID:** `TASK 2.4.1` (Likely one primary function initially)
* **Detailed Execution Steps:**
    1. **Retrieve Spec:** Get the finalized validation logic spec from TASK 1.4.
    2. **Prepare Context:** Ensure the spec details (XSD path handling, caching mechanism via dict, `lxml` usage, error parsing, custom exception) are clear.
    3. **Formulate Prompt:** Create a prompt based directly on the TASK 1.4 spec. Include the function signature, specific library (`lxml.etree`), caching logic, validation steps, success/failure behavior, custom exception definition, and specific error handling requirements (parsing `error_log`).
    4. **Execute Prompt:** Send to the AI model.
    5. **Receive & Pre-Check Code:** Get the generated Python code for the validation function.
    6. **HITL Review (Mandatory):** Proceed to detailed HITL.
    7. **Integrate & Format:** Add the function to a suitable utility module (e.g., `utils/validation.py`), define the custom exception class, ensure imports, and run formatter/linter.
    8. **Commit:** Commit the reviewed and formatted validation code.
* **Agent Role:** Code Generator (Python/lxml)
* **Example Prompt:** (Based on strategy example and assuming TASK 1.4 finalized the spec)

    ```
    "Generate a Python function and a custom exception class based on this specification:

    Custom Exception Class:
    - Name: XMLValidationError
    - Inherits from: Exception
    - Attributes: Should store the detailed error log from lxml.

    Validation Function:
    - Name: validate_xml
    - Arguments:
        - xml_string: str (The XML content to validate)
        - xsd_path: str (Filesystem path to the XSD schema file)
        - schema_cache: dict (A dictionary used for caching parsed lxml.etree.XMLSchema objects)
    - Logic:
        1. Define the 'XMLValidationError' exception class as specified above.
        2. Inside 'validate_xml': Check if 'xsd_path' is a key in the 'schema_cache' dictionary.
        3. If YES: Retrieve the cached 'lxml.etree.XMLSchema' object from the dictionary.
        4. If NO:
           - Attempt to parse the XSD file using 'lxml.etree.XMLSchema(file=xsd_path)'.
           - Handle potential 'IOError' or 'lxml.etree.XMLSchemaParseError' during XSD parsing by raising a standard Python exception (e.g., ValueError) with an informative message including the path.
           - If successful, store the newly parsed schema object in 'schema_cache' with 'xsd_path' as the key.
        5. Attempt to parse the input 'xml_string' into an lxml etree object using 'lxml.etree.fromstring()'. Handle potential 'lxml.etree.XMLSyntaxError' by raising the custom 'XMLValidationError' containing the syntax error details.
        6. Validate the parsed XML tree using the (potentially cached) schema object's 'validate()' method.
        7. If 'schema.validate(xml_tree)' returns False:
           - Get the detailed error log using 'schema.error_log'.
           - Raise the custom 'XMLValidationError', passing the 'error_log' to it.
        8. If validation succeeds (returns True): Return True.
    - Imports: Include necessary imports from 'lxml.etree'.
    "
    ```

* **Recommended Models & Justification:**
  * **Gemini 2.5 Pro / GPT-4.1/4.5 Series:** Reliable for generating Python code involving specific library usage (`lxml`) and following structured logic like caching and error handling.
* **HITL Integration:**
  * **Trigger:** Agent generates the validation function code.
  * **Purpose:** **Mandatory Code Review & Validation.** Ensure correct `lxml` usage, robust caching, and proper error handling/reporting.
  * **Activities:**
    * Verify correct use of `lxml.etree.XMLSchema` and `lxml.etree.fromstring`.
    * Check the caching logic (dict key check, retrieval, storage).
    * Confirm the `schema.validate()` method is used correctly.
    * **Crucially:** Verify that on validation failure, the `schema.error_log` is correctly accessed and passed to the custom `XMLValidationError`.
    * Check exception handling for XSD parsing (`IOError`, schema parse errors) and XML parsing (`XMLSyntaxError`).
    * Ensure the function returns `True` only on successful validation.
    * Test manually with sample valid/invalid XML and XSDs if feasible early on.
  * **Context:** Finalized Validation Logic Spec (TASK 1.4 output), Agent's generated code, `lxml` documentation.
* **Guardrails & Quality Checks:**
  * **Correct `lxml` API Use:** Uses library functions as intended.
  * **Effective Caching:** Implements the specified caching mechanism correctly.
  * **Robust Error Handling:** Catches expected errors, raises the specified custom exception with required details (`error_log`).
  * **Spec Compliance:** Fully implements the logic defined in TASK 1.4.
  * **Linter/Formatter Pass.**
* **Anticipated Errors & Mitigation:**
  * **Incorrect Error Log Handling:** Agent might fail to capture or pass the `error_log` correctly. **Mitigation:** Specific HITL check on this step. Write unit tests (Stage 3) to assert error log content in exceptions.
  * **Inefficient Caching:** Logic flaws in checking/updating the cache. **Mitigation:** HITL walkthrough of caching logic.
  * **Swallowing Errors:** Might use a broad `except Exception:` that hides specific `lxml` errors. **Mitigation:** HITL check for specific exception handling.
* **Output / Artifact:** Reviewed, validated, formatted, and committed Python code file containing the `validate_xml` function and `XMLValidationError` exception class.

---

**Sub-Stage 2.5: Packaging Logic Implementation (`zipfile`)**

* **Task Objective:** Implement the function to create the IMS Content Package ZIP file using the `zipfile` module, based on Project Plan Section 8 and requirements from TASK 1.1.
* **Task ID:** `TASK 2.5.1` (Likely one primary function initially)
* **Detailed Execution Steps:**
    1. **Retrieve Spec:** Consolidate requirements for packaging from Project Plan Section 8 and TASK 1.1 output (manifest name/location, resource structure, relative paths, `zipfile` usage, compression).
    2. **Prepare Context:** Define the function signature clearly (inputs: manifest XML string, dict of item XML strings, dict of asset file paths, output zip path).
    3. **Formulate Prompt:** Create a prompt based on the spec. Detail the required ZIP structure (manifest at root), how to handle item files (write string content to `<identifier>.xml` at root), how to handle asset files (add from source path to relative destination path within ZIP), and the required compression (`ZIP_DEFLATED`).
    4. **Execute Prompt:** Send to the AI model.
    5. **Receive & Pre-Check Code:** Get the generated Python code for the packaging function.
    6. **HITL Review (Mandatory):** Proceed to detailed HITL.
    7. **Integrate & Format:** Add the function to a suitable utility module (e.g., `utils/packaging.py`), ensure imports, and run formatter/linter.
    8. **Commit:** Commit the reviewed and formatted packaging code.
* **Agent Role:** Code Generator (Python/zipfile)
* **Example Prompt:** (Based on strategy example and plan requirements)

    ```
    "Generate a Python function to create an IMS Content Package ZIP file using the 'zipfile' module, based on this specification:

    Function Name: create_ims_package
    Arguments:
    - manifest_xml: str (The complete imsmanifest.xml content as a string)
    - item_files: Dict[str, str] (Key: resource identifier (UUID string), Value: assessmentItem XML content as a string)
    - asset_files: Dict[str, str] (Key: Relative path within ZIP (e.g., 'assets/image1.png'), Value: Source filesystem path to the asset file)
    - output_zip_path: str (The full path where the output ZIP file should be saved)

    Logic:
    1. Create a new ZIP archive at 'output_zip_path'. Use 'zipfile.ZIP_DEFLATED' compression. Ensure the zipfile is properly closed (e.g., using a 'with' statement).
    2. Write the 'manifest_xml' string content to a file named exactly 'imsmanifest.xml' at the ROOT level inside the ZIP archive.
    3. Iterate through the 'item_files' dictionary:
       - For each key-value pair (identifier, xml_content):
       - Construct the filename within the ZIP as '<identifier>.xml' (e.g., 'uuid_string.xml').
       - Write the 'xml_content' string to this file at the ROOT level inside the ZIP archive.
    4. Iterate through the 'asset_files' dictionary:
       - For each key-value pair (zip_relative_path, source_filesystem_path):
       - Add the file from 'source_filesystem_path' into the ZIP archive.
       - The path of the file INSIDE the ZIP archive MUST be exactly 'zip_relative_path' (preserving any directory structure specified in the key).
    5. Handle potential IOErrors during file writing or reading (e.g., if source asset doesn't exist or output path is invalid) by raising appropriate standard Python exceptions.

    Include necessary imports (zipfile, Dict, os - if needed for path manipulation, though zipfile often handles it).
    "
    ```

* **Recommended Models & Justification:**
  * **Gemini 2.5 Pro / GPT-4.1/4.5 Series:** Capable of generating correct Python code for file I/O and library usage like `zipfile`.
* **HITL Integration:**
  * **Trigger:** Agent generates the packaging function code.
  * **Purpose:** **Mandatory Code Review & Validation.** Ensure correct ZIP structure, file placement, path handling, and compression.
  * **Activities:**
    * Verify correct `zipfile` API usage (`ZipFile`, `writestr`, `write`, context manager `with`).
    * Confirm `imsmanifest.xml` is written to the root.
    * Check item file naming (`<identifier>.xml`) and placement (root).
    * **Crucially:** Verify asset file handling – check that the `arcname` argument in `zipfile.write` is correctly set to the *key* from `asset_files` (the relative path) and the file source is the *value*.
    * Confirm `ZIP_DEFLATED` compression is used.
    * Check basic error handling (e.g., does it assume source files exist?).
    * Generate a test ZIP file manually with this function using sample data and inspect its contents using standard OS tools or Python's `zipfile` module again.
  * **Context:** Packaging requirements (Plan Sec 8 / TASK 1.1), Agent's generated code, `zipfile` documentation, Sample manifest/item/asset data.
* **Guardrails & Quality Checks:**
  * **Correct ZIP Structure:** Manifest at root, items at root (or specified path), assets at correct relative paths.
  * **File Content:** Verify string data is written correctly (`writestr`) and files are added correctly (`write`).
  * **Path Handling:** `arcname` must match the intended relative path inside the ZIP.
  * **Spec Compliance:** Adheres to IMS CP structure requirements.
  * **Linter/Formatter Pass.**
* **Anticipated Errors & Mitigation:**
  * **Incorrect Asset Paths:** Agent might confuse source path and destination `arcname`, or fail to create nested directories within the ZIP. **Mitigation:** Specific HITL check on the `zipfile.write(source_path, arcname=zip_relative_path)` call; test generation and inspection of the ZIP structure.
  * **Manifest Placement Error:** Putting manifest in a subdirectory. **Mitigation:** HITL check and test ZIP inspection.
  * **Compression Ignored:** Forgetting to specify `ZIP_DEFLATED`. **Mitigation:** HITL check.
* **Output / Artifact:** Reviewed, validated (via test generation + inspection), formatted, and committed Python code file containing the `create_ims_package` function.

---

**Conclusion of Stage 2:**

Upon completing iterations within Stage 2 (ideally starting with an MVP slice and expanding), you will have:

1. Runnable, reviewed, and formatted Python code for your Pydantic input models.
2. Runnable, reviewed, and formatted Python code for your FastAPI endpoints.
3. Reviewed and validated Jinja2 templates for generating the XML components.
4. A reviewed and validated Python function for performing `lxml` schema validation.
5. A reviewed and validated Python function for packaging content into an IMS CP ZIP file.

All generated artifacts have passed mandatory HITL review and basic checks (linting, formatting, potentially early rendering/validation tests for templates). The codebase is managed in Git. You are now ready to move to **Stage 3: Integration & Testing**, where these components will be tied together more formally, comprehensive tests written (including agent assistance for boilerplate), and crucial validation against Blackboard Ultra will occur.
