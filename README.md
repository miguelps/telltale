telltale
========

Detection of resized images, uncalibrated scanners and other problems with digitized images.


Up-scaled images
================

Up-scaled images is defined here as bitmaps that at some point has been resized to a dimension larger
than the original. This is sometimes done behind the scenes by scanner software, sometimes explicitly
by operators caring more about the bottom line than delivering the best product to customers.

ResizeDetect is currently detecting simple x2 up-scales, with the typical case being 200DPI scans
resized to 400DPI. The detection is extremely reliable for the trivial Nearest-Neighbour scaling and
fairly reliable for a real-world sample from a scanner which used what looks like a mix between
Nearest-Neighbour and Bi-linear interpolation (sample hvide_19681224_1.png in the test resources).

The working hypothesis is that the alleged up-scaled image is made up of 2x2 pixel super-pixels where
the colors (or greyscales) of the pixels within a super-pixel are fairly uniform. The trivial example
is again Nearest-Neighbour, where all the pixels within a super-pixel are identical.

Consider the 4x4 pixel image

    RrGg
    rRgG
    BbTt
    BbtT

Where each letter represents a color and `R` is close to `r` etc. As can be intuitively seen, the
image is made up of 2x2 super-pixels. One problem is that the image can be cropped other otherwise
manipulated so that the super-pixels are offset by 1 pixel horizontally, vertically or both. So we
need to check offsets (0, 0), (0, 1), (1, 0) and (1,1).

For each offset, four hypothesises are stated:
1. The super-pixel is made up of uniform pixels: Offset (0, 0) in the sample.
2. The super-pixel is made up of two pairs of pixels, with the two top pixels being one pair and the
   two bottom pixels being the second pair: Offset (0, 1) in the sample.
3. The super-pixel is made up of two pairs of pixels, with the two left pixels being one pair and the
   two right pixels being the second pair: Offset (1, 0) in the sample.
4. The super-pixel is made up of non-uniform pixels: Offset (1, 0) in the sample.

The chosen method for comparison of pixels is the standard deviation. For hypothesis 1 and 4, this is
simple calculated over all 4 pixels. For 2 and 3, it is calculated for each group (2 pixels each) and
the mean is used. Finally the mean of all the standard deviations is calculated.

There is not hard science behind the choice of calculations: Taking the mean for the two standard
deviations in case 2 and 3 might be wrong. Maybe it would be better to use the median instead of the
mean for the overall calculation. This really calls for a sanity check by someone with a better grasp
of statistics.

When means has been calculated for all 4 offsets, we are able to guess whether our overall theory of
Nearest-Neighbour-like up-scaling is plausible: Each offset should match one and only one of the 4
hypothesises and no hypothesis should be matched by 2 offsets. As hypothesis 1 and 4 uses the same
calculation, this means that we have three numbers for all offsets: ABCD (all 4 pixels), AB_CD (two
pairs of pixel rows) and AC_BD (two pairs of pixel columns). The numbers for the different hypothesis
should thus be

| Hypothesis | ABCD        | AB_CD | AC_BD |
|------------|-------------|-------|-------|
|          1 | Low         | Low   | Low   |
|          2 | Medium/high | Low   | High  |
|          3 | Medium/high | High  | Low   |
|          4 | High        | High  | High  |

So what are low, medium and high values? That depends on the overall image. The higher the overall
contrast, the higher the spread of the numbers will be. Conversely, images with very low contrast
will result in lower spread.

A potential improvement would be a calculation of overall contrast in the images - maybe the average
deviation for 16x16 pixel blocks? This would make it easier to judge whether the hypothesis numbers
are due to chance or up-scaling.


- Toke Eskildsen, te@statsbiblioteket.dk (work) / te@ekot.dk (private)
