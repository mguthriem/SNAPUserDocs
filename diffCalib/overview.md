(label_difcal)=
# Diffraction Calibration

## The goal of Diffraction Calibration

Powder diffraction measurements embody a rich source of information on the atomic structure of the sample of interest. The presence and position of Bragg peaks encode information on the crystallographic unit cell (and the effects of any stress it is under). The intensity of the peaks encode the details of the nuclear density within the unit cell convolved with the orientational distribution of the sample crystallites. While the width and shape of the Bragg peaks encode details of sample strain fields, particle size and shape{cite}`ITOCH`.  

As these properties of Bragg peaks are also modified by purely instrumental artifacts,the goal of calibration measurements is determine these fully, in order to deconvolve those effects from the sample measurement. The process of diffraction calibration has two main goals: to determine the relationship between crystallographic d-space and measured time-of-flight of detected neutrons and to fully quantify the native peak width and shape of the instrument. 

This section specifies the process used by `SNAPRed` for the former goal. The latter goal is captured via separate workflows currently using external Rietveld packages, such as GSAS {cite}`GSAS`.

## Inputs

The inputs to `Diffraction Calibration` are:

1. A set of neutron powder diffraction data from a known calibrant material, measured in a given instrument state. This is specified via its run number.
2. Crystallographic information on the calibrant material (see below).
3. A pixel grouping scheme (e.g. `Column` or any arbitrary scheme) that is used during calibration (as described below), but does not impose any constraint on any subsequent choice of grouping used during reduction.   
4. A set of parameters governing the calibration workflow. Some of these are fixed and some are variable. The ultimate goal is to fix everything possible to fully automate the calibration process.
 
A note on input data: Despite it's best efforts, SNAPRed will always be limited to the quality of the input calibration data. Known issues have been observed under the following circumstances: 

* **Input data with insufficient counting statistics.** both components of the diffraction calibration workflow include execution of algorithms that will fail if input data are too noisy.
* **Calibrant sample too far off instrument centre.** initial guesses for peak positions will fail if the physical location of the calibrant sample is too far away from the nominal instrument centre. Best practice is to identify the instrument centre and deploy aligment approaches guaranteed to repeatedly position the calibrant sample in this absolute location.
* **Crystallographic properties of calibrant.** SNAPRed uses the mantid alogrithm `PDCalibration`, which is limited to fitting isolated peaks. At least two non-overlapping peaks *in every pixel group* are required for it to run.    
* **Input data with poor signal-to-background** the pixel calibration requires the determination of a peak centre from a cross-correlation, which can become unreliable for high-background data.   

 
##  Overall methodology

The basic approach is to use the diffraction signal from a known calibrant material in order to establish the relationship between the measured variable of time-of-flight and the crystallographic parameters of the calibrant. SNAPRed borrows the diffraction calibration methodology from [an existing approach](https://docs.mantidproject.org/v6.1.0/concepts/calibration/PowderDiffractionCalibration.html), developed for `POWGEN` and `NOMAD`. This approach consists of two operations:

1. Make the diffraction focused peaks as sharp as possible 
2. Make the resulting (sharp) peaks match as closely as possible the correct d-spacing 

The first step is referred to as "pixel calibration", the second step "group calibration". The word group calibration implies the existence of a "Pixel Grouping Scheme", this is nothing other than an assigned grouping ID to SNAP pixels. For example, grouping can be defined according to detector components (such as "Banks" or "Columns"), but SNAPRed is not limited to this and can calibrate with any arbitrary grouping definition

The pixel grouping scheme is chosen by the user at the onset of the Diffraction Calibration workflow.

## Calibrant samples.

The choice of calibrant sample is fundamental to the calibration measurement. For example, any uncertainty in the absolute values of the calibrant lattice parameter will propagate - via the diffractometer constants - into error in the subsequent measurement of any sample that uses that calibration. Similarly, the use of calibrant samples with intrinsic strain, or small particle sizes will interfere with accurate characterisation of the native instrumental peak shape. An additional consideration is the range of d-spaces to be study, which on SNAP is highly variable, depending on instrument state. 

For these reasons, `SNAPRed` uses an established library of calibrant samples. Each calibrant sample is represented by a `json` file containing relevant information on the sample including its crystal structure (and a citation to the source of this information), and its geometry, material and nuclear properties. Each calibrant has a unique ID and an easily redable name. 

A future goal is to record the calibrant within the experimental process variable logs via use of RFID tags.


## Essential background theory

A sufficient theoretical background to follow diffraction calibration process is obtained from Bragg's Law: 

$\lambda = 2d\sin\theta$

where $d$ is a d-spacing corresponding to the separation of a particular set of cystallographic planes (labelled by Miller indices $hkl$) and $\theta$ is half of the scattering angle, and the deBroglie relation for non-relativistic neutrons:

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