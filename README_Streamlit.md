# Enhanced Band-Gap Prediction + Crystal Structure Streamlit App

## 1. What was added

The original Colab project was preserved and extended with:

- Electronic/composition descriptors:
  - weighted atomic number
  - atomic-number range
  - weighted atomic mass
  - mass range
  - weighted electronegativity
  - electronegativity range
  - weighted periodic row/group
  - periodic row/group ranges
  - weighted atomic radius
  - atomic-radius range
- Enhanced Random Forest band-gap model.
- Feature-importance analysis.
- Crystal-structure descriptors from CIF/POSCAR:
  - crystal system
  - space group
  - space-group number
  - lattice a/b/c
  - alpha/beta/gamma
  - cell volume
  - density
  - nearest-neighbour distance statistics
- Interactive 2D crystal-structure visualization with Plotly.
- Streamlit web interface.

## 2. Important scientific point

The original Matbench experimental band-gap table used in the notebook is composition-based. It does not provide a crystal structure/CIF for every training row.

Therefore:

**The enhanced ML model uses composition/electronic descriptors for training and prediction.**

Crystal descriptors are calculated from a real uploaded CIF/POSCAR and shown in the application.

Do not create fake crystal descriptors for the Matbench rows. If you later obtain a dataset where every row has both a band gap and a crystal structure, a genuinely structure-aware model can be trained by adding the crystal descriptors to X.

## 3. Google Colab procedure

### Step 1 — Upload the enhanced notebook

Open Google Colab and upload:

`project2_enhanced_bandgap.ipynb`

Run the cells from top to bottom.

### Step 2 — Run the original notebook

Run the original cells normally until the original model/plots have completed.

### Step 3 — Run the new enhancement cells

At the end of the notebook, run the seven added cells in this order:

1. Enhanced installation
2. Electronic descriptors
3. Enhanced model training
4. Crystal-structure descriptors
5. 2D visualization
6. Complete enhanced analysis
7. Enhanced feature importance

The enhanced model will create:

- `materials_bandgap_enhanced_model.pkl`
- `materials_bandgap_enhanced_scaler.pkl`
- `materials_bandgap_enhanced_features.pkl`
- `bandgap_enhanced_feature_importance.csv`

### Step 4 — Test a formula

Use a formula such as:

`GaAs`

or:

`TiO2`

The notebook will calculate the enhanced descriptors and predict the band gap.

### Step 5 — Test a crystal structure

Use the existing CIF upload cell or upload a CIF in the new structure-analysis section.

You will obtain:

- space group
- crystal system
- lattice parameters
- cell volume
- density
- nearest-neighbour statistics
- 2D structure plot

## 4. Run Streamlit locally

Put these files in one folder:

- `app.py`
- `requirements.txt`
- `materials_bandgap_enhanced_model.pkl`
- `materials_bandgap_enhanced_scaler.pkl`
- `materials_bandgap_enhanced_features.pkl`

Then run:

```bash
pip install -r requirements.txt
streamlit run app.py
```

Open the local address shown by Streamlit, normally:

```text
http://localhost:8501
```

## 5. Streamlit workflow

1. Enter a chemical formula.
2. Click **Predict / Analyze**.
3. The app calculates electronic/composition descriptors.
4. The trained Random Forest predicts the band gap.
5. The app converts the predicted band gap to an estimated absorption-edge wavelength using:

   λ(nm) ≈ 1240 / Eg(eV)

6. Upload a CIF/POSCAR if you want structure analysis.
7. The app then calculates crystal descriptors.
8. The app renders the crystal as an interactive 2D Plotly plot.

## 6. Deploy on Streamlit Community Cloud

1. Create a GitHub repository.
2. Upload:
   - `app.py`
   - `requirements.txt`
   - the three `.pkl` model files
3. Open Streamlit Community Cloud.
4. Create a new app.
5. Select your GitHub repository.
6. Select `app.py` as the main file.
7. Deploy.

## 7. How to make it a true structure-aware ML project

For a stronger research version, prepare a table like:

| formula | cif_file | band_gap_eV |
|---|---|---:|
| GaAs | CIF/GaAs.cif | 1.42 |
| TiO2 | CIF/TiO2.cif | 3.0 |
| ... | ... | ... |

Then calculate crystal descriptors for every row and train:

`band_gap = f(composition/electronic descriptors + crystal descriptors)`

That would directly support the project statement:

**"Prediction of band gap and structure/optical properties using crystal-structure and electronic-structure descriptors."**

