# Diffraction Calibration

## Inputs

The inputs to `Diffraction Calibration` are:

1. A set of neutron powder diffraction data from a known calibrant material, measured in a given instrument state. 
2. Crystallographic information on the calibrant material, specified by a cif file
3. A pixel grouping scheme (e.g. `Column` or any arbitrary scheme) that is used during calibration (as described below), but does not impose any constraint on any subsequent choice of grouping used during reduction.   
4. A set of parameters governing the calibration workflow. Some of these are fixed and some are variable. The ultimate goal is to fix everything possible to fully automate the calibration process
 
A note on input data: Despite it's best efforts, SNAPRed will always be limited to the quality of the input calibration data. Known issues have been observed under the following circumstances: 

* **Input data with insufficient counting statistics.** both components of the diffraction calibration workflow include execution of algorithms that will fail if input data are too noisy.
* **Calibrant sample too far off instrument centre.** initial guesses for peak positions will fail if the physical location of the calibrant sample is too far away from the nominal instrument centre
* **Crystallographic properties of calibrant.** SNAPRed uses the mantid alogrithm `PDCalibration`, which is limited to fitting isolated peaks. At least two non-overlapping peaks *in every pixel group* are required for it to run.    
* **Input data with poor signal-to-background** the pixel calibration requires the determination of a peak centre from a cross-correlation, which can become unreliable for high-background data.   

 
##  Overall methodology

SNAPRed borrows the diffraction calibration methodology from [an existing approach](https://docs.mantidproject.org/v6.1.0/concepts/calibration/PowderDiffractionCalibration.html), developed for `POWGEN` and `NOMAD`. This approach consists of two operations:

1. Make the diffraction focused peaks as sharp as possible 
2. Make the resulting (sharp) peaks match as closely as possible the correct d-spacing 

The first step is referred to as "pixel calibration", the second step "group calibration". The word group calibration implies the existence of a "Pixel Grouping Scheme", this is nothing other than an assigned grouping ID to SNAP pixels. For example, grouping can be defined according to detector components (such as "Banks" or "Columns"), but SNAPRed is not limited to this and can calibrate with any arbitrary grouping definition

The pixel grouping scheme is chosen by the user at the onset of the Diffraction Calibration workflow.


## Essential background theory

A sufficient theoretical background to follow diffraction calibration process is obtained from Bragg's Law: 

$\lambda = 2d_{hkl}\sin\theta$

where $d_{hkl}$ is a d-spacing corresponding to the separation of the cystallographic planes with Miller indices $hkl$ and $\theta$ is half of the scattering angle, and the deBroglie relation for non-relativistic neutrons:

$\lambda = \frac{h}{p_n}$

Here $p_n$ is the neutron momentum, which is given by the product of the neutron mass ($m_n$) and its velocity ($v_n$). A neutron travelling at a constant velocity will traverse a known pathlength $L$ in a time $T$. For neutrons at a pulsed source, it is possible to measure $T$ as the "time-of-flight" between neutron generation and detection.

By substituting this above and equating the two equations, we see that a given Bragg peak with indices $hkl$ will be observed at a corresponding time of flight $T^{hkl}$ given by:

$T_{hkl}=(\frac{2m_n}{h})L\sin\theta d_{hkl}$

For any given pixel, $i$, this can be written as

$T_{hkl,i}=C_i d_{hkl,i}$  

Where $C_i=(\frac{2m_n}{h})L_i\sin\theta_i$ is a constant for pixel i at fixed scattering angle $2\theta_i$ and for which a detected neutron will have traversed a total distance of $L_i$ after scattering from the sample.

In general, details including the finite thickness of the neutron source (a moderator) and time offsets between proton pulse and neutron generation require a more complex relation where the relation from between $T^{hkl}$ and $d^{hkl}$ is parabolic, follow the definition in the GSAS manual.

$T_{hkl,i}=A_i {d_{hkl,i}}^2 + C_i d_{hkl,i} + Z$

And the set of values $A_i$, $C_i$ and $Z$ are called the diffractometer constants, specified as arrays for each of the $N_{pix}$ pixels. 

However, at present, SNAPRed calibration is based on the approximation $T_{hkl}=C_i d_{hkl,i}$.   