# Resting-state frontal alpha asymmetry in major depressive disorder

**Exploratory secondary analysis of the Mumtaz EEG dataset**  
7 November 2026

## Abstract

**Background.** Frontal alpha asymmetry (FAA) is a measure of how alpha power differs between homologous frontal EEG electrodes. Its association with major depressive disorder (MDD) varies across studies.

**Methods.** We used the public Mumtaz dataset (Figshare version 2) to analyze eyes-closed resting EEG from 29 participants labeled MDD and 27 healthy controls. We filtered 19 scalp channels, applied an average reference, screened four-second windows for gross artifact, and used Welch spectra to compute one F4-minus-F3 log alpha power ratio per participant. We tested the mean group difference with a two-sided Welch t-test and assessed a broader spectral classifier with five-fold participant-level cross-validation.

**Results.** Mean FAA was 0.0358 in MDD and 0.0417 in controls. The MDD-minus-control difference was −0.0059 log units (95% CI −0.1332 to 0.1214, p = 0.9259). With the native EDF reference, the contrast was −0.1796 (95% CI −0.3172 to −0.0420). The exploratory classifier achieved 82.3% mean balanced accuracy. Source channel count alone reached 88.7% balanced accuracy.

**Conclusions.** Average-referenced FAA showed a near-zero mean difference, with uncertainty compatible with differences in either direction. The effect depended on referencing, and recording configuration complicated interpretation of classifier performance.

## 1. Introduction

Frontal alpha asymmetry measures the difference between logarithmic alpha power at right and left frontal sites. The F3/F4 measure can be computed from standard scalp EEG. Its value can depend on the reference, artifact handling, and recording conditions [3]. A meta-analysis found a negligible pooled F3/F4 effect after addressing heterogeneity and reported variability across studies [4].

The primary question was whether mean eyes-closed FAA differed between the MDD and control groups. We tested a zero mean difference using a two-sided test. The secondary question was whether a simple classifier could distinguish the source labels using a broader set of spectral features. The group contrast and classifier evaluate different quantities. The first uses a single left-right ratio. The second uses power levels across frequencies and electrodes.

## 2. Materials and methods

### 2.1 Study design and data source

This observational, cross-sectional reanalysis used *MDD Patients and Healthy Controls EEG Data (New)* by Wajid Mumtaz, Figshare accession **4244171**, version **2**, under CC BY 4.0 [1]. The associated paper describes the original EEG study [2]. The deposit includes eyes-closed, eyes-open, and auditory-task recordings. We used the eyes-closed condition and assigned groups from the deposit's H and MDD filename prefixes. Numeric identifiers recur across groups, so the participant key combines group and number. We did not link clinical or demographic covariates to the EDFs.

### 2.2 Cohort construction and data integrity

The versioned manifest listed 64 eyes-closed EDFs. Six listed files returned HTTP 404 during retrieval. Their file IDs are fixed in the notebook for reproducible cohort selection. Two same-group pairs had matching file checksums. We kept the lower numeric identifier from each pair and ran a sensitivity analysis that excluded both pairs entirely. The remaining EDFs passed byte-size and MD5 checks against the repository metadata. A supplementary exact-match check found no duplicate first 60 seconds of decoded scalp signals among the retained files. These checks detect exact matches and leave near-duplicates or mislabeled unique files unresolved.

### 2.3 EEG preprocessing and quality control

We loaded the EDFs with MNE-Python [5]. We retained the common 19-channel scalp montage at 256 Hz (Fp1, Fp2, F7, F3, Fz, F4, F8, T3, C3, Cz, C4, T4, T5, P3, Pz, P4, T6, O1, and O2) and excluded auxiliary channels. We applied a 1–40 Hz zero-phase Hamming-windowed FIR filter with 1 Hz and 10 Hz transition widths. Common average referencing then subtracted the mean across the 19 channels at each time point.

We trimmed two seconds from each recording edge and formed four-second windows with a two-second hop. A window was rejected if any filtered native-reference channel varied by less than 0.5 µV peak to peak, any filtered average-referenced channel exceeded 200 µV peak to peak, or the window overlapped a bad or edge annotation. A participant needed at least 20 accepted windows. The amplitude and flatness thresholds screen for gross artifacts. We used all passing windows, and the amount of accepted EEG could differ by participant.

### 2.4 Spectral features and primary endpoint

We removed the mean from each retained window, applied a Hamming taper, and averaged the modified periodograms following Welch's method [6]. The 1,024-point Fourier transform gave a 0.25 Hz grid. Within each band, we summed the density bins and multiplied by bin width to obtain absolute power in V².

| Band | Frequency range |
|---|---|
| Delta | 1 ≤ f < 4 Hz |
| Theta | 4 ≤ f < 8 Hz |
| Alpha | 8 ≤ f < 13 Hz |
| Beta | 13 ≤ f < 30 Hz |

The endpoint was **FAA = ln(alpha power at F4) − ln(alpha power at F3)**. A positive score means greater alpha power at the right frontal electrode F4 relative to left F3. This is a dimensionless log power ratio measured at the scalp. Using the guide's mean alpha density would give the same FAA because the common scale factor cancels in the log difference.

### 2.5 Statistical analysis

We compared participant FAA values with a two-sided Welch independent-samples t-test at α = 0.05. We report group means and sample SDs, the MDD-minus-control contrast and its 95% Welch confidence interval, and Hedges' *g* using the pooled sample SD and small-sample correction. The participant was the independent unit. We averaged overlapping windows within each participant before testing. Three descriptive sensitivity analyses used a 150 µV peak-to-peak cutoff, retained the native EDF reference on the primary set of clean windows, or omitted both duplicate pairs entirely. The 150 µV analysis reapplied the 20-window minimum.

### 2.6 Exploratory prediction and recording-layout checks

The classifier used 76 predictors consisting of natural-log power for four bands at 19 electrodes. We fixed L2 logistic regression at C = 1 and used shuffled, stratified five-fold cross-validation with seed 42. Standardization and model fitting occurred within each training fold. We calculated mean test-fold accuracy, balanced accuracy (the mean of the two class recalls), and ROC AUC. We compared accuracy with a training-fold majority predictor and balanced accuracy with a 999-permutation label null. The permutation procedure re-stratified the permuted labels and reran the pipeline each time. The p-value adds one to both the number of null scores at least as large as observed and the number of permutations. Fold SD describes spread within the chosen split.

The original EDFs had either 20 or 22 channels. We checked whether the source channel count could itself distinguish groups by fitting a balanced depth-one decision tree on channel count alone in the same five folds. We also reevaluated the EEG model in the 20-channel subgroup, which included both labels. Both layout checks were post hoc. There was no hyperparameter search or independent test cohort.

## 3. Results

### 3.1 Cohort and EEG quality

| Cohort step | Control | MDD |
|---|---:|---:|
| Eyes-closed EDFs listed | 30 | 34 |
| Unavailable files | 2 | 4 |
| Duplicate aliases removed | 1 | 1 |
| Passed EEG quality screening | **27** | **29** |

Unavailable records were H S12, H S18, MDD S4, MDD S8, MDD S12, and MDD S16. The primary analysis retained H S27 from the H S27/H S30 pair and MDD S33 from the MDD S33/MDD S34 pair. All 56 eligible recordings passed both the primary and stricter quality screens.

| Quality measure | Control | MDD |
|---|---:|---:|
| Recording duration, median (range), s | 301 (292–376) | 300 (180–303) |
| Median candidate windows | 147 | 147 |
| Accepted windows, median (range) | 147 (124–185) | 144 (83–148) |
| Median proportion of windows accepted | 99.3% | 98.6% |
| Original 20-channel / 22-channel recordings | 5 / 22 | 28 / 1 |

### 3.2 Primary FAA contrast

| Group | Participants | Mean FAA | SD |
|---|---:|---:|---:|
| Healthy control | 27 | 0.0417 | 0.2815 |
| MDD | 29 | 0.0358 | 0.1745 |

The estimated MDD-minus-control difference was **−0.0059 log units** (95% CI **−0.1332 to 0.1214**), Welch *t*(42.84) = −0.094, *p* = 0.9259, Hedges' *g* = −0.025. The estimated difference was near zero and the interval included zero and effects in both directions.

Figure 1 in the notebook shows each participant's FAA alongside group means and 95% confidence intervals. The control scores show greater spread and include a high positive value. Figure 2 shows the participant-median PSD at F3 and F4. Both groups display a peak around 10–11 Hz. The control group's median peak is higher at both electrodes, illustrating how absolute power can differ even when the mean within-person right-to-left ratio is similar. The spectral curves are descriptive.

### 3.3 Sensitivity analyses

| Analysis | Control / MDD n | Difference | 95% CI | p | Hedges' g |
|---|---:|---:|---:|---:|---:|
| Primary, average reference at 200 µV | 27 / 29 | −0.0059 | [−0.1332, 0.1214] | 0.9259 | −0.025 |
| Average reference at 150 µV | 27 / 29 | −0.0025 | [−0.1281, 0.1231] | 0.9684 | −0.011 |
| Native EDF reference, same clean windows | 27 / 29 | −0.1796 | [−0.3172, −0.0420] | 0.0123 | −0.725 |
| Average reference, omit both alias pairs | 26 / 28 | −0.0033 | [−0.1356, 0.1290] | 0.9598 | −0.014 |

The two alternate average-reference analyses remained close to zero. Keeping the native reference produced a larger negative estimate using the same participants and windows. The native-reference result is a descriptive sensitivity comparison. It shows that the observed group contrast depends on referencing.

### 3.4 Exploratory classification

| Five-fold test metric | Mean | Fold SD |
|---|---:|---:|
| Accuracy | 82.3% | 12.5 percentage points |
| Balanced accuracy | 82.3% | 12.7 percentage points |
| ROC AUC | 0.891 | 0.116 |
| Majority-class accuracy | 51.8% | 4.1 percentage points |

The permutation null mean was 50.4% balanced accuracy (*p* = 0.001, 999 permutations). The permutation check measures association with the recorded labels under the analysis procedure. Any cohort-specific acquisition signal correlated with the labels could also contribute to that association.

| Post hoc model | Control / MDD n | Accuracy | Balanced accuracy | ROC AUC |
|---|---:|---:|---:|---:|
| Source channel count only | 27 / 29 | 89.1% | 88.7% | 0.887 |
| EEG model, 20-channel source only | 5 / 28 | 90.5% | 78.3% | 0.920 |

Channel count alone outperformed the EEG model on mean balanced accuracy in the full cohort. The 20-channel subgroup still showed separation, but each test fold had just one control. The subgroup's high raw accuracy partly reflects its class imbalance and should be read alongside balanced accuracy.

## 4. Discussion

The average-referenced result provides little evidence for a mean FAA difference in this cohort under the primary pipeline. The small point estimate is consistent with the near-zero pooled estimate reported by van der Vinne and colleagues, although the datasets and preprocessing differ [4]. The interval leaves room for nonzero effects, and no equivalence margin was specified.

The reference sensitivity changes the scientific interpretation. A common average and the native reference produce different signals at both frontal electrodes, so their alpha power ratios can differ even when computed over the same time windows. The analysis provides a concrete example of the reference dependence discussed by Smith and colleagues [3]. It cannot identify which acquisition or physiological factors account for the native-reference contrast.

The exploratory model uses absolute spectral power from many channels and bands, so it can separate labels through patterns outside FAA. The source-channel-count result makes an acquisition explanation plausible even though the EEG model used only the common 19 channels. The metadata check does not isolate how much of the EEG model's score came from recording factors. The matched-layout subgroup is small and imbalanced, and the recordings within it could still differ in acquisition or clinical factors.

### Limitations

- **Selection and precision.** The six unavailable records could differ from the analyzed cohort. Sample size and between-person variation limit precision for small effects and subgroup analyses.
- **Clinical covariates.** The group comparison was unadjusted. Age, sex, medication, symptom severity, and comorbidities could confound associations or modify them.
- **Measurement.** The amplitude screen can leave low-amplitude eye and muscle artifacts. We used fixed alpha boundaries, did not perform ICA or manual artifact review, and analyzed differing numbers of usable windows.
- **Generalizability.** This was a single-cohort, non-preregistered reanalysis. The prediction results are internal estimates. External data acquired under a common protocol would be needed to evaluate generalization.

## 5. Conclusion

In 56 eyes-closed recordings, average-referenced F3/F4 FAA was nearly identical on average across the MDD and control groups. Changing to the native reference produced a larger group difference. The classifier separated the dataset labels, while original recording layout also strongly predicted those labels. These findings support reporting the preprocessing choice and acquisition checks alongside the headline results.

## 6. Data, code, and reproducibility

The accompanying **EEG_Frontal_Alpha_Asymmetry.ipynb** includes saved execution outputs, figures, dependency pins, and run instructions. Running all cells fetches the versioned manifest and verifies EDFs, then writes a source manifest, participant quality and feature tables, FAA contrasts, cross-validation results, layout diagnostics, permutation scores, plots, and a JSON summary. The source manifest preserves file IDs, checksums, and exclusions. Raw recordings are available from the original deposit [1].

**Environment.** Python 3.12.14, MNE-Python 1.10.2, NumPy 2.3.5, SciPy 1.17.0, pandas 2.2.3, Matplotlib 3.10.8, scikit-learn 1.8.0, and threadpoolctl 3.6.0.

**Ethics and attribution.** This analysis reused publicly licensed recordings and involved no new data collection or participant contact. The data are credited to Mumtaz and the original research team [1, 2].

## References

1. Mumtaz W. *MDD Patients and Healthy Controls EEG Data (New).* Figshare, version 2, 2016. [doi:10.6084/m9.figshare.4244171.v2](https://doi.org/10.6084/m9.figshare.4244171.v2)
2. Mumtaz W, Xia L, Ali SSA, Yasin MAM, Hussain M, Malik AS. Electroencephalogram (EEG)-based computer-aided technique to diagnose major depressive disorder (MDD). *Biomedical Signal Processing and Control.* 2017, 31, 108–115. [doi:10.1016/j.bspc.2016.07.006](https://doi.org/10.1016/j.bspc.2016.07.006)
3. Smith EE, Reznik SJ, Stewart JL, Allen JJB. Assessing and conceptualizing frontal EEG asymmetry. An updated primer on recording, processing, analyzing, and interpreting frontal alpha asymmetry. *International Journal of Psychophysiology.* 2017, 111, 98–114. [doi:10.1016/j.ijpsycho.2016.11.005](https://doi.org/10.1016/j.ijpsycho.2016.11.005)
4. van der Vinne N, Vollebregt MA, van Putten MJAM, Arns M. Frontal alpha asymmetry as a diagnostic marker in depression. Fact or fiction? A meta-analysis. *NeuroImage: Clinical.* 2017, 16, 79–87. [doi:10.1016/j.nicl.2017.07.006](https://doi.org/10.1016/j.nicl.2017.07.006)
5. Gramfort A, Luessi M, Larson E, et al. MEG and EEG data analysis with MNE-Python. *Frontiers in Neuroscience.* 2013, 7, 267. [doi:10.3389/fnins.2013.00267](https://doi.org/10.3389/fnins.2013.00267)
6. Welch PD. The use of fast Fourier transform for the estimation of power spectra. A method based on time averaging over short, modified periodograms. *IEEE Transactions on Audio and Electroacoustics.* 1967, 15(2), 70–73. [doi:10.1109/TAU.1967.1161901](https://doi.org/10.1109/TAU.1967.1161901)
