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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd cell of AdvancedLaneLines.ipynb

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

Before and afters for each calibration can be found in the 3rd cell of AdvancedLaneLines.ipynb.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I saved the camera calibration parameters as a pickle, so that the calibration only has to be performed once.

In the 4th cell of AdvancedLaneLines.ipynb, I retrieve the parameters and use them to undistort highway images.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. All of the thresholding methods are defined
in the 5th cell.  Examples of thresholded images are in the 6th cell.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I selected 4 points in straight_lines1.jpg--2 points on the left lane, 2 on the right, 2 near the car, and 2 far away.  
These points were used as the source points for the perspective transformation.  I used the cv2.getPerspectiveTransform
function to create a transformation matrix that mapped the points so that the new perspective was an aerial view in which
the left and right lane points ran parallel.  I used cv2.warpPerspective method and the aforementioned transformation matrix
to do all future perspective shifts.

| Source        | Destination   |
|:-------------:|:-------------:|
| 286, 666      | 200, 715        |
| 1020, 666      | 1000, 715      |
| 527, 500     | 200, 300      |
| 761, 500      | 1000, 300        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image
and its warped counterpart to verify that the lines appear parallel in the warped image.  This is done in cell 4.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I created a histogram of all nonzero pixels in the warped binary images.  I interpreted the highest peak in the left half of the
image as the left lane and the peak from the right half as the right lane.  I used sliding window technique to find all of the
points that made up the right and left lanes.  I used np.polyfit to fit a 2nd degree polynomal to each set of lane points.
I then used the cv2.fillPoly function to draw a lane polygon bounded by the two polynomials.  Finally, I unwarped the lane polygon
using the inverse transformation matrix and overlayed it on the original image using cv2.addWeighted.

The code for this is in the 9th cell.  

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of the curvature in the 8th cell.  I used the formula provided in the lectures.  I calculated the position
of the vehicle relative to the center of the lane in the second code block of cell 11.  I calculated the center of the lane by
averaging the points nearest to the vehicle in the right and left lanes.  I then subtracted this value from the center of the
image, which I took to be equivalent to the vehicle's position.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the process_image method in cell 11.
---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Look at cell 13 of AdvancedLaneLines.ipynb for my video results.

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline works pretty well on the project_video but falls on either of the challenge videos.  I think the thresholding
functions are failing to weed out shadows and other non-lane color gradients.  I could experiment more with the threshold
values being used.  

Right now my sanity check only looks at radius of curvature value.  I should add other safeguards to ensure that the lane
spacing makes sense and that the lanes are relatively parallel.
