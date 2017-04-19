## Running SDAccel

It is beneficial to make a custom script for setting up the SDAccel environment, as the default one generated by Xilinx sets `LD_LIBRARY_PATH`, which generally is bad practice, and will conflict with binaries compiled under a different environment.

The following is sufficient as of SDx 2016.3:

```shell
#!/bin/sh
export XILINX_OPENCL=<path to DSA folder>/xbinst/pkg/pcie
export XCL_PLATFORM=<DSA string (e.g. "xilinx_adm-pcie-7v3_1ddr_3_0")>
unset XILINX_SDACCEL
unset XCL_EMULATION_MODE
```

## Things to look out for

### Bundled libc++

When linking against the Xilinx OpenCL libraries, the linker must add the runtime library folder to the library search path. This folder contains an ancient version of libc++.so, which will break compilation when using a newer compiler.
To avoid this, backup or delete the library file located at:
```
<SDx or SDAccel folder>/runtime/lib/<architecture>/libstdc++.so
```

### relax\_ii\_for_timing

SDAccel sets the option `relax_ii_for_timing`, and a conservative clock uncertainty of 27% of the target timing. This means that it will silently increase the initiation interval if this more conservative constraint is not met, resulting a slowdown of 2x to the resulting performance. Check the HLS log file to see if your design was throttled, located at:
```
_xocc_<source file>_<kernel file>.dir/impl/build/system/<kernel file>/bitstream/<kernel file>_ipi/ipiimpl/ipiimpl.runs/impl_1/<"vivado_hls" or "runme">.log
```

### Power report crash

While building a project, Vivado generates a number of GUI-reports, even when running `xocc` on the command line. These reports have an internal limit of 64 MB per _section_. For very large projects these sections can exceed 64 MB, causing Vivado to crash with an error like "Unable to write <report name>.rpx as it exceeds maximum size of 64 MB".
So far there does not seem to be a solution to this error.

