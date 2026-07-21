# `base.csv` schema

CSV-delimited by `;`. One row per administrative region.

## Columns

| Column | Description                                                                                                                                                                         |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Code` | Region code from Permendagri 300.2.2-2138/2025. Dot-separated, length encodes level: `NN` (province), `NN.NN` (kab/kota), `NN.NN.NN` (kecamatan), `NN.NN.NN.NNNN` (kelurahan/desa). |
| `Name` | Sanitised region name.                                                                                                                                                              |
| `Type` | Integer level code (1–6), see below.                                                                                                                                                |

## `Type` values & totals

| Type | Level         | Indonesian | Count  |
| ---- | ------------- | ---------- | ------ |
| 1    | Province      | Provinsi   | 38     |
| 2    | Regency       | Kabupaten  | 416    |
| 3    | City          | Kota       | 98     |
| 4    | District      | Kecamatan  | 7,285  |
| 5    | Urban village | Kelurahan  | 8,496  |
| 6    | Village       | Desa       | 75,266 |

**Total rows:** 91,599

> Type derived from code length + digit position (`parse_code` in `base.ipynb`):
> kab vs kota by 4th digit (0–6 = kabupaten, 7–9 = kota); kelurahan vs desa by 10th digit (1 = kelurahan, 2/3 = desa).
