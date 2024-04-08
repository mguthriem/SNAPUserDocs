(cisMode)=
# CIS Mode

In `SNAPRed` some effort has been taken to manage intermediate (mantid) workspaces both to optimise performance, reduce RAM usage, and streamline the user interaction. Often, this boils down to running a clean up of uneeded workspaces during or after workflow execution. However, sometimes situations arise where it is helpful to inspect these intermediate workspaces. This has been enabled by creating the parameter `cis_mode` found in `application.yml`. By setting `cis_mode=true` intermediate workspaces will be retained in the mantid workspace tree.   

```note
CIS mode is not meant to be user facing, so the output workspaces likely have names that are confusing without prior knowledge of the underlying algorithms.
```



  