# Index

- [1. TP2 Student Starter Pack](#1-tp2-student-starter-pack)
- [2. Project Setup with `uv`](#2-project-setup-with-uv)
	- [2.1 Install `uv`](#21-install-uv)
	- [2.2 Clone the repository](#22-clone-the-repository)
	- [2.3 Create the environment and install dependencies](#23-create-the-environment-and-install-dependencies)
	- [2.4 Activate the virtual environment](#24-activate-the-virtual-environment)
	- [2.5 Run the project](#25-run-the-project)
	- [2.6 Adding new dependencies](#26-adding-new-dependencies)
	- [2.7 Updating the local environment](#27-updating-the-local-environment)
	- [2.8 Files that should be committed](#28-files-that-should-be-committed)
	- [Minimal setup command summary](#minimal-setup-command-summary)

# 1. TP2 Student Starter Pack

Files:

- `TP2_StarterPack_Students.ipynb`: Colab/VS Code starter notebook.
- `tp2-chosen/`: copy of the TP2 target images.
- `tp2-chosen.zip`: optional zip with the same target images.
- `outputs/`: local output folder placeholder.

Recommended Google Drive layout for Colab / VS Code Colab extension:

```text
MyDrive/GENAI_TP2/tp2-chosen/*.png
```

or:

```text
MyDrive/GENAI_TP2/tp2-chosen.zip
```

The notebook mounts Google Drive, searches these paths, extracts the zip if needed, and saves generated outputs to:

```text
MyDrive/GENAI_TP2/outputs/
```

The LCM settings match the TP2 target generation setup:

- model: `SimianLuo/LCM_Dreamshaper_v7`
- seed: parsed from target filename
- inference steps: `8`
- guidance scale: `8.0`
- `lcm_origin_steps`: `50`
- resolution: `768x768`

If Colab raises an error such as:

```text
cannot import name '_Ink' from 'PIL._typing'
```

restart the runtime/kernel and rerun the notebook from the first cell. The install cell pins `Pillow<12` to avoid that Diffusers/Pillow compatibility issue.

The install cell also pins `pandas<3` to avoid dependency conflicts with packages commonly preinstalled in Colab, such as Gradio.

# 2. Project Setup with `uv`

This project uses [`uv`](https://docs.astral.sh/uv/) to manage Python, the virtual environment, and dependencies.

The environment is defined by:

```text
pyproject.toml
uv.lock
.python-version
```

The `.venv/` directory is intentionally not committed to Git. It will be recreated automatically by `uv`.

---

## 2.1 Install `uv`

If `uv` is not installed yet, install it with:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Then restart your terminal, or run:

```bash
source ~/.bashrc
```

Check that `uv` is available:

```bash
uv --version
```

---

## 2.2 Clone the repository

```bash
git clone <REPOSITORY_URL>
cd <PROJECT_FOLDER>
```

Replace `<REPOSITORY_URL>` with the Git URL of this project.

---

## 2.3 Create the environment and install dependencies

Run:

```bash
uv sync
```

This command will:

- read the required Python version from `.python-version`;
- create a local `.venv/` virtual environment if it does not exist;
- install the exact dependency versions from `uv.lock`.

---

## 2.4 Activate the virtual environment

On Linux/macOS:

```bash
source .venv/bin/activate
```

On Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Check that the environment is active:

```bash
python --version
which python
```

The Python executable should point to the local `.venv` folder.

---

## 2.5 Run the project

After activating the environment, run the project normally. For example:

```bash
python main.py
```

If the project uses a different entry point, replace `main.py` with the correct file or command.

You can also run commands directly through `uv` without manually activating the environment:

```bash
uv run python main.py
```

---

## 2.6 Adding new dependencies

To add a new dependency, use:

```bash
uv add package-name
```

For example:

```bash
uv add numpy
```

This updates both:

```text
pyproject.toml
uv.lock
```

After adding dependencies, commit the updated files:

```bash
git add pyproject.toml uv.lock
git commit -m "Update dependencies"
```

---

## 2.7 Updating the local environment

When pulling changes from Git, dependencies may have changed. After pulling, run:

```bash
git pull
uv sync
```

This updates the local `.venv/` to match the committed `uv.lock` file.

---

## 2.8 Files that should be committed

The following files should be committed:

```text
pyproject.toml
uv.lock
.python-version
README.md
source code files
```

The following files should not be committed:

```text
.venv/
__pycache__/
*.pyc
.ipynb_checkpoints/
.env
```

Make sure `.venv/` is included in `.gitignore`.

---

## 2.9 Minimal setup command summary

For a fresh clone, the complete setup is:

```bash
git clone <REPOSITORY_URL>
cd <PROJECT_FOLDER>
uv sync
source .venv/bin/activate
```

Or, without activating manually:

```bash
git clone <REPOSITORY_URL>
cd <PROJECT_FOLDER>
uv sync
uv run python main.py
```