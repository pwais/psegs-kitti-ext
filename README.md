# PSegs KITTI Extension

[![License: CC BY-NC-SA 3.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%203.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/3.0/)

This project contains code and data related to the
[KITTI](http://www.cvlibs.net/datasets/kitti/index.php) dataset and is
licensed the same as KITTI.  This project is an extension 'module' atop
the [PSegs](https://github.com/pwais/psegs) project.

# Structure
 * [main.ipynb](main.ipynb) - A Jupyter Notebook used to study the density
     of KITTI labels and produce the `assets` described next.
 * [assets](assets) - A few Parquet-encoded tables that store the mapping
     between the KITTI Object and Tracking Benchmarks and the Raw Sync data.
 * [ps_external_test_fixtures](ps_external_test_fixtures) - These are fixtures
     created using KITTI data and code in [PSegs](https://github.com/pwais/psegs).
     They are hosted here and licensed using the same KITTI-compatible license
     used in this repository.

# Key Results

The KITTI Detection and Tracking Benchmarks contain copies of subsets of the Raw Data.  One
goal of this project is to study the Benchmark <=> Raw Data mapping to gain a better understanding
of the benchmarks.  For an introduction to the KITTI dataset see
[the KITTI dataset paper](http://www.cvlibs.net/publications/Geiger2013IJRR.pdf).  You can find 
code for each of the results described below in [main.ipynb](main.ipynb).

#### The `test` Split Raw Data is Private
The Benchmarks contain `train` and `test` splits, and we find that the `train` splits sample data
exclusively from the publicly available Raw Sync Data.  For the `test` splits, we verify that there is
**no** Raw Data available.  Consequences:
 * Timestamps for images and velodyne scans are thus only available publicly for the `train` split
    (and we recover those timestamps in this project).
 * It is unknown how correlated the `test` and `train` splits are; we can't determine if they were
    recorded on the same days.

#### There Are Missing Velodyne Scans
The Object Benchmark appear to have 100% Velodyne coverage, but the Tracking Benchmark is missing
4 Velodyne scans.  Be sure you either drop these frames or make your training pipeline robust
to missing data.  
```
+--------------------------+--------+-----+
|benchmark                 |topic   |num  |
+--------------------------+--------+-----+
|data_object_image_2.zip   |image_02|14999|
|data_object_velodyne.zip  |velodyne|14999|
|data_tracking_image_2.zip |image_02|19103|
|data_tracking_velodyne.zip|velodyne|19099|
+--------------------------+--------+-----+
```

#### Sensor Sample Rates are Consistently 10Hz
The camera-lidar delay is a fairly consistent ~6ms, and sensors are synchronized at 10Hz
consistently across all Tracking Benchmark segments.  Here's a sample:

```
Segment 2011_09_26_drive_0028_sync
Start 2011-09-26 13:22:48 	Duration 44.445416223 sec
Num Images 430 (Lidars 430)
            Series   Freq Hz  Diff img (msec)  std (msec)   Duration  Support
0         image_00  9.655935         6.474308    0.678468  44.428635      430
1         image_01  9.655925         6.392096    0.660889  44.428678      430
2         image_02  9.655940         0.000000    0.000000  44.428613      430
3         image_03  9.655937         0.555579    0.096519  44.428622      430
4             oxts  9.655230         6.653511    3.385045  44.431878      430
5  velodyne_points  9.655946        10.507161    0.138202  44.428582      430

Segment 2011_09_26_drive_0064_sync
Start 2011-09-26 14:38:23 	Duration 58.865393697 sec
Num Images 570 (Lidars 570)
            Series   Freq Hz  Diff img (msec)  std (msec)   Duration  Support
0         image_00  9.668892         6.536892    0.794485  58.848523      570
1         image_01  9.668857         6.396500    0.656120  58.848734      570
2         image_02  9.668954         0.000000    0.000000  58.848144      570
3         image_03  9.668933         0.541377    0.065483  58.848272      570
4             oxts  9.669516         6.597285    3.088882  58.844726      570
5  velodyne_points  9.668951        10.471518    0.157921  58.848163      570
```

Here's a segment with missing velodyne scans:
```
Segment 2011_09_26_drive_0009_sync
Start 2011-09-26 13:08:24 	Duration 46.184051925000006 sec
Num Images 447 (Lidars 443)
            Series       Freq Hz  Diff img (msec)    std (msec)      Duration  Support
0         image_00  9.660596e+00     6.480731e+00  6.871421e-01  4.616692e+01      447
1         image_01  9.660584e+00     6.377872e+00  6.091369e-01  4.616698e+01      447
2         image_02  9.660614e+00     0.000000e+00  0.000000e+00  4.616684e+01      447
3         image_03  9.660577e+00     5.473424e-01  6.646240e-02  4.616701e+01      447
4             oxts  9.659563e+00     6.396858e+00  3.182116e+00  4.617186e+01      447
5  velodyne_points -3.386375e-07     1.178562e+10  1.240293e+11 -1.317043e+09      447
```

#### The `test` and `train` Sets Do Not Contain The Same Data
We computed the SHA-1 of all available files (and even the SHA-1 of the *decoded* images / velodyne scans)
and verified there is zero overlap between `test` and `train`.

#### The Object Benchmark Overweights Certain Raw Data Segements
The Object Benchmark is *not* a uniform sampling from available data.  Here's a snapshot showing
that this benchmark has bias towards a few drive segments:
```
+-----+-------------------------+-----------+------------------------------+---------+-------+--------------------+
|split|benchmark                |category   |segment                       |n_labeled|n_total|frac_labeled        |
+-----+-------------------------+-----------+------------------------------+---------+-------+--------------------+
|train|data_object_image_2.zip  |city       |2011_09_26_drive_0014_sync.zip|235      |314    |0.7484076433121019  |
|train|data_object_image_2.zip  |road       |2011_09_26_drive_0015_sync.zip|215      |297    |0.7239057239057239  |
|train|data_object_image_2.zip  |city       |2011_09_26_drive_0056_sync.zip|197      |294    |0.6700680272108843  |
|train|data_object_image_2.zip  |city       |2011_09_26_drive_0009_sync.zip|287      |447    |0.6420581655480985  |
|train|data_object_image_2.zip  |city       |2011_09_26_drive_0095_sync.zip|166      |268    |0.6194029850746269  |
|train|data_object_image_2.zip  |road       |2011_09_26_drive_0101_sync.zip|577      |936    |0.6164529914529915  |
|train|data_object_image_2.zip  |city       |2011_09_28_drive_0001_sync.zip|65       |106    |0.6132075471698113  |
|train|data_object_image_2.zip  |city       |2011_09_26_drive_0018_sync.zip|154      |270    |0.5703703703703704  |
|train|data_object_image_2.zip  |residential|2011_09_26_drive_0023_sync.zip|258      |474    |0.5443037974683544  |
|train|data_object_image_2.zip  |city       |2011_09_26_drive_0096_sync.zip|243      |475    |0.511578947368421   |
|train|data_object_image_2.zip  |person     |2011_09_28_drive_0078_sync.zip|2        |37     |0.05405405405405406 |
|train|data_object_image_2.zip  |road       |2011_09_26_drive_0052_sync.zip|4        |78     |0.05128205128205128 |
|train|data_object_image_2.zip  |residential|2011_09_26_drive_0079_sync.zip|5        |100    |0.05                |
|train|data_object_image_2.zip  |campus     |2011_09_28_drive_0016_sync.zip|9        |186    |0.04838709677419355 |
|train|data_object_image_2.zip  |person     |2011_09_28_drive_0077_sync.zip|2        |42     |0.047619047619047616|
|train|data_object_image_2.zip  |person     |2011_09_28_drive_0071_sync.zip|2        |43     |0.046511627906976744|
|train|data_object_image_2.zip  |person     |2011_09_28_drive_0066_sync.zip|1        |29     |0.034482758620689655|
|train|data_object_image_2.zip  |person     |2011_09_28_drive_0156_sync.zip|1        |30     |0.03333333333333333 |
|train|data_object_image_2.zip  |person     |2011_09_28_drive_0138_sync.zip|2        |68     |0.029411764705882353|
|train|data_object_image_2.zip  |person     |2011_09_28_drive_0065_sync.zip|1        |39     |0.02564102564102564 |
|train|data_object_image_2.zip  |person     |2011_09_28_drive_0070_sync.zip|1        |39     |0.02564102564102564 |
+-----+-------------------------+-----------+------------------------------+---------+-------+--------------------+
```

#### The Tracking Benchmark is Derived from a Few Specific Raw Data Segments

The Tracking Benchmark consists of a few particular Raw Data segments that were fully labeled:
```
+-----+-------------------------+-----------+------------------------------+---------+-------+--------------------+
|split|benchmark                |category   |segment                       |n_labeled|n_total|frac_labeled        |
+-----+-------------------------+-----------+------------------------------+---------+-------+--------------------+
|train|data_tracking_image_2.zip|city       |2011_09_26_drive_0091_sync.zip|340      |340    |1.0                 |
|train|data_tracking_image_2.zip|city       |2011_09_28_drive_0001_sync.zip|106      |106    |1.0                 |
|train|data_tracking_image_2.zip|city       |2011_09_28_drive_0002_sync.zip|376      |376    |1.0                 |
|train|data_tracking_image_2.zip|campus     |2011_09_28_drive_0021_sync.zip|209      |209    |1.0                 |
|train|data_tracking_image_2.zip|campus     |2011_09_28_drive_0043_sync.zip|145      |145    |1.0                 |
|train|data_tracking_image_2.zip|road       |2011_09_29_drive_0004_sync.zip|339      |339    |1.0                 |
|train|data_tracking_image_2.zip|city       |2011_09_29_drive_0071_sync.zip|1059     |1059   |1.0                 |
|train|data_tracking_image_2.zip|road       |2011_10_03_drive_0047_sync.zip|837      |837    |1.0                 |
+-----+-------------------------+-----------+------------------------------+---------+-------+--------------------+
```

#### The Object and Tracking Benchmarks Do Overlap

We find that about 18% of the train sets overlap, and about 20% of the test sets overlap:
```
+---------+-----------+----+-------------+-----------+
|obj_split|track_split|n   |frac_tracking|frac_object|
+---------+-----------+----+-------------+-----------+
|null     |test       |7542|0.395        |0.0        |
|null     |train      |4910|0.257        |0.0        |
|test     |null       |3965|0.0          |0.264      |
|test     |test       |3553|0.186        |0.237      |
|train    |null       |4383|0.0          |0.292      |
|train    |train      |3098|0.162        |0.207      |
+---------+-----------+----+-------------+-----------+
```

#### The Tracking Benchmark is a Subset of Available Tracklet Labels
The KITTI Raw Data includes `tracklet.xml` labels for many drive segments.  We find there are about 17
Raw Data segments that have `tracklet.xml` labels yet are absent from the Tracking Benchmark.

