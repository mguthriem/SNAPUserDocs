(group_cal)=
# Diffraction Calibration Part 2: Group Calibration

The input data for Group Calibration is a diffraction focussed (Mantid algorithm `DiffractionFocusing`), using the CC-corrected diffractometer constants, and applying the specified pixel grouping scheme (PGS). This process combines the neutron counts in all pixels in each group into a single spectrum, reducing the total number of spectra in the dataset to the number of sub groups in the PGS, $N_{grp}$. Due to the application of the cross correlation correction, the Bragg peaks in each diffraction-focussed group should be as sharp as possible, however, the absolute value of their d-spacings will, in general, still be incorrect. This is dealt with using a process of `Group Calibration` which employs the mantid algorithm [`PDCalibration`](https://docs.mantidproject.org/nightly/algorithms/PDCalibration-v1.html) applied independently for each subgroup in the PGS. The output of this process is a complete set of diffractometer constants that ensure the sharpest possible peaks at the correct positions in d-space.  

The approach applied by `PDCalibration` is to fit the Bragg peaks in each input spectrum. This works well on the focussed dataset as the statistical quality of the data in each $focused$ spectrum are much better than in individual pixels.

`PDCalibration` fits the data as a function of TOF and unit conversion must be done on the diffraction focused data prior to input into the algorithm. It operates by conducting a series of individual peak fits and, thus, has a requirement that the peaks selected for fitting do not overlap. To enable this, `SNAPRed` orchestrates the peak fitting in the following way for each pixel group:

* The peak positions in d-spacing are calculated for the calibrant sample via the user-selected cif file. The resultant list is truncated between the limits of the minimum and maximum d-spacings accessible to each pixel group, which `SNAPRed` calculates, respecting both the grouping scheme and the instrument state. Bragg peak intensity is estimated approximately by applying the TOF-powder specific Lorentz correction to the calculated structure factors squared and weak peaks can then be removed via a user-specified threshold ($I_{thresh}$) entered as a percentage of the estimated intensity of the strongest calibrant Bragg peak.

* The peak extents are estimated by first calculating the FWHM of each peak to be fitted - deriving this from the known diffraction resolution, respecting the pixel group properties and the instrument state. Second, the exponent of the exponential trailing edge tail of the TOF Bragg peak shape is determined. This latter follows the same parameterisation used by the GSAS TOF Peak Profile Function 3 {cite:p}`GSAS`, whereby the exponent $\beta$ is determined from its parameterised d-space dependence, converted to its equivalent in d-space $\beta_d$ and the corresponding 1/e length calculated. These calculated values are then used to define a left and right extent ($E_L$ and $E_R$ respectively) by applying three multipliers (constants fixed for the instrument): $M_{FWHM-L}$, $M_{FWHM-R}$ and $M_{tail}$ in the following way:

    $E_L = FWHM*M_{FWHM-L}$

    $E_R = FWHM*M_{FWHM-R}+M_{tail}/\beta _d$

* Using the calculated peak extents, the list of peaks is purged of any peaks that overlap. Since the calculation respects the varying (angular-dependent) diffraction resolution, different peaks may be purged in different groups. At least two peaks per group are required for `PDCalibration` to execute, but a larger number of peaks are recommended.

Prior to execution, the user is able to fine tune the list of peaks by adjusting the minimum and maximum d-spacing of the list and the peak intensity threshold $I_{thresh}$. `SNAPRed` provides a visualisation of data and peak extents to support optimising the peak list prior to execution of group calibration.

```{note}
An idea, not yet implemented, is to record the optimal settings for each calibrant in its `.json` file. This will further automate the calibration process. At present there's insufficient knowledge of what the best settings are, not how these vary with PGS and instrument state.
``` 

Lastly, the user is allowed the possiblity to select a peak shape. At present, due to a limitation of `PDCalibration` only symmetric shapes are supported: Gaussian, Lorentzian and psuedoVoigt. This is a known limitation but the best solution isn't yet settled as use of asymmetric peak shapes will have additional impacts on subsequent analysis of reduced data (see [here](todo)). 

`PDCalibration` executes by fitting all of the specified peaks in each pixel group. This provides a set of pairs of $T_{hkl}:d_{hkl}$ values (where $d_{hkl}$ are known for the reference calibrant material) that can be used to determine the "correct" diffractometer constant $C^{CC,PD}_j$ for the $j^{th}$ group. Since - post cross correlation correction - all pixels in the group have their peaks at the same d-spacings, multiplying each diffractometer contant by the ratio $\frac{C^{CC,PD}_j}{C^{CC}_i}$ will shift the peak for _every pixel_ to the correct d-spacing. 
