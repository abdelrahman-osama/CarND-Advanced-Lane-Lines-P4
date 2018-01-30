```
1638.   895.Advanced Lane Finding Project**
```

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # "Image References"

[image1]: ./writeup_images/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is included in the 2nd and the 3rd code cells of Advanced-Lane-Finding.ipynb.

We needed two arrays of points; `imgpoints` which are 2D points, and `objpoints` which are 3D points but since the chessboard is on a fixed plane we neglected the z direction since it's fixed for all the images and all chessboards have the same points so the `objpoints` are simply repeated for all the images and the `imgpoints` are the points that are successfully detected using the `findChessboardCorners()` function.

Then using the previous two arrays we compute the camera calibration and distortion coefficients using the `calibrateCamera()` function



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is in the 13th code cell, I used a combination between  B channel from the LAB model, L channel from the HLS model and the sobel-gradient in the x direction.

The B channel used mainly to detect the yellow lanes line and the L channel to detect the white lines.

The sobel-gradient on the x direction used to handle cases when B channel and L channel fail.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for transformation is included in the  16th code cell, the function `perspective_transform` uses the `src` and `dst`points calculated int cells 14 and 15, and takes two inputs the `undistorted` to be transformed and `inverse` which indicates if the image is a warped image and need inverse operation or it's a normal image and need warping.

I used some kind of robust technique to generate my `src` points:

`midpoint` using the fact that the camera is at the center of the lane so w get the midpoint of the image 

by`shape[1]//2 `

`nearoffset` is how much want to consider from the left and right sides if the `midpoint`near the bottom of the image

`faroffset` is the same like `nearoffsert` but at distance `length` from the bottom of the image

I hard coded the `dst` values to give me the appropriate results 

```python
src = np.float32([[midpoint-nearOffset, example_image.shape[0]-bottom_margin],                           [midpoint+nearOffset+30, example_image.shape[0]-bottom_margin],
                  [midpoint-farOffset, example_image.shape[0]-length],                                   [midpoint+farOffset, example_image.shape[0]-length]])

dst = np.float32([[25, example_image.shape[0]-25], 
                  [example_image.shape[1]-25, example_image.shape[0]-25],
                  [25,25], 
                  [example_image.shape[1]-25, 25]])
```

This resulted in the following source and destination points:

|  Source  | Destination |
| :------: | :---------: |
|  0,895   |   25,870    |
| 1638,895 |  1584,870   |
| 644,597  |    25,25    |
| 964,597  |   1584,25   |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

the code for the lane-line pixels included in code cells 19 and 21, I use `histo_finding_lane()` function on the first frame to find an initial polynomial function that fit the curves and then use `margin_finding_lane()`that uses previously computed polynomials in the next frames

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cells 23 and 24. I defined conversions in x and y from pixels space to meters and fitted new polynomials to x,y in world space. Moreover, calculated the radius of curvature in meters.

`car` is the location of the car which is the center of the image assuming the camera is  mounted on the center of the car

`lane_center` is calculated using the polynomials of the image values to get the value at the bottom of the image for the two bounding lines of the lane and get their average

`center_distance` is  the position of the vehicle with respect to center calculated by subtracting the `lane_center` from the `car` and multiply by meters per pixel in x dimension

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

the code for the pipeline the found in the code cell number 28

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Most of the problems i faced was due to the light conditions and some defects in the road, for example in the challenge video the black lines next to the white lines of the lane wee considered as lane lines till i used the LAB and HLS models with the help of sobel gradient in the x direction


