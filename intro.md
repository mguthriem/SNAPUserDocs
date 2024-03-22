# SNAPRed User Docs v 0.2

This is a draft set of documentation for Phase 2 release of SNAPRed.

## What is SNAPRed

SNAPRed is a computational tool that supports instrument calibration and data reduction for the SNAP instrument on Beamline 3 of the Spallation Neutron Source, Oak Ridge National Laboratory.

SNAPRed is built on the [mantid code base](https://www.mantidproject.org/) and uses many of its algorithms and data visualisation widgets.

The current release on SNAPRed is in **Phase 2** of the project
	
## What can Phase 2 do? 

Phase 2 can: 

1. Execute calibration of the SNAP instrument. Calibration includes two processes: 

* "Diffraction Calibration", the production of diffractometer constants that allow the conversion of TOF to crystallographic units of d-spacing (d) or momentum transfer (Q)

* "Normalisation Calibration", the production of a function that can be used to normalise diffraction data for wavelength ($\lambda$)-dependent instrumental artefacts. 

2. Support numeric assessment of quality of calibration specifically, accuracy of peak positions and sharpness of peaks
3. Save a calibration record that records the entirity of the calibration process along with metrics capturing the calibration quality 
4. Manage instrument configurations ("States") including:

* Creation of states on the basis of an input run number corresponding to a specific instrument configuration. 
* Automatic configuration of the calibration workflow on the basis of state properties (e.g. diffraction resolution, minimum or maximum d-spacing etc.)
* Saving and indexing of calibration output according to state, such that subsequent data reduction can locate the correct calibration parameters and data. 

5. Define of unique calibrant materials with defined, customisable properties (including material, geometry and crystallographic information)


## What can Phase 2 not do?

Planned scope for SNAPRed that Phase 2 does not include: 

1. Performant management of high event rate data 
2. Execution of powder diffraction data reduction.
3. Management of complex sample container effects
4. a GUI 

To emphasis point 4: SNAPRed primarily exists as a "Backend". To facilitate testing, a "Test Panel" is provided. This is a minimalist GUI and only provides limited interactivity.

## How do I run SNAPRed?

SNAPRed Phase 2 can be invoked from within a session on `analysis.sns.gov` by opening a terminal and typing:

```
mantidworkbench --SNAPRed (TODO: find correct command)
```

## Futher information

The entire SNAPRed codebase is available on the [SNAPRed github repo](https://https://github.com/neutrons/SNAPRed/).

For detailed questions try Malcolm Guthrie on guthriem@ornl.gov  

```{tableofcontents}
```
