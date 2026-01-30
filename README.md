# AI-Assisted-Metadata-Review-Tool
An AI-assisted content and metadata QA workflow that flags quality issues and produces review-ready fix suggestions at scale. An evolution of my last Metadata Review Tool.
## Overview

This project builds on my last Metadata Review Tool. This tool is a **human-in-the-loop metadata review tool** that audits records from the Internet Archive, flags the common metadata issues, and produces **AI-assisted fix suggestions** with confidence scores and explicit human-review gating.

It is designed as a **review assistant** and not an automated overwrite system.

The demo notebook targets the Internet Archive's _magazine_rack_ collection (the *Pulp Magazine Archive*, but the workflow is collection-agnostic.

---

## Quick Start

1. Open `ia_metadata_tool_with_ai.ipynb` in [Google Colab](https://colab.research.google.com)
2. Configure Cell 4:
    - Set `COLLECTION` to your Internet Archive collection name
    - Set `MAX_ITEMS` (recommend 50-200 for testing)
    - Set `PROVIDER` to your LLM provider (see [LLM Configuration](#llm-configuration))
3. Add your API key in Cell 5 (see [Setup Instructions](#setup--installation) for secure options)
4. Run all cells sequentially
5. Download the CSV outputs from the Colab file browser:
    - `ia_pulp_snapshot.csv` - raw metadata
    - `ia_pulp_issues_report.csv` - detected issues
    - `ia_pulp_ai_field_fixes_report.csv` - AI suggestions
    - `ia_pulp_proposed_cleaned.csv` - preview of fixes (optional)

---

## Setup & Installation

### Prerequisites

- Python 3.7+
- Google Colab (recommended) or local Jupyter environment
- API key from your chosen LLM provider (see [LLM Configuration](#llm-configuration))

### Installation

**Option 1: Google Colab (Recommended)**

1. Upload the notebook to Google Colab or open it directly from GitHub
2. Dependencies are installed automatically when you run Cell 2
3. Configure your API key using Colab Secrets (secure), or configure directly into Cell 5 (not recommended for shared notebooks)

**Option 2: Local Jupyter**

1. Install dependencies:
    
    ```bash
    pip install pandas requests tqdm google-genai openai anthropic
    ```
    
2. Launch Jupyter:
    
    ```bash
    jupyter notebook ia_metadata_tool_with_ai.ipynb
    ```
    
3. Configure your API key in Cell 5

### API Key Configuration

**Security Warning: Never commit API keys to version control**

**Secure Method (Google Colab):**

1. In Colab, click the key icon in the left sidebar
2. Add a secret named `GEMINI_API_KEY` (or equivalent for your provider)
3. Uncomment the secure option in Cell 5:
    
    ```python
    from google.colab import userdataGEMINI_API_KEY = userdata.get('GEMINI_API_KEY')
    ```
    

**Direct Method (Testing Only):**

- Replace the placeholder in Cell 5 with your actual API key
- **Do not commit or share the notebook after adding your key**

**Before sharing this notebook:**

- Remove any hardcoded API keys from Cell 5
- Use environment variables or secrets management instead
- Consider adding `*.ipynb` with embedded keys to `.gitignore`

---

## LLM Configuration

The tool supports three LLM providers, configured via the `PROVIDER` variable in Cell 4:

|Provider|Config Value|API Key Required|Model Used|Get API Key|
|---|---|---|---|---|
|**Google Gemini**|`"gemini"`|`GEMINI_API_KEY`|gemini-2.5-flash|[ai.google.dev](https://ai.google.dev/)|
|OpenAI|`"openai"`|`OPENAI_API_KEY`|gpt-4.1-mini|[platform.openai.com](https://platform.openai.com/)|
|Anthropic|`"anthropic"`|`ANTHROPIC_API_KEY`|claude-3-5-sonnet-latest|[console.anthropic.com](https://console.anthropic.com/)|

**Default:** Gemini (free tier available)

To switch providers, change the `PROVIDER` value in Cell 4 and ensure the corresponding API key is configured in Cell 5.

---

## What This Tool Does

### 1. Fetches metadata at scale

- Pulls item identifiers from a specified Internet Archive collection
- Retrieves detailed per-item metadata via the Internet Archive metadata API
- Saves a reproducible snapshot (`CSV`) for audit and comparison

### 2. Validates metadata and generates an issues report

The validation phase flags common, high-impact quality problems:

**HIGH Severity:**

- Missing required fields (identifier, title)
- Date values from the future
- Critical formatting errors

**MEDIUM Severity:**

- Non-standard date formats (e.g., "19040217" instead of "1904-02-17")
- Date values before 1800 or after 2100
- Leading/trailing whitespace in text fields

**LOW Severity:**

- Fields containing serialized lists instead of normalized values (e.g., "[item1, item2]")
- Minor formatting inconsistencies

Each issue is recorded with:

- Severity level (HIGH / MEDIUM / LOW)
- Field name
- Current value
- Notes explaining why it was flagged

### 3. Produces AI-assisted fix suggestions tied to real issues

The second pass uses a large language model to propose **field-level fixes**, but only in response to issues already identified by validation.

For each suggested fix, the tool records:

- The specific issue being addressed
- Current value → suggested value
- Reasoning
- Confidence score (0.0 to 1.0)
- Whether human review is required

Simple fixes (such as whitespace normalization) are handled deterministically without AI when possible.

### 4. Preserves human oversight

- AI suggestions are **never auto-applied by default**
- High-confidence, low-risk fixes can optionally be applied to create a _proposed_ cleaned snapshot
- All outputs are designed for spreadsheet-based or manual review

---

## What This Tool Does **Not** Do

- It does **not** automatically overwrite source metadata
- It does **not** attempt authoritative cataloging
- It does **not** claim AI-generated metadata is correct without review
- It does **not** write changes back to the Internet Archive

This tool is intentionally assistive, not autonomous. All final decisions on the metadata have to be made by a human.

---

## Outputs

The complete demo run produces the following artifacts:

|File|Description|
|---|---|
|`ia_pulp_snapshot.csv`|Raw metadata snapshot pulled from the Internet Archive|
|`ia_pulp_issues_report.csv`|Validation report listing detected metadata issues|
|`ia_pulp_ai_field_fixes_report.csv`|AI-assisted fix suggestions with confidence and review flags|
|`ia_pulp_proposed_cleaned.csv` _(optional)_|Proposed cleaned metadata with only safe fixes applied|

**Note:** Output filenames use the `ia_pulp_` prefix by default. You can customize these in Cell 4 (`SNAPSHOT_CSV`, `ISSUES_CSV`, etc.) to match your collection name.

---

## Performance Notes

The approximate runtime for 200 items:

- **Metadata fetch:** ~60-90 seconds (with 0.2s delay between items)
- **Validation:** < 1 second
- **AI fix suggestions:** ~2-5 minutes (depends on number of issues and LLM provider)

**Tip:** Start with `MAX_ITEMS = 50` to verify configuration before scaling up.

---

## Ethical & Design Principles

This project follows a **responsible AI** approach:

- **It's human-in-the-loop by design**  
    All AI suggestions are reviewable and traceable.
- **There's no silent automation**  
    Nothing is overwritten without explicit approval.
- **Transparency over authority is present**  
    The system explains _why_ something is flagged or suggested.
- **The source is respected**  
    Original metadata is preserved and attributed to the Internet Archive.

---

## Use Cases

This prototype is intended to demonstrate workflows applicable to:

- Metadata QA and cleanup
- Content operations and auditing
- Data quality review pipelines
- AI-assisted review tools with human oversight
- Large-scale catalog or content migrations

---

## Status & Limitations

This is a **demonstration prototype** suitable for:

- Portfolio presentation
- Metadata quality research
- Workflow experimentation

**It's not production-ready for:**

- Large-scale processing (> 1,000 items without modification)
- Automated pipelines
- Mission-critical metadata systems

**The known limitations:**

- Rate limiting is basic (0.2s between calls)
- No resume capability for interrupted runs
- API errors halt the entire process
- Output filenames are hardcoded to `ia_pulp_` prefix

---

## Data Source

Metadata is retrieved from the **Internet Archive** using publicly available APIs.  
This project is not affiliated with or endorsed by the Internet Archive.

---

## Troubleshooting

**"Missing GEMINI_API_KEY" error:**

- Ensure you've set your API key in Cell 5
- If using Colab Secrets, verify the secret name matches exactly

**Empty or incomplete results:**

- Check that `COLLECTION` in Cell 4 is a valid Internet Archive collection identifier
- Verify your Internet Archive collection is publicly accessible
- Try reducing `MAX_ITEMS` to test with a smaller sample

**Rate limit errors:**

- Increase `SLEEP_BETWEEN_CALLS` in Cell 4 (e.g., from 0.2 to 0.5)
- For LLM rate limits, consider switching providers or waiting before retrying

**API connection timeouts:**

- Check your internet connection
- Verify the Internet Archive is accessible from your location
- Try running the notebook again (temporary service issues)

---

## License & Attribution

This project uses an MIT license and is intended for educational and portfolio purposes. Additional information about the license is in the LICENSE file.

Metadata accessed through this tool is sourced from the Internet Archive and subject to their terms of use and any applicable licenses for individual items.
