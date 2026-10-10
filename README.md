# Physics-Informed 3D Crystal Structure Band-Gap Analyzer

A professional Streamlit dashboard for a materials-science machine-learning
project: **"Composition + Crystal Structure + Optical Analysis"**.

Predict experimental **band gaps (eV)** from chemical composition using
physics-informed descriptors, analyse real **3D crystal structures** from
CIF / POSCAR / VASP files, estimate the approximate **optical absorption edge**
(`λ ≈ 1240 / Eg`), and — only when a separately trained model is supplied — run
true **structure-aware ML** predictions.

**Student:** Muhammad Fayaz · **Supervisor:** Sir Noman · **KPIT Board** ·
**UET Mardan**

---

## 1. Project objective

Design a fast, physics-guided screening tool that maps chemical composition to
the experimental band gap of inorganic materials, and enrich the analysis with
real crystal-structure descriptors whenever a structure file is available.

The application is careful about what it claims:

| Mode | What it actually does |
|---|---|
| **Composition Only** | Predicts the band gap from physics-informed composition/electronic descriptors. |
| **Composition + 3D Structure** | Same composition model prediction **plus** a full 3D structure analysis and visualization of an uploaded CIF/POSCAR. The structure is analysed, but is **not** fed into the composition model. |
| **Structure-Aware ML** | Only active if `models/materials_bandgap_3d_model.pkl` (+ imputer + features) exist. Uses composition **and** 3D crystal descriptors with a saved imputer to predict. |

The application never claims that a 2D chemical formula uniquely determines a
3D crystal structure, never fabricates lattice parameters / coordinates / space
groups, and only shows 3D descriptors computed from a real uploaded structure.

## 2. Dataset

**MatBench `matbench_expt_gap`** (4604 experimental band gaps, in eV), the
standard experimental band-gap benchmark. See
`src/features.py` / `src/features_enhanced.py` for the shared data cleaning and
feature engineering.

> The MatBench experimental dataset is **composition-based**: it does not
> include a crystal structure (CIF) for every training row. For this reason the
> trained ML model predicts from **composition/electronic descriptors**.
> A genuine 3D model would require a dataset where every row has an
> experimentally matched crystal structure.

## 3. Composition features (core 8)

- `num_elements` — number of distinct elements
- `num_atoms` — atoms per formula unit
- `avg_atomic_mass` — average atomic mass (g/mol)
- `max_atomic_number` / `min_atomic_number`
- `max_electronegativity` / `min_electronegativity`
- `electronegativity_difference`

## 4. Physics-informed (enhanced) descriptors

Additional atom-fraction **weighted** + **range** electronic descriptors
(20 total with the core 8):

- `weighted_atomic_number` / `atomic_number_range`
- `weighted_atomic_mass` / `mass_range`
- `weighted_electronegativity` / `electronegativity_range`
- `weighted_periodic_row` / `periodic_row_range`
- `weighted_periodic_group` / `periodic_group_range`
- `weighted_atomic_radius` / `atomic_radius_range`

These capture ionic character, atomic size, and periodic-table trends that
correlate with the band gap. They are still **proxy composition descriptors** —
polymorphs with the same formula can have different real band gaps.

## 5. 3D crystal descriptors (from an uploaded CIF / POSCAR)

Computed with **pymatgen** only from a real structure file:

- reduced formula, number of sites
- crystal system, space group, space-group number
- lattice a / b / c and α / β / γ
- cell volume, **volume per atom**, density
- nearest-neighbour distance statistics (mean / min / max / std, periodic
  minimum-image distances)
- coordination statistics (mean / min / max, geometric covalent-radius based)

The app renders an interactive **Plotly 3D** unit cell: atoms coloured by
element with hover coordinates, approximate bonds, unit-cell edges drawn from
the **actual lattice matrix** (never a fake cubic cell), lattice-vector axes,
and full rotate/zoom inspection.

## 6. Band-gap prediction

Loaded from:

```
models/materials_bandgap_enhanced_model.pkl   # Random Forest regressor
models/materials_bandgap_enhanced_scaler.pkl  # StandardScaler
models/materials_bandgap_enhanced_features.pkl# exact feature order (20 cols)
```

The app constructs exactly the saved feature order, applies the saved scaler,
and predicts. The result is clearly labelled **"Composition/Electronic Model"** —
even when a crystal structure is present, so the app never implies the 3D
structure was used.

## 7. Optical absorption-edge approximation

From the predicted band gap:

```
λ_edge (nm) ≈ 1240 / Eg (eV)
```

Classified approximately: `<400 nm` Ultraviolet, `400–700 nm` Visible,
`700–2500 nm` Near Infrared, `>2500 nm` Infrared.

This is an **approximate absorption-edge conversion, not a complete
optical-property model** — it does not predict the absorption spectrum,
refractive index, or extinction coefficient.

## 8. Composition model vs structure-aware model

- **Composition/Electronic model** — trained on `matbench_expt_gap` using the
  20 composition/electronic descriptors. Available in every mode.
- **Structure-aware model** — would be trained on a dataset pairing each band
  gap with a real crystal structure, using composition **+** 3D crystal
  descriptors. Only usable when its artifacts exist:

```
models/materials_bandgap_3d_model.pkl      # trained regressor
models/materials_bandgap_3d_imputer.pkl    # fitted imputer
models/materials_bandgap_3d_features.pkl   # exact feature order
```

If these files are absent the app disables **Structure-Aware ML** and shows:

> "3D structure-aware prediction is not available because a paired
> structure-aware training model has not been supplied. The current
> experimental MatBench dataset is composition-based. A genuine 3D model
> requires experimentally matched crystal structures."

The app never trains a model inside Streamlit and never fuses a CIF with an
experimental gap from a different source.

## 9. Required model files

```
models/
├── materials_bandgap_enhanced_model.pkl      # required — Random Forest
├── materials_bandgap_enhanced_scaler.pkl     # required — StandardScaler
├── materials_bandgap_enhanced_features.pkl   # required — feature order
├── bandgap_enhanced_metadata.json            # optional — metrics display
└── bandgap_enhanced_feature_importance.csv   # optional — importance chart
```

Regenerate with: `python train_model_enhanced.py`.

Optional comparison tables in `results/` or `models/`
(`model_comparison.csv`, `enhanced_model_comparison.csv`) power the
**Model Performance** tab.

## 10. Run locally

```bash
pip install -r requirements.txt
streamlit run app.py
```

Open `http://localhost:8501`.

Workflow:

1. Enter a formula (e.g. `GaAs`) in the sidebar — or pick an example.
2. Optionally upload a `.cif`, `.vasp` or `POSCAR` file for 3D analysis.
3. Pick an analysis mode and click **Run analysis**.
4. Inspect the tabs: Overview, Composition, 3D Crystal, Band Gap,
   Optical Analysis, Model Performance.
5. Download the feature table, crystal descriptors, prediction summary
   (CSV/JSON) or export the uploaded structure as CIF.

## 11. Deploy on Streamlit Community Cloud

1. Push a GitHub repository containing `app.py`, `requirements.txt`,
   `src/` and the `models/` artifacts above.
2. Open [Streamlit Community Cloud](https://streamlit.io/cloud).
3. **Create app** → select the repository → set the main file to `app.py`.
4. Deploy and open the generated URL.

Packages are pinned in `requirements.txt`. TensorFlow is intentionally **not**
required by the Streamlit app.

---
See also `colab_training_prompt.md` for the Colab deep-learning training cell
and `README_Streamlit.md` for the enhanced Colab procedure.