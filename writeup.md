## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./writeup_images/undistorted_subplot.png "Undistorted subplot"
[image2]: ./test_images/test6.jpg "Road"
[image3]: ./writeup_images/undistorted_test6.jpg "Undistorted Road"
[image4]: ./writeup_images/distorted_diff.jpg "Diff normal and distorted"
[image5]: ./writeup_images/pipeline_straight_lines2.jpg "L and S layers after thresholding and gradient"
[image6]: ./writeup_images/pipeline_undist_straight_lines2.jpg "Summed L and S layers"
[image7]: ./writeup_images/pipeline_processed_straight_lines2.jpg "Warped processed image"
[image8]: ./writeup_images/searched_lines.jpg "Colored lines pixels with fitted polynomial"
[image9]: ./writeup_images/found_lane_test5.jpg "Found lane transformed back to road"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup

So here it is!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

Code for this step is in first cell of IPython notebook located in './project.ipynb'. I used the same code as specified
in example, so I prepared object points, that are corners of chessboard fields in real world, I assume that chessboard is flat.
Then using `cv2.findChessboardCorners()` on grayscale image I obtained (x, y) coordinates of the same corners in pictures,
which I used with `cv2.calibrateCamera()` to calculate distortion matrix. Running `cv2.undistort()` on calibration images
I checked if distortions will be corrected and I receive flat images. Image below present few of calibration images,
that were corrected. They aren't ideal, but it seems, that such correction should be enough, as by now straights are
rather straights than curves.
![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

After taking calibration values for camera on chessboard I can use them for all images - camera distortions wouldn't
change. So I applied cv2.undistort on test images and then I included it as first part of my pipeline. Below I present
original and undistorted image.
![alt text][image2]
![alt text][image3]
It's really hard to find differences by means of lines, they can be visible on cars, but lines look pretty similar. To
see what changed I applied cv2.`absdiff()`
![alt text][image4]
As for lines, they weren't changed much, so I assume that even without correcting distortions I would have nice results.


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used transformation to HLS space and thresholding light and saturation layers. I also tried HSV space and thresholding
values, but I got much worse results on brighten roads, so I've took HLS. Then I applied Sobel filter for in x direction,
and used it to threshold x gradients, so I hope it'll identify lines that are more or less vertical (and then used thresholding).
Second thing, I thresholded saturation channel, to obtain only that areas, that are relatively bright, so I hoped there
were be only lanes.
![alt text][image5]
It looked as good base for whole pipeline if sum those layers and mask them, so next I summed them, and applied my
function `region of interest`. I chose coordinates for masking by visual evaluation, no algorithms for finding them.
So I received something as below (original image is has mistakern R and B layers).
![alt text][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I tried to create function that would identify points for warping itself, but I couldn't manage to get warped image that
I'd be glad of. So after all I decided to take point from straight lines and then to create destination, that would be
700px wide (I assumed that my 700px would be then even to 3.7m in real world), and will have height of whole image,
and will be more less centered. I've took source points that are only part of line, not whole one, as image quality
gets worse when getting closer to horizon

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:| 
| 580, 460      | 300, 0        |
| 195, 720      | 300, 720      |
| 1120, 720     | 1000, 720      |
| 700, 460      | 1000, 0        |

When I got image that I thought that is enough quality to discover lines I decided that my preprocessing pipeline is ready.
Lines for straights are parallel, vertical and I hoped that would have enough pixels for discovery.

![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used histogram in 9 slices of test images, to find windows, where should be pixels for lines, and implemented it
as function `find_windows`. After taking all pixels in regions of those windows I fit second order polynomial using
`np.polyfit`. For video I'm using these polynomial extended by some margins instead of finding windows for next images,
as long as my sanity check goes ok.

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Next in function `transform_back_to_original()` I'm created filled polygon from found lines, and then making perspective
transform (I used inversed matrix by calculating transformation from destination to source).

Here is an example of my result on a test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./writeup_images/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Main issue in current pipeline is finding lines on bright road. Main improvements could be done in image pre processing,
as I think that rest is good enough. If pre-processed images are not that good, the rest sophisticated algorithms are
rather useless. I think that I should try some combination of earlier warping and then additional processing, maybe in
different way, and I think that some algorithms for filling holes in lines also should improve whole pipeline much.
My main problem was when dashed line was found only on really short segment ending with really weird (generally small)
curvature, but I managed to get rid of such segment by providing sanity checks for curvature differences between lines,
and minimal acceptable curvature. If line doesn't fulfill sanity check conditions, first I'm searching again for sliding
windows, and if it's still not good, I take averaged coefficients from previously found lines.