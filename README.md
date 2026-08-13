# Major-Axis Fluorescence Intensity Analysis

**Code Developed by Dora `<Shiyue Liu>`
MATLAB code for measuring **width-averaged fluorescence intensity along the curved major axis** of elongated objects (e.g. organoids, spheroid chains, elongated cells) in a 2-D microscopy image.

For each user-drawn region of interest (ROI), the software traces the object's medial axis, extends it to the object's true tips, and reports the mean image intensity over a cross-section perpendicular to that axis at every point along it. The output is a one-dimensional profile of intensity versus distance along the object.

---

## 1. System requirements

### Software dependencies and operating systems

| Component | Requirement |
|---|---|
| Image Processing Toolbox | **Required** — `bwskel`, `bwmorph`, `bwdist`, `imfreehand`, `createMask`, `improfile`, `imshow` |
| Statistics and Machine Learning Toolbox | **Required** — `pdist`, `squareform`, `nanmean` |
| Operating system | Any platform supported by MATLAB: Windows 10/11, macOS 11 or newer, Linux (e.g. Ubuntu 20.04/22.04). The code is pure MATLAB, with no compiled components and no platform-specific calls. |

A graphical display is required, because the regions of interest are drawn interactively.

> `imfreehand` is still available in current MATLAB releases but is no longer recommended by MathWorks (its replacement is `drawfreehand`). It may be removed in a future release.

### Versions the software has been tested on 
Image Processing Toolbox  Version 24.2 (R2024b)
-----------------------------------------------------------------------------------------
MATLAB Version: (R2024b) 
| MATLAB release | Operating system | Status |
| R2024b | Windows 11 (64-bit) | Tested |
| R2024b | macOS 14 (Apple Silicon) | Tested |

### Required non-standard hardware

None. The code runs on a standard desktop or laptop computer. No GPU, cluster, or specialised acquisition hardware is required; approximately 4 GB of RAM is sufficient for images up to 2048 × 2048 pixels.
---

## 2. Installation guide

### Instructions

1. Download or clone this repository:

   ```bash
   git clone <https://github.com/doraliusy/Major-axis-fluorescence-intensity-analysis>        
   ```

   or download the ZIP archive and extract it.

2. Start MATLAB and change to the folder containing the three `.m` files:

   ```matlab
   cd /path/to/this/folder
   ```

That is the whole installation. `Fluoresent_Intensity_Calculation.m` calls `addpath(genpath(pwd))` on its first line, so the two helper functions are found automatically as long as MATLAB's current folder is this one.
### Typical install time on a "normal" desktop computer

**Under 1 minute** (the time to copy the files). This excludes installing MATLAB itself.

---

## 3. Demo

### Instructions to run on the demo data

1. Put the demo image in this folder under the name `organoid.png`.
  
2. In MATLAB, with this folder as the current folder, run:

   ```matlab
   Fluoresent_Intensity_Calculation
   ```

3. A window opens showing the image and a message box explaining the
   interaction. Then **trace one object by holding down the left
   mouse button** and releasing when the contour is complete.
4. Answer **"Keep this ROI?"** — choose *No* to discard the trace and draw it
   again, *Yes* to accept it.
5. Answer **"Draw another ROI?"** — choose *Yes* to add a further object,
   *No* to finish and start the analysis.

### Expected output

* **"Curved major axis extended to true tips"** — the input image with a red
  curve drawn along each ROI, running tip to tip through the object. This is
  the main quality-control view: each curve should follow the long direction
  of its object and reach both ends.
* **"Cross-sectional mean intensity"** — one curve per ROI, mean intensity
  (arbitrary units) against distance along the axis in pixels.
* **Two further figures per ROI** — the binary mask that was drawn, and its
  intensity profile plotted against sample index.
* In the MATLAB workspace, `means{i}` holds the intensity profile of ROI *i*
  and `positions{i}` the matching distance along the axis, in pixels. Export
  them with, for example:

  ```matlab
  writematrix([positions{1}, means{1}], 'roi01_profile.csv');
  ```

### Expected run time for the demo on a "normal" desktop computer

| Step | Typical time |
|---|---|
| Drawing 2 ROIs by hand | 15–30 s of user interaction |
| Analysis of 2 ROIs of ~200 px length in a 1300 × 1300 image | **< 30 s** |
| Figure rendering | < 2 s |

---

## 4. Instructions for use

### How to run the software on your data

1. Copy your image into this folder.
2. Open `Fluoresent_Intensity_Calculation.m` and edit one line so it names
   your file:

   ```matlab
   I = imread('organoid.png');      % <- your image here
   ```

   Any format `imread` supports may be used (PNG, TIFF, JPEG …). RGB images
   are converted to grayscale automatically; multi-channel stacks should be
   split beforehand and analysed one channel at a time.
3. Optionally adjust the averaging width on the `w` line:

   ```matlab
   w = 1000;    % upper bound on the HALF-width of the cross-section, in px
   ```

   The half-width actually used at each point is `min(w, distance from that
   point to the ROI boundary)`. A deliberately large value therefore means
   *"average over the full local width of the object"*, which is the intended
   default. Set `w` to a small number (e.g. `10`) to restrict the averaging
   to a narrow band around the axis.
4. Run the script and draw the ROIs as described in the Demo section above.

### File overview

| File | Role |
|---|---|
| `Fluoresent_Intensity_Calculation.m` | Top-level script: loads the image, collects ROIs, runs the analysis, plots the results. Start here. |
| `MaskROI_multiple.m` | Interactive freehand ROI selection; returns one logical mask per region. |
| `Majorline_intensity.m` | Core analysis: traces each ROI's curved major axis and computes the width-averaged intensity profile along it. |

### How to draw good ROIs

* Enclose **one connected, elongated object per ROI**. The analysis extracts
  a single tip-to-tip axis per region, so a region containing two separate
  objects has no well-defined major axis.
* Leave **a small margin of ROI around both tips** of the object. The axis is
  extrapolated beyond the ends of the skeleton until it exits the ROI, and
  that step assumes there is some region left to extend into.
* Avoid very round regions. An object whose skeleton has fewer than two free
  ends (a disc, or a closed ring) has no major axis; such a region is
  reported with the warning `ROI n has fewer than 2 skeleton endpoints` and
  is skipped.
* If any ROI is skipped, note that the two summary figures at the end of
  `Majorline_intensity` plot the regions up to the last one processed;
  inspect `intensityPlots` and `arclengths` directly if a region was skipped.


**Tip for exact reproducibility:** hand-drawn ROIs cannot be reproduced by
another user. To make an analysis fully repeatable, save the masks the first
time and re-use them afterwards:

```matlab
% once, after drawing
save('myImage_rois.mat', 'roiMasks');

% on every later run, in place of the MaskROI_multiple call
load('myImage_rois.mat', 'roiMasks');
[I2, means, positions] = Majorline_intensity(I, w, roiMasks);
```

---



## 5. Repository contents

```
.
├── README.md                            this file
├── Fluoresent_Intensity_Calculation.m   top-level script (start here)
├── MaskROI_multiple.m                   interactive ROI selection
├── Majorline_intensity.m                major-axis intensity analysis
└── organoid.png                         demo image  <-- ADD BEFORE SUBMISSION
```

---

## 6. Licence

Released under the **MIT Licence** (an [Open Source Initiative](https://opensource.org/licenses)–approved licence). See [`LICENSE`](LICENSE).

**Code repository:** `<https://github.com/doraliusy/Major-axis-fluorescence-intensity-analysis>` 

## 7. Authors and contact

This code was developed by **Shiyue Liu.


Questions about the code should be addressed to `<doraliusy@gmail.com>`.
