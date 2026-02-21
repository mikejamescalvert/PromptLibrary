# Environment Tool Check — Verify and Install Dependencies

## Purpose

Audit the current development environment for essential tools and libraries that Claude Code needs to be fully functional — Python, pip, PDF/Office/image processing, SQL database access, and GitHub CLI. Identify anything missing and install it so the session starts from a known-good baseline.

## Prompt

```
Audit my development environment and make sure you have everything you need to be
fully functional. Check for each category below, report what you find, and install
anything that is missing or broken.

1. **Python & pip**
   - Verify `python` (or `python3`) is on PATH and report the version.
   - Verify `pip` (or `pip3`) is available and report the version.
   - If either is missing, install or fix it before continuing.

2. **PDF reading**
   - Check for a Python library that can extract text from PDFs (e.g., `PyMuPDF`
     / `fitz`, `pdfplumber`, or `pymupdf4llm`).
   - Install one if none is found. Prefer `PyMuPDF` for speed and broad support.

3. **Word documents (.docx)**
   - Check for `python-docx`.
   - Install it if missing.

4. **Excel files (.xlsx / .xls)**
   - Check for `openpyxl` (xlsx) and optionally `xlrd` (legacy xls).
   - Install any that are missing.

5. **SQL databases**
   - Check for database driver libraries:
     - `pyodbc` or `pymssql` (MSSQL / SQL Server)
     - `psycopg2` or `psycopg2-binary` (PostgreSQL)
     - `sqlite3` (built-in — just confirm it imports)
   - Install any that are missing. Use the `-binary` variants where available to
     avoid native build issues.

6. **Image reading and analysis**
   - Check for `Pillow` (PIL).
   - Optionally check for `pytesseract` (OCR) if Tesseract is installed on the
     system.
   - Install `Pillow` if missing.

7. **GitHub CLI (`gh`)**
   - Verify `gh` is on PATH and report the version.
   - Check authentication status with `gh auth status`.
   - If `gh` is missing, let me know — do NOT attempt to install it automatically
     (it requires admin privileges).

8. **Summary report**
   After all checks, print a table like this:

   | Category          | Tool / Library     | Status  | Version / Notes     |
   | ----------------- | ------------------ | ------- | ------------------- |
   | Python            | python             | OK / MISSING | 3.x.x          |
   | Package manager   | pip                | OK / MISSING | 24.x            |
   | PDF               | PyMuPDF            | OK / INSTALLED | 1.x.x        |
   | Word              | python-docx        | OK / INSTALLED | 0.x.x        |
   | Excel             | openpyxl           | OK / INSTALLED | 3.x.x        |
   | SQL (MSSQL)       | pyodbc             | OK / INSTALLED | 5.x.x        |
   | SQL (PostgreSQL)  | psycopg2-binary    | OK / INSTALLED | 2.x.x        |
   | SQL (SQLite)      | sqlite3 (built-in) | OK / MISSING | —             |
   | Images            | Pillow             | OK / INSTALLED | 10.x.x       |
   | GitHub CLI        | gh                 | OK / MISSING | 2.x.x          |

   Flag anything that could not be installed and suggest manual steps to fix it.
```

## Parameters

None — this prompt runs against the current environment as-is.

## Expected Outcome

1. Every listed tool and library is checked and its status reported.
2. Missing Python libraries are installed automatically via `pip install`.
3. A summary table shows the final state of every dependency.
4. Any items that could not be resolved are called out with next steps.
