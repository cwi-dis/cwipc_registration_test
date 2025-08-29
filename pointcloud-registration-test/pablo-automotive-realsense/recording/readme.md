## Aligning automotive realsense Pablo

First thing tried was to redo the coarse calibration. This was no success: the result was pretty similar to the previous coarse registration.

Opened <https://github.com/cwi-dis/cwipc/issues/234> to test the hypothesis that the underlying problem is an issue with using the wrong Depth frame (for the wrong camera) when converting RGB image `(u, v)` coordinates to 3D `(x, y, z)` coordinates.

Used this capture anyway to test cwipc_register floor alignment.

Proceeded with using this capture with `cwipc_register --guided`, selecting various viewpoints and algorithms.

Got as far as `01-cameraconfig.json`, but I have the feeling we can do better.

## Trying again

Back to `00-cameraconfig.json`.

Align to floor. Keep as `11-cameraconfig.json`.
Capture `11-pablo.ply`.

Can't do anything with it: the algorithm just concentrates on the floor and rotates so that it matches best.

Got rid of the floor, keep as `12-cameraconfig.json`, capture `12-pablo.ply`.
Different problem: front and back are still mapped to each other, now by raising one pointcloud off the floor.

Enabled the orientation-filter. Seems to help somewhat, but not enough.

