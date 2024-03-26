# Diffraction Calibration part 1: pixel calibration

The first step of diffraction calibration, Pixel Calibration, is based on the mathematical process of cross correlation (CC) between the data in a reference pixel and adjacent pixels. In SNAPRed, this is governed by the selected Pixel Grouping Scheme. For example, if "Column" grouping is selected, the algorithm will progress through each of the corresponding 6 pixel groups (one for each detector column). _In each group_, a reference pixel is selected (ensuring that this is always the same pixel for every group in a given pixel grouping scheme), and then the diffraction data in every other pixel in the group is cross-correlated against those in the reference.

Prior to the cross-correlation calculation, input data that are measured as a function of TOF are converted to d-spacing (this is currently done using the default instrument definition, but will be extended to use any pre-existing calibration) and histogrammed, using a logarithmic binning scale. Logarithmic binning is essential to enable to full-pattern cross correlation conducted by SNAPRed, discussed below. The selection of binning parameters occurs automatically, with SNAPRed selecting values that are appropriate for the relevant instrument state and selected pixel group.

The cross-correlation itself is conducted using the mantid algorithm [`CrossCorrelate`](https://docs.mantidproject.org/nightly/algorithms/CrossCorrelate-v1.html). The operation is a way to identify how similar two signals are as a function of their offset relative to each other along a shared x-axis. In our case, the input data are histogrammed neutron counts _versus_ d-spacing.

Since the input spectra contain sharp Bragg peaks at the same d-spacings, the CC will _also_ have a sharp peak. In a perfectly calibrated instrument, the Bragg peaks in any spectra should occur at the same d-spacings and the CC peak would be centred on an offset of zero. However, if errors in calibration are present, they will cause this peak to shift from zero by a certain number of d-space bins, $N^{off}$. By correcting for this non-zero offset, we obtain optimal agreement between the observed d-spacings in every pixel.

An important advantage of the CC approach is that it is independent of any assumption of peak shape _and_ a reliable CC signal can be obtained even when the input data are rather noisy (especially when using whole-pattern cross correlation). This is a significant boon when trying to minimise data collection time needed for calibration.

In real data, the Bragg signal will be combined with experimental background that (normally) will be broadly varying as a function of d-spacing. This background propagates into the CC with the corollary that the algorithm will work better for high signal-to-background input data (n.b. During testing, specific issues were observed with input silicon powder data measured in a kapton container). It is intended that `SNAPRed` will calculate and subtract a background prior to CC calculation, but this feature isn't yet available. 

## Determining the offset 

Once the CC has been obtained, a second step is to determine the centre of the sharp CC peak corresponding to the value of offset  ($N^{off}$), that maximises the overlap of the diffraction data in the two spectra. Again, SNAPRed uses a mantid algorithm [`GetDetectorOffsets`](https://docs.mantidproject.org/nightly/algorithms/GetDetectorOffsets-v1.html), which performs peak fitting to extract a peak centre of the CC to return ($N^{off}$). SNAPRed currently uses a Gaussian peak shape to fit the CC. `GetDetectorOffsets` will create a workspace of offsets, containing a the numerical value of the determined offset for each pixel.

Once CC has executed across all pixel groups, and ($N^{off}_i$)  has been determined for every pixel $i$, the final step in pixel calibration is to apply this to the initial values of the diffractometer constants $C_i$ used to convert each i from TOF to d-spacing. This calculation is conducted by the mantid algorithm [`ConvertDiffCal`](https://docs.mantidproject.org/nightly/algorithms/ConvertDiffCal-v1.html) and returns a new set of $C_i$ values that include the offset correction. 

## Single peak cross correlation

The most common approach when conducting cross-correlation while calibrating a TOF diffractometer is to select a single Bragg peak. This pwK must be common to all pixels within the group of pixels that are being cross correlated. 

In such a calculation, it is appropriate that the input data are a function of d-space and have linear binnning with step size $\Delta d_{lin}$. The resultant applied CC correction corresponds to an _additive_ shift of the x-values by $N^{off}_i \Delta d_{lin}$ of the input data for pixel $i$ to maximise overlap with the data in the reference pixel.

The offset is currently managed in `Mantid` by replacing the initial diffractometer constant $C^{init}_i$ by a new constant $C^{CC}_i$ that includes the offset correction. This immediately creates consideration that the diffractometer constants are applied by multiplication versus in contrast to an offset, which is additive (a shift of the y-values by $N^{off}$ bins). These two distinct operations can only agree at a single point in d-space, $d'$(see {ref}`Figure <CC_singlePeak>`).   

```{figure} static/CC_singlePeak.png
---
height: 400px
name: CC_singlePeak
---
The solid blue line shows the initial relationship between $\mathbf{T}_i$ and $\mathbf{d}_i$ for pixel $i$: a straight line with gradient $\frac{1}{C^{init}_i}$. Instead of applying an additive offset, which would shift the all values of d by a constant (as indicated by the dashed blue line), the CC-corrected diffractometer constant $C^{CC}_i$ must be calculated by considering the offset at a specific d-value $d'$.
```

To enable this, a value for $d'$ must be manually specified when calling the mantid algorithm `GetDetectorOffsets` and, from this, the corresponding corrected diffractometer constant $C^{CC}_i$ can be obtained. This calculation is supported by chosing the  _Relative Offset_ output mode of `GetDetectorOffsets`. 

## Whole pattern cross correlation

SNAPRed adopts a different approach, whereby the entire pattern is cross correlated. This utilises the counts from all present Bragg peaks, reducing the collection time by a factor of 2-3, which is important for the frequent recalibrations necessay on SNAP.

This approach hinges on ensuring that the input diffraction data for each pixel have been binned $logarithmically$. Due to the property of a TOF diffractometer that the diffraction resolution $\delta d/d$ is approximately constant for a given detector, logarithmic binning ensures that each Bragg peak is (approximately) sampled with the same number of histogram bins. Consequently, intensity from each Bragg peak in the pattern will contribute to the total CC peak according to its own intensity.

Since the cross-correlation is not conducted using a single peak, there's no obvious way to select a $d'$ to scale the offset. However, this issue is also solved by log binning. This is seen from the (mantid) definition of log binning, where successive bin edges are related by the corresponding binning parameter is $\Delta d_{log}$ according to:

$d_{j+1}=d_j(1+\Delta d_{log})$

and, the $j^{th}$ d value can be calculated from the first first d value, $d_0$, which is a constant via

$d_j = d_0(1=\Delta d_{log})^n

In this case, the CC offset of of $N^{off}_j$, is applied in the exponent:  

$d_j^{CC} = d_0(1=\Delta d_{log})^(n+N^{off}_j)$

This is equivalent to a multiplication and so the extracted $N^{off}_i$ can be used   directly to scale the diffractometer constants. This will be returned by `GetDetectorOffsets` by choosing the $signed$ output mode. 

## Applying the cross correlation

After running `CrossCorrelation` and `GetDetectorOffsets`, every pixel for which these operations have been successful will have a numerical value of the measured offset, stored in an `offsets workspace`. 

The next step is to apply these offsets via an operation that returns the corrected set of diffractometer constant for each pixel $\mathbf{C^{CC}}$. This is done using the mantid algorithm `ConvertDiffCal` noting that `SNAPRed` returns offsets in using the "Signed" mode. Subsequently, the set of CC-corrected diffractometer constants $C^{CC}$, which are then applied to the input data set (Mantid algorithm `ApplyDiffCal`). The data are then converted from the measured TOF to d-space (Mantid algorithm `ConvertUnits`) with the result that Bragg peaks _in each pixel within a specific subgroup_ will all have the same d-values, equal to those of the corresponding reference pixel. These values may still be incorrect, due to any present (and unknown) offset of the reference pixel. This necessitates the second step of the diffraction calibration process described below.

## Masking

A mask is automatically created for any pixels where this operation fails and. This mask is persisted to disk as a property of the calibration and, subsequently, these pixels will not be included in data reduction using that calibration. During the calibration process, it's important to inspect the output mask to identify pathlological issues (high background in the input data, for instance) with cross correlation that have been observed to lead to large numbers of pixels being masked.