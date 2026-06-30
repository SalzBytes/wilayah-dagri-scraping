# Permendagri 300.2.2-2138/2025 data structured data extraction

Permendagri 300.2.2-2138/2025 is a ministerial decree that is the latest edition of Indonesia's administrative region codes.

The raw dataset is a single ~58MB PDF which consists of:
* The ministerial decree itself
* An appendix which contains the region codes (this is where the data will be extracted from)

## Dependencies

If you're on Windows, using WSL would make installing the dependencies a lot easier. [You can use Visual Studio Code with WSL too for the best development experience.](https://code.visualstudio.com/docs/remote/wsl)

### pdftotext

pdftotext is used for finding relevant pages from the raw PDF file. There are [several dependencies](https://pypi.org/project/pdftotext/) that need to be installed beforehand.

    sudo apt install build-essential libpoppler-cpp-dev pkg-config python3-dev
    pip install pdftotext

### tabula-py

We use [tabula-py](https://pypi.org/project/tabula-py/), which is a wrapper for [tabula-java](https://github.com/tabulapdf/tabula-java).

Java needs to be installed and available from your `PATH`.
