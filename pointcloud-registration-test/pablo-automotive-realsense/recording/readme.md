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

## And again.

Starting with `11-cameraconfig.json`, copy to `21-cameraconfig.json` and add capturing of the floor.

Align the floor, save the result as `22-cameraconfig.json`. Capture `22-pablo.ply`.

Use `8` as base. Then align `4` (not `2` as suggested).

This looks very promising. Next `2`. Again good.

And `1` also looks good.

> **Note**: the only interactive choice made was to do 4 in stead of 2 as the second step.
> 
> Also, most step pre/post results showed all (or almost all) statistics better post than pre.
>
> Note also that I must try later with point2point and maybe point2plane in stead of generalized ICP.
> 

Save as `23-cameraconfig.json`, capture `23-pablo.ply`.

Created the alignment plots for 22 and 23 pablo. They clearly show a grand improvement. Command used:

```
ython -m cwipc.scripts.cwipc_analyze_registration --measure median --measure mean --plot --ignore_floor 23-pablo.ply
```

Pablo-22 statistics:

```
Alignment 0x1 to 0xfe: correspondence: 0.0213, count: 14004, percentage: 52%, mean=0.0230, stddev=0.0166, tmean=0.0213, median=0.0203
Alignment 0x2 to 0xfd: correspondence: 0.0228, count: 15493, percentage: 56%, mean=0.0252, stddev=0.0200, tmean=0.0228, median=0.0198
Alignment 0x4 to 0xfb: correspondence: 0.0308, count: 14666, percentage: 55%, mean=0.0344, stddev=0.0288, tmean=0.0308, median=0.0267
Alignment 0x8 to 0xf7: correspondence: 0.0213, count: 17019, percentage: 55%, mean=0.0234, stddev=0.0175, tmean=0.0213, median=0.0191

```

Pablo-23 statistics:

```
Alignment 0x1 to 0xfe: correspondence: 0.0203, count: 15646, percentage: 58%, mean=0.0244, stddev=0.0230, tmean=0.0203, median=0.0165
Alignment 0x2 to 0xfd: correspondence: 0.0183, count: 16163, percentage: 58%, mean=0.0219, stddev=0.0202, tmean=0.0183, median=0.0149
Alignment 0x4 to 0xfb: correspondence: 0.0216, count: 15214, percentage: 57%, mean=0.0263, stddev=0.0252, tmean=0.0216, median=0.0179
Alignment 0x8 to 0xf7: correspondence: 0.0216, count: 17548, percentage: 56%, mean=0.0240, stddev=0.0189, tmean=0.0216, median=0.0184
```

The medians show a pretty good improvement, the means are a bit of a mixed bag.

Let's see if making another pass leads to more improvements.

Order `8`, `4` (not `2` as suggested), `2`, `1`.
Visually tile `4` and `1` seem to have improved a little. Plots and statistics only show significant improvement for `1`.

Save `24-cameraconfig.json`, captured `24-pablo.ply`, created `24-pablo.png`. Here are the statistics:

```
Alignment 0x1 to 0xfe: correspondence: 0.0211, count: 16476, percentage: 61%, mean=0.0257, stddev=0.0258, tmean=0.0211, median=0.0150
Alignment 0x2 to 0xfd: correspondence: 0.0187, count: 17065, percentage: 61%, mean=0.0229, stddev=0.0230, tmean=0.0187, median=0.0140
Alignment 0x4 to 0xfb: correspondence: 0.0212, count: 16257, percentage: 61%, mean=0.0270, stddev=0.0287, tmean=0.0212, median=0.0162
Alignment 0x8 to 0xf7: correspondence: 0.0223, count: 17698, percentage: 57%, mean=0.0251, stddev=0.0205, tmean=0.0223, median=0.0188
```

The medians have gone down, the means have gone up a little. Again. 

> Thinking about this: maybe it is to be expected, because the long tail has a relatively large effect on the mean.
> 
> Maybe I should try a "modified mean", where I ignore the bottom and top `5%` of the points when computing the mean?
> Ah, that is called the _Trimmed Mean_.

Implemented `tmean`. Re-computed the three sets of statistics above.

Now from 22-pablo to 23-pablo the trimmed means do go down (unlike the means). For 23 to 24 it doesn't seem to make a difference.
