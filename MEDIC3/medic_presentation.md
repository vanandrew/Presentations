---
title: Framewise distortion correction in Multi-echo Echo Planar Imaging
author: Andrew Van, PhD Candidate in Biomedical Engineering
institute: Dosenbach Lab, Washington University in St. Louis
date: August 21, 2023
---

## Echo Planar Imaging suffers from distortion due to B0 inhomogeneities

![Simulation of distortion effects caused by B0 inhomogeneities.](imgs/distortion_simulation.png){ width=80% }

Correcting distortion requires measuring B0 field change.

## Current B0 field mapping approaches

Current approaches<sup>[2]</sup>:

|         Phase Difference       |        PEpolar         |
| :----------------------------: | :--------------------: |
| ![](imgs/phase_difference.png) |    ![](imgs/rpe.png)   |

Both require separate field map acquisitions before/after a EPI scan!

<p style="font-size: 10px;">[2] Wang F, Dong Z, Reese TG, Bilgic B, Katherine Manhard M, Chen J, Polimeni JR, Wald LL, Setsompop K. Echo planar time-resolved imaging (EPTI). Magnetic Resonance in Medicine. 2019;81(6):3599–3615.</p>

## The Multi-Echo DIstortion Correction (MEDIC) algorithm

MEDIC is an algorithm that extracts field maps from the phase 
information of multi-echo EPI data. This removes the need for a 
separate field map acquisition:

<div class="r-stack">
:::{.element: class="fragment current-visible"}

![Example of PEpolar field map acquisition. Field mapping is done before/after an EPI run.](imgs/traditional_fmap.png)
:::
:::{.element: class="fragment current-visible"}

![In MEDIC, field maps are computed from the phase information of ME-EPI, which allows for measurement of the field frame-to-frame.](imgs/medic_fmap.png)
:::
</div>

## How does MEDIC work?

1. Correct phase offsets and phase wraps
2. Temporal Correction
3. Field Map Estimation and SVD Filtering
4. Invert Field Map to Undistorted Space

## How does MEDIC work?

1. Correct phase offsets and phase wraps<sup>[3,4]</sup>

<div class="r-stack">
:::{.element: class="fragment current-visible"}

![Wrapped Phase](imgs/phase3.png)
:::
:::{.element: class="fragment current-visible"}

![Unwrapped Phase](imgs/echo3.png)
:::
</div>

<p style="font-size: 10px;">[3] Eckstein K, Dymerska B, Bachrata B, Bogner W, Poljanc K, Trattnig S, Robinson SD. Computationally Efficient Combination of Multi-channel Phase Data From Multi-echo Acquisitions (ASPIRE). Magnetic Resonance in Medicine. 2018;79(6):2996–3006.</p>
<p style="font-size: 10px;">[4] Dymerska B, Eckstein K, Bachrata B, Siow B, Trattnig S, Shmueli K, Robinson SD. Phase unwrapping with a rapid opensource minimum spanning tree algorithm (ROMEO). Magnetic Resonance in Medicine. 2021;85(4):2294–2308.</p>

## How does MEDIC work?

2. Temporal Correction

<div class="r-stack">
:::{.element: class="fragment current-visible"}
<p>Find frames similar in head position (R = 0.98) and correct phase to nearest $2\pi$ multiple of average phase.</p>

![Example of unwrapped phase solutions settling on $2\pi$ offset away from other solutions.](imgs/pre_gmoc_timeseries.png)
:::
:::{.element: class="fragment current-visible"}
<p>Find frames similar in head position (R = 0.98) and correct phase to nearest $2\pi$ multiple of average phase.</p>

![Bad solution in question.](imgs/pre_gmoc_correction.png)
:::
:::{.element: class="fragment current-visible"}
<p>Find frames similar in head position (R = 0.98) and correct phase to nearest $2\pi$ multiple of average phase.</p>

![After temporal correction.](imgs/post_gmoc_correction.png)
:::
</div>

## How does MEDIC work?

3. Field Map Estimation and SVD Filtering

<div class="r-stack">
:::{.element: class="fragment current-visible"}
The field map is computed by using weighted least squares:

$$ W_{mag} \boldsymbol{\phi} = W_{mag} \gamma \Delta B_0 \mathbf{t} $$

where $W_{mag}$ is the magnitude at each echo.
:::
<div class="fragment current-visible">

Noisy temporal components are removed using SVD filtering:

<video width="720" height="480" controls loop>
<source src="videos/svd.mp4" type="video/mp4">
</video>
</div>
</div>

## How does MEDIC work?

4. Invert Field Map to Undistorted Space

Field maps computed on ME-EPI data are in the distorted space, so we must invert the field 
map to get it into the undistorted space:

$$y_{undistorted} = y_{distorted} - \Delta r(x, y_{distorted})$$

<div class="r-stack">
:::{.element: class="fragment current-visible"}

![Field map in distorted space.](imgs/fmap_native.png)
:::
:::{.element: class="fragment current-visible"}

![Field map in undistorted space.](imgs/fmap.png)
:::
</div>

## Acquisition Parameters and Datasets

- CMRR Multi-Echo EPI Sequence
  - TR: 1.761 s, TEs: 14.2, 38.93, 63.66, 88.39, 113.12 ms, 72 Slices, FOV: 110x110, Voxel Size: 2.0 mm, Multi-Band: 6
- Spin Echo PEpolar Field maps
  - TR: 8 s, TE: 66 ms, 72 Slices, FOV: 110x110, Voxel Size: 2.0mm
  - Field map solutions computed with FSL TOPUP
- T1w/T2w Anatomical Scans
  - T1w: Multiecho MPRAGE, TR: 2.5 s, TEs: 1.81, 3.6, 5.39, 7.18 ms, 208 Slices, FOV: 300x300, Voxel Size: 0.8 mm, Bandwidth: 745 Hz/px
  - T2w: T2 SPACE, TR: 3.2, TE: 565 ms, 176 Slices, Turbo Factor: 190, FOV: 256x256, Voxel Size: 1 mm, Bandwidth: 240 Hz/px

- Datasets (Resting State Only)
  - WashU Midnight Scan Club (MSC) (N=1)
  - UPenn preliminary data (N=1)
  - UMinn data (N=1)
  - ASD/ADHD Dataset (N=21)

- Comparisons between MEDIC and static PEpolar (TOPUP) correction.

## MEDIC can measure field changes due to head position.

<div class="r-stack">
<div class="fragment current-visible">
<video width="720" height="480" controls loop>
<source src="videos/rot-X.mp4" type="video/mp4">
</video>
<p>-X Rotation</p>
</div>
<div class="fragment current-visible">
<video width="720" height="480" controls loop>
<source src="videos/rot+X.mp4" type="video/mp4">
</video>
<p>+X Rotation</p>
</div>
<div class="fragment current-visible">
<video width="720" height="480" controls loop>
<source src="videos/rot-Y.mp4" type="video/mp4">
</video>
<p>-Y Rotation</p>
</div>
<div class="fragment current-visible">
<video width="720" height="480" controls loop>
<source src="videos/rot+Y.mp4" type="video/mp4">
</video>
<p>+Y Rotation</p>
</div>
<div class="fragment current-visible">
<video width="720" height="480" controls loop>
<source src="videos/rot-Z.mp4" type="video/mp4">
</video>
<p>-Z Rotation</p>
</div>
<div class="fragment current-visible">
<video width="720" height="480" controls loop>
<source src="videos/rot+Z.mp4" type="video/mp4">
</video>
<p>+Z Rotation</p>
</div>
<div class="fragment current-visible">
![Field Map Differences (Rotated Head Position - Neutral Position)](imgs/fieldmap_differences.png)
</div>
</div>

## MEDIC corrects data in the presence of head position changes.

![Functional Connectivity MEDIC vs. TOPUP vs. Grouth Truth](imgs/head_position_concat.png)

## MEDIC corrected data is more similar to group data than TOPUP corrected data.

![Similarity Comparison to ABCD Group Template](imgs/group_template_compare.png)

## MEDIC correction has greater correspondence to cortical anatomy than PEpolar correction

<div class="r-stack">
:::{.element: class="fragment current-visible"}

![UMinn: MEDIC corrected](imgs/anatomical_align/UMinn_medic.png)
:::
:::{.element: class="fragment current-visible"}

![UMinn: PEpolar (TOPUP) corrected](imgs/anatomical_align/UMinn_topup.png)
:::
:::{.element: class="fragment current-visible"}

![Penn MEDIC corrected](imgs/anatomical_align/UPenn_medic.png)
:::
:::{.element: class="fragment current-visible"}

![Penn PEpolar (TOPUP) corrected](imgs/anatomical_align/UPenn_topup.png)
:::
:::{.element: class="fragment current-visible"}

![MSC: MEDIC corrected](imgs/anatomical_align/MSCHD02_medic.png)
:::
:::{.element: class="fragment current-visible"}

![MSC: PEpolar (TOPUP) corrected](imgs/anatomical_align/MSCHD02_topup.png)
:::
:::{.element: class="fragment current-visible"}

![ASD/ADHD: MEDIC corrected](imgs/anatomical_align/washu2_medic.png)
:::
:::{.element: class="fragment current-visible"}

![ASD/ADHD: PEpolar (TOPUP) corrected](imgs/anatomical_align/washu2_topup.png)
:::
</div>

## MEDIC captures additional off-resonance effects.

![Field map Difference (MEDIC - TOPUP)](imgs/fieldmap_comparison.png)

### MEDIC captures additional off-resonance effects.

<div class="r-stack">
:::{.element: class="fragment current-visible"}

Comparison of ME-EPI data to GRE PEpolar field map shows additional off-resonance effects in ME-EPI data.

![Spin Echo PEpolar Image (UPenn)](imgs/se_upenn_fmap.png)
:::
:::{.element: class="fragment current-visible"}

Comparison of ME-EPI data to GRE PEpolar field map shows additional off-resonance effects in ME-EPI data.

![Gradient Echo PEpolar Image (UPenn)](imgs/gre_upenn_fmap.png)
:::
:::{.element: class="fragment current-visible"}

Comparison of ME-EPI data to GRE PEpolar field map shows additional off-resonance effects in ME-EPI data.

![ME-EPI Image, 1st Echo (UPenn)](imgs/me_epi_upenn_fmap.png)
:::
:::{.element: class="fragment current-visible"}

![Comparison of Field Map from PEpolar EPI (TOPUP), FLASH (ROMEO), and ME-EPI (MEDIC) data.](imgs/fieldmap_compare.png)
:::
</div>

## MEDIC correction produces greater correspondence to anatomy in both global and local alignment metrics.

![Global and local anatomical alignment metrics](imgs/alignment_metrics.png)

## MEDIC correction produces greater correspondence to anatomy in both global and local alignment metrics.

<div class="r-stack">
:::{.element: class="fragment current-visible"}

Global alignment metrics (whole brain) for ASD/ADHD Dataset. ✅ indicates best metric that was
statistically significant (p < 0.05). Correlation metric is the squared Pearson correlation between
the functional and anatomical data. Grad. Corr. is the Pearson correlation between the gradient
of the functional and anatomical data. Norm. MI is the normalized mutual information between the
functional and anatomical data.

|     Metric      |        MEDIC        |     PEpolar     | t-stat | p-value |  df  |
|:---------------:|:-------------------:|:---------------:|:------:|:-------:|:----:|
|   T1w Corr.^2   | ✅**0.062 (0.029)** |  0.060 (0.028)  | 5.259  | < 0.001 |  184 |
|   T2w Corr.^2   | ✅**0.456 (0.053)** |  0.453 (0.057)  | 2.374  |  0.019  |  184 |
| T1w Grad. Corr. | ✅**0.430 (0.028)** |  0.414 (0.036)  | 11.473 | < 0.001 |  184 |
| T2w Grad. Corr. |     0.637 (0.039)   |  0.638 (0.054)  | -0.728 |  0.467  |  184 |
|  T1w Norm. MI   |     0.873 (0.029)   |  0.872 (0.029)  | 0.796  |  0.427  |  184 |
|  T2w Norm. MI   |     0.837 (0.026)   |  0.838 (0.026)  | -0.885 |  0.377  |  184 |

:::
:::{.element: class="fragment current-visible"}

Local alignment metrics (whole brain) for ASD/ADHD Dataset. ✅ indicates best metric that was
statistically significant (p < 0.05). Spotlight analysis was performed by computing the average squared
local correlation between the functional and anatomical images across all voxels within a 3 voxel radius sphere.

|       Metric        |        MEDIC        |     TOPUP     | t-stat | p-value |  df  |
|:-------------------:|:-------------------:|:-------------:|:------:|:-------:|:----:|
|  T1w Spotlight R^2  | ✅**0.077 (0.007)** | 0.075 (0.009) |  6.528 | < 0.001 |  184 |
|  T2w Spotlight R^2  | ✅**0.098 (0.011)** | 0.097 (0.014) |  2.365 |  0.019  |  184 |

:::
:::{.element: class="fragment current-visible"}

Segmentation based metrics for ASD/ADHD Dataset. ✅ indicates best metric that was
statistically significant (p < 0.05). Each metric examined the separability of tissue
types using anatomical segmentation labels applied to functional data. Separability was
measured by thresholding the functional data along the tissue boundary, comparing the
classification against the anatomical segmenation, and computing the AUC of the resulting
ROC curve.

|     Metric      |        MEDIC        |     TOPUP     | t-stat | p-value |  df  |
|:---------------:|:-------------------:|:-------------:|:------:|:-------:|:----:|
|   Gray/White    | ✅**0.691 (0.031)** | 0.686 (0.036) |  6.307 | < 0.001 |  184 |
| Brain/Exterior  | ✅**0.736 (0.035)** | 0.729 (0.034) | 11.982 | < 0.001 |  184 |
| Ventricle/White |    0.828 (0.057)    | 0.829 (0.062) | -0.273 |  0.785  |  184 |

:::
</div>

<!-- ## <span style="font-size: 1.5rem;">MEDIC frame-to-frame correction does not add significant temporal variation compared to static correction.</span>

<div class="r-stack">
:::{.element: class="fragment current-visible"}

![tSNR comparison between TOPUP and MEDIC (MSC).](imgs/tSNR.png){ width=60% }
:::
:::{.element: class="fragment current-visible"}

In ASD/ADHD dataset, tSNR was not significantly different between MEDIC and TOPUP corrected data.

|     Metric      |        MEDIC        |     TOPUP       | t-stat | p-value |  df  |
|:---------------:|:-------------------:|:---------------:|:------:|:-------:|:----:|
|      tSNR       |   38.993 (16.130)   | 39.345 (16.166) | -1.747 |  0.082  |  184 |

:::
</div> -->

## Conclusions

- MEDIC provides superior distortion correction performance over PEpolar (i.e. TOPUP) method
  - Likely due to additional correction of eddy-currents

- MEDIC field maps are coupled in space and time to the same EPI data it is correcting.
  - No need for co-registration of field map to EPI data
  - Removes separate sequence for field map acquisition
  - Can account for field changes due to head motion

## Extras

## MEDIC corrects repiration effects in motion parameters.

<div class="r-stack">
:::{.element: class="fragment current-visible"}
![Motion Parameters Pre/Post MEDIC Correction](imgs/motion_traces.png)
:::
:::{.element: class="fragment current-visible"}
![Power Spectral Density of Motion Parameters Pre/Post MEDIC Correction](imgs/power_spectra.png)
:::
:::{.element: class="fragment current-visible"}
![Framewise Displacement Pre/Post MEDIC Correction](imgs/FD.png)
:::
</div>

## Signal crosstalk in field map correlation?

<div class="r-stack">
:::{.element: class="fragment current-visible"}
![Seed Correlation, 72 Slice Interleave (Top) and Ascending (Bottom)](imgs/interleave_72_and_ascending.png)
:::
:::{.element: class="fragment current-visible"}
![Seed Correlation, 78 Slice Interleave](imgs/interleave_78.png)
:::
</div>
