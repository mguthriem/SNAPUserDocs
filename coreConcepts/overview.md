(coreConcepts)=
# Core Concepts in SNAPRed

## Instrument States

The highly re-configurable nature of the SNAP instrumentation necessitates automated data management, that associates each run to a specific configuration and thus sets up the appropriate reduction parameters. SNAPRed does this via creation and tracking of "Instrument States". These are defined on the basis of a unique combination of:

* East detector angle
* West detector angle
* Incident optics status (e.g. Guide)
* Wavelength band
* Source frequency

SNAPRed can automatically identify the state from its run number, via accessing the relevant metadata stored along with the diffraction data. Should a specific state not exist in the database, i.e. the instrumnt is being used in a new configuration, SNAPRed can also create and instantiate a new state automatically, although this functionality is limited to users who have write privilege to the `/SNS/SNAP/shared` directory (i.e. Beamline Staff). A "human-readable" name can be given to any state but, internally, SNAPRed manages these via the allocation of 16 character hexadecimal string ID. This approach allows for future modification of the way that States are defined.

All pertinent information for a state is stored in an associated folder, located in `/SNS/SNAP/shared/Malcolm/Calibration/` and the output of calibration operations in the state are stored and indexed here. SNAPRed is thus able to locate the appropriate calibrations for any input run number.  

```{warning}
This method of categorising states relies on the recording of the related process variables(PVs). Since these have not always been recorded for all of the above state variables, this imposes the limiation that SNAPRed will only work for runs with numbers exceeding or equal to 46342. Furthermore, if the readback is not correct, for example due to a sensor failure, the PV should be updated with the proper value _in the raw data *.nxs_ for that run.
```

## Data types

At the SNS, neutron data are recorded in event mode, which should be considered the native data type. Event mode data are essentially arrays that record a set of properties for each detected neutron. These properties are the neutron's time-of-flight, the ID of the pixel in which it was detected and the ID of the pulse from which it was generated (this latter can be converted to a "wall clock" time).  

An alternate mode of storing data is via histograms of observation: whereby a series of discrete bins for a property of interest (say TOF) is predefined and every neutron event is assigned to a bin. 

An important consideration that follows from the nature of the event mode data type is that the size of the data is proportional to the number of neutrons detected. In contrast, the size of histogrammed data is constant (dictated by the chosen number of bins). Consequently, on low flux instruments where number of events per dataset is small, events can be a highly efficient way to store the data. However, on instruments with a high flux and large detector coverage event data can become extremely large volume. SNAP sits on the borderline of these two scenarios so SNAPRed takes careful measures to ensure that the reduction metodology takes into account various datasets it orchestrates. [NOTE: SNAPRed does not "store" that is why I prefered the wording of the reduction aspect]

A design requirement of SNAPRed is to efficiently manage events upon loading data for reduction or calibration . _This is not yet available in Phase 2._

(label_lite)=
## Lite mode

During prototyping of SNAPRed, in part addressing the efficient management of events describe above, the concept of `Lite mode` was developed. This is a process where the entire input event list is relabelled such that events within a fixed 8x8 grid of native pixels on the detector phase are assigned the same "super pixel" ID. The output of this is process is a "Lite workspace" that contains the same number of events as the original, but has 64 times fewer pixels (18 modules of 32x32=18432 _versus_ 18 modules of 256x256=1179648), with almost imperceptible loss of diffraction resolution.

Lite mode greatly accelerates all calibration and reduction operations that iterate through pixels (for example vanadium absorption correction taking 60 minutes in regular SNAP occurs in ~1 minute for SNAP Lite. Furthermore, Lite mode creates the potential to greatly enhance the effectiveness of event compression (via mantid algorithm `CompressEvents`), although this functionality is not implemented in Phase 2

An important consequence is that the activation of `Lite Mode` impacts all files or workspaces that depend on pixel IDs. Notably, these include: calibration workspaces, masking workspaces and grouping workspaces (and associated files containing these that are saved to disk). SNAPRed is designed to automatically handle this, but the user should be aware that selecting `Lite Mode` is conceptually switching to a different (virtual) instrument with coarser pixels.

## Pixel Grouping Definitions

SNAP's diffraction detector system consists of two movable detector banks, each conprising a 3x3 array of Anger cameras (area resolved TOF-capable neutron detectors). This creates a natural flexibility in how detector pixels can be grouped together. As a general rule, larger pixel groups improve counting statistics but, eventually, at the expense of diffraction resolution. A consequence of this is a user should have the ability to easily switch between different pixel grouping schemes [schema?] (PGS) that describe how pixels are to be combined. Each PGS will consist of a number of subgroups each with their own ID.

Thus, SNAPRed is intended to be able to manage multiple different PGS and to reduce data from a specified list of these (allowing users multiple views on their data after reduction). The "standard" PGS based on combinations of detector components (`All`,`Bank`,`Column`,`2-4`) exist as defaults, however, it is also possible to specify unique PGS for any given instrument state. An example of this might be a scheme based on pixels sharing a scattering angle range. Such schemes differ from detector-component based schemes as the corresponding pixel ID's will change if the detector moves. [NOTE: The pixel ID changes if the detector moves? Or did you mean the Grouping ID? If I am getting this all wrong, maybe consider rewording]

As they involve pixels IDs, separate PGS _must_ be specified separately for both Lite and non-Lite modes. 

(binning_considerations)=
## Data binning considerations

Appropriate management of large event TOF diffraction datasets requires optimized histogramming and/or compression of the raw event data. In general, this operation is lossy and so the binning parameters used must be chosen correctly. In SNAPRed, the binning parameters are properties of _each subgroup_ for a given state and are calculated for each of these. This calculation is done on the basis of a set of defined instrument parameters, namely, the incoming wavelength band and the instrument diffraction resolution, the latter is parameterised by the TOF-resolution equation - {cite:p}`WORLTON1976`- and a specification of the relevant value for SNAP.

The User can then specify the number of bins `NBin` within the FWHM of a measured Bragg peak and SNAPRed establishes the optimal binning parameters. This approach exploits logarithmic binning combined with the approximately linear dependence of resolution on wavelength for a TOF diffractometer and means that the chosen `NBin` number of points will be constant for any peak in any subgroup in any state of the instrument. The value used should reflect the final approach to fitting peak shapes and be sufficient to support the numbner of parameters needed. Typically, this would include a Rietveld fit using a back-to-back exponential convolved with a psuedovoigt peak model, which has at least 5 parameters for each peak (position, width, gauss-Lorentz mixing,and the leading and trailing edge exponents) in addition to a background model. 

By default, the FWHM used in combination with `NBin` to determine binning is taken from measurements of an ideal sample - taken to be NIST Silicon (640d). All data will be binned during reduction to match this native diffraction resolution of the instrument state. After reduction, it is possible increase the size of binning, downsampling the diffraction resolution, as this may be appropriate in the presense of sample-dependent peak broadening. This downsampling is controlled via specification of a multiplier of `NBin` to ensure that every peak in the output data is sampled with the same binning.

Note that, when handling the data as hystogrammed, the length of an array of x-values will have one extra value, when compared to the corresponding y-values, as the x-array contains the pair of bin edges for each y-value, as opposer to the conventional _x-y_ point data. 

## A note on indexing [I found this \confusing, is there a graphical way to describe this?]

An important consideration is indexing due to the complex nature of TOF datasets. Each pixel is an independent, TOF-resolving detector that collects a dataset that can be represented as an array: a list of events or a histogram. So, in general, we need to give an index for a specific pixel _and_ an index for a element in an associated data array.

As a specific example, consider the case where the events in the $i^{th}$ pixel are histogrammed according to an array of TOF bins. We can refer to this array as $\mathbf{T}_i$ where $0\leq i \leq (N_{pix}-1)$ and where $N_{pix}=1179648$ for the native instrument and $N_{pix}=18432$ for the Lite instrument (i.e. the number of pixels). The bold font indicates this quantity is an array.

In turn, we would then refer to the $j^{th}$ element within the $i^{th}$ array as $T_{i,j}$, no longer using a bold font as this is now a single value. Here, it's important to remember that, in general, the size of the data arrays can vary between pixels.    

An important operation in data handling within `SNAPRed` is diffraction focusing, whereby the data in multiple spectra are combined, effectively replacing many pixels with a single "super pixel" for each group as defined by a pixel grouping scheme. We will represent diffraction focussed data by enclosing the affected array in $\{$ and $\}$ brackets but otherwise use a similar indexing, using capital letters for the indices. Thus, if we used a pixel grouping scheme with $N_{grp}$ groups of pixels, this will result in $N_{grp}$ arrays containing d-spacings and the array for the $I^{th}$ group would be $\{\mathbf{d}\}_I$. As above, we can refer to the $J^{th}$ element within this array using $\{d\}_{I,J}$

## Symbols used in this book

Below are set of symbols used hroughout this document. 

### The symbols [NOTE: do we ever mention wall clock or regular time "small t" in seconds]

| Symbol       | Units         | Description |
|-------       |-------        |-------------|
|$h$ | J.Hz$^{-1}$ | Planck's Constant |
| $m_n$ | kg | neutron mass |
|$\lambda$  | Å | wavelength values|
|$T$  | $\mu$s        | TOF values|
|$d$  | Å        | d-spacing values|
|$hkl$| | Miller indices|
|$Z$  | $\mu$s          | zeroth order diffractometer constant |
|$C$  | $\mu$s.Å$^{-1}$          | first order diffractometer constant |
|$A$  | $\mu$s.Å$^{-2}$          | second order diffractometer constant |  

