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

[image1]: ./myexamples/undistort_output.png "Undistorted"
[image2]: ./myexamples/test1.jpg "Road Transformed"
[image3]: ./myexamples/binary_combo_example.jpg "Binary Example"
[image4]: ./myexamples/warped_straight_lines.jpg "Warp Example"
[image5]: ./myexamples/color_fit_lines.jpg "Fit Visual"
[image6]: ./myexamples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"
[image7]: ./myexamples/test1_undist.jpg "Road Transformed"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell 2 of the IPython notebook located in `advanced_lane_lines.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I saved the `mtx` and `dist` outputs from `cv2.calibrateCamera` and used those to undistort the test image -

![alt text][image7]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at cells 28 and 29 of `advanced_lane_lines.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

I used absolute sobel thresholds along x and y axes and s (of hls) and v (of hsv) for thresholding.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp_img()`, which appears in cell 3 of `advanced_lane_lines.ipynb`.  The `unwarp_img()` function takes as inputs an image (`img`), and transformer object M (or Minv).  I chose the hardcode the source and destination points in the following manner:

```python
offset=200
src = np.float32([(580,450),
                  (250,665),
                  (1080,665),
                  (700,450)])
dest = np.float32([[offset, 0],
                  [offset, img_size[0]], 
                  [img_size[1]-offset, img_size[0]], 
                  [img_size[1]-offset, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 450      | 200, 0        | 
| 250, 665      | 200, 720      |
| 1080, 665     | 1060, 720     |
| 700, 450      | 1060, 0       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then in cell 61 of `advanced_lane_lines.ipynb` I used convolution methods to find histogram peaks in the left and right lines by sliding window technique and fit the centroids of the windows with with a 2nd order polynomials. Then I got this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in cell 61 in my code in `advanced_lane_lines.ipynb`. After fitting a second order polynomial, the radius of curvature was calculated using standard formula.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 61 in my code in `advanced_lane_lines.ipynb`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output1_tracked.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

1) I was using a hardcoded trapezoid too close to a straight lane line image. This will cause issues with any other video that may not have the same resolution. I observed, if I take my trapezoid slightly further from the lane lines, it produced better results for curved lines. Going forward, I should use src points using ratios in image frame
2) My tracker faltered in bright light. This problem can be fixed using more sophisticated image pipeline
