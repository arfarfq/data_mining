# ACDC Data Mining Project

This repository contains the exploratory data analysis for the NOVA IMS Data Mining project. The Phase 1 notebook is `notebooks/EDA.ipynb`, and its input is `data/acdc_dataset.csv`.

## Set up from a fresh clone

1. Install [uv](https://docs.astral.sh/uv/getting-started/installation/) and Git.
2. Clone this repository and open a terminal in its root directory (the folder containing `pyproject.toml`).
3. Install the Python version specified in `.python-version` and create the project environment from the committed lockfile:

   ```sh
   uv python install 3.12
   uv sync --locked
   ```

   `uv sync` creates `.venv` and installs the analysis packages plus the development tools, including JupyterLab. The `.venv` folder is ignored by Git.

4. Start JupyterLab from the repository root:

   ```sh
   uv run --locked jupyter lab
   ```

5. Open `notebooks/EDA.ipynb` and select the Python kernel from this project's `.venv` environment. Run the cells from the top. The notebook locates `data/acdc_dataset.csv` whether the kernel starts in the repository root or the `notebooks` folder.

If using VS Code instead, open the repository root, install the Python and Jupyter extensions, and select the `.venv` interpreter for the notebook.

## Quick environment check

From the repository root, this command verifies the notebook's imports and data path:

```sh
uv run --locked python -c "import pandas, scipy, statsmodels, matplotlib, sklearn; data = pandas.read_csv('data/acdc_dataset.csv', sep=';'); print(data.shape)"
```

The CSV is semicolon-separated. The expected shape of the current file is `(15465, 31)`.

## Updating dependencies

Add or remove packages with `uv add <package>` or `uv remove <package>`, then commit both `pyproject.toml` and `uv.lock`. For notebook-only tools, use `uv add --dev <package>`. Other group members can then rerun `uv sync --locked`.
