# Permendagri 300.2.2-2138/2025 structured data extraction

Permendagri 300.2.2-2138/2025 is a ministerial decree that is the latest edition of Indonesia's administrative region codes.

The raw dataset is a single ~58MB PDF which consists of:

- The ministerial decree itself
- An appendix which contains the region codes (this is where the data is extracted from)

The extracted result is written to [`dist/base.csv`](./dist/base.csv). See [`SCHEMA.md`](./SCHEMA.md) for column descriptions and totals.

---

## Requirements

| Dependency | Version | Why |
| --- | --- | --- |
| **Python** | 3.10+ | Runs the notebook. |
| **Java** (JRE/JDK) | 8+ | Required by `tabula-py`. Must be on your `PATH`. |
| **poppler** | — | C++ library required by `pdfplumber`. |

Python packages are listed in [`requirements.txt`](./requirements.txt).

---

## Setup

Pick the section for your OS, run the blocks top to bottom. Every block is copy-paste ready.

### Step 1 — System dependencies

<details open>
<summary><b>Linux (Debian / Ubuntu)</b></summary>

```bash
sudo apt update
sudo apt install -y build-essential libpoppler-cpp-dev pkg-config python3-dev default-jre
```

</details>

<details>
<summary><b>Linux (Fedora / RHEL)</b></summary>

```bash
sudo dnf install -y gcc-c++ poppler-cpp-devel pkgconfig python3-devel java-latest-openjdk
```

</details>

<details>
<summary><b>macOS (Homebrew)</b></summary>

```bash
brew install pkg-config poppler
brew install --cask temurin
```

If `pip install` later cannot find poppler, point it at Homebrew, then re-run pip:

```bash
export PKG_CONFIG_PATH="$(brew --prefix poppler)/lib/pkgconfig:$PKG_CONFIG_PATH"
```

</details>

<details>
<summary><b>Windows</b></summary>

**Recommended:** use [WSL](https://code.visualstudio.com/docs/remote/wsl) (install Ubuntu) and follow the Linux steps above.

**Native Windows** — install Java and poppler:

```powershell
winget install EclipseAdoptium.Temurin.21.JDK
winget install oschwartz10612.Poppler
```

If a pip wheel fails to build, the most reliable native route is conda:

```powershell
conda install -c conda-forge poppler
```

</details>

### Step 2 — Python environment

<details open>
<summary><b>Linux / macOS</b></summary>

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

</details>

<details>
<summary><b>Windows (PowerShell)</b></summary>

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

</details>

### Step 3 — Verify the toolchain

```bash
java -version
python -c "import pdfplumber, tabula; print('ok')"
```

You should see a Java version and `ok`.

---

## Configuration

Paths and tuning parameters live in [`base.ipynb`](./base.ipynb).

| Setting | Cell | Default | Purpose |
| --- | --- | --- | --- |
| `input_path` | config cell | `./raw.pdf` | Source PDF to extract from. |
| `output_path` | config cell | `./dist/base.csv` | Where the final CSV is written. Create `dist/` first. |
| `RANGE` / `area` | `extract_tables` cell | `[80, 0, 600, 601]` | `[top, left, bottom, right]` crop region (PDF points) for tabula. Recompute with the *"Find the area parameter"* utility cell if the layout changes. |

---

## Running

```bash
mkdir -p dist
jupyter notebook base.ipynb
```

Then run the cells top to bottom. Place the source PDF at `./raw.pdf` (or update `input_path`).

### What each step does

1. **Find relevant pages** — `pdfplumber` scans the PDF and collects pages whose tables contain desa-level detail (match `NAMA PROVINSI /` + `JUMLAH`).
2. **Extract tables** — `tabula-py` reads those pages within `RANGE` into pandas DataFrames.
3. **Parse rows** — each row is reduced to a `(code, raw_name)` pair where `code` matches the region-code regex.
4. **Sanitise names** — strips ordinal prefixes, stray carriage returns, and in-padded text like `P A P U A`; normalises quotes; de-duplicates by code.
5. **Dump CSV** — writes `Code;Name;Type` rows to `output_path`.

Utility cells at the bottom help recompute the tabula `area` and tally counts per level.
