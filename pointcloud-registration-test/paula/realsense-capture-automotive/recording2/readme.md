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

