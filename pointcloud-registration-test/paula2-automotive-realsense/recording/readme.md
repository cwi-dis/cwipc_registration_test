## New camerapositions in automotive

Captured paula with 5fps, all preprocessing turned off.

- Cameraconfig is `01-cameraconfig.json`. Grabbed 5th frame as `01-paula.ply`. Plotted median-toself, mean and mode.
- Run
  ```
  cwipc_register --nograb 01-paula.ply --algorithm_multicamera MultiCameraToFloor
  ```
  
  Save as `02-cameraconfig.json`.
  
  Grab `02-paula-flooraligned.ply`.
  
  Plot mean and mode, with floor and without.
  
- Worst tile nofloor is 2, `mean+stddev=0.0386`.
- Try to align using that:
  ```
  cwipc_register --nograb 02-paula-flooraligned.ply --correspondence 0.0386
  ```
  
  Save as `03-cameraconfig.json`.
  
  Grab `03-paula-corr0386.ply`. Visually clear that her front and back have been squashed. But plotting mean without floor anyway.
  Little improvements in the graph, if any.
- Back to `02-cameraconfig.json`, now try interactive, so we can judge which tiles are aligned in which order. To see if alignment is possible at all.
- Run
  ```
  cwipc_register --nograb 02-paula-flooraligned.ply --algorithm_multicamera MultiCameraIterativeInteractive
  ```
  - Start with `3` (suggested)
  - Step 1: Use `2` and `corr=0.004` (suggested)
	  - Did a little move, in the wrong direction.
	  - Plot suggests `0.15` might be the right correspondence.
  - Retry: use `2` and `corr=0.15`.
  - That is worse, because now it has aligned her front to her back.
  - Retry with `0`. Exactly the same problem.
  - Give up.
  
### See if we can do a partial registration

- Back to `02-cameraconfig.json` And `02-paula-flooraligned.ply`.
- Inspect the colored capture pointcloud. Camera `4` and `2` look like they may have a chance at success.
