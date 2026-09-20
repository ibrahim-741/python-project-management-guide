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
- [Advanced uv Features (Often Missed)](#-advanced-uv-features-often-missed)
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

## 🧩 Advanced uv Features (Often Missed)

<details>
<summary><strong>🐍 uv python — Manage Python versions</strong></summary>

```bash
uv python install 3.12
uv python list
uv python pin 3.12
```

Replaces `pyenv` for many use cases.

</details>

<details>
<summary><strong>🛠️ uv tool / uvx — Run CLI tools in isolation</strong></summary>

```bash
uv tool run ruff check .
uv tool install ruff
uvx ruff check .          # shortcut for `uv tool run`
uvx black .
```

Like `npx` for Python.

</details>

<details>
<summary><strong>🌳 uv tree — Inspect dependency graph</strong></summary>

```bash
uv tree
```

Equivalent to `pipdeptree` but built-in.

</details>

<details>
<summary><strong>🧪 Dev dependencies (--dev)</strong></summary>

```bash
uv add --dev pytest
uv add --dev ruff
```

Recorded separately in `pyproject.toml`. Not installed in production.

</details>

<details>
<summary><strong>🏗️ CI/CD flags (--frozen, --no-dev)</strong></summary>

```bash
uv sync --frozen      # don't update lockfile
uv sync --no-dev      # skip dev dependencies
```

Useful for reproducible builds.

</details>

<details>
<summary><strong>📦 uv run --with — Temporary dependency</strong></summary>

```bash
uv run --with requests python script.py
```

Run a command with a one-off dependency.

</details>

<details>
<summary><strong>🧹 uv remove also cleans transitive dependencies</strong></summary>

- `pip uninstall requests` removes only `requests`.
- `uv remove requests` also removes now-unused transitive dependencies.

Keeps your environment clean automatically.

</details>

<details>
<summary><strong>📌 Don't run uv init inside an existing repo</strong></summary>

Running `uv init` inside a repo that already has `pyproject.toml` will create a **second project definition**, confusing the tooling.

Always inspect the repo first.

</details>

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

<details>
<summary><strong>🐍 Traditional pip Cheat Sheet</strong></summary>

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

</details>

<details>
<summary><strong>🚀 uv Cheat Sheet</strong></summary>

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

</details>

<details>
<summary><strong>📥 After Cloning a Repo Cheat Sheet</strong></summary>

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

</details>

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