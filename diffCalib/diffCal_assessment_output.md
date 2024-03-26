# Calibration assessment, completion and output

## Calibration assessment

At this stage in diffraction calibration, the resultant set of diffractometer coefficients, generated via the two step pixel and group calibration approach, $\mathbf{C^{CC,PD}}$ have to be assessed. This is done by applying them to the input data (Mantid algorithm `ApplyDiffCal`) and then diffraction focusing (using the specified PGS). The peaks available in the resultant diffraction focused groups are fitted using a Gaussian peak shape. Although the Gaussian shape is not ideal for TOF data, it is sufficient for relative comparisons between different calibrations.

### Overall calibration quality

For each PGS, the resultant set of Gaussian peak positions are subtracted from the equivalent set of expected values for the calibrant sample and the result divided by the peak width, these values are then averaged over all peaks in the group. This quantity is called "strain" in analogy with the physical effect of elongating or compressing a material. Similarly, the Gaussian peak widths are divided by their positions to give an estimate of diffraction resolution $\delta d/d$ which is also averaged over all peaks in the group, this quantity is called "sigma" in reference to the Gaussian standard deviation. 

The resultant two values are captured for each group in the PGS as a function of scattering angle %2\theta$ in workspaces with names similar to these (for calibrant run 57474): 

```
calib_metrics_sigma_057474_ts1711137415314
calib_metrics_strain_057474_ts1711137415314
```

Here the final part of the workspace name is a time stamp. `SNAPRed` allows the calibration process to be repeated iteratively allowing, the optimal calibration to be accepted. 

It is also possible to compare these metrics with earlier calibrations for the same state, allowing a more global assessment of quality between calibrations conducted at different times, or for different calibrant materials. 

When a calibration is finalised, these metrics are saved as part of the calibration record.

## Detailed inspection: Group Calibration 

The fit parameters generated during execution of `PDCalibration` are stored in a grouped workspace `fitPeaksWSGroup`. These contain table workspaces for each pixel group and for each peak fitted in the group, including $\chi^2$ and should be inspected. A graphic interface for this has been developed, whereby a single plot window shows the data and fitted spectra for each group. As a visual guide, peak fits with $\chi^2$ above a threshold value of 100 are coloured red.

```{note}
In testing it was noticed that very high $\chi^2$ values can be found for long datasets. This is due to random errors due to Poisson counting statistics becoming very small relative to systematic errors from using incorrect peak shape (at present, `PDCalibration` can only use symmetric peak models, which do not perfectly fit the TOF peakshape).

Modificartion of the threshold value of $\chi^2$ can be achieved via editing `application.yml`   
```

### Detailed inspection CIS mode

More detailed diagnostic information is accessible by through the `cis_mode` option. This is currently accesible in the `application.yml` and, if set `True` will preserve intermediate workspaces, which can be inspected. 

* *CC correlation* the values of the offsets are stored in workspaces with the name `offsets_12345_0` where `12345` is the run number and 0 is the calibration iteration. These contain the value of the offset (in units of absolute number of bins)for every pixel and can be insepcted using either the `Plot/Bin` or `Show Instrument` methods to look for geometric trends. In particular, the maximum offset is currently hardwired to not exceed 10: if multiple spectra have offsets equal to this hard limit, it likely indicates an issue with the input data. 

* DIFC 

TODO: complete this section

## Diffraction calibration outputs

When satisfied with the results of the calibration, the resultant output is saved to disk: 

* a set of calibrated diffractometer constants stored in an mantid diffcal .h5 file (this includes the calibration mask)
* a calibration record capturing all other parameters from the calibration.
* a copy of the "strain" and "sigma" metrics and final diffraction-focused dataset.

It is also required that the user specify to which run number the calibration applies _from_. This defaults to the run number of the diffraction calibrant however, it can be specified to an earlier run number (for example when a calibration is conducted after a sample dataset was measured). This is then stored in the calibration index for the state to allow for the automatic location of the appropriate calibration to be used when reducing sample data.