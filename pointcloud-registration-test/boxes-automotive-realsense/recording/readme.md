# Aligning realsenses in automotive with boxes

This went pretty well.

Here are the steps taken:

- Ensure we have floor. Captured `01-boxes-with-floor.ply`. Cameraconfig saved as `cameraconfig-01.json`.
- Run
  ```
  cwipc_register --nograb 01-boxes-with-floor.ply --algorithm_multicamera MultiCameraToFloor
  ```
- Saved cameraconfig as `cameraconfig-02.json`. Captured `02-boxes-with-aligned-floor.ply`.
- Analyzed alignment for `mean`, both with floor and with `--ignorefloor`. Results are in `02-boxes-with-aligned-floor-mean.png` and `02-boxes-with-aligned-floor-mean-ignorefloor.png`.
  - Visually it is clear that ignoring the floor leads to a different PDF, with two humps in stead of one.
  - Visually it also appears that the cumulative graphs for `--ignorefloor` are much more different than for including the floor.
  - Maybe `median` could make this clear numerically? Unsure, this may be due to the fact that these are boxes (not a human).
- Decided on a max correspondence of `0.034`, beging `mean+sd` of the worst camera (number 1). Ran
  ```
  cwipc_register --nograb 02-boxes-with-aligned-floor.ply --correspondence 0.034
  ```
  Saved the config as `cameraconfig-03.json` and captured `03-boxes-corr034.ply`.
- Success!!
	- `03-boxes-corr034-mean-ignorefloor.png` shows that the second hump is gone. So that means we can start looking at the `mode` again.
	- Created 4 different `mode` plots:
	  - `03-boxes-corr034-mode-toself.png` shows the intra-camera error
	  - `03-boxes-corr034-mode-toself-nth2.png` shows the second-nearest-point intra-camera error
	  - `03-boxes-corr034-mode.png` shows the inter-camera error with floor
	  - `03-boxes-corr034-mode-ignorefloor.png` shows the inter-camera error without floor
  - This is really good! the four distance distribution plots have a very similar size, and reasonably similar numerical values too.
    - I _think_ this means that if we get to this situation we can conclude that our work is done.

### Next steps

Let's apply these cameraconfigs (-2 and -3) to the Paula captures, and see if we get similar graphs.
