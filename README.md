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
[image2]: ./examples/road_transform.png "Road Transformed"
[image3]: ./examples/binary_combo_example.png "Binary Example"
[image4]: ./examples/warped_straight_lines.png "Warp Example"
[image6]: ./examples/example_output1.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the notebook section marked **Calculate object points for camera calibration**.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I verified the image distortion correction against the test images in the code section marked **Test calibration on the test images**.  Here is one example:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image in the code section marked **Create thresholded binary images**.  I chose gradient and color saturation thresholds from the class examples.  Here is one example:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I scanned an example file and manually chose the following transformation points based on experimentation:

| Source        | Destination   |
|:-------------:|:-------------:|
| 582, 467      | 280, 0        |
| 714, 467      | 1120, 0      |
| 1120, 719     | 1120, 719      |
| 280, 719      | 280, 719       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code can be found in the `Line` class (modified from the class example) in the code section marked **Helper Line class**.

For line fitting, I used the histogram method from the class example.  This method calculates which pixels on the x-axis are the most heavily drawn.  By choosing the peaks on the left and right sides, I was able to make a reasonable assumption about where to place line coordinates starting with the bottom of the image.  The method then uses "sliding windows" to move up the image and search horizontally within a relatively narrow margin for the next set of coordinates.  The code can be found in the `fit_line_histogram` method (which works for both left and right lines).

The `Line` class also keeps track of previous line fits (I chose to keep the last 8).  Rather than use the histogram method for all of the frames, I made use of the previous set of frames as a starting point for the search.  This code can be found in the `fit_line` method.

If there are no previous reads or fitting the line from previous reads doesn't work, then the program uses the histogram method.  If the histogram method doesn't work, then the fit is taken to be the average of the previous few reads.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature in the Line class in the method `calculate_radius_of_curvature`.  I found this method to be useful as a sanity check.

The code to calculate the vehicle position is in the section marked **Calculate the vehicle position from the left and right lines**.  It is not in the `Line` class because it needs data from both the left and right lines.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the code section marked **Test the frame processor on the example images**.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had relatively few problems after following the class examples.  One lesson I've taken to heart from previous projects is to analyze the data first!  On the first attempt for the video, the right line crossed over the left in a few places.  After keeping track of previous line fits and making use of the curvature radius I was able to address the problem.

To make the code more robust, I would make use of the vehicle position (for more sanity checks).  I'd also want to test against poor conditions like night driving or bad weather. 
