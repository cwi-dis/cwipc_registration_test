### Initial capture, align to floor

- Use `cameraconfig-orig.json`, but with floor added.
- capture `paula-with-floor.ply`
- Run 
  ```
  cwipc_register --nograb paula-with-floor.ply --plot --verbose --algorithm_multicamera MultiCameraToFloor
  ```
- plot `paula-with-floor.png` and `paula-with-floor-after.png`
- Captured `paula-with-floor-after.ply`
- Edit cameraconfig, get rid of floor.
- Capture `paula-flooraligned.ply`
- Keep config as `cameraconfig-afterflooralign.json`

### Check whether we can align to ground truth

- Run
  ```
  python -m cwipc.scripts.cwipc_analyze_registration --method q=95 --plot --togroundtruth paula-flooraligned.ply ../../paula-miraco.ply
  ```
  Plot alignment to `paula-miraco-to-flooraligned.png`
- Run 
  ```
  python -m cwipc.scripts.cwipc_find_transform --output paula-miraco-aligned.ply --correspondence 0.16 --plot ../../paula-miraco.ply paula-flooraligned.ply
  ```
  Plot alignment to `paula-miraco-aligned.png`
- Run
  ```
  python -m cwipc.scripts.cwipc_analyze_registration --method q=90 --plot --togroundtruth paula-miraco-aligned.ply paula-flooraligned.ply
  ```
  Plot alignment to `paula-to-miraco-aligned-pre.png`.
  Worst correspondence was camera 1 with `0.044`.
- Test whether we can use this ground truth `paula-miraco-aligned.ply` to align the individual tiles. Run
  ```
  python -m cwipc.scripts.cwipc_test_aligner --togroundtruth paula-miraco-aligned.ply --correspondence 0.044 --plot paula-flooraligned.ply paula-flooraligned-miracoaligned.ply
  ```
  Plot alignment to `paula-to-miraco-aligned-post.png`.
  There is a gap on her left.
- Tried various things to get rid of the gap (like upping the correspondence, or repeating the ground-truth registration). Nothing worked. Maybe I accidntally moved paula's arm, so the ground truth isn't an actual ground truth?

### Use the boxes registration

Let's  a completely different angle. We have the good calibration from `../../../boxes-automotive-realsense/recording`.

- First we copy the floor-aligned cameraconfig (also to cameraconfig-01.json`) and capture `02-paula-with-aligned-floor.ply`.
- Analyze alignment for `mean`, both with floor and with `--ignorefloor`.
  - Results are similarly-shaped as for the boxes.
  - I have the impression (_need to verify_) that for most tiles it holds that `mode < mean-stddev`. Maybe not for camera 1, but it has a very large `mean`, and also its `mode` is the "second peak". We may be able to detect this.

