# Calibration assessment and output

## Calibration assessment

At this stage in diffraction calibration, the resultant set of diffractometer coefficients, generated via the two step pixel and group calibration approach, $C^{CC,PD}_i$ have to be assessed. This is done by applying them to the input data (Mantid algorithm `ApplyDiffCal`) and then diffraction focusing. The peaks available in the resultant diffraction focused groups are fitted - using a Gaussian peak shape - to extract their position and their widths (defined as the Gaussian standard deviation). These are used to calculate ... 

## Diffraction calibration outputs

At the end of the calibration process