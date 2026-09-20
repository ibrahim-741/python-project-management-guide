# 🐍 Python Project Management: pip vs uv

![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![uv](https://img.shields.io/badge/uv-latest-orange)
![pip](https://img.shields.io/badge/pip-latest-green)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

> A practical, no-nonsense guide to Python virtual environments, pip, uv, dependencies,
> `pyproject.toml`, `uv.lock`, `requirements.txt`, and what to do after cloning a repo.

---

## 📖 Table of Contents

- [The Core Distinction](#-the-core-distinction)
- [Traditional pip Workflow](#-traditional-pip-workflow)
- [Modern uv Workflow](#-modern-uv-workflow)
- [Direct Comparison](#-direct-comparison)
- [Key Files Explained](#-key-files-explained)
- [Why uv is Fast](#-why-uv-is-fast)
- [uv pip vs uv add](#-uv-pip-vs-uv-add)
- [Advanced uv Features](#-advanced-uv-features)
  - [🐍 uv python — Manage Python Versions](#-uv-python--manage-python-versions)
  - [🛠️ uv tool / uvx — Run CLI Tools in Isolation](#️-uv-tool--uvx--run-cli-tools-in-isolation)
  - [🌳 uv tree — Inspect Dependency Graph](#-uv-tree--inspect-dependency-graph)
  - [🧪 Dev Dependencies (--dev)](#-dev-dependencies---dev)
  - [🏗️ CI/CD Flags (--frozen, --no-dev)](#️-cicd-flags---frozen---no-dev)
  - [📦 uv run --with — Temporary Dependency](#-uv-run---with--temporary-dependency)
  - [🧹 uv remove Also Cleans Transitive Dependencies](#-uv-remove-also-cleans-transitive-dependencies)
  - [📌 Don't Run uv init Inside an Existing Repo](#-dont-run-uv-init-inside-an-existing-repo)
- [Cloning a Repository: What to Do](#-cloning-a-repository-what-to-do)
- [Cheat Sheets](#-cheat-sheets)
- [Golden Rules](#-golden-rules)
- [Resources](#-resources)

---

## 🧠 The Core Distinction

| Tool | What it is |
|------|------------|
| **pip** | Primarily Python's package installer. Traditionally used with `venv` and `requirements.txt`. |
| **uv** | A faster, broader Python project manager that handles dependencies, virtual environments, lockfiles, and command execution in one workflow. |

**One-liner:**  
`pip` installs packages. `uv` manages your entire project.

---

## 🏛️ Traditional pip Workflow

The classic stack:

```
Python
  │
  ├── venv              → create virtual environment
  │
  ├── pip               → install packages
  │
  └── requirements.txt  → record dependencies
```

### Step-by-step

```bash
# 1. Create project folder
mkdir my-project
cd my-project

# 2. Create virtual environment
python3 -m venv .venv

# 3. Activate it
source .venv/bin/activate        # macOS/Linux
.venv\Scripts\activate           # Windows

# 4. Install packages
pip install fastapi
pip install uvicorn
pip install openai
pip install python-dotenv

# 5. Freeze dependencies
pip freeze > requirements.txt

# 6. Run your project
python main.py
uvicorn app.main:app --reload
```

**Project structure:**

```
my-project/
├── .venv/
├── requirements.txt
├── main.py
└── ...
```

---

## 🚀 Modern uv Workflow

The uv stack:

```
uv
 │
 ├── project initialization
 ├── virtual environment
 ├── package installation
 ├── dependency resolution
 ├── lockfile
 ├── Python versions
 └── running commands
```

### Step-by-step

```bash
# 1. Create project
uv init my-project
cd my-project

# 2. Add dependencies
uv add fastapi uvicorn openai python-dotenv

# 3. Run your project
uv run python main.py
uv run uvicorn app.main:app --reload
```

> 💡 **No need to manually activate `.venv`** — `uv run` handles it.

**Project structure:**

```
my-project/
├── .venv/
├── pyproject.toml
├── uv.lock
├── main.py
└── ...
```

---

## ⚖️ Direct Comparison

| Task | Traditional (pip) | uv |
|------|-------------------|-----|
| Create project | manually | `uv init` |
| Create venv | `python3 -m venv .venv` | `uv` manages it |
| Activate venv | `source .venv/bin/activate` | optional |
| Install package | `pip install requests` | `uv add requests` |
| Remove package | `pip uninstall requests` | `uv remove requests` |
| Record dependencies | `pip freeze > requirements.txt` | `uv.lock` + `pyproject.toml` |
| Recreate environment | `pip install -r requirements.txt` | `uv sync` |
| Run project | activate → `python` | `uv run python` |
| Run tools | activate → `uvicorn` | `uv run uvicorn` |

---

## 📄 Key Files Explained

### `requirements.txt` (Traditional)

A flat list of installed packages:

```txt
fastapi==0.120.0
openai==1.107.0
uvicorn==0.35.0
```

> "Here are the packages I installed."

### `pyproject.toml` (Modern)

Declares what your project **needs**:

```toml
[project]
name = "my-project"
version = "0.1.0"
dependencies = [
    "fastapi>=0.120",
    "uvicorn>=0.35",
    "openai>=1.0"
]
```

> "What does my project need?"

### `uv.lock` (Modern)

Records the **exact resolved dependency graph**:

> "What exact dependency resolution should be used?"

**Commit `uv.lock` for applications.**  
For libraries, you may omit it to let consumers resolve.

---

## ⚡ Why uv is Fast

1. **Written in Rust** — very fast dependency resolution.
2. **Global cache** — packages downloaded once, reused across all projects.
3. **Parallel installation** — installs multiple packages simultaneously.
4. **Smart resolver** — resolves the entire dependency graph at once.

Example of caching:

```
Project A → requests
Project B → requests
Project C → requests
(Downloaded only once, reused everywhere)
```

---

## 🔀 `uv pip` vs `uv add`

| Command | Meaning |
|---------|---------|
| `uv pip install requests` | Pip-style installation into the current environment. No project tracking. |
| `uv add requests` | Adds `requests` as a project dependency in `pyproject.toml` + `uv.lock`. |

> ✅ For a **uv project**, always prefer `uv add`.  
> ✅ For a **pip-style repo**, use `uv pip install -r requirements.txt`.

---

## 🧩 Advanced uv Features

These are the features most people miss when learning uv. Each one solves a real problem.

### 🐍 `uv python` — Manage Python Versions

uv can install and manage Python interpreters directly, replacing tools like `pyenv`.

**Commands:**

```bash
uv python install 3.12        # Install Python 3.12
uv python install 3.11 3.12   # Install multiple versions
uv python list                # View available and installed versions
uv python pin 3.12            # Pin project to 3.12 (.python-version)
```

**Example:** Your project needs Python 3.12, but your system default is 3.10. No manual Python installation needed:

```bash
uv python install 3.12
uv python pin 3.12
# All subsequent uv commands automatically use 3.12
uv run python --version
# Output: Python 3.12.x
```

---

### 🛠️ `uv tool` / `uvx` — Run CLI Tools in Isolation

`uvx` is an alias for `uv tool run`. It runs tools in temporary isolated environments — like `npx` for Python. Nothing gets installed into your project.

**Commands:**

```bash
uvx ruff check .              # Run without installing
uvx black .                   # Format code
uvx ruff@0.6.0 check .        # Run a specific version
uv tool install ruff          # Install persistently to PATH
uv tool upgrade ruff          # Upgrade an installed tool
uv tool list                  # List installed tools
```

**Example:** You want to lint your code with `ruff`, but you don't want it in your project dependencies:

```bash
uvx ruff check .
# After it finishes, the temporary environment is cleaned up.
# Your project's .venv has no ruff package.
```

---

### 🌳 `uv tree` — Inspect Dependency Graph

Visualize your project's dependency tree. Equivalent to `pipdeptree`, but built-in.

**Commands:**

```bash
uv tree                       # Full dependency tree
uv tree --depth 2             # Limit depth
uv tree --invert requests     # Reverse: who depends on requests?
```

**Example output:**

```
my-project v0.1.0
├── fastapi v0.120.0
│   ├── pydantic v2.10.0
│   └── starlette v0.41.0
├── uvicorn v0.35.0
│   └── click v8.1.0
└── openai v1.107.0
    └── httpx v0.28.0
```

---

### 🧪 Dev Dependencies (`--dev`)

Development dependencies like `pytest` and `ruff` are recorded separately and are not installed in production.

**Commands:**

```bash
uv add --dev pytest ruff      # Add dev dependencies
uv remove --dev pytest        # Remove dev dependency
uv sync --no-dev              # Skip dev deps (production)
```

**Example:** Add pytest for testing, but exclude it from production builds:

```bash
uv add --dev pytest
uv sync --no-dev
# pytest is NOT installed, but fastapi and uvicorn are.
```

Dev dependencies are stored in `pyproject.toml` under `[dependency-groups]`:

```toml
[dependency-groups]
dev = [
    "pytest>=8.0",
    "ruff>=0.6"
]
```

---

### 🏗️ CI/CD Flags (`--frozen`, `--no-dev`)

Use these flags in CI/CD pipelines to ensure reproducible, production-ready environments.

**Commands:**

```bash
uv sync --frozen              # Use uv.lock as-is, don't update it
uv sync --no-dev              # Skip dev dependencies
uv sync --frozen --no-dev     # Production-ready sync
```

**Example GitHub Actions workflow:**

```yaml
- name: Install uv
  run: curl -LsSf https://astral.sh/uv/install.sh | sh

- name: Sync dependencies
  run: uv sync --frozen --no-dev

- name: Run tests
  run: uv run pytest
```

**Behavior:**

| Flag | Effect |
|------|--------|
| `--frozen` | Errors if `uv.lock` needs updating. Guarantees the lockfile is respected exactly. |
| `--no-dev` | Skips dev dependencies. Smaller, faster installs for production. |

---

### 📦 `uv run --with` — Temporary Dependency

Run a script with a one-off dependency without modifying your project.

**Commands:**

```bash
uv run --with requests python script.py
uv run --with 'rich>12,<13' python report.py
uv run --with pandas --with numpy python analysis.py
```

**Example:** You have a quick script that needs `requests`, but you don't want to add it to your project:

```bash
uv run --with requests python fetch_data.py
# requests is installed in a temporary overlay, not in pyproject.toml
```

---

### 🧹 `uv remove` Also Cleans Transitive Dependencies

This is a key difference from `pip uninstall`.

**Comparison:**

```bash
pip uninstall requests        # Removes only requests
uv remove requests            # Also removes unused transitive dependencies
```

**Example:** Suppose `requests` depends on `urllib3` and `certifi`. If nothing else uses them:

```bash
uv remove requests
# uv also removes urllib3 and certifi if they are no longer needed.
```

This keeps your environment clean automatically. `pip` leaves orphaned packages behind.

---

### 📌 Don't Run `uv init` Inside an Existing Repo

If a repository already has `pyproject.toml`, running `uv init` will error or create a conflicting project definition.

**What happens:**

```bash
cd existing-repo/
uv init
# error: Project is already initialized in /path/to/existing-repo (pyproject.toml file exists)
```

**Correct approach:** Inspect the repo first, then choose the right command.

```bash
# 1. Check what files exist
ls

# 2. If it's a uv project (pyproject.toml + uv.lock)
uv sync

# 3. If it's a pip-style repo (requirements.txt)
uv pip install -r requirements.txt

# 4. If it has pyproject.toml but no uv.lock
cat README.md
cat pyproject.toml
# Follow the documented tool (Poetry, Hatch, PDM, etc.)
```

> ⚠️ **Never run `uv init` on a cloned repository.** Respect the existing workflow.

---

## 📥 Cloning a Repository: What to Do

### 🧭 Decision Tree

```
Clone repo
    │
    ▼
Check the files
    │
    ├── requirements.txt          → pip-style workflow
    │
    ├── pyproject.toml + uv.lock  → uv workflow
    │
    └── pyproject.toml only       → inspect README / tooling
```

### Case A: Repo uses pip-style (only `requirements.txt`)

```bash
git clone <repo-url>
cd my-project
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python main.py
```

**Or use uv for speed without changing the repo:**

```bash
git clone <repo-url>
cd my-project
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
uv run python main.py
```

### Case B: Repo uses uv (`pyproject.toml` + `uv.lock`)

```bash
git clone <repo-url>
cd my-project
uv sync
uv run python main.py
```

### Case C: Repo has `pyproject.toml` but no `uv.lock`

Inspect the README. It might use Poetry, Hatch, PDM, or another manager.

```bash
cat README.md
cat pyproject.toml
```

Follow the documented setup.

### ⚠️ Warning

**Do not run `uv init` inside an existing repository.**  
Respect the repo's existing workflow unless converting is part of your task.

---

## 📋 Cheat Sheets

### 🐍 Traditional pip Cheat Sheet

```bash
# Create environment
python3 -m venv .venv

# Activate (macOS/Linux)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Install packages
pip install fastapi
pip install openai

# Freeze dependencies
pip freeze > requirements.txt

# Another developer:
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 🚀 uv Cheat Sheet

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh
# or: pip install uv

# Create new project
uv init my-project
cd my-project

# Add dependencies
uv add fastapi openai

# Add dev dependency
uv add --dev pytest

# Remove dependency
uv remove openai

# Sync environment
uv sync

# Run commands
uv run python main.py
uv run uvicorn app.main:app --reload

# Run tools
uv run pytest

# Create venv manually
uv venv

# Pip-compatible interface
uv pip install -r requirements.txt
uv pip freeze > requirements.txt
```

### 📥 After Cloning a Repo Cheat Sheet

```bash
# 1. Clone
git clone <repo-url>
cd my-project

# 2. Inspect
ls
cat README.md

# 3a. If requirements.txt exists (pip-style):
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python main.py

# 3b. If requirements.txt exists (using uv for speed):
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
uv run python main.py

# 3c. If uv.lock exists (uv-style):
uv sync
uv run python main.py

# 3d. If pyproject.toml exists but no uv.lock:
cat README.md
cat pyproject.toml
# Follow documented tool (Poetry, Hatch, PDM, etc.)
```

---

## 🏆 Golden Rules

1. **Read the README first.**
2. **Inspect the project files** (`requirements.txt`, `pyproject.toml`, `uv.lock`).
3. **Identify the project's package manager.**
4. **Follow the repo's documented setup.**
5. **Always use an isolated environment.**
6. **Never commit `.venv/` or API keys.**
7. **Don't choose a package manager based on personal preference — match the repo.**

---

## 🔗 Resources

- [uv Official Documentation](https://docs.astral.sh/uv/)
- [pip Documentation](https://pip.pypa.io/)
- [Python venv Documentation](https://docs.python.org/3/library/venv.html)
- [pyproject.toml Specification](https://peps.python.org/pep-0621/)

---

> **Remember:** `pip` is not bad. `uv` is not a replacement for everything.  
> They coexist. Learn both, and respect the repository you're working with.

---

<p align="center">
  Made with ❤️ for Python developers
</p>