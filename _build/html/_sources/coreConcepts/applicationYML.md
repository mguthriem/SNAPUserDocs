(SNAPRedConfig)=
# Global configuration

An important design feature of `SNAPRed` is to automate and standardise workflows. To this end, it is necessary to specify, in advance, a set of parameters. These parameters can be grouped into sets

**Instrument Specific Parameters (ISP):** These are global parameters that `SNAPRed` will use for all runs that it operates on. These include details on standard file names and locations but also extend to physical instrument parameters (such as wavelength bandwidth). The latter are not expected to change frequently but, of course, can change following a significant instrument modification (such as chopper or focusing-guide replacement).

**Application parameters:** This is another set of global parameters but, rather than reflecting the physical instrument, they govern details of individual algorithms called by `SNAPRed` during execution. Examples are parameters used by individual mantid algorithms.

**State Specific Parameters:** These are parameters that are unique to a particular state (examples might include limiting d-range values). When a state is created, `SNAPRed` initialises these values, typically deriving them from the `Instrument Specific Parameters` and fundamental State Parameters (such as detector position). Subsequently, these values are fine-tuned during calibration. The set of `State Specific Parameters` should be common for every run measured in a particular state.

**Run Specific Parameters:** These are unique to a given experimental dataset, an example might be a pixel mask applied to remove diamond anvil reflections or to mask pixels occluded by Paris-Edinburgh cell anvils as they move during a compression.

## SNAPInstPrm.json

On the basis of above, it's clear that the ISP's include a set of very important numbers typically determined via careful calibration and experimentally verified.

The set of ISP's are stored in a file called `SNAPInstPrm.json` located in the main `Calibration` directory of SNAPRed, which is located in `/SNS/SNAP/shared` and, thus, visible by all SNAP users, but only editable by instrument scientists. At present, this file is not version controlled and any edits made will affect the subsequent creation of any new state. 

```{important}
Currently the ISP values have been derived through prototype testing of the calibration algorithms. They should be considered a good starting point but are not necessarily the final "best" values. It is expected that during field testing of `SNAPRed` their values may change a little, but the idea is that, once these are chosen, the should remain mostly fixed (until something changes on the instrument).
```
```{important}
In light of the above comment, after sufficient testing has been done to conclude that a validated set of ISP's has been found, these should be managed in a different way that is version controled.
```

## application.yml

As a general rule, the application parameters are stored in `application.yml` which is a file stored inside the `SNAPRed` repo and, therefore, held under version control. Correspondingly, a user can always modify their own copy of `application.yml` and there version of `SNAPRed` will use this. However, to change values in the _deployed_ version of the file requires a formal "Pull Request". This process ensures that a level of change management is exerted over the software.

## how SNAPInstPrm.json and application.yml are used 

It is important to understand that `application.yml` and `SNAPInstPrm.json` act as as _templates_ that are then applied during the creation of a new state. Thus, once a new state is instantiated, subsequent edits to  either file will not subsequently affect the existing state. 

When a state is [created](coreConcepts), the relevant information from these template files are copied. In addition, a set of State Specific Parameters are calculated, which depend on parameters such as detector angles and chopper central-wavelength settings. The complete set of parameters is treated as a "version 1" diffraction calibration. This isn't a real calibration, but uses calculations of  that are then enable the first real calibration. The parameters are stored in a file called `CalibrationParameters.json` in subdirectry of the state at: `/diffraction/v_0001`. It is from this copy that `SNAPRed` then draws the ISP's for the first calibration. Thus, editing this file will modify the values that `SNAPRed` uses during the first calibration. 

Each subsequent n$^{th}$ diffraction calibration of the state will create a new folder `/diffaction/v_000n`. `SNAPRed` will automatically select the correct folder to use on the basis of the `calibrationIndex` that it maintains for each state. The calibration indexes on run number  
  