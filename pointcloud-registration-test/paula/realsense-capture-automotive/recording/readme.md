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
  python -m cwipc.scripts.cwipc_find_transform --output paula-miraco-aligned.ply --correspondence 0.24 --plot ../../paula-miraco.ply paula-flooraligned.ply
  ```
  Plot alignment to `paula-miraco-aligned.png`
  
- Run
  ```
  python -m cwipc.scripts.cwipc_find_transform --output paula-miraco-aligned.ply --correspondence 0.24 --plot ../../paula-miraco.ply paula-flooraligned.ply
  ```
  Plot alignment to `paula-to-miraco-aligned-pre.png`
- Test whether we can use this ground truth `paula-miraco-aligned.ply` to lign the individual tiles. Run
  ```
  python -m cwipc.scripts.cwipc_test_aligner --togroundtruth paula-miraco-aligned.ply --correspondence 0.071 --plot paula-flooraligned.ply paula-flooraligned-miracoaligned.ply
  ```
  Plot alignment to `paula-miraco-aligned-post.png`.
  The results are pretty good.

### Check whether we can get similar alignment without ground truth

- Determine within-camera precision with
  ```
  python -m cwipc.scripts.cwipc_analyze_registration --method median --plot --toself paula-flooraligned.ply
  ```
  result saved as `paula-flooraligned-toself.png`.
  
  Looks believable: median of `4mm`, and about `5%` of the points have noise up to `3mm`.
- Determine one-camera-to-all-others distances with
  ```
  python -m cwipc.scripts.cwipc_analyze_registration --method q=70 --plot paula-flooraligned.ply
  ```
  > Note that the `q=70` is a magic number: it was determined by looking at the graph for another method, and then seeing approximately the percentage of points we expect to overlap between each camera and all other cameras.
  
  Results are in `paula-flooraligned-q70.png`, and we are going to use the per-tile correspondences in the next step.
  
  Actually, I didn't: I used interactive but accepted all the choices.
  
- Ran
  ```
  python -m cwipc.scripts.cwipc_test_aligner --algorithm_multicamera MultiCameraIterativeInteractive paula-flooraligned.ply paula-flooraligned-interactive.ply
  ```
  ```
  python -m cwipc.scripts.cwipc_analyze_registration --method q=70 --plot paula-flooraligned-interactive.ply
```
  This had the problem that tile `4` (index `3`) which is the back of paula was pushed forward too far, because apparently the correspondence chosen allowed back points to be mapped to front points.
- Ran
  ```
  python -m cwipc.scripts.cwipc_test_aligner --algorithm_multicamera MultiCameraIterativeInteractive paula-flooraligned.ply paula-flooraligned-interactive2.ply
  ```
  this time selecting **half** the suggested correspondence. Why? Just a hunch.
  
  Did not make much of a difference, if any.Still the back tile is squashed onto the front.
- Inspected the graph again, let us try the "err on the side of cautiousness" method. At a distance of `0.015` we should match about 30% of the points.
  ```
  python -m cwipc.scripts.cwipc_test_aligner --algorithm_multicamera MultiCameraIterativeInteractive paula-flooraligned.ply paula-flooraligned-interactive3.ply
  ```
  Still not a success.
- Inspected the original capture and saw that the captured tiles already had this problem: the "paula's back" tile is too much towards the "paula's front" tile.

-  **Brainwave**: the problem is that for Point2Plane it does not take into account the direction of the plane! So "paula front" points and "paula back" points should be very difficult to match, but in stead they are very easy to match.

  Need to inspect the "normal invention" algorithm. If we can have another one (for example based on the centroid of the per-camera point cloud) we should be getting much better results.
- While setting out to implement this I realised I made a serious operator error: I always selected `no` for accepting the alignment results. Also, there was a bug in the code so while using `cwipc_test_aligner` it probably used point2point in stead of point2plane as the algorithm.

  Trying again, now non-interactive.
- First let's try non-interactive with a correspondence of `0.035` (which is the maximum gleaned from `paula-flooraligned-q70.png`:
  ```
  python -m cwipc.scripts.cwipc_test_aligner --correspondence 0.035 paula-flooraligned.ply paula-flooraligned-035.ply
  ```
  
  This clearly shows that we need a better normal estimation: it now managed to match paula's front and back to a much finer level:-)
- Fixed `RegistrationComputer_ICP_Point2Plane` to estimate normal direction by looking at the centers of the source and terget point cloud to align.
- Ran
  ```
  python -m cwipc.scripts.cwipc_test_aligner --correspondence 0.035 --verbose paula-flooraligned.ply paula-flooraligned-normal035.ply
  ```
  It makes no difference whatsoever!
  
  So maybe we need to try GeneralizedICP in stead of PointToPlane.
  
- Implemented that, and ran
  ```
  python -m cwipc.scripts.cwipc_test_aligner --correspondence 0.035 --verbose --algorithm_fine RegistrationComputer_ICP_Generalized paula-flooraligned.ply paula-flooraligned-generalized035.ply
  ```
  Tiles are pairwise-aligned pretty nicely. But overall there is a huge error.
- Realized that the `70%` is toolarge an overlap for some of these camera-pairs. 

  Let's try the max-correspondence of `0.0133` that comes out of there:
  ```
    python -m cwipc.scripts.cwipc_test_aligner --correspondence 0.0133 --verbose --algorithm_fine RegistrationComputer_ICP_Generalized paula-flooraligned.ply paula-flooraligned-generalized0133.ply
  ```
  Very similar results. Sigh.