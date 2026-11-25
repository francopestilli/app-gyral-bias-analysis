# Gyral Bias Analysis (app-gyral-bias-analysis)

<!-- brainlife-app:begin (generated from the App manifest; edit the manifest, not this block) -->

> **Computes parcel-wise gyral bias metrics by combining FreeSurfer cortical curvature with tractography streamline counts and densities, optionally restricted to visual areas and polar-angle meridians.**

Projects FreeSurfer surface curvature to the volume with mri_surf2vol, computes mean curvature and volume per parcel, counts the streamlines of each parcel's tractogram (track<ID>.tck), optionally keeps only streamlines connecting visual_area_a and visual_area_b, and reports streamline density per parcel with summary plots by meridian. Use it to test whether streamline terminations are biased towards gyral crowns. Needs parc (with label), tcks and the FreeSurfer directory; parc_vareas is required when visual area filtering is requested. Note: input_mode and normalize, and the tractogram and prf inputs, are not read by main.

| | |
|---|---|
| 📥 **Inputs** | `neuro/freesurfer` · `neuro/tcks` (optional) · `neuro/parcellation/volume` (optional) ×2 · `neuro/track/tck` (optional) · `neuro/prf` (optional) |
| 📤 **Outputs** | `neuro/tractmeasures` · `generic/images` |
| 🐳 **Container** | `gamorosino/tract_align:v1.0` |
| ⚙️ **Resources** | 1 CPUs · 3 h walltime |
| 🔑 **Requires** | `FREESURFER_LICENSE` |
| 🔗 **Run it** | [doi.org/10.25663/brainlife.app.914](https://doi.org/10.25663/brainlife.app.914) |

#### 📥 Inputs

| Input | Datatype | Required | Description |
|---|---|---|---|
| `freesurfer` | `neuro/freesurfer` | yes | FreeSurfer recon-all subject directory (curvature and aparc+aseg are used). |
| `tcks` | `neuro/tcks` | no | Collection of per-parcel tractograms named track<ID>.tck. |
| `parc_vareas` | `neuro/parcellation/volume` | no | Visual area parcellation (for example Benson14 varea) used to filter streamlines by visual area. |
| `parc` | `neuro/parcellation/volume` | no | Parcellation whose parcels the metrics are computed for, with its label file. |
| `tractogram` | `neuro/track/tck` | no |  |
| `prf` | `neuro/prf` | no |  |

#### 📤 Outputs

| Output | Datatype | Description |
|---|---|---|
| `output` | `neuro/tractmeasures` | Table (tractmeasures.csv) of per-parcel curvature, volume, streamline counts and densities. |
| `plots` | `generic/images` | Summary plots of curvature and streamline density by meridian and eccentricity. |

<details>
<summary><b>⚙️ Parameters</b> (5)</summary>

| Parameter | Type | Default | Description |
|---|---|---|---|
| `visual_area_a` | string | `V1` | First visual area name (for example V1) used to filter streamlines. |
| `visual_area_b` | string |  | Second visual area name; when set, only streamlines connecting area A and area B are kept. |
| `meridian_mode` | string | `HM, VM, LVM, UVM` | Which polar-angle meridian groups to compare in the plots (for example HM, VM, LVM, UVM). |
| `input_mode` | enum | `precomputed_tcks` | Options: `precomputed_tcks`, `whole_tractogram`. |
| `normalize` | boolean | `false` |  |

</details>

<!-- brainlife-app:end -->

This app computes parcel-wise gyral bias metrics by combining cortical curvature (from FreeSurfer) with tractography-derived streamline counts and densities. Optional filtering allows analysis restricted to specific visual areas and ROI ordering.

---

## Summary

For each parcel, the app computes:

* Mean cortical curvature
* Parcel volume and voxel count
* Total streamline count
* Filtered streamline count (optional)
* Streamline density (filtered / volume)
---
## Author

Gabriele Amorosino (g.amorosino@gmail.com)

---

## Inputs

| Name     | Type                 | Description                                |
| -------- | -------------------- | ------------------------------------------ |
| `parc`   | `neuro/parcellation` | Parcel segmentation (NIfTI)                |
| `label`  | `json`               | Label mapping (parcel ID → name)           |
| `tcks`   | `track/tck` (dir)    | Directory of tractograms (`track<ID>.tck`) |
| `output` | `neuro/freesurfer`   | FreeSurfer subject directory               |

---

## Optional Inputs

| Name          | Type                 | Description                           |
| ------------- | -------------------- | ------------------------------------- |
| `parc_vareas` | `neuro/parcellation` | Visual area segmentation              |
| `lh_curv`     | `neuro/mgz`          | Left hemisphere curvature (override)  |
| `rh_curv`     | `neuro/mgz`          | Right hemisphere curvature (override) |

If curvature files are not provided, they are generated from FreeSurfer using `mri_surf2vol`.

---

## Parameters

| Parameter       | Type   | Description                           |
| --------------- | ------ | ------------------------------------- |
| `visual_area_a` | string | First visual area (e.g., `V1`)        |
| `visual_area_b` | string | Second visual area                    |
| `roi_order`     | bool   | Enforce A→B / B→A streamline ordering |

---

## Outputs

| Name                        | Type        | Description                                  |
| --------------------------- | ----------- | -------------------------------------------- |
| `curvature_streamlines.csv` | `table/csv` | Parcel-wise curvature and streamline metrics |

---

## Output Fields

* `parcel_id`
* `parcel_name`
* `hemi` (lh/rh)
* `voxel_count`
* `parcel_volume`
* `mean_curvature`
* `streamline_count_total`
* `streamline_count_filtered`
* `streamline_density_filtered`
* `visual_area_a`
* `visual_area_b`
* `roi_order`

---

## Processing Steps

1. Convert FreeSurfer surface curvature to volume (`mri_surf2vol`)
2. Split parcel segmentation into binary masks
3. Compute mean curvature per parcel
4. Load corresponding tractograms (`track<ID>.tck`)
5. Optionally filter streamlines:

   * Area A only
   * A ↔ B (unordered)
   * A → B / B → A (ordered endpoints)
6. Compute streamline density:

   ```
   density = filtered_count / parcel_volume
   ```
7. Write results to CSV

---

## Filtering Logic

| Mode                | Behavior                            |
| ------------------- | ----------------------------------- |
| None                | All streamlines used                |
| A only              | Keep streamlines touching area A    |
| A + B               | Keep streamlines connecting A and B |
| A + B + `roi_order` | Keep A→B or B→A based on endpoints  |

---

## Assumptions

* Each parcel has a corresponding tractogram:

  ```
  track<ID>.tck
  ```
* Parcel IDs match tractogram filenames
* FreeSurfer directory contains:

  * `surf/`
  * `mri/aparc+aseg.mgz`

---

## Example `config.json`

```json
{
  "parc": "parc.nii.gz",
  "label": "labels.json",
  "tcks": "tcks/",
  "output": "freesurfer_subject",

  "visual_area_a": "V1",
  "visual_area_b": "V2",
  "roi_order": true,
  "parc_vareas": "parc_vareas.nii.gz"
}
```

---

## Dependencies

* Fischl, B. (2012). FreeSurfer. Neuroimage, 62(2), 774-781.
* Tournier, J. D., et al. (2019). MRtrix3: A fast, flexible and open software framework for medical image processing and visualisation. Neuroimage, 202, 116137.
* Garyfallidis, E., et al. (2014). Dipy, a library for the analysis of diffusion MRI data. Frontiers in neuroinformatics, 8, 8.
