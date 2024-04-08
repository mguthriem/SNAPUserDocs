# Test Panel: User Guide

## What is the Calibration Test Panel?

Most modern applications can be viewed as comprising a backend (typically where data are managed and calculations done) and a frontend (e.g. a graphical user interface, or data visualisation tools). In its phase 2 release, `SNAPRed` primarily exists as a backend. In other words: _THERE IS NO GUI YET!_

The test panel acts as a minimalist interface with the backend, designed to enable testing and, consistent with this, there are limitations in what it allows in terms of access to data and other backend processes. To increase testability, the test panel is shipped within mantid workbench, making all of the mantid methods available to interact with the generated workspaces.

## Opening the Panel

The test panel can be opened from within an instance of `workbench` by selecting the `Interfaces` option and then `Diffraction` and then `SNAPRed` from the drop down menu. 

```{figure} static/SNAPRed_UI.png
---
height: 400px
name: snapUI
---
The default UI for SNAPRed.
```

From here, the Panel can be opened by clicking the `Open Calibration Panel` button. This will open the Panel with the `Diffraction Calibration` (hereafter refered to as `DifCal`) tab selected.


