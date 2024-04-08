# The DifCal Tab

The `DifCal` tab provides access to [the diffraction calibration workflow](label_difcal). At certain points in the workflow, user interaction is needed and this is facilitated by four different tabs in the interface. The workflows are linear, beginning on the first tab and transiting one tab at a time through the entire workflow. The `Cancel` button can be clicked at any time and will reinitialise the interface (deleting any existing workspaces).

## First tab: Diffraction Calibration

The first tab in the panel allows the configuration of the calibration, requiring a set of inputs specified either via text fields or drop down menus. All inputs must have valid values in order to progress to the next tab, which is done by pressing `Continue`.

```{figure} static/difcal1.png
---
height: 350px
name: difcal1
---
The first tab in the difCal workflow used to configure the initial calibration
```
### Explanation of fields

**Run Number:** This is the run number of a dataset from a crystalline calibrant sample. Value must be >= 46342

**Lite Mode:** This toggle enables or disables [Lite Mode](label_lite) (default is `ON`)

**Convergence Threshold:** This is an input to the [`CrossCorrelation algorithm`](label_CC). The algorithm is set up to reexecute iteratively, determining and then applying the offset correction in each iteration. This should result in smaller and smaller offsets. This will continue until the average offset (measured in number of logarithmic bins) becomes less than the requested `Convergence Threshold`, or when a define maximum number of cycles is reached (default is 5, can be adjusted via the parameter `maximumIterations` in `application.yml`).

**Peak Intensity Threshold:** This allows user specification of the minimum (estimated) intensity of Bragg peaks to use as a fraction of the intensity of the strongest peak. Setting to 0.0 will include all peaks. The effect of this parameter can be inspected on the next tab where it can be adjusted.

```{note}
In testing, it was noted that (as a result of the huge $F_{obs}^2$ of the diamond 111 reflection) a value of 0 works best for this calibrant. In contrast, 0.01 is found to be a good value for silicon, but this needs to be fine tuned as it depends on the counting statistics of the measurement, with longer collections allowing the inclusion of weaker peaks. 
```

**Bins Across Peak Width:** This parameter provides _global control_ of the output binning and works by allowing user specification of the number of (logarithmic) bins across the FWHM of any given Bragg peak, at native instrument resolution for a given state as described [here](binning_considerations).

```{note}
The intension is to make `Bins Across Peak Width` a fixed number for all data reduction. However, this parameter has been exposed to enable testing. The final value should be sufficient to allow Rietveld refinement of GSAS Profile Function 3 (or equivalent). The possiblity to _down sample_ from this native resolution will also be supported, however, the default binning parameter should reflect the native resolution of the instrument.
``` 
**Sample:** This allows user selection of the available diffraction calibrants.

**Grouping File:** This allows user selection of the available pixel grouping schemes that can be used for calibration. This is a fundamental input to the diffraction calibration as both the pixel calibration _and_ the group calibration are conducted group by group. The available groups consist of the "standard" detector-component based groupings (e.g. `Bank`, `Column`...) however it's also possible to use arbitrary pixel grouping schemes, specified for a given instrument state. The latter allows the use of 2$\theta$-based grouping, which may give better quality calibrations, particularly for states where detectors are at extreme angles.

**Peak Function:** This allows selection of the peak shape to use during group calibration. At present, only symmetric peak shapes are available. The best choice varies for different calibrants (e.g. the SNAP diamond calibrant exhibits a strong Lorentzian profile). The current plan would be to ultimately transition to a more realistic TOF peak shape, but this requires modification of the underlying mantid algorithm `PDCalibration` to allow constraints including the fixing of global values of the exponential α and β terms. (NOTE: the `psuedovoigt` peakshape appears to diverge most of the time).

## Second tab: Tweak Peak Peek

```{figure} static/difcal2.png
---
height: 400px
name: difcal2
___
The "Peak Peek" tab, allows fine tuning of peak selection and corresponding peak fitting extents
```

The `Tweak Peak Peek` tab allows visual inspection of the diffraction focused data (using the calibration pixel grouping scheme) enabling configuration of the subsequent [Group Calibration](group_cal) part of the algorithm. Each spectrum is associated with a list of calibrant bragg peak positions. This list is derived from the calibrant cif file and can be filtered by user specification of d-spacing limits (and indicated by vertyical red lines on each spectrum) and `intensity threshold`. Each peak has an associated "peak extent", which delimits left and right hand boundaries that will be used for fitting. Since the underlying mantid algorithm `PDCalibration` is based on single peak fitting, peaks with overlapping extents are excluded.

(n.b. In order to apply any modified parameters, it's necessary to press the `Recalculate` button)

When the `Tweak Peak Peek` view is generated, an initial round of peak fitting is conducted and the resulting fits are overlayed across the diffraction data for each peak. "Good" fits - those that yield a $\chi^2$ value less than a threshold (default is 100, this can be adjusted using the parameter `maxChiSq` in `application.yml`) - are coloured purple/blue and those that are bad coloured red . The goal is to achieve as many good peaks as possible. In {ref}`Figure <difcal2>` several issues are immediately noticed. Firstly, the peak extent of the large peaks is clearly not sufficient to capture the long (Lorentzian) tails of the diamond peak shape. Secondly, many of the lower d-spacing peaks are coloured red indicating poor quality fits.

It is possible to zoom in and inspect details in any of the panels. Doing this in Group (ID: 2) ({ref}`Figure <difcal2_zoom>`) shows that, not only are the peak extents too narrow, but the peaks in the focused calibration data are not well aligned with these. This is indicative of error in the uncalibrated `DIFC` values.  

```{figure} static/difcal2_zoom.png
---
height: 400px
name: difcal2_zoom
___
When zooming it, it is evident that the peak extents are not well aligned with the input data, reflecting the error in the uncalibrated values. Where this leads to a poor fit, those peaks are coloured red.
```

At this point, it's possible for users to fine tune the peak extents. As described [here](group_cal), the (native resolution) FWHM of each peak is custom calculated for each peak and the symmetric part of the peak extent is governed by applying an independent multiplier for the left and right side of the peaks: `FWHM Left` and `FWHM Right`.

In this case, by adjusting these to `FWHM Left = 3` and `FWHM Right=5` it was possible to make almost every peak turn purple/blue indicating a good quality of fit. (n.b. Again, to apply any modified parameters, it's necessary to press the `Recalculate` button)

```{figure} static/difcal2_zoom2.png
---
height: 400px
name: difcal2_zoom2
___
After adjusting the peak extents all peaks are now successfully fitted, without loss of peaks due to extent overlap.
```

```{note}
At present, `SNAPRed` does not allow the possibility to use a pre-existing calibration for a given state to be used as a starting point for a subsequent calibration. Currently, this means that the starting point for calibration is instead the idealised detector locations defined in the instrument definition file.   
```

```{note}
It might be useful to have sample-dependent peak profile parameters in addition to the current instrument-dependent peak profile. These could follow GSAS convention {cite:p}`GSAS` and could be stored in as parameters in the calibrant definition files. This would allow easy management of calibrants where sample-dependent broadening gives peaks that are significantly wider than the instrument resolution. 
```

```{note}
In testing it was found that for some long data collections the random error due to counting statistics becomes very small relative to the systematic error due to the peak shape. This can lead to $\chi^2$ in excess of 1000. In this case, it may be necessary to increase the $\chi^2$ threshold to allow the peaks to pass. As noted above, a longer term plan is to improve the peak shape used.  
```

When the peak fitting configuration has been optimised, the `Continue` button should be used to initiate the Group Calibration component of the diffraction calibration workflow.  

## Third tab: Assessing

This tab allows the user to load and pre-existing calibrations for the state to allow comparison with the present calibrations

## Fourth tab: Saving


```{figure} static/difcal_save.png
---
height: 400px
name: difcal_save
___
The saving tab.
```

Once the calibration is deemed complete. The user has to input some information to persist the output to disk. The only two mandatory inputs are `Comments` (used to record any useful observations during calibration or at least a statement like "OK") and `Author` (the person who conducted the calibration). In addition, if multiple iterations were attempted during the process, the user must select which of these to persist.

Another useful field is the `Applies To:` field. This tells the software which run number the calibration applies to. While it defaults to the present calibration run, it is possible to specify a different run. A logical operator is expected to precede the number entered e.g. `>=12345` with `>=` indicating that runs equal to or greater than run number 12345 will use this calibration. This information is written to the calibration index for the state, which is used during reduction to identify the correct calibration files and parameters.



