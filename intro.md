# SNAPRed User Docs v 0.2

This is a draft set of documentation for Phase 2 release of SNAPRed.

## What is SNAPRed

SNAPRed is a computational tool that supports instrument calibration and data reduction for the SNAP instrument on Beamline 3 of the Spallation Neutron Source, Oak Ridge National Laboratory.

SNAPRed is built on the [mantid code base](https://www.mantidproject.org/) and uses many of its algorithms and data visualisation widgets.

The current release on SNAPRed is in **Phase 2** of the project
	
## What can Phase 2 do? 

Phase 2 can: 

1. Execute calibration of the SNAP instrument. This calibration includes two processes: 

* "Diffraction Calibration", the production of optimized diffractometer constants that allow the conversion of TOF to crystallographic units of d-spacing (d) or momentum transfer (Q).

* "Normalisation Calibration", the production of a function that can be used to normalise diffraction data for wavelength ($\lambda$)-dependent neutron flux and instrumental artefacts. 

2. Support auntitative diagnostics of quality of calibration, specifically with respect to accuracy of the location of the extracted Bragg reflections and sharpness of diffraction peaks.
3. Save a calibration record that stores the entirity of the calibration process along with metrics capturing the calibration quality. 
4. Manage instrument configurations ("States") including:

*Identify if a specific run has an associated state already calibrated
* Creation of states on the basis of an input run number corresponding to a specific instrument configuration. 
* Automatic configuration of the calibration workflow on the basis of state properties (e.g. diffraction resolution, minimum or maximum d-spacing etc.).
* Saving and indexing of calibration output according to state, such that subsequent data reduction can locate the correct calibration parameters and data (NOTE: does this mean the calibration data for example for masking, or actual run data? I think it is ambiguous). 

5. Support the definition of unique calibrant materials with specific, customisable properties (including material, geometry and crystallographic information)


## What can Phase 2 not do?

The planned scope for SNAPRed that Phase 2 does not include: 

1. Performant management of high event rate data. 
2. Execution of powder diffraction data reduction.
3. Management of complex sample container effects, such as attenuation correction or diamond peak masking.
4. a GUI 

To emphasisize point 4: SNAPRed primarily exists as a "Backend". To facilitate testing, a "Test Panel" is provided. This is a minimalist GUI and only provides limited interactivity.

## How do I run SNAPRed?

SNAPRed Phase 2 can be invoked from within a session on `analysis.sns.gov` by opening a terminal and typing:

```
mantidworkbench --env snapred-qa
```
## Specific planned features for Phase 2 not yet built and known defects that will be fixed

1. Apply a calibration to new/different data (run). This is an integral part of the reduction phase, but will also allow a _previous_ calibration to be a starting point for a _subsequent_ calibration. (EWM 3143) 

2. Iterative calibration. It should be possible to iteratively run a calibration, each iteration improving on the former. Although there's an "iterate" button, at the moment, the preceding calibration is not applied to the data prior to execution, effectively reseting the calibration to the instrument defaults. This will be enabled in the future (with an option to "reset" a calibration retained if necessary)[NOTE: does "reset" mean "revert"? also does this mean "with an option to overwrite a previous calibration"? ]

3. Applying an _apriori_ detector mask to a calibration sequence. It is anticipated that the presence of sample environment, detector errors etc may lead to the need to apply a detector mask _before_ calibration begins. This option will be included in the future. (EWM 2866)

4. TOF Peak shapes. Calibration includes the fitting of mathematical functions describing diffraction peaks to the measured diffraction data. At the moment, the only avilable functions describe symmetric peaks (Gaussian, Lorentzian and PsuedoVoigt), however, TOF neutron data are known to have more complex, asymmetric peaks that require more sophisticated mathematical models. This possiblity continues to be investigated.  As a related note, it is known that symmetric peaks shapes give particularly poor fits to LaB<sub>6</sub> data and other samples comprising "large" unit cells, with reflections at long d-spacings. 

```{note}
An important consideration is that the d-spacing assignment of a reflection with a typical assymetric peak shape (e.g. [`Bk2Bk2ExpConvPV`](https://docs.mantidproject.org/nightly/fitting/fitfunctions/Bk2BkExpConvPV.html)) is significantly shifted to the left the peak centre of mass (see {ref}`Figure <profile3>`). If such a peak shape is used during calibration, subsequent estimation of a peak position by inspection of the peak centre of mass will lead to large systematic errors. 

Whichever peak shape is used, it will be essential to use the same peak function choice for both the data used to derive an instrument parameter file and for the sample dataset itself used for subsequent Rietveld analysis (e.g. by GSAS-II)  
```
```{figure} static/profile3.png
---
height: 400px
name: profile3
___
The figure shows a fit to a LaB<sub>6</sub> Bragg peak using the GSAS-2 TOF Peak Profile 3 (back to back exponentionals convolved with a psuedoVoigt). The vertical dotted blue line indicates the peak position, which is significantly shifted to the left of the peak centre-of-mass.
```

5. Limitations of [`PDCalibration`](https://docs.mantidproject.org/nightly/algorithms/PDCalibration-v1.html). The main diffraction calibration algorithm is mMantid's `PDCalibration`. This has significant limitations in that it can only fit non-overlapped peaks, which is particulary limiting for calibrants other than extremely high symmetry diamond and $\alpha$-silicon phases, requiring the exclusion of strong diffraction peaks that would otherwise benefit the calibration. It will be benefitial to do a full profile refinement (similar to GSAS-2), which would allow overlapping fits to be included. The option to conduct a fully automated profile refinement is being investigated. 

6. Background subtraction prior to the cross correlation step. The removal of background, prior to cross-correlation, is especially important in datasets that resulted in poor signal-to-background, such as shorter runs or silicon measured in a kapton container. `SNAPRed` has the functionality (used in calculating the vanadium correction) to separate peaks and background and this could be used to subtract a large background. However, this is scheduled to be implemented only at a later date, in Phase 4, as it will be deployed as part of container management. (EWM 3098)

7. Calibrant sample specific parameters. Due to its reconfigurability, SNAP needs a range of calibrants that cover different regions of $Q$-space. However, these calibrants have their own scattering characteristics that affect the resultant calibration measurement. For example, the diamond calibrant has a pronounced Lorentzian contribution to the Bragg peak shape and also wider peaks than silicon. It is intended to capture these properties by adding them to the `json` [NOTE: this is the first and only mention of the JSON files in this page. Maybe needs some introduction, for example what is already in these JSON files.] files that contain the other parameters of the calibrants. This will ensure they are applied systematically. (EWM 3146,3409)

8. Math on ragged workspaces. [Mantid Ragged Workspaces](https://docs.mantidproject.org/nightly/concepts/Ragged_Workspace.html) are necessary frameworks to store TOF diffraction data from instruments, such as SNAP, that span wide ranges of diffraction angle. They allow each spectra within a dataset to have different binning parameters (start, stop and step). However, currently, `Mantid` supports these only poorly and many basic operations (addition, subtraction, scaling and saving) are not supported for ragged workspaces. This will be enabled once the underlying mantid routines are upgraded. 

9. Managing instrument upgrades through time. For example, SNAP is planning to install new bandwidth choppers that will change the wavelength bandwidth. `SNAPRed` needs a way to manage the resultant temporal change in instrument parameters.

10. Consolidating [global configuration](SNAPRedConfig). Currently the configuration description is distributed over two files, one of which is not under version control. These should be consolidated and version control applied to all parameters. (EWM 3408)

11. Enable Lite/Native data options. Currently, `SNAPRed` uses lite-mode as default and doesn't fully support native (i.e. _as collected_) mode. It will be ensured that native mode is fully supported.

12. User-friendly logging. Currently `SNAPRed` is hardwired to report the logging output to terminal in the most verbose "debug" mode. This is useful during code development but is very user unfriendly. This will be changed to allow easy reconfiguration of logging 

## Futher information

The entire SNAPRed codebase is available on the [SNAPRed github repo](https://https://github.com/neutrons/SNAPRed/).

For detailed questions try Malcolm Guthrie on guthriem@ornl.gov  
