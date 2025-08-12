Steps taken:

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
