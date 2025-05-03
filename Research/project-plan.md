**Project Plan: IMS QTI 2.1 Assessment Package Generator**

**Document Version:** 2.0
**Date:** April 28, 2025
**Prepared For:** Solutions Team Implementation
**Prepared By:** AI Expert Consultant (Educational Technology Standards & Content Interoperability)
**Context:** Planning for a web application to generate IMS QTI 2.1 compliant assessment packages using Python and Jinja2, focusing on standards compliance, Blackboard Ultra compatibility, and future readiness. This version incorporates feedback and detailed strategies from initial review.

**1. Introduction & Executive Summary**

This document outlines the architectural plan and development strategy for a web application designed to generate robust, valid IMS QTI 2.1 compliant assessment packages (ZIP files containing test/question bank XML and associated resources). The system will utilize a Python backend (FastAPI recommended) with the Jinja2 templating engine to create `assessmentItem`, `assessmentTest`, and `imsmanifest.xml` files based on structured input data. Key features include support for various standard QTI interaction types, accurate handling and embedding of Presentation MathML, management of associated static assets (images, etc.), and strict adherence to IMS QTI 2.1 and IMS Content Packaging (v1.2) specifications.

The primary goal is to create a reliable, maintainable system that produces highly interoperable QTI content. While aiming for broad QTI 2.1 compliance, **successful, reliable, and functionally correct import into the current version of Blackboard Ultra is a primary acceptance criterion.** The system architecture aims to facilitate potential future migration to newer QTI versions (2.2/3.0), but the immediate generation target is **strictly QTI 2.1**.

**2. Core Problem & Objectives**

The creation of valid and complex QTI assessment packages is challenging due to the intricacies of the XML schemas, the need for unique identifiers, correct packaging including associated resources, ensuring content fidelity (especially for mathematical notation), and achieving reliable import into specific LMS platforms like Blackboard Ultra. Manual creation or inadequate tools often lead to errors, validation failures, and import issues.

**Objectives:**

* **QTI 2.1 Compliance:** Generate `assessmentItem` and `assessmentTest` XML that strictly validates against the official IMS QTI 2.1 XSD schema.
* **IMS CP Compliance:** Generate `imsmanifest.xml` and package content into ZIP archives that strictly validate against the official IMS Content Packaging v1.2 XSD schema.
* **Content Fidelity:** Accurately represent question prompts, choices, feedback, and metadata, with specific focus on correctly rendering and embedding Presentation MathML and associated static assets (images, media).
* **Interaction Support:** Support a configurable range of standard QTI 2.1 interaction types (e.g., choice, text entry, order, match, extended text).
* **Robustness & Reliability:** Implement comprehensive validation (input data, generated XML) and error handling to minimize invalid package generation.
* **Blackboard Ultra Compatibility:** Ensure generated packages import reliably and function correctly within Blackboard Ultra, serving as the primary target platform for validation beyond basic schema compliance.
* **Accessibility Compliance:** Aim for WCAG 2.1 AA conformance where applicable within the QTI structure (e.g., support for alternative text for images).
* **Maintainability & Extensibility:** Design a modular architecture that simplifies updates, adding new interaction types, and potential future migration to newer QTI versions.
* **Testability:** Ensure the core generation logic, components, and API are designed for unit and integration testing.
* **Usability:** Provide a straightforward mechanism for content authors to input question data, including options for bulk/programmatic import (e.g., via structured JSON).

**3. Guiding Principles**

* **Standards First:** Prioritize strict adherence to official 1EdTech specifications (QTI 2.1, IMS CP 1.2, IMS LOM, Presentation MathML, XHTML) as the primary measure of success.
* **Validation is Non-Negotiable:** Integrate mandatory input validation (Pydantic) and output XSD schema validation (`lxml`) at critical points in the workflow. Generation fails on validation errors.
* **Target Platform Awareness:** While "Standards First" is paramount, rigorously test against Blackboard Ultra to identify and document specific interpretations, quirks, or limitations *after* ensuring standard compliance. Use this feedback to refine generation patterns within the bounds of the standard.
* **Modularity:** Design components (templates, code modules, data models) for reusability and ease of maintenance.
* **Clear Error Handling:** Provide specific and actionable feedback when validation or generation fails, pinpointing the source of the error where possible.
* **Interoperability:** Aim for generated packages to be functional across multiple QTI 2.1-compliant platforms, using Blackboard Ultra as the primary, stringent testbed.
* **Pragmatism:** Employ tested best practices for handling common issues (e.g., MathML embedding via CDATA, asset path management) based on known platform behaviors.

**4. Proposed Architecture**

A web application architecture is proposed, separating concerns for scalability and maintainability.

* **Backend:**
  * **Language:** Python 3.x
  * **Framework:** **FastAPI** (Recommended) - Offers built-in data validation (Pydantic), async capabilities, automatic API documentation. Flask is an alternative.
  * **Responsibilities:** API endpoints, input data validation, orchestration of the generation process, interaction with the database and asset storage, task management.
* **Data Modeling (Input):**
  * **Technology:** **Pydantic** models defined in Python.
  * **Structure:** Define clear, validated Python classes representing the logical structure of questions, tests, metadata, feedback, scoring rules, and asset references, abstracting from raw XML. See Section 5 for details.
* **Database:**
  * **Technology:** **PostgreSQL** (Recommended) or similar robust relational database.
  * **Schema:** Store the *validated Pydantic input data models*, user information, package generation history/metadata, asset metadata, and links to generated files/assets. Avoid storing generated XML directly.
* **Templating Engine:**
  * **Technology:** **Jinja2**
  * **Responsibilities:** Render XML (`assessmentItem`, `assessmentTest`, `imsmanifest.xml`) based on the validated input data and configuration provided by the backend. See Section 6 for details.
* **Frontend:**
  * **Initial Implementation:** Server-rendered HTML forms (using Jinja2 via backend framework) for manual entry and basic management. A modern JS framework (Vue, React, Svelte) could be used for a richer UI later.
  * **Input Methods:**
    * UI Forms tailored to specific question types.
    * Rich Text Editor (e.g., TinyMCE, CKEditor) configured for clean XHTML output, potentially with MathML input support/plugin.
    * **Crucial:** File upload/download and direct text area input for structured data (**JSON** strongly recommended, mirroring Pydantic models) for bulk operations.
  * **User Management:** Standard authentication/authorization mechanisms.
* **Static Asset Handling:**
  * **Requirement:** Images, audio, video files referenced in QTI content must be managed and included in the package.
  * **Strategy:**
        1. **Data Model:** Pydantic models for questions/items will include a list field for associated asset metadata (e.g., original filename, unique stored identifier/path, desired path within the package ZIP, alt text).
        2. **Storage:** Use a reliable storage mechanism (e.g., local filesystem directory accessible by the app, cloud storage like AWS S3). Store assets using unique identifiers (e.g., UUIDs) to avoid filename collisions.
        3. **Upload Interface:** Frontend requires an asset upload component linked to question creation/editing.
        4. **Backend Logic:**
            *Associate uploaded assets with the corresponding question data model in the database.
            * During package generation: Retrieve necessary asset files from storage.
            *Include asset files in the ZIP archive at the correct relative path specified in the metadata.
            * Generate `<file href="path/to/asset.png"/>` entries within the relevant `<resource>` element in `imsmanifest.xml`, matching the relative path in the ZIP.
            * Ensure QTI content (e.g., `<img>`, `<object>`) uses the correct *relative* `src` or `data` attribute matching the manifest `href`.
* **Configuration Management:**
  * **Requirement:** Manage application settings (XSD paths, Pandoc path, namespaces, default values, database connection strings, asset storage config) securely and flexibly.
  * **Strategy:** Use environment variables (preferred for sensitive data) possibly combined with a configuration file (e.g., YAML, `.env`) loaded at application startup (e.g., via Pydantic's Settings Management). Avoid hardcoding configurations.
* **Task Queue (Optional but Recommended for Scale):**
  * **Technology:** Celery with Redis/RabbitMQ.
  * **Purpose:** Offload potentially time-consuming QTI generation, validation, and packaging tasks from the web request cycle to background workers, improving UI responsiveness for large jobs or complex validation.

**5. Data Modeling Strategy (Input - Pydantic)**

* **Core Principle:** Use Pydantic for defining the expected input structure, providing automatic validation, type safety, clear documentation (via models), and seamless integration with FastAPI.
* **Structure:**
  * Define base models for common elements (e.g., `BaseItem`, `FeedbackBlock`, `LomMetadata`).
  * Create specific models inheriting from bases for each supported interaction type (e.g., `ChoiceInteractionData`, `TextEntryInteractionData`), containing fields relevant only to that type (e.g., `choices` list, `correct_response_value`, `shuffle`, `expected_length`).
  * **Pedagogical Intent:** Design models around the *meaning* and *structure* of the assessment content (e.g., question text, options, correct answer, scoring rule type) rather than mirroring the QTI XML structure rigidly. The mapping to XML occurs in the templates.
  * **Fields:** Models will typically include `title`, `identifier` (UUID, auto-generated if not provided), `prompt` (string, potentially XHTML/MathML), `interaction_type` (enum/string), interaction-specific data structures, scoring rules (points, mapping, standard template selection), feedback blocks, LOM metadata object, and a list of associated `AssetReference` objects.
  * **Feedback:** Define structured models for different feedback types (e.g., `ModalFeedback`, `InlineFeedback`) allowing association with specific outcomes (e.g., correct, incorrect, specific choices).
  * **Scoring:** Model the scoring *intent*. Initially focus on common scenarios: assigning a score outcome based on correctness using standard QTI `responseProcessing` templates (`MATCH_CORRECT`, `MAP_RESPONSE`). Avoid modeling complex custom response processing logic initially.
  * **Metadata (LOM):** Define a `LomMetadata` Pydantic model reflecting a chosen subset of the IMS LOM schema (e.g., technical, educational, classification). Prioritize fields known to be recognized/mapped by Blackboard Ultra (test and document this mapping: likely `general.title`, `general.description`, `general.keyword`, `educational.difficulty`).
  * **Asset References:** Include a list field (e.g., `assets: List[AssetReference]`) where `AssetReference` is another Pydantic model holding asset metadata (unique ID, package path, type, alt text).

**6. Templating Strategy (Jinja2)**

* **Modular Organization:** Essential for maintainability.
  * `templates/base/`: `base_item.xml`, `base_test.xml`, `base_manifest.xml` (skeletons using Jinja2 blocks).
  * `templates/interactions/`: Snippets or full files defining the specific interaction block (e.g., `_choiceInteraction.xml`, `_textEntryInteraction.xml`). Included/called via macros.
  * `templates/includes/` or `templates/macros/`: Reusable components:
    * Response/Outcome Declarations (`_responseDeclaration.xml`, `_outcomeDeclaration.xml`).
    * Response Processing (`_responseProcessing.xml` - macros supporting standard templates).
    * Metadata (`_lom_metadata.xml` - rendering the `LomMetadata` model).
    * Asset embedding logic (e.g., macro for `<img>` tags with relative paths and alt text).
    * MathML rendering logic.
* **Technical Details & Best Practices:**
  * **XML Autoescaping:** **Critically important.** Configure the Jinja2 environment specifically for XML autoescaping to prevent invalid XML and potential injection issues. Use `jinja2.select_autoescape(enabled_extensions=('xml',), default_for_string=True, default=True)`. Rely on this over manual escaping where possible.
  * **Namespaces:** Define all required XML namespaces (QTI 2.1, XHTML, MathML, LOM, XML Schema Instance) centrally (e.g., as Jinja2 globals/context variables) and declare them explicitly in the root elements of generated XML files (`assessmentItem`, `assessmentTest`, `imsmanifest.xml`). Ensure the MathML namespace (`xmlns="http://www.w3.org/1998/Math/MathML"`) is *always* present on the root `<math>` element itself.
  * **Custom Filters & Globals:**
    * `generate_uuid()`: Generate `uuid.uuid4()` strings.
    * `render_mathml(mathml_string)`: Takes a Presentation MathML string. **Strategy:** Ensure the string includes `<math xmlns="http://www.w3.org/1998/Math/MathML">...</math>`. When embedding this within an XHTML context (`<p>`, `<div>`, `<itemBody>`, `<simpleChoice>`, etc.), **wrap the entire MathML string in `<![CDATA[...]]>`** for robustness against parsing conflicts and special characters. Test this thoroughly with Blackboard Ultra's MathJax rendering.
    * Macros for generating standard QTI structures based on input models.
  * **XHTML Content Handling:** Input designated as XHTML (e.g., in prompts, choices, feedback) **must be valid and well-formed**.
    * **Strategy:** Configure any frontend Rich Text Editor to output clean XHTML. Consider server-side validation/sanitization using a library like `lxml` or `bleach` *before* storing or passing the content to Jinja2, removing potentially harmful or invalid tags/attributes.
  * **Whitespace Control:** Utilize Jinja2's whitespace control features (`{%-`, `-%}`) judiciously to produce cleaner, more readable XML output, especially important for preventing unwanted whitespace within elements where it might be significant.

**7. Validation Strategy**

Validation is multi-layered and non-negotiable:

1. **Input Data Validation (Pydantic):** Automatically performed when data is received at the API endpoint or loaded into Pydantic models. Catches structural errors, type mismatches, and missing required fields *before* any generation logic runs. Errors here immediately stop processing and return informative messages to the user.
2. **XML Schema (XSD) Validation (lxml):** The **primary technical validation** for generated output.
    * **Tool:** Python's `lxml.etree` library.
    * **Process:**
        * Validate *each* generated `assessmentItem` XML against the official QTI 2.1 XSD.
        * Validate the generated `assessmentTest` XML (if applicable) against the official QTI 2.1 XSD.
        * Validate the generated `imsmanifest.xml` against the official IMS Content Packaging v1.2 XSD.
    * **Schemas:** Obtain official XSD files from 1EdTech. Store them locally accessible to the application.
    * **Caching Strategy:** Parse schemas once using `lxml.etree.XMLSchema()` and cache the resulting validator objects in memory for reuse to improve performance during batch generation.
    * **Outcome:** **Generation fails if *any* XSD validation fails.**
3. **Blackboard Ultra Compatibility Testing (Manual/Automated):**
    * **Purpose:** Verify successful import and correct rendering/functionality within the target LMS, which may have interpretations beyond basic XSD compliance.
    * **Process:** Regularly import generated test packages (covering various question types, MathML, assets, metadata) into a Blackboard Ultra test instance. Verify item rendering, scoring, feedback display, and asset linking.
    * **Feedback Loop:** Document any BB Ultra import errors or rendering issues, even for XSD-valid packages. Use this feedback to refine Jinja2 templates or identify unsupported QTI patterns for BB Ultra, while staying within the standard.
4. **Schematron Validation (Potential Future Enhancement):** Implement Schematron rules for enforcing best practices or constraints not expressible in XSD (e.g., accessibility rules).
5. **Error Reporting Strategy:**
    * Validation failures must produce clear, user-friendly error messages.
    * **XSD Errors:** Parse the `lxml` error log (`validator.error_log`). Extract line numbers, the specific validation message, and ideally map the error back to the item identifier (`assessmentItem/@identifier`) and potentially the input data field that caused the issue.
    * **Input Errors:** Pydantic/FastAPI provide detailed information about which input field failed validation and why.

**8. Packaging Strategy**

* **Standard:** **IMS Content Packaging v1.2**. Adhere strictly to this specification.
* **`imsmanifest.xml` Generation:**
  * Use a dedicated Jinja2 template (`base_manifest.xml`).
  * Dynamically populate based on the items/tests and assets being packaged.
  * **Structure:** Correctly nest `<manifest>`, `<metadata>` (referencing IMS LOM schema), `<organizations>`, `<resources>`.
  * **`<resource>` Elements:** Each resource (QTI item, test, image, etc.) requires a `<resource>` entry:
    * `identifier`: **Must exactly match** the `identifier` of the corresponding QTI item/test root element (use the same UUID). Generate unique identifiers for assets.
    * `type`: Use official identifiers: `imsqti_item_xmlv2p1`, `imsqti_test_xmlv2p1`. For assets, use appropriate MIME types (e.g., `image/png`, `image/jpeg`, `application/pdf`).
    * `href`: The **relative path** to the file *within the ZIP archive*. This path must be correct.
    * `<file>` Child Elements: List the actual file(s) making up the resource using `<file href="..."/>`. The `href` here matches the resource `href` for single-file resources.
    * Dependencies: List dependencies if needed (though often not required for simple item banks).
* **ZIP Archive Creation:**
  * **Tool:** Python's standard `zipfile` module.
  * **Structure:**
    * The `imsmanifest.xml` file **must** be at the root level of the ZIP archive.
    * All referenced resources (`href` targets in the manifest) must exist within the ZIP at the specified relative paths. Maintain directory structures within the ZIP as specified by `href` attributes.
  * Use appropriate compression (e.g., `ZIP_DEFLATED`).

**9. MathML Handling**

* **Input Format:** Accept pre-rendered **Presentation MathML** strings.
* **Conversion Tool (Optional but Recommended):** Use **Pandoc** (via command-line or Python wrapper) server-side to convert formats like LaTeX (`--mathml` flag) or AsciiMath to Presentation MathML *before* data is stored or passed to templates.
  * **Dependency Management:** Ensure Pandoc is installed in the deployment environment. Implement robust error handling for Pandoc execution failures (e.g., capture stderr, check exit codes).
* **Storage:** Store the final, valid Presentation MathML string (including the root `<math>` element with namespace) in the database field associated with the relevant content (prompt, choice, feedback).
* **Embedding Strategy (Jinja2 Filter/Macro):**
  * The `render_mathml` function/filter receives the stored MathML string.
  * It ensures the `<math xmlns="http://www.w3.org/1998/Math/MathML">` element is present.
  * When embedding within XHTML contexts (most common case in QTI body elements), it wraps the *entire* MathML string in `<![CDATA[...]]>`.
* **Rendering Quirks:** Be aware that rendering relies on the LMS's engine (typically MathJax for Blackboard Ultra). Test complex MathML structures in BB Ultra. Stick to common Presentation MathML constructs for best compatibility.

**10. Platform Compatibility & Interoperability**

* **Primary Goal:** Generate standard-compliant IMS QTI 2.1 / IMS CP 1.2 packages.
* **Primary Test Target:** Blackboard Ultra. Use successful import and function in BB Ultra as the key acceptance criterion.
* **Secondary Testing:** Test generated packages in other QTI 2.1 compliant LMSs (e.g., Moodle, Canvas) periodically to gauge broader interoperability.
* **Blackboard Ultra Specific Considerations & Testing Strategy:**
  * **Known Potential Pain Points:** Be aware of potential limitations or quirks in BB Ultra's QTI 2.1 implementation, such as:
    * *Response Processing:* May have limited support for complex custom logic; standard templates (`MATCH_CORRECT`, `MAP_RESPONSE`) are most reliable.
    * *Interaction Types:* Complex configurations of less common types (matching, ordering, gap-fill) may require careful testing. Prioritize common types.
    * *Metadata Mapping:* Only a subset of IMS LOM fields might be displayed or used. Identify and test mapping for key fields.
    * *Feedback Display:* Test rendering and triggering of different feedback types (modal, inline).
    * *MathML Rendering:* Relies on BB's MathJax; test complex equations.
    * *Asset Path Handling:* Ensure relative paths work consistently.
  * **Testing Strategy:**
        1. **Prioritize Core Features:** Focus initial development on well-supported interaction types (`choiceInteraction`, `textEntryInteraction`, `extendedTextInteraction`), standard response processing, basic metadata, and MathML/image embedding.
        2. **Targeted Testing:** Create specific test cases for features suspected or known to be problematic in BB Ultra (e.g., complex matching, specific LOM fields, different feedback scopes, nested assets).
        3. **Documentation:** Maintain internal documentation detailing confirmed BB Ultra behaviors, successful QTI patterns, identified limitations, and unsupported features based *on testing results*.
        4. **Avoid Non-Standard Workarounds:** Resist implementing BB Ultra-specific XML structures that violate the QTI standard. If a standard approach fails *only* in BB Ultra, document the issue thoroughly. Seek standards-compliant alternatives first.

**11. Future Considerations (QTI 2.2 / 3.0)**

* **Namespace Configuration:** Make QTI XML namespace declarations easily configurable (e.g., via settings).
* **Modular Design:** The proposed template/code modularity and Pydantic abstraction are key to isolating changes needed for future QTI versions.
* **Data Model Abstraction:** Focusing Pydantic models on pedagogical intent helps buffer against underlying XML schema changes.
* **Adaptation Planning:** Recognize that QTI 3.0 (HTML5, PCI) represents a major shift requiring significant dedicated effort for migration/support. The core backend architecture (Python, FastAPI, DB, Jinja2, validation workflow) remains fundamentally applicable.

**12. Development Workflow & Process**

Follow an iterative development process, starting with an MVP and expanding incrementally.

```mermaid
graph TD
    subgraph User Interaction (Frontend/API)
        A[/"Input Data (UI Form / File Upload: JSON/CSV/YAML)"/] --> B{Validate Input Format/Schema};
        B -- Valid --> C[Send Validated Data Structure to Backend API];
        B -- Invalid --> A_Error[Show Input Format Error];
        API_Response -- Success --> G_Success[Show Success / Provide Download Link for ZIP];
        API_Response -- Failure --> G_Error[Show Detailed Generation/Validation Error];
    end

    subgraph Backend Processing (Python/FastAPI + Optional Celery)
        C --> D{Parse & Validate Input Data (Pydantic Models)};
        D -- Valid --> E[Store/Update Validated Data & Asset Metadata in DB (PostgreSQL)];
        D -- Invalid --> API_Response_Error1[Return Pydantic Validation Error];
        E --> F{Trigger QTI Generation Task (Sync/Async)};
        F --> H[Load Data, Config, Assets from DB/Storage];
        H --> I{Generate Presentation MathML (if needed, e.g., Pandoc)};
        I --> J[Render assessmentItem XMLs (Jinja2 -> QTI 2.1 Spec)];
        J --> K{Validate Item XMLs (lxml + Cached QTI 2.1 XSD)};
        K -- Valid --> L[Render assessmentTest XML (Optional, Jinja2 -> QTI 2.1 Spec)];
        K -- Invalid --> Error_ItemXSD[Log Error, Prepare Item XSD Error Response];
        L --> M{Validate Test XML (lxml + Cached QTI 2.1 XSD)};
        M -- Valid --> N[Render imsmanifest.xml (Jinja2 -> IMS CP 1.2 Spec)];
        M -- Invalid --> Error_TestXSD[Log Error, Prepare Test XSD Error Response];
        N --> O{Validate Manifest XML (lxml + Cached IMS CP 1.2 XSD)};
        O -- Valid --> P[Create IMS CP 1.2 ZIP Package (zipfile, including assets)];
        O -- Invalid --> Error_ManifestXSD[Log Error, Prepare Manifest XSD Error Response];
        P --> Q[Store Package Info/Link in DB];
        Q --> API_Response_Success[Return Success Response + Link];

        Error_ItemXSD --> API_Response_Error2[Return Generation/XSD Validation Error];
        Error_TestXSD --> API_Response_Error2;
        Error_ManifestXSD --> API_Response_Error2;

        API_Response_Success --> API_Response;
        API_Response_Error1 --> API_Response;
        API_Response_Error2 --> API_Response;

    end

    style K fill:#bbf,stroke:#00f,stroke-width:2px; label:"Validate Item XMLs (lxml + Cached QTI 2.1 XSD)"
    style M fill:#bbf,stroke:#00f,stroke-width:2px; label:"Validate Test XML (lxml + Cached QTI 2.1 XSD)"
    style O fill:#bbf,stroke:#00f,stroke-width:2px; label:"Validate Manifest XML (lxml + Cached IMS CP 1.2 XSD)"

```

**Testing Strategy:**

1. **Unit Tests:** Use `pytest`. Test Pydantic models (validation logic), Jinja2 filters/macros (render with mock data, assert output structure/content), validation logic wrappers, utility functions, individual modules. Aim for high coverage of core logic.
2. **Integration Tests:** Test FastAPI API endpoints (using `TestClient`), database interactions, the end-to-end generation process for simple cases (input JSON -> generated ZIP). Include automated XSD validation checks within these tests using sample valid and invalid inputs. Test asset handling logic.
3. **Manual/Acceptance Testing:** **Crucial for BB Ultra validation.**
    * Generate packages with diverse content: all supported interaction types, complex MathML, various asset types (images, potentially small audio/video if supported), different feedback configurations, edge cases (empty fields, special characters).
    * Manually import these packages into a **current Blackboard Ultra instance**.
    * Verify:
        * Successful import without errors.
        * Correct rendering of question text, prompts, choices, MathML, and assets in the student/instructor views.
        * Correct functionality: interaction behavior, scoring, feedback display based on responses.
        * Metadata visibility/usage in BB Ultra.
    * Document all findings rigorously.

**Sample Data Strategy:**

* Develop and maintain a comprehensive suite of test input data (e.g., as JSON files mirroring Pydantic models or CSV for simpler structures).
* Include examples covering:
  * Each supported interaction type.
  * Valid and complex MathML.
  * Items with and without assets.
  * Various feedback configurations.
  * Boundary conditions and potential edge cases (long text, special characters, different scoring values).
  * Known BB Ultra sensitivity points.
* Use this data for automated integration tests and as a basis for manual acceptance testing.

**13. Key Decisions Summary**

* **Primary Standard:** IMS QTI 2.1
* **Packaging Standard:** IMS Content Packaging v1.2
* **Backend Technology:** Python / FastAPI (Recommended)
* **Input Data Modeling:** Pydantic (focus on pedagogical intent)
* **Database:** PostgreSQL
* **Templating:** Jinja2 (Modular Structure, XML Autoescaping)
* **Validation:** Mandatory Input (Pydantic) + Output (Official XSDs via `lxml`) + Target Platform Testing (Blackboard Ultra)
* **MathML Handling:** Presentation MathML (Pandoc for conversion, CDATA embedding)
* **Identifiers:** UUIDs (Universally Unique Identifiers)
* **Static Asset Storage:** TBD (e.g., Local Filesystem, AWS S3) - Requires decision based on deployment environment and scale.
* **Primary Target LMS:** Blackboard Ultra

**14. Identified Risks and Mitigation Strategies Summary**

| Risk                                       | Mitigation Strategy                                                                                                                               |
| :----------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Invalid XML Generation**                 | Strict XSD validation (`lxml`), correct Jinja2 XML autoescaping, careful namespace management, unit/integration testing of templates.               |
| **Inconsistent Identifiers**             | Centralized UUID generation, passing IDs via context, template logic enforcing consistency between items/resources/manifest.                       |
| **MathML Issues**                          | Use Presentation MathML, Pandoc for reliable conversion, embed via Jinja2 filter (adding namespace + CDATA), extensive testing in BB Ultra/MathJax. |
| **Static Asset Mismanagement**           | Defined process: Pydantic model fields, chosen storage solution, upload mechanism, backend logic for packaging & manifest (relative paths).       |
| **Blackboard Ultra Import Failures**       | Prioritize standards, use BB Ultra as primary testbed, document quirks, favor known compatible QTI patterns, targeted testing, clear error analysis. |
| **Poor Error Feedback**                    | Catch errors at all stages, parse `lxml` validation logs, map errors back to input data/items, provide specific messages via API responses.       |
| **Template/Code Complexity**             | Strict modular design (templates, Python modules), Pydantic abstraction, clear separation of concerns, comprehensive testing.                          |
| **External Tool Dependency (Pandoc)**    | Ensure installation in deployment, robust error handling for tool execution (subprocess management, error checking).                                |
| **XHTML Validity in User Content**         | Configure RTE for clean output, server-side validation/sanitization (e.g., `lxml`, `bleach`) before storing/rendering.                              |
| **Configuration Management Issues**        | Use environment variables and/or config files; avoid hardcoding paths/credentials.                                                                |
| **Performance Bottlenecks (Large Scale)** | Implement XSD caching, consider asynchronous task queue (Celery) for generation/validation.                                                         |

**15. Next Steps**

1. Review and formally adopt this enhanced plan with the solutions team.
2. Define the specific set of QTI interaction types and LOM metadata fields for the Minimum Viable Product (MVP).
3. Finalize the detailed Pydantic data models for core structures, assets, and initial interaction types.
4. Make a decision on the Static Asset Storage mechanism.
5. Set up the initial project structure, development environment (including Python, FastAPI, DB, Pandoc), acquire official XSD schemas, and establish version control.
6. Begin implementation of the MVP core logic (data loading -> render -> validate -> package) focusing on 1-2 simple interaction types, MathML, and basic asset handling.
7. Establish the initial testing framework (pytest, sample data, BB Ultra test instance access).

---
