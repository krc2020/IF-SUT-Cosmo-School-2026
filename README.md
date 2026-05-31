# Spectroscopic Capabilities in Thailand for Extragalactic Astronomy

Short-course materials for the **IF-SUT Cosmo School 2026** — a 3-hour
introduction to the optical spectroscopic facilities at the Thai National
Observatory (TNO) and the Thai Robotic Telescope (TRT) network, with worked
hands-on examples of exposure-time calculation and data reduction.

**Instructor:** Krittapas Chanchaiworawit (NARIT)
**Audience:** Bachelor's to PhD students in Physics & Astronomy
**Duration:** 3 hours
**Cycle:** anchored to TNT/LRS **Cycle 14** (October 2026 – May 2027)

---

## Quick start

```bash
# 1. Clone or download the repository
git clone https://github.com/<your-user>/IF-SUT_Cosmo_School.git
cd IF-SUT_Cosmo_School

# 2. Install dependencies
pip install numpy scipy matplotlib astropy ipywidgets astroscrappy

# 3. Open the notebooks
jupyter lab        # or `jupyter notebook`
```

Open **`01_LRS_ETC_v5.ipynb`** first (Exposure-Time Calculator) and
**`02_LRS_data_reduction_v4.ipynb`** second (end-to-end reduction of the
shipped M82 frames).

---

## What's in this repository

### Lecture deck

| File | Description |
|---|---|
| `Lesson_Blocks1to6_Thai_Spectroscopy.pptx` | 36-slide deck covering Blocks 1–6 of the syllabus |
| `Syllabus_Spectroscopy_Thailand_Extragalactic.md` | The course syllabus (3-hour breakdown) |

### Jupyter notebooks (run these)

| Notebook | What it does | Result on the example data |
|---|---|---|
| `01_LRS_ETC_v5.ipynb` | Photon-noise-limited Exposure-Time Calculator. Source spectrum, noise budget, throughput, S/N per pixel, **redshift series for the M82 template (z = 0 – 0.5)**. | Median S/N/pixel for M82 (3 × 600 s) → 1269 |
| `02_LRS_data_reduction_v4.ipynb` | End-to-end LRS reduction: bias → flat → CR-reject → wavelength calibration (auto-degree, two-brightest-line anchor) → nucleus finder → **continuum tracing** → **trace-following + optimal extraction** → **flux calibration over 4000–8000 Å (reliably calibrated 4626–7996 Å)** → save. | M82 nucleus at row 150, **12 HgAr lines matched, RMS = 0.91 Å**, Hα residual = −0.9 Å, standard trace drift 13 pix across the chip, optimal-vs-aperture S/N gain +11%, **median S/N/pix = 8.5** over the reliably-calibrated band (covers Hβ 4861, [O III] 4959/5007, [N II] + Hα, [S II]) |

### Example data (M82, 1.8″ slit, PA = 0°)

`example_frames/` — 15 raw FITS frames (~7.7 MB total) that let the
reduction notebook run end-to-end:

```
example_frames/
├── bias/                       5 frames @ 0 s            (runs 167-171)
├── flat/                       3 sky-flats @ 20 s        (runs 137-139)
├── arc/                        5 HgAr arcs @ 60 s        (runs 149-153)
├── science_M82/                3 frames @ 60 s, PA=0     (runs 26-28)
├── standard_BD75d325/          3 frames @ 60 s           (runs 71-73)
├── M82_template_3500_9000.txt  M82-like template spectrum used by the ETC
└── M82_reduced.fits            reduced 1D spectrum (output of notebook 2)
```

The bundle was extracted from the 2026-04-01 LRS commissioning run on the
2.4-m TNT. The original archive contains 5-frame stacks at all three slit
widths and both position angles — see `LRS/Data_Reduction_Pipeline/file_map.py`.

### M82 template (input for the ETC)

`example_frames/M82_template_3500_9000.txt` — a 2-column ASCII
(wavelength_Å, f_λ in erg s⁻¹ cm⁻² Å⁻¹), 5501 samples at 1 Å spacing.
The continuum is a Calzetti+ 2000 attenuated stellar SED anchored to the
LRS-reduced M82 continuum at 5500 Å; emission lines (Hα, [N II], [O III],
Hβ, [O II], [S II], [Ar III]) and stellar absorption (Ca II triplet, Mg b,
Na D, Ca H/K) are added at literature strengths.

References inside the file header: Kennicutt+ 1994, Förster Schreiber+
2003, Origlia+ 2004, SDSS DR17.

## Notebook walk-throughs

### `01_LRS_ETC_v5.ipynb`

Nine sections, total runtime ~30 s:

1. **Site, telescope, instrument constants** — TNT 2.4 m, f/10, 0.09 area
   obstruction, system-throughput anchors.
2. **Helpers** — `ab_mag_to_flam`, `slit_loss`, `run_etc`,
   `solve_exptime_for_snr`.
3. **Smoke test** — g ≈ 20 AGN at z ≈ 0.3.
4. **Noise budget** — source/sky/dark/read variance breakdown at Hβ.
5. **System throughput** curve.
6. **M82 input spectrum** — loaded from `example_frames/M82_template_3500_9000.txt`.
7. **S/N per pixel** — `snr_per_pixel(t_per_frame, n_frames, …)`.
8. **Plot S/N per pixel vs wavelength** — overlays of multiple strategies.
9. **Quick lookup** — `solve_t_per_frame_for_snr_pp(target_snr, N, λ_aim)`.
10. **Redshift series** — `redshift_spectrum(M82_SPECTRUM, z)` and S/N
    predictions at z = 0, 0.05, 0.10, 0.20, 0.50.
11. **Exposure budget vs z** — exposure needed to keep S/N/pix = 10 at
    the redshifted Hα line.

### `02_LRS_data_reduction_v4.ipynb`

Seven steps, runtime ~20s on the example bundle:

1. **Master bias** — median-combine 5 frames (~1.2 ADU rms).
2. **Master flat** — slit-interior normalisation.
3. **Stack + flat + CR-reject** — science and standard frames.
4. **Wavelength calibration** — collapses the arc using the *slit interior*
   (not the bright slit-edge holes), and anchors the initial dispersion on
   the **two brightest detected peaks** (Hg 5460.74 Å and Ar 7635.11 Å),
   so the dispersion is measured from the data rather than assumed. Lines
   are then matched against a 15-line NIST HgAr reference list with
   iterative tolerance shrinking, fits a deg=1/2/3 polynomial (auto-picked
   by RMS), and sigma-clips at min(5 Å, 2 σ). Result on the example data:
   **12 lines matched, RMS = 0.91 Å, Hα residual = −0.9 Å**.
5. **Find the nucleus** by **emission-excess inside the slit interior**.
   The slit's machined edges have two bright "holes" that look like
   point sources but are not; the algorithm masks them and locates the
   M82 nucleus (~row 150) by the Hα-over-continuum excess.
5b. **Trace the continuum** of both the science object and the
   spectrophotometric standard. For each binned column, fit an
   intensity-weighted centroid in a small spatial window around the
   nucleus row, sigma-clip outliers, and fit a deg=2 polynomial
   `row_centroid = f(col)`. The standard drifts ~13 rows across the
   chip from atmospheric differential refraction; the science object's
   trace is fit independently because it sat at a different airmass /
   parallactic angle. The mean spatial profile measured here doubles as
   the optimal-extraction weight in Step 6.
6. **Extract & flux-calibrate**. Two estimators run side-by-side: a
   trace-following ±8-row aperture sum, and a Horne (1986) optimal
   extraction that weights each row by `P/V`. The optimal estimator
   gains ~+11% in S/N for free. Sensitivity is calibrated against the
   BD+75°325 CALSPEC reference flux over the full 4000–8000 Å range.
7. **Save** to `M82_reduced.fits` (binary table: WAVELENGTH, FLUX, ERROR,
   COUNTS).

---

## Dependencies

Minimum:
```
numpy           ≥ 1.20
scipy           ≥ 1.7
matplotlib      ≥ 3.4
astropy         ≥ 5.0
```
Recommended:
```
ipywidgets      ≥ 7   # widgets in ETC §6 (M82 plot)
astroscrappy    ≥ 1.0 # L.A.Cosmic CR rejection in reduction notebook
```

Install everything in one go:
```bash
pip install numpy scipy matplotlib astropy ipywidgets astroscrappy
```

Or via a conda environment:
```bash
conda create -n ifsut python=3.11 numpy scipy matplotlib astropy ipywidgets
conda activate ifsut
pip install astroscrappy
```

---

## Tested results (on the shipped example_frames)

```
Master bias: median = 300.0 ADU,  rms = 1.22 ADU
Slit illuminates rows 42–251  (123 rows)
Slit interior used for spectral shape: rows 57–236
Normalized-flat rms inside the slit interior: 0.046
M82           stacked 3 frames  (flat-fielded, CR-rejected)
BD+75°325     stacked 3 frames  (flat-fielded, CR-rejected)
HgAr arc      stacked 5 frames  (no CR)
Total on-source: M82 = 180 s, BD+75°325 = 180 s

Slit interior (used for arc + source finding): rows 57–236
Arc 1D background MAD: 0.40 ADU,  threshold 3.0
Detected 13 peaks
Brightest peak at col 388.3  →  Hg 5460.74 Å
Second-brightest at col 891.0  →  Ar 7635.11 Å
  ⇒ initial linear dispersion = 4.326 Å/pixel
Final solution: deg = 3, 12 lines used, RMS = 0.91 Å
λ range: 3906–8216 Å,  median dispersion: 4.28 Å/pixel

Hα sanity check (rest 6562.8 Å):
  Predicted at column 646 → λ = 6561.89 Å (residual -0.91 Å)

M82 nucleus: row 150  (spatial FWHM of emission excess ≈ 4 pix)
BD+75°325:   row 154

Standard trace: rms = 0.11 pix, spatial FWHM = 10.0 pix,
                trace span = 147.98 → 160.53  (13 px drift!)
M82      trace: rms = 0.72 pix, spatial FWHM = 11.0 pix,
                trace span = 145.44 → 153.16  ( 8 px drift)

Median S/N before flux cal — aperture: 6.6, optimal: 7.3 (gain: +11%)
Spectrum well-calibrated over 4626 – 7996 Å (789 pixels)
Median S/N per pixel: 8.5
Saved M82_reduced.fits
```

---

## Citation / Acknowledgment

If you use these materials in a publication, please cite:

> Chanchaiworawit, K. (2026). *Spectroscopic Capabilities in Thailand for
> Extragalactic Astronomy* — IF-SUT Cosmology School course materials,
> National Astronomical Research Institute of Thailand.

Underlying LRS instrument-design + commissioning references:
- Phetra et al. 2016, *Opt. Express* **24**, 1416 — TNT focal-reducer design
- Prasit et al. 2019, *SPIE* **11116**, 111161A — TNT focal-reducer commissioning
- Paenoi et al. 2019, Current Applied Science and Technology, 20, 1 — LRS Mrk-II's design and on-sky performance 

CoLoRS commissioning references:
- Thomrungpiyathana et al. 2025, *SPIE* **13599**, 135990N — optomechanical design
- Chanchaiworawit et al. 2025, *SPIE* **13624**, 136241M — on-sky performance

NIST atomic-line list (HgAr): A. Kramida, Yu. Ralchenko, J. Reader, and
NIST ASD Team (2025), *NIST Atomic Spectra Database* (version 5.11).
https://physics.nist.gov/asd

BD+75°325 flux: STScI CALSPEC archive.

---

## License

Course content (notebooks, figures, text): **CC BY 4.0**.
Code (the helper functions inside the notebooks): **MIT**.

---

## Contact

Questions, fixes, pull requests welcome.
Email: **krittapas [at] narit.or.th**
