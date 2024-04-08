# The NormCal Tab

The `NormCal` tab provides access to [the normalisation calibration workflow](normCal). At certain points in the workflow, user interaction is needed and this is facilitated by four different tabs in the interface. The workflows are linear, beginning on the first tab and transiting one tab at a time through the entire workflow. The `Cancel` button can be clicked at any time and will reinitialise the interface (deleting any existing workspaces).

## First tab: Diffraction Calibration

The first tab in the panel allows the configuration of the calibration, requiring a set of inputs specified either via text fields or drop down menus. All inputs must have valid values in order to progress to the next tab, which is done by pressing `Continue`.

```{figure} static/normcal1.png
---
height: 350px
name: normcal1
---
The first tab in the normCal workflow used to configure the initial calibration
```
### Explanation of fields

**Run Number**: This is the run number of a dataset from a incoherent calibrant sample (typically vanadium or vanadium-niobium). Value must be >= 46342

**Background Run Number**: This is the run number of an equivalent empty instrument measurement for the incoherent sample. It is mandatory that the background is measured from the same state as the incoherent calibrant and steps should be taken to insure that all other details of the measurement match (e.g. slit settings or any manually applied upstream collimation)  

**Lite Mode*: This toggle enables or disables [Lite Mode](label_lite) (default is `ON`) 

**Select Sample**: This allows user selection of the incoherent calibrant used. Each calibrant has its own unique ID allowing, for example, different geometries of vanadium to be used.

**Select Grouping File**: While this selection is not used to generate the raw vanadium output, it is used to inspect the diffraction focused vanadium. Focusing according to this group is necessary to configure the peak stripping and smoothing parameters (see [here](normCal))

### The raw vanadium correction

When the user presses the continue button, the algorithm will calculate the raw vanadium correction. 

```{warning}
The calculation of the raw vanadium correction includes the calculation of an absorption correction, which can be quite time consuming to execute. Notably, since the calculation runs over each pixel it is much faster for `Lite` mode versus `Native` mode. Typical numbers are ~ 60s for `Lite` and ~1 hour for `Native`.
```
```{note}
Vanadium measurements can often contain a very large number of events, leading to ~10's Gb file sizes. These are both slow to load and, slow to operate on. In Phase 3, event management (including logarithmic event compression) should greatly accelerate the entire workflow.
```    

## Second tab: Tweak Parameters

After calculation of the "raw vanadium correction" is complete, the result is diffraction focused to allow inspection and optimization of peak removal and smoothing of the pattern

```{figure} static/normcal2.png
---
height: 350px
name: normcal2
---
This tab provides a view of the diffraction focused raw vanadium correction. This is what function that will be used to normalise data during reduction
```

Similar to the "tweak peek" view in the `DifCal` workflow, a grid of diffraction focused datasets is shown depending on the selected grouping scheme. The grouping scheme can be changed by selecting from the drop down menu and pressing `Recalculate`. The expected peak positions and extents are shown as coloured blue/purple regions. These can be modified via the `dMin`, `dMax` and `intensity threshold` parameters in the same way as the "tweak peek" tab. Any data _within the blue regions_ is ignored when smoothing is calculated. 

```{note}
if the incoherent calibrant is vanadium-niobium no peak regions will appear as the calculated Bragg intensity is zero.
```

The dashed orange line is the smoothed correction. The level of smoothing is adjusted by varying the `Smoothing` slider and hitting `Recalculate`. The goal here is to optimise smoothing so that the functional form of the normalisation correction is faithfully followed, but statical noise is removed. In {ref}`Figure <normcal2>`, the default level of smoothing is clearly observed to be to aggresive, such that the smoothed data do not closely follow the measured data. 

```{figure} static/normcal2_after.png
---
height: 350px
name: normcal2_after
---
After reducing the smoothing parameter, a much better fit to the data is observed. Here, the zoom view on `Group ID=5` shows the smoothed form accurately traced the functional form while removing high-frequency statistical noise.
```

In addition to optimising the smoothing parameters, the ends of the d-range of each diffraction focused normalisation correction should be expected. These should terminate where the normalisation correction is positive and not too small. Since the diffraction focused normalisation correction is applied via division any present very small values will result in an ill-conditioned correction.

## Third tab:

The final tab facilitates saving of the raw vanadium correction, updating of a calibration index (associating which runs this correction applies to) and writing of a calibration record. The calibration record captures all parameters used during the workflow allowing these to be used during subsequent data reduction.

### Explanation of fields

**Version:** This probably should be removed. It is copy pasted from the `DifCal` view, which has an iterative workflow allowing for multiple iterations ("versions") to exist of a given calibration operations.

**Applies to:** This allows the specification via logical operators `>` or `>=` and a run number (e.g. `>12345`) of which sample runs this correction should be applied to. The default is that it shall apply to any run-number of greater than that of the vanadium incoherent calibration dataset itself. (n.b. this facilitates the conduction of a calibration _after_ the measurement of a sample dataset)

**Comments:** (Mandatory) a free text field to comment on the conditions or other observations relating to the calibration

**Author:** (Mandatory) a free text field to record who conducted the calibration (normally an instrument scientist)

### Completion

After filling out the fields on the final tab and pressing `Continue` the output data will be written to disk and `SNAPRed` will reset so it's ready to conduct another calibration 