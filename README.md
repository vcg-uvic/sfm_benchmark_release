# Image Matching: Local Features and Beyond: Phototourism dataset

**The submission format has changed slightly as of April 24, 2019. Please follow
the current instructions if you plan to submit a new entry.**

This repository contains source code and instructions to access the data and
format submissions for the Phototourism dataset published as part of the IMW2019
Challenge. The challenge will remain open after the workshop takes place. For
details on the current edition please refer to the website:

* [CVPR 2019 workshop on Image Matching: Local Features and Beyond](https://image-matching-workshop.github.io)
* [Phototourism dataset: description, tasks, and download links](https://image-matching-workshop.github.io/challenge)
* [Leaderboards](https://image-matching-workshop.github.io/leaderboard)

# Downloading the data

* [Download link for the training data: 127 Gb](http://https://gfx.uvic.ca/nextcloud/index.php/s/75JNSoggacQhOkQ)
* [Download link for the test data: 450 Mb](http://webhome.cs.uvic.ca/~kyi/files/2019/image-matching/imw2019-test.tar.gz)

# Resources for reading the training data

For some examples on how to access the training data (images, poses, depth maps)
please check `parse_data.ipynb`. Dependencies are given in `deps.txt`.

# Challenge submissions

Results can be submitted here:

* [Submission link](https://gfx.uvic.ca/nextcloud/index.php/s/1z3K0JzRY4w2cfk)

The submission website is password-protected to prevent abuse, please contact
the organizers at [imagematching@uvic.ca](mailto:imagematching@uvic.ca) for the
password (please account for short delays in answering and uploading close the
deadline).

Submissions may fall under one of three categories: features (F), features and
matches (M), and poses (P). Task 1 (stereo) accepts all 3. Task 2 accepts F and
M, but not P, which are generated by COLMAP over sampled subsets which will
remain private, for fairness. Please note that the maximum number of keypoints
per image is 8000. Entries will be broken down by the number of keypoints, with
the following categories: up to 256, 512, 1024, 2048, and 8000 keypoints. We
will however not make a distinction for the purpose of awarding prizes.

Multiple submissions can be allowed but please refrain from parameter search on
the test set, running the evaluation is very time-consuming.

Only complete submissions will be evaluated. We will contact you if we detect
problems with your submission. However, please account for delays in evaluating
your submissions; this is a costly process!

# Formating your submissions

When submitting, please zip the results into a single file along with a text
file with some meta-data.
  * **Method name:** should be short.
  * **Details.** description of the method.
  * **Type:** F (features), M (matches) or P (poses, currently unavailable). For sparse methods, F and M
      are possible, in which case we'll generate our matches from your features in addition to the matches you provide and evaluate both variants. Dense methods which retrieve the pose directly (P) will be allowed in future editions of the challenge, please contact the organizers if you are interested.
  * **Descriptor dimensionality:** e.g. 128 float32, 256 uint8.
  * **List of authors:** this information is mandatory, please specify if you wish the
      submission to be anonymous.
  * **Contact e-mail:** Please specify a contact address. We will not publish it
      if you wish to remain anonymous but we need it to get back to you.
  * **Link:** if desired.

You can find an example of a mock submission in the `example` folder. It
contains four images for the `reichstag` dataset, for which we have extracted
features with SIFT and matches using simple nearest-neighbour matching. Let's
consider a toy set with only four images:

```bash
$ ls imw2019-test/reichstag
06373813_9127207861.jpg
63790741_1504116525.jpg
77023786_7168337563.jpg
92481126_2062182859.jpg
```

A local-feature based submission will require HDF5 files containing keypoints,
descriptors, and optionally the matches:

```bash
$ cd example; tar xvf submission-example.tar.gz > /dev/null

$ cd submission-example/reichstag/your_method

$ ls
descriptors.h5  keypoints.h5  matches.h5
```

The keypoint/descriptor files must contain one key for each image. The keypoint
file must contain an array with N keypoints with 0-indexed (x, y) coordinates,
with the origin in the top left corner. We do not use scale or orientation but
feel free to include them. If you sort them by score we can run multiple
evaluations subsampling the number of keypoints.

```
>>> import h5py
>>> with h5py.File('keypoints.h5') as f 
>>>     for k, v in f.items():
>>>         print((k, v.shape))

('06373813_9127207861', (512, 2))
('63790741_1504116525', (512, 2))
('77023786_7168337563', (512, 2))
('92481126_2062182859', (513, 2))
```

Descriptors should be consistent with the list of keypoints. We expect them to
be stored in a floating point format, and use the L2 distance for comparisons.
If you require a different set-up (e.g. binary descriptors and the Hamming
distance) please specify it when you contact us.

```
>>> import h5py
>>> with h5py.File('descriptors.h5') as f 
>>>     for k, v in f.items():
>>>         print((k, v.shape))

('06373813_9127207861', (512, 128))
('63790741_1504116525', (512, 128))
('77023786_7168337563', (512, 128))
('92481126_2062182859', (513, 128))
```

If you want to specify your own matches, you will have to provide matches for
every possible pair of images in each sequence. The match file should contain a
key for every match using the convention `LARGEST_KEY-SMALLEST_KEY`. For
instance, for this toy set the file would contain six keys, as follows:

```
>>> import h5py
>>> with h5py.File('matches.h5') as f 
>>>     for k, v in f.items():
>>>         print((k, v.shape))

('63790741_1504116525-06373813_9127207861', (2, 512))
('77023786_7168337563-06373813_9127207861', (2, 512))
('77023786_7168337563-63790741_1504116525', (2, 512))
('92481126_2062182859-06373813_9127207861', (2, 513))
('92481126_2062182859-63790741_1504116525', (2, 513))
('92481126_2062182859-77023786_7168337563', (2, 513))
```

Each of these entries stores a list of matches between the image encoded by the
first key and the second key, such that:

```
>>> with h5py.File('matches.h5') as f:
>>>    print(f['63790741_1504116525-06373813_9127207861'].value)

[[  0   1   2 ... 509 510 511]
 [398  87  18 ... 502 467 458]]
```

So that keypoint 0 in `63790741_1504116525.jpg` matches with keypoint 398 in
`06373813_9127207861.jpg` and so on. If this list is not specified, we will use
simple nearest neighbour matching as the default strategy. Whether matches are
provided or not, this will be followed by robust matching strategies (RANSAC or
SfM with COLMAP).

The instructions to submit poses for the stereo track will be published after
the first edition of the challenge.

