# Diffraction Calibration Part 2: Group Calibration

The input data set, as a function of d-spacing, is diffraction focussed (Mantid algorithm `DiffractionFocusing`), using the specified pixel grouping scheme (PGS). This process combines the neutron counts in all pixels in the group into a single spectrum and reduces the total number of spectra in the dataset to the number of sub groups in the PGS, $N_{grp}$. Due to the application of the cross correlation correction, any Bragg peaks in each diffraction-focussed group should be as sharp as possible, however, the absolute value of their d-spacings will, in general, still be incorrect. This is dealt with using a process of `Group Calibration` which employs the mantid algorithm [`PDCalibration`](https://docs.mantidproject.org/nightly/algorithms/PDCalibration-v1.html) indepenedently for each subgroup in the PGS. The output of this process is a complete set of diffractometer constants that should yield sharp peaks at the correct position.  

The approach applied by `PDCalibration` is to fit the Bragg peaks in each focused spectrum. This works well on the focussed dataset and the statistical quality of the data in each $focused$ spectrum are much better than in individual pixels, due to a greatly increased number of neutron counts.

`PDCalibration` performs fits the data as a function of TOF and unit conversion must be done on the diffraction focused data prior to input into the algorithm. It operates by conducting a series of individual peak fits and, thus, has a requirement that the peaks selected for fitting do not overlap. `SNAPRed` orchestrates the peak fitting in the following way for each pixel group:

* The peak positions in d-spacing are calculated for the calibrant sample via the user provided cif file. The resultant list is truncated between the limits of the minimum and maximum d-spacings accessible to each pixel group (and respecting the instrument state). Weak peaks are removed by estimating Bragg peak intensity, by applying a Lorentz correction to the calculated structure factors squared, and allowing a specified threshold ($I_{thresh}$) as a percentage of the strongest peak.

* The peak extents are estimated by first calculating the FWHM of each peak to be fitted - deriving this from the known diffraction resolution, respecting the pixel group properties and the instrument state. Second, the exponent of the exponential trailing edge tail of the TOF Bragg peak shape is determined. This follows the same parameterisation used by the GSAS TOF peak profile function 3, whereby the exponent $\beta$ is determined, converted to its equivalent in d-space $\beta_d$ and the corresponding 1/e length calculated. These calculated values then used to define a left and right extent ($E_L$ and $E_R$ respectively) by applying three multipliers (constants fixed for the instrument): $M_{FWHM-L}$, $M_{FWHM-R}$ and $M_{tail} in the following way:

    $E_L = FWHM*M_{FWHM-L}$, and

    $E_R = FWHM*M_{FWHM-R}+M_{tail}/\beta _d$

* Using the calculated peak extents, the list of peaks is purged of any peaks that overlap. Since the calculation respects the varying (angular-dependent) diffraction resolution, different peaks maybe purged in different groups (with typically higher angle groups retaining more peaks) 

Prior to execution, the user is able to fine tune the list of peaks by adjusting the minimum and maximum d-spacing of the list and the peak intensity threshold $I_{thresh}$. `SNAPRed` provides a visualisation of data and peak extents to support optimising the peak list prior to execution of group calibration.

Lastly, the user is allowed the possiblity to select a peak shape. At present, due to a limitation of `PDCalibration` only symmetric shapes are supported: Gaussian, Lorentzian and Psuedo Voigt.

`PDCalibration` executes by fitting all of the specified peaks in each pixel group. This provides a set of pairs of $T_{hkl}:d_{hkl}$ values (where $d_{hkl}$ are known for the reference calibrant material) that can be used to determine the "correct" diffractometer constant $C^{CC,PD}_j for the $j^{th}$ group. Since - post cross correlation correction - all pixels in the group have their peaks at the same d-spacings, multiplying each diffractometer contant by the ratio $\frac{C^{CC,PD}_j}{C^{CC}_i} will shift the peak for every pixel to the correct d-spacing. 

## Calibration assessment

At this stage in diffraction calibration, the resultant set of diffractometer coefficients, generated via the two step pixel and group calibration approach, $C^{CC,PD}_i$ have to be assessed. This is done by applying them to the input data (Mantid algorithm `ApplyDiffCal`) and then diffraction focusing. The peaks available in the resultant diffraction focused groups are fitted - using a Gaussian peak shape - to extract their position and their widths (defined as the Gaussian standard deviation). These are used to calculate ... 

## Diffraction calibration outputs

At the end of the calibration process