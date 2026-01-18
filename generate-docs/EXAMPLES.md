# Documentation Generation Examples

This file shows example invocations and the expected output.

---

## Basic usage

### Example 1: Default output directory

```
/generate-docs
```

The following files are generated under `docs/`:
- `docs/index.md`
- `docs/overview.md`
- `docs/api-reference.md`
- `docs/setup-guide.md`

### Example 2: Custom output directory

```
/generate-docs documentation/
```

Files are generated under `documentation/`.

---

## Example analysis process

### Example: Python FastAPI project

#### Step 1: Understand the structure

```
Inspecting project structure...

Files:
├── app/
│   ├── main.py          <- entry point detected
│   ├── config.py        <- config detected
│   ├── models.py        <- ORM models detected
│   ├── db.py            <- DB connection detected
│   └── ...
├── docker-compose.yml   <- Docker config detected
├── requirements.txt     <- dependencies detected
└── README.md            <- existing docs detected
```

#### Step 2: Extract key elements

```
Detected elements:

Framework: FastAPI
Database: PostgreSQL (SQLAlchemy)
Auth: session-based
External integrations:
  - Dify (LLM)
  - Pleasanter (business system)

Endpoints: 15
Models: 4
Environment variables: 20
```

#### Step 3: Generate documentation

```
Generating...

✓ docs/index.md (2.1 KB)
✓ docs/overview.md (15.3 KB)
✓ docs/api-reference.md (8.7 KB)
✓ docs/setup-guide.md (6.2 KB)

Done: 4 files generated (32.3 KB)
```

---

## Output samples

### `overview.md` opening section

```markdown
# Pleasanter-Dify Connector - Technical Overview

## 1. Project Overview

### 1.1 Purpose
This project integrates **Pleasanter** (a business management system) and **Dify** (an LLM platform)
to help automate email summarization and support document/report creation in a web application.

### 1.2 Key Features
1. **Fetch email**: Retrieve case-linked emails from Pleasanter
2. **Clean**: Remove unnecessary parts from the email body
3. **AI summarize**: Run LLM summarization via Dify
4. **Review/edit**: Let the user review and edit the summary result
5. **Save**: Persist the edited result back to Pleasanter
```

### `api-reference.md` endpoint example

```markdown
### Email summarization

```
POST /api/summarize_email
Content-Type: application/json
```

**Description**: Submit the email body and run AI summarization.

**Parameters**:
| Name | Type | Required | Description |
|------|------|----------|-------------|
| email_text | string | required | email body to summarize |
| conversation_id | string | - | existing conversation ID (empty = create new) |

**Request example**:
```json
{
  "email_text": "Hello team — following up on the item from last week...",
  "conversation_id": ""
}
```

**Response example**:
```json
{
  "conversation_id": "abc123",
  "answer": "{\"llm_comment\":\"Summarized\",...}",
  "parsed": {
    "llm_comment": "Summarized",
    "DescriptionA": "Summary of the support request"
  }
}
```
```

---

## Detection patterns by project type

### Web applications

Detection keywords:
- `FastAPI`, `Flask`, `Django`, `Express`, `NestJS`
- `routes/`, `controllers/`, `views/`
- `templates/`, `static/`

Generated diagrams:
- client/server architecture
- request handling sequence

### CLI tools

Detection keywords:
- `argparse`, `click`, `typer`, `commander`
- `bin/`, `cli/`, `commands/`

Generated diagrams:
- command execution flow
- subcommand structure

### Libraries / SDKs

Detection keywords:
- `setup.py`, `pyproject.toml` with `[project]`
- `src/`, `lib/`
- `__init__.py` with exports

Generated diagrams:
- module dependency graph
- class hierarchy

---

## Error handling

### When generation fails

```
Error: Entry point not found

Checks:
 1. Check for an entry point like `main.py`, `app.py`, or `index.js`
 2. Check for a dependency manifest like `package.json` or `requirements.txt`

Fix:
- Run from the project root directory
- Manually specify an entry point: /generate-docs --entry src/main.py
```

### Partial generation

```
Warning: Some information could not be extracted

- API endpoints: extracted (15)
- Data model: extracted (4)
- Environment variables: partial (not documented in README.md)

Continue? You can add missing environment variables manually later.
```
