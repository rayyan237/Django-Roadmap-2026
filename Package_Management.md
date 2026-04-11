# Chapter 6: Package Management — Dependency Resolution & CI/CD

In Chapter 5, we built the fortress: an isolated Virtual Environment. In this chapter, we supply the fortress. 

Modern software is built on the shoulders of giants. You will rarely write your own cryptographic hashing functions or database drivers. You will import them. But managing external code introduces massive risks: version conflicts, security vulnerabilities, and deployment failures. This chapter covers how to safely acquire, lock, and manage external code for enterprise deployment.

---

## LEVEL 1: THE ABSOLUTE BASICS (Zero Prior Experience)

### 1.1 PyPI and `pip`
The **Python Package Index (PyPI)** is the official public repository for Python software. It hosts hundreds of thousands of open-source packages.

**`pip`** (Pip Installs Packages) is the command-line tool built into Python that connects to PyPI, downloads the package, and places it into your active Virtual Environment.

### 1.2 Basic Installation Commands
You must activate your virtual environment before running these commands.

```bash
# Install the absolute latest version of a package
pip install requests

# Upgrade an already installed package to the latest version
pip install --upgrade requests

# Uninstall a package
pip uninstall requests

# List everything currently installed in your environment
pip list
```

### 1.3 The Danger of Unpinned Versions
Running `pip install django` in an enterprise setting is a critical mistake. If you don't specify a version, `pip` grabs the newest one. If you run that command today, you get Django 4.0. If a teammate runs it next year, they get Django 5.0, and their server crashes because the code is fundamentally different.

You must always "pin" your versions.

```bash
# 1. Exact Pinning (Safest for production)
pip install django==4.2.0

# 2. Minimum Version Pinning (Needs at least this version to work)
pip install django>=4.0.0

# 3. Compatible Release Pinning (Allows minor bug fixes, blocks major breaking changes)
# This installs 4.2.0, allows 4.2.9, but blocks 4.3.0 or 5.0.0
pip install django~=4.2.0
```

---

## LEVEL 2: INTERMEDIATE MECHANICS (Trees & Automation)

When you install a package, you are rarely just installing one thing. You are installing a web of dependencies.

### 2.1 The Dependency Tree
If you run `pip install fastapi`, `pip` doesn't just download FastAPI. It looks at FastAPI's source code, realizes FastAPI needs a tool called `pydantic` to work, and automatically downloads `pydantic` too. 

* **Top-Level Dependencies:** The packages *you* explicitly installed (FastAPI).
* **Sub-Dependencies (Transitive):** The packages your packages installed (`pydantic`, `starlette`).

### 2.2 `pip freeze` vs. `requirements.txt`
To share your project, you must give other developers a list of your required packages. 

```bash
# pip freeze reads every single package inside your virtual environment 
# and dumps the exact version numbers into a text file.
pip freeze > requirements.txt
```

When a new developer downloads your code, they build an empty `.venv`, activate it, and run:
```bash
# The '-r' flag stands for 'read'. It installs everything from the file.
pip install -r requirements.txt
```

### 2.3 Development vs. Production Dependencies
You use tools like `pytest` (for testing) and `black` (for formatting) while coding. **You must never install these on a production server.** They bloat the server's memory and increase the surface area for hackers to attack.

Professionals split their requirements into two files:
1. `requirements.txt` (Strictly what the server needs to run: FastAPI, Postgres drivers).
2. `requirements-dev.txt` (Contains dev tools, and includes the main file).

*Inside `requirements-dev.txt`:*
```text
-r requirements.txt 
pytest==7.4.0
black==23.7.0
```

### 2.4 Editable Installs (Local Monorepos)
If you are building a massive application split into multiple folders (e.g., `core_api` and `payment_gateway`), you might need to install your own local folder as a package. 

If you just run `pip install ./core_api`, pip copies the files. If you change the code, you have to reinstall it.
Instead, use an **Editable Install** (`-e`):

```bash
# The period '.' means "the folder I am currently in"
pip install -e .
```
This creates a symlink (a shortcut). If you change the code in your Python file, the virtual environment instantly sees the change without needing a reinstall.

### 2.5 The Global Cache Trap
When `pip` downloads a package, it saves a hidden copy to your OS's global cache. The next time you create a `.venv` and install that package, `pip` skips the internet and uses the cached copy.

If your virtual environment is acting possessed, or throwing random C-compiler errors, your cache is likely corrupted.

```bash
# Purge the global pip cache to force fresh downloads from the internet
pip cache purge
```

---

## LEVEL 3: ADVANCED ENGINEERING (Enterprise Architecture)

In modern enterprise environments, raw `pip` is considered a legacy tool. It lacks strict mathematical dependency resolution and advanced security features.

### 3.1 Dependency Resolution & "Backtracking"
Raw `pip` often hangs for 10 minutes trying to install a single package. Why? 
**The Resolver Nightmare:**
1. Package A requires `urllib3 v1.26`.
2. Package B requires `urllib3 v2.0`.
3. `pip` downloads Package A and installs `v1.26`. 
4. `pip` looks at Package B, realizes `v2.0` is required, panics, uninstalls `v1.26`, installs `v2.0`, looks back at Package A, realizes it broke Package A, and starts "Backtracking" through hundreds of package versions trying to find a mathematical compromise.

This is why enterprises abandon raw `pip` for advanced tools.

### 3.2 The Modern Standard: `pyproject.toml`
Historically, Python projects used a chaotic mix of `setup.py`, `setup.cfg`, and `requirements.txt`. The modern, unified standard is the **`pyproject.toml`** file. 

It acts as the single source of truth for your dependencies, project metadata, and tool configurations.

*Example `pyproject.toml`:*
```toml
[project]
name = "skillswap-backend"
version = "1.0.0"
dependencies = [
    "fastapi>=0.100.0",
    "uvicorn~=0.23.0"
]
```

### 3.3 Deterministic Builds and Lock Files (`poetry`)
If you define `fastapi>=0.100.0`, different servers might install different versions depending on what day they deploy. That inconsistency causes random production crashes.

Enterprise tools like **Poetry** solve this using **Lock Files**.
When you use Poetry, you define your broad requirements in `pyproject.toml`. Poetry calculates the exact, mathematically perfect versions of every single sub-dependency required, and permanently writes those exact versions into a `poetry.lock` file.

* `pyproject.toml`: For humans to read (broad rules).
* `poetry.lock`: For servers to read (strict, immutable hashes and versions).

If you commit `poetry.lock` to GitHub, you guarantee that every developer and every production server will install the exact same bytecode, ensuring a **Deterministic Build**.

### 3.4 Ultra-Fast Tooling (`uv`)
Python packaging has historically been slow. In 2024, the industry experienced a massive shift toward **`uv`**, an incredibly fast Python package manager written in the Rust programming language.

It acts as a drop-in replacement for `pip`, but executes dependency resolution and package installations 10x to 100x faster. In a CI/CD pipeline where server time costs money, reducing dependency installation from 2 minutes to 2 seconds is a massive architectural upgrade.

```bash
# Using uv is exactly like pip, just blazingly fast.
uv pip install fastapi
```

### 3.5 Wheels (`.whl`) vs. Source Distributions (`sdist`)
When you download a package, what exactly are you downloading?
* **Source Distribution (`.tar.gz`):** The raw code. If the package contains C++ extensions (like `cryptography`), your computer must physically compile that C code during installation. If you lack a C++ compiler, it violently crashes.
* **Wheels (`.whl`):** A pre-compiled, ready-to-run binary package. The developer already compiled the C code for Windows, Mac, and Linux. 

**Enterprise Rule:** Always configure CI/CD pipelines to prefer Wheels. They install in milliseconds and never crash due to missing OS compilers.

### 3.6 Private Enterprise Package Indexes
Enterprise companies (like Banks or Healthcare) do not upload their proprietary Python code to the public PyPI internet. They host their own private PyPI servers (like AWS CodeArtifact or JFrog Artifactory).

To install internal company packages, you must pass secure authentication tokens and point `pip` (or `uv`) to the private URL.

```bash
# Pointing pip to an internal enterprise server instead of the public internet
pip install secret-company-auth-tool \
    --index-url [https://username:secure_token@artifactory.mycompany.com/pypi/simple](https://username:secure_token@artifactory.mycompany.com/pypi/simple)
```

### 3.7 Supply Chain Security (Hashes and Audits)
"Supply Chain Attacks" happen when a hacker uploads a malicious package to the public PyPI (e.g., naming it `requestts` instead of `requests`) to steal environment variables.

Enterprise pipelines use **Hash-Checking** to prevent this. When a lock file is generated, it calculates the SHA-256 cryptographic hash of the original package. 

```bash
# Example of strict hash-checking in an enterprise requirements file
requests==2.31.0 \
    --hash=sha256:942c5a758f98d790eaed1a29cb6eefc7ffb0d1ce0faa05...
```
When the production server downloads `requests`, it hashes the downloaded file. If it does not mathematically match the hash in the lock file, the server assumes the package was tampered with by a hacker and immediately aborts the deployment.
