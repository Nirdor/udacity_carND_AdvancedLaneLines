# Writeup

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

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./output_images/test1_undistorted.png "Road Transformed"
[image3]: ./output_images/thresholds.png "Thresholds"
[image4]: ./output_images/warped_straight_lines.png "Warp Example"
[image5]: ./output_images/color_fit_lines.png "Fit Visual"
[image6]: ./output_images/example_output.png "Output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in [AdvancedLaneLines.ipynb](./AdvancedLaneLines.ipynb) 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

The complete pipeline for finding lanes in an image is in the 6th code cell in [AdvancedLaneLines.ipynb](./AdvancedLaneLines.ipynb). Helper functions are in the 3rd code cell.

#### 1. Provide an example of a distortion-corrected image.

For this step, I use the OpenCV functions `cv2.getOptimalNewCameraMatrix` and `cv2.undistort` on the distorted image.
Here is the Result:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate three binary images. One with thresholding white pixels one with thresholding yellow pixels and one using a combination of thresholds and gradient thresholds. (thresholding steps at the beginning of the 6th code cell in the IPython notebook. Here's an example of my output for this step. 

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform()`, which appears in lines 36 through 44 in the 3rd code cell of the IPython notebook.  The `transform()` function takes as input an image (`img`) and returns a transformed image. I chose to pick the  source points by hand and calculate the destination points in the following manner:

```python
h, w = img.shape[:2]
dx = 200
dst = np.float32([[dx, w], [dx, 0], [h - dx, 0], [h - dx, w]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 265, 685      | 200, 1280     | 
| 620, 440      | 200, 0        |
| 680, 440      | 520, 0        |
| 1050, 685     | 520, 1280     |


My resulting image after the transform changes the dimensions in x and y direction (height 1280, width 720), because the upright format is a more realistic representation of the lane and leads to better results.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identified lane-line pixel using a sliding window algorithm (lines 56 through 160 in 3rd code cell). I detected the two bottom starting points based on the lanes from the last frame. If this was not possible by using a histogram.

Going up from these two points I used sliding windows to detect connected parts of the lane. After detecting all points I fit a second order polynomial to represent the lane.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines 163 through 182 in my code in the 3rd code cell of the IPython notebook.
I transformed the lane coordinates in world space and calculated the curvature using the formular:

`((1 + (2*A*p*ym_per_pix + B)**2)**1.5) / np.absolute(2 * C)` where A, B and C are the parameters of the transformed polynomial and p is the evaluated point.

I use the average curvature over 5 points in the bottom 100 pixels <> 5m of my transformed image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 87 through 124 in my code in the 6th code cell of the IPython notebook. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced the issue that combining different color and gradient thresholds often leave to many non-lane pixels. To solve this issue i first try to find the lanes only using yellow and white color thresholds and only if that fails I try to find the lane using the complete combination of all methods.

My pipeline will likely fail if it encounters a curve that is too narrow, so that it will leave the transformed image patch on the side. To make it more robust in these cases I could choose different source points for the transformation that include more space on the side.
