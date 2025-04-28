# Blackboard Ultra MathML Quiz Tool

**Version:** 0.1.0 (Initial Documentation)
**Date:** 2025-04-27

---

## 1. Problem Statement

Creating assessments with mathematical notation (e.g., using LaTeX: $\int_0^\infty f(x) \, dx$) for Blackboard Ultra can be inefficient. The standard "Upload questions from file" feature accepts tab-delimited `.txt` files but lacks native support for automatically rendering embedded LaTeX code. This forces instructors to manually edit each question containing math after upload using the platform's Math Editor (WIRIS/MathType), which is time-consuming and error-prone for large question sets. Alternative methods like Question Bank/Pool imports introduce additional complexities and fidelity issues.

---

## 2. Discovery: Direct MathML Embedding Works

Through systematic testing (April 2025), a critical workaround was discovered:

**Embedding standard Presentation MathML (`<math xmlns="http://www.w3.org/1998/Math/MathML">...</math>`) directly inside the question/answer fields of the `.txt` file causes automatic rendering of mathematical notation upon upload.**

Key conditions:

- MathML must be correctly formatted and standards-compliant.
- The MathML must be provided as a clean, single-line string within the tab-delimited field.

This enables efficient offline preparation of upload-ready `.txt` files with embedded MathML.

---

## 3. Goal

This project aims to:

- [x] Document the discovered MathML embedding method.
- [x] Define a workflow for converting LaTeX-authored quizzes into Blackboard Ultra uploadable `.txt` files.
- [ ] Develop automation scripts to streamline the conversion process.

---

## 4. Proposed Workflow

### Step 1: Authoring (`src/` directory)

- Organize source files by course/module structure:
  `src/COURSE/MODULE/quiz_name.txt`
- Write questions using the Blackboard Ultra tab-delimited format.
  [Official Format Guide →](https://help.blackboard.com/Learn/Instructor/Ultra/Tests_Pools_Surveys/Reuse_Questions/Upload_Questions) *(verify link validity)*
- Embed LaTeX math using `<latex> ... </latex>` tags on a **single line**.

Example source line (Multiple Choice):

```

MC What is <latex>\lim_{x \to 0} \frac{\sin x}{x}</latex>? 1 correct 0 incorrect <latex>\infty</latex> incorrect undefined incorrect

```

- Commit source files to Git.

---

### Step 2: Conversion (`scripts/convert_to_mathml.py`)

- **Recursively** scan `src/` directory for `.txt` files.
- For each file:
  - Create a mirrored structure in `output/`.
  - Read line by line.
  - Find all `<latex>...</latex>` blocks using regex.
  - For each LaTeX snippet:
    - Trim whitespace.
    - Convert to Presentation MathML using Pandoc:

      ```bash
      echo 'Your LaTeX Snippet' | pandoc -f latex -t mathml | tr -d '\n'
      ```

    - Replace `<latex>...</latex>` block with the generated MathML.
    - Handle errors gracefully (log failures, skip problematic entries).
  - Write processed lines to output file.

---

### Step 3: Verification (`output/` directory)

- Inspect a few `.txt` files manually to verify:
  - MathML embedding integrity.
  - Tab structure preservation.

---

### Step 4: Upload to Blackboard Ultra

- Navigate to your course > Test/Pool > Upload Questions from File.
- Select a `.txt` file from `output/`.
- Verify automatic rendering of math expressions.

---

## 5. Project Structure

```

your-repo-name/
├── .git/
├── .gitignore          # Ignore /output/
├── README.md            # This documentation
├── scripts/
│   └── convert_to_mathml.py
├── src/
│   └── COURSE_ID/
│       └── MODULE_ID/
│           └── quiz_name.txt
└── output/
    └── COURSE_ID/
        └── MODULE_ID/
            └── quiz_name.txt

```

---

## 6. Tool Requirements

- **Pandoc**: For LaTeX → MathML conversion
  [Official site →](https://pandoc.org/)
- **Git**: For version control.
- **Python 3.x** (or shell scripting capabilities):
  - `os`
  - `re`
  - `subprocess`
  - `pathlib`

---

## 7. Important Notes

- **Undocumented Behavior**: Direct MathML rendering from `.txt` uploads in Blackboard Ultra is undocumented as of April 2025. Future Blackboard updates may change or remove this functionality.
- **LaTeX Compatibility**:
  - Pandoc's LaTeX parsing may differ slightly from WIRIS/MathType parsing.
  - Avoid complex/nonstandard LaTeX constructs.
  - Prioritize clean, basic LaTeX math commands (fractions, exponents, limits, integrals).

---

## 8. Future Development Roadmap

- [x] Document discovery and manual workflow.
- [ ] Implement `convert_to_mathml.py`.
- [ ] Add logging and robust error handling.
- [ ] Add CLI arguments (input/output dir, overwrite).
- [ ] Create example quizzes in `examples/`.
- [ ] Optimize Pandoc command for ideal MathML output.
