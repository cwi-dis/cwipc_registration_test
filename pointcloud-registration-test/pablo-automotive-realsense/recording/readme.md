## Aligning automotive realsense Pablo

First thing tried was to redo the coarse calibration. This was no success: the result was pretty similar to the previous coarse registration.

Opened <https://github.com/cwi-dis/cwipc/issues/234> to test the hypothesis that the underlying problem is an issue with using the wrong Depth frame (for the wrong camera) when converting RGB image `(u, v)` coordinates to 3D `(x, y, z)` coordinates.

Used this capture anyway to test cwipc_register floor alignment.

Proceeded with using this capture with `cwipc_register --guided`, selecting various viewpoints and algorithms.

Got as far as 