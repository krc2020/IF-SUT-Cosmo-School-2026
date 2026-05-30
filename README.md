# Spectroscopic Capabilities in Thailand for Extragalactic Astronomy

Short-course materials for the **IF-SUT Cosmo School 2026** — a 3-hour
introduction to the optical spectroscopic facilities at the Thai National
Observatory (TNO) and the Thai Robotic Telescope (TRT) network, with worked
hands-on examples of exposure-time calculation and data reduction.

**Instructor:** Krittapas Chanchaiworawit (NARIT)
**Audience:** Bachelor's to PhD students in Physics & Astronomy
**Duration:** 3 hours (one 10-min break)
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
| `02_LRS_data_reduction_v4.ipynb` | End-to-end LRS reduction: bias → flat → CR-reject → wavelength calibration (deg=2) → nucleus finder → flux calibration → save. | M82 nucleus at row 150, 7 HgAr lines matched, RMS = 6.8 Å, median S/N/pix = 9.0 |

Earlier versions (`*_v2`, `*_v3`, `*_v4` for ETC; `*`, `*_v2`, `*_v3` for
reduction) are kept for reference — open the highest version number for
the canonical content.

### Example data (M82, 1.8″ slit, PA = 0°)

`example_frames/` — 15 raw FITS frames (~7.7 MB total) that let the
reduction notebook run end-to-end:

```
example_frames/
├── bias/                       5 frames @ 0 s            (runs 167-171)
├── flat/                       3 sky-flats @ 20 s        (runs 137-139)
├── arc/                        1 HgAr arc @ 60 s         (run 148)
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

### Figures and infographics

| File | Used in |
|---|---|
| `Plot1a_…png/pdf`, `Plot1b_…`, `Plot1c_…zoom`, `Plot1d_…zoom_with_Thai` | Aperture vs wavelength coverage of major optical/NIR/MIR spectrographs |
| `Plot2a_…png/pdf` through `Plot2d_…zoom_with_Thai` | Continuum AB-mag limit vs spectral resolution R |
| `Spectrograph_landscape_SOURCES.md` | Citation list for every number in plots 1–2 |
| `LRS_raytrace_overlay.png` + `LRS_raytrace_schematic.{png,pdf}` | LRS optical-train ray-trace overlay (transparent + standalone) |
| `LRS_throughput_plot.{png,pdf}` + `LRS_throughput.csv` + `LRS_throughput_assumptions.md` | Per-component LRS+TNT throughput model |
| `SNR_dashboard.{png,pdf}` | 6-panel S/N infographic (t, N, η, F_src, F_sky, R) |
| `Sci_pathway_1_AGN_diagnostics.{png,pdf}` | AGN single-epoch diagnostics infographic |
| `Sci_pathway_2_Reverberation.{png,pdf}` | TNT + TRT reverberation-mapping infographic |
| `Sci_pathway_3_Galaxy_surveys.{png,pdf}` | Galaxy-survey use-cases infographic |

---

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

Seven steps, runtime ~20 s on the example bundle:

1. **Master bias** — median-combine 5 frames (~1.2 ADU rms).
2. **Master flat** — slit-interior normalisation.
3. **Stack + flat + CR-reject** — science and standard frames.
4. **Wavelength calibration** — collapses the arc using the *slit interior*
   (not the bright slit-edge holes), matches against a 20-line NIST HgAr
   reference list with iterative tolerance shrinking, fits a deg=2
   polynomial.
5. **Find the nucleus** by **emission-excess inside the slit interior**.
   The slit's machined edges have two bright "holes" that look like
   point sources but are not; the algorithm masks them and locates the
   M82 nucleus (~row 150) by the Hα-over-continuum excess.
6. **Extract & flux-calibrate** with the BD+75°325 sensitivity function.
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
Master flat rms (inside slit): 0.269
M82           stacked 3 frames  (flat-fielded, CR-rejected)
BD+75°325     stacked 3 frames  (flat-fielded, CR-rejected)
HgAr arc      stacked 1 frame   (no CR)
Total on-source: M82 = 180 s, BD+75°325 = 180 s

Slit interior (used for arc + source finding): rows 62–231
Arc 1D background MAD: 0.81 ADU,  threshold 3.0
Detected 59 peaks
Final wavelength solution: deg = 2, 7 lines matched, RMS = 6.75 Å
λ range: 3678–7926 Å,  median dispersion: 4.15 Å/pixel

M82 nucleus: row 150  (spatial FWHM of emission excess ≈ 15 pix)
BD+75°325:   row 154

Median S/N per pixel (4900–7800 Å): 9.0
Saved M82_reduced.fits
```

---

## Citation / acknowledgement

If you use these materials in a publication, please cite:

> Chanchaiworawit, K. (2026). *Spectroscopic Capabilities in Thailand for
> Extragalactic Astronomy* — IF-SUT Cosmology School course materials,
> National Astronomical Research Institute of Thailand.

Underlying LRS instrument-design + commissioning references:
- Phetra et al. 2016, *Opt. Express* **24**, 1416 — TNT focal-reducer design
- Prasit et al. 2019, *SPIE* **11116**, 111161A — TNT focal-reducer commissioning

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
