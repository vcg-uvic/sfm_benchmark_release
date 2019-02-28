# Image Matching: Local Features and Beyond: Phototourism dataset

This repository contains source code and instructions to access the data and format submissions for the Phototourism dataset published as part of the IMW2019 Challenge. For details please refer to the website:

* [CVPR 2019 workshop on Image Matching: Local Features and Beyond](https://image-matching-workshop.github.io)
* [Phototourism dataset: description, tasks, and download links](https://image-matching-workshop.github.io/challenge)
* [Leaderboards](https://image-matching-workshop.github.io/leaderboard)

# Downloading the data

* [Download link for the training data: SOON](http://)
* [Download link for the test data: 450 Mb](http://webhome.cs.uvic.ca/~kyi/files/2019/image-matching/imw2019-test.tar.gz)

# Resources for reading the training data

For some examples on how to access the training data (images, poses, depth maps)
please check `parse_data.ipynb`. Dependencies are given in `deps.txt`.

# Formating your submissions

Submissions may fall under one of three categories: features (F), matches (M), and
poses (P). Task 1 (stereo) accepts all 3. Task 2 accepts F and M, but not P,
which are generated by COLMAP over sampled subsets which will remain private,
for fairness.

When submitting, please zip the results into a single file along with a text
file with some meta-data.
  * **Method name:** should be short.
  * **Details.** description of the method.
  * **Task:** stereo, mvs, or both.
  * **Type:** F (features), M (matches) or P (poses). For sparse methods, F and M
      are possible, in which case we'll generate our matches from your features in addition to the matches you provide and evaluate both variants.
  * **List of authors:** this information is mandatory, please specify if you wish the
      submission to be anonymous.
  * **Contact e-mail:** please note that it will be listed in the website (unless
      the submission is anonymous).
  * **Link:** if desired.

You can find an example in the `example` folder. It contains four images for the
`reichstag` dataset, for which we have extracted features with SIFT and matches
using simple nearest-neighbour matching. Let's consider a toy set with only four
images:

```bash
$ ls imw2019-test/reichstag
06373813_9127207861.jpg
63790741_1504116525.jpg
77023786_7168337563.jpg
92481126_2062182859.jpg
```

A local-feature based submission will require a list of keypoints and
descriptors:

```bash
$ cd example; tar xvf submission-example.tar.gz > /dev/null

$ cd submission-example/reichstag

$ ls keypoints
06373813_9127207861.h5
63790741_1504116525.h5
77023786_7168337563.h5
92481126_2062182859.h5

$ ls descriptors
06373813_9127207861.h5
63790741_1504116525.h5
77023786_7168337563.h5
92481126_2062182859.h5
```

Keypoints should contain an array with N keypoints with 0-indexed (y, x)
coordinates, with the origin in the top left corner. We do not use keypoint
score, orientation or scale but feel free to include that information in the
array if you want.

```
>>> import h5py
>>> f = h5py.File('keypoints/06373813_9127207861.h5', 'r')
>>> [k for k in f]
['keypoints']
>>> f['keypoints'].shape
(512, 2)
>>> f['keypoints'][:3]
array([[338.39126587, 413.37594604],
       [428.95111084, 332.44958496],
       [452.37286377, 982.00183105]])

```

Descriptors are stored in a separate file but should be consistent with the list
of keypoints. We expect them to be stored in a floating point format, and use
the L2 distance for comparisons. If you require a different set-up (e.g. binary
descriptors and the Hamming distance) please specify it when you contact us.

```
>>> f = h5py.File('descriptors/06373813_9127207861.h5', 'r')
>>> [k for k in f]
['descriptors']
>>> f['descriptors'].shape
(512, 128)
>>> f['descriptors'][0]
array([ 80.,   6.,   3.,  73.,  33.,   0.,   0.,   2., 154.,  50.,   9.,
         1.,   1.,   0.,   0.,  12.,  46.,  44.,  22.,   4.,   5.,   0.,
         0.,   1.,   0.,   1.,   3.,  18.,  13.,   0.,   0.,   0., 107.,
         8.,   5.,  33.,  36.,   2.,   1.,  38., 152., 142.,  96.,  13.,
        16.,   3.,   1.,  20.,  84., 154.,  94.,   5.,   2.,   0.,   0.,
         1.,  21.,  27.,   7.,  18.,   8.,   0.,   0.,   0., 101.,   3.,
         2.,  13.,  23.,   4.,   3.,  88.,  64.,  14.,  13.,  38.,  86.,
        13.,   3.,  33., 154.,  55.,   7.,   4.,  10.,   2.,   1.,  31.,
        74.,   8.,   0.,   5.,   8.,   2.,   0.,   6.,  37.,   8.,  11.,
        24.,   7.,   3.,   3.,  33.,  61.,  10.,  17.,  79.,  16.,   0.,
         0.,   5., 154.,   6.,   2.,   3.,   1.,   0.,   0.,  31.,  49.,
         2.,   1.,   5.,   6.,   2.,   0.,   6.], dtype=float32)

```

If you want to specify your own matches, you will have to provide files matching
every possible pair of images in each sequence. Please create the pairwise h5
files following the `LARGEST_KEY-SMALLEST_KEY.h5` convention, without
duplicates. For instance, for this toy set the list would contain six files:

```bash
$ ls matches
63790741_1504116525-06373813_9127207861.h5
77023786_7168337563-06373813_9127207861.h5
77023786_7168337563-63790741_1504116525.h5
92481126_2062182859-06373813_9127207861.h5
92481126_2062182859-63790741_1504116525.h5
92481126_2062182859-77023786_7168337563.h5
```

Please note that this is only meant to generate a list of putative matches from
the NxN possible matches. Our default strategy is nearest neighbour matching
from the first image to the second. The list will be processed by robust
matching strategies such as RANSAC afterwards.
The format of a matches file is as follows:

```
>>> f = h5py.File('matches/63790741_1504116525-06373813_9127207861.h5', 'r')
[k for k in f]
['matches']
>>> f['matches'].shape
(2, 512, 1)
>>> f['matches'][:,:10,:]
array([[[  0],
        [  1],
        [  2],
        [  3],
        [  4],
        [  5],
        [  6],
        [  7],
        [  8],
        [  9]],

       [[398],
        [ 87],
        [ 18],
        [ 18],
        [127],
        [242],
        [345],
        [199],
        [280],
        [392]]])
```

Where keypoint 0 in `keypoints/63790741_1504116525.h5` matches with keypoint 398
in `06373813_9127207861.h5` and so on.

The instructions to submit poses for the stereo track will be published soon, please stay tuned.
