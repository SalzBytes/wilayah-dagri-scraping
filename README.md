# Permendagri 300.2.2-2138/2025 structured data extraction

Permendagri 300.2.2-2138/2025 is a ministerial decree that is the latest edition of Indonesia's administrative region codes.

The raw dataset is a single ~58MB PDF which consists of:
* The ministerial decree itself
* An appendix which contains the region codes (this is where the data will be extracted from)

## Requirements

* **Python** 3.10+
* **Java** (JRE/JDK) 8+, available on your `PATH` — required by `tabula-py`
* **poppler** (C++ library) — required to build/run `pdftotext`

Python packages are listed in [`requirements.txt`](./requirements.txt).

## Installing dependencies

The two tricky dependencies are `pdftotext` (needs the poppler C++ headers/library) and `tabula-py` (needs Java). Follow the section for your OS, then install the Python packages.

### Linux (Debian/Ubuntu)

```bash
sudo apt update
sudo apt install -y build-essential libpoppler-cpp-dev pkg-config python3-dev default-jre
```

### Linux (Fedora/RHEL)

```bash
sudo dnf install -y gcc-c++ poppler-cpp-devel pkgconfig python3-devel java-latest-openjdk
```

### macOS (Homebrew)

```bash
brew install pkg-config poppler
brew install --cask temurin   # Java; or: brew install openjdk
```

If `pip install pdftotext` cannot find poppler, point it at Homebrew:

```bash
export PKG_CONFIG_PATH="$(brew --prefix poppler)/lib/pkgconfig:$PKG_CONFIG_PATH"
```

### Windows

Easiest path is **WSL** — install Ubuntu, then follow the Linux steps above. [VS Code works great with WSL.](https://code.visualstudio.com/docs/remote/wsl)

Native Windows (without WSL):

```powershell
# Java
winget install EclipseAdoptium.Temurin.21.JDK
# poppler (provides headers + DLLs)
winget install oschwartz10612.Poppler   # or: conda install -c conda-forge poppler
```

`pdftotext` has no Windows wheels for some versions; the most reliable native route is conda:

```powershell
conda install -c conda-forge pdftotext poppler
```

### Install the Python packages

After the system dependencies are in place:

```bash
python -m venv .venv
# Linux/macOS:
source .venv/bin/activate
# Windows PowerShell:
# .venv\Scripts\Activate.ps1

pip install -r requirements.txt
```

Verify the toolchain:

```bash
java -version            # should print a version
python -c "import pdftotext, tabula; print('ok')"
```

## File configuration

Paths and tuning parameters live in [`base.ipynb`](./base.ipynb). The defaults assume this repo layout:

| Setting | Cell | Default | Purpose |
| --- | --- | --- | --- |
| `input_path` | config cell | `./raw.pdf` | Source PDF to extract from. |
| `output_path` | config cell | `./dist/base.csv` | Where the final CSV is written. **Create the `dist/` folder first** (`mkdir dist`) — Python's `open(..., 'w')` will not create it for you. |
| `RANGE` / `area` | `extract_tables` cell | `[80, 0, 600, 601]` | `[top, left, bottom, right]` crop region (in PDF points) passed to tabula. Recompute with the *"Find the area parameter"* utility cell at the bottom if the layout changes. |

Place the source PDF at `./raw.pdf` (or update `input_path`).

## Running

1. Put the source PDF at `./raw.pdf`.
2. Create the output dir: `mkdir dist`
3. Launch Jupyter: `jupyter notebook base.ipynb`
4. Run the cells top to bottom.

### What each step does

1. **Find relevant pages** — `pdftotext` scans the PDF and collects page numbers whose tables contain the desa-level detail (matches `NAMA PROVINSI /` + `JUMLAH`).
2. **Extract tables** — `tabula-py` reads those pages within `RANGE` into pandas DataFrames.
3. **Parse rows** — each row is reduced to a `(code, raw_name)` pair where `code` matches the region-code regex.
4. **Sanitise names** — strips ordinal prefixes, stray carriage returns, and in-padded text like `P A P U A`; normalises quotes; de-duplicates by code.
5. **Dump CSV** — writes `code,name` rows to `output_path`.

Utility cells at the bottom help recompute the tabula `area` and tally counts per level (province/city/district/subdistrict).
