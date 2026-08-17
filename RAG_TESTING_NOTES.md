# Promptfoo RAG Testing Notes

## 1. How Knowledge Grounding Testing Works

In promptfoo, tests like `01-knowledge-grounding.yaml` use a **hardcoded context**. This is designed to test the **LLM layer** in isolation. 

**The 3 Layers of RAG Testing:**
1. **Layer 1: LLM Grounding (Unit Test)**
   - **How:** Hardcoded context in YAML.
   - **What it tests:** "If I give the LLM perfect context, does it still hallucinate?"
   - **Assertions:** `context-faithfulness`, `context-relevance`, `icontains`.
2. **Layer 2: Retrieval Quality (Integration Test)**
   - **How:** Script-based vars (a Python script calls your retriever and feeds dynamic context into promptfoo).
   - **What it tests:** "Does my vector DB return the right docs for this query?"
   - **Assertions:** `context-recall`, `context-relevance`.
3. **Layer 3: Full End-to-End**
   - **How:** Custom provider (your entire RAG app IS the promptfoo provider).
   - **What it tests:** "Does my full RAG app give correct answers?"
   - **Assertions:** `llm-rubric`, `factuality`.

**How context-* Assertions Work Internally:**
- **`icontains`**: Simple string search (No LLM needed).
- **`context-faithfulness`**: Calls a **grader LLM** to check: "Does every claim in the answer have support in the context?" (Returns 0.0-1.0 score).
- **`context-relevance`**: Calls a **grader LLM** to check: "Is the context relevant to answering this query?"
- **`context-recall`**: Calls a **grader LLM** to check against a reference answer: "Does the context contain enough info to produce this reference answer?"

---

## 2. Loading External Documents as Context

You don't have to hardcode context in your YAML files. You can load context from real files (`.txt`, `.pdf`, `.xml`, `.json`) using 3 main methods.

### Method 1: `file://` Reference (Simplest — for text files)
Promptfoo lets you point any var to a file using `file://`:
```yaml
vars:
  context: file://documents/company-policy.txt     # ← loads the file content
  query: "What is the refund policy?"
```
*Works with:* `.txt`, `.md`, `.csv`, `.xml`, `.html`, `.json`.

### Method 2: JSON File for Bulk Test Cases
Instead of writing each test in YAML, load them from a JSON file:
```yaml
tests: file://test-cases.json
```
```json
[
  {
    "description": "Refund policy from company doc",
    "vars": {
      "context": "file://documents/company-policy.txt",
      "query": "What is the refund policy?"
    },
    "assert": [
      { "type": "context-faithfulness", "threshold": 0.7 }
    ]
  }
]
```

### Method 3: Python Script (For PDFs and complex files)
PDFs are binary — promptfoo can't read them directly. Write a small Python script that extracts text and returns test cases:
```yaml
tests: file://load_pdf_tests.py
```
```python
# load_pdf_tests.py
import PyPDF2

def read_pdf(path):
    with open(path, 'rb') as f:
        reader = PyPDF2.PdfReader(f)
        text = "".join(page.extract_text() for page in reader.pages)
    return text

def get_test_cases():
    return [
        {
            "description": "HR policy from PDF",
            "vars": {
                "context": read_pdf("documents/hr-policy.pdf"),
                "query": "How many sick leaves do employees get?"
            },
            "assert": [
                {"type": "context-faithfulness", "threshold": 0.7}
            ]
        }
    ]
```
