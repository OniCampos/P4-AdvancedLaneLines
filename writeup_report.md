## Writeup Report

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

[image1]: ./images/calibration_image_corrected.png "Undistorted"
[image2]: ./images/distortion_correction.png "Road Transformed"
[image3]: ./images/combined_thresholds.png "Binary Example"
[image4]: ./images/warped_image.png "Warp Example"
[image5]: ./images/warped_binary_image.png "Fit Visual"
[image6]: ./images/warped_back_image.png "Output"
[video1]: ./output_videos/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./lane_finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps is contained in the third code cell of the IPython notebook located in "./lane_finding.ipynb"). I decide to use for the gradient threshold a combination of x and y gradients and for the color threshold a combination of saturation (HLS space) and red (RGB space) channels. Here's an example of my output for this step:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 4th code cell of the IPython notebook (./lane_finding.ipynb).  The `warper()` function takes as inputs an image (`image`), as well as source (`src`) and destination (`dst`) points. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([
    [(img_size[1] / 2) - 48, img_size[0] / 2 + 90],
    [(img_size[1] / 2) + 48, img_size[0] / 2 + 90],
    [(img_size[1] * 4 / 5) + 110, img_size[0]],
    [(img_size[1] / 5) - 75, img_size[0]]
])
dst = np.float32([
    [(img_size[1] / 4), 0],
    [(img_size[1] * 3 / 4), 0],
    [(img_size[1] * 3 / 4), img_size[0]],
    [(img_size[1] / 4), img_size[0]]
])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 592, 450      | 320, 0        | 
| 688, 450      | 960, 0        |
| 1134, 720     | 960, 720      |
| 181, 720      | 320, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for identify the lane-lines and fit the 2nd order polynomial is contained in the 5th cell of the IPython notebook. To identifty the lane-line pixels, I used the sliding windows algorithm included on the function `slidingWindows()` that takes as arguments a binary warped image `binary` and a boolean input `draw_windows` that draw the windows used on the algorithm if True. The function returns `left_fit` and `right_fit` that are the coefficients of the 2nd order polynomial fit, `leftpx` and `rightpx` that are the position of the pixels that contains the lane lines and `out_img` that is the image of the warped binary image.
Here is the warped binary image with the sliding windows:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 5th cell in my code in "./lane_finding.ipynb" between the lines 94 and 137. I defined a method called `laneCurvaturesAndOffset()` that has the arguments image, leftpx and rightpx that are respectively the binary warped image of the road and the left and right pixels positions on the image. The function returns two tuples with the left and right radius of curvature and distance from the vehicle center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 5th cell in my code in "./lane_finding.ipynb" in the function `drawLaneArea()`.  Here is an example of my result on a test image generated in the 6th cell fo the IPython notebook:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

For the first part of the project where I calibrate the camera using `cv2.calibrateCamera()` and I correct the distortion of images using `cv2.undistort()`, it was very straightfoward applying the same pipeline used on the lessons. 
For the thresholded binary image, I tried a lot of combinations of gradient and color's channels of images. The best combinations was with the gradient with respect of x AND the gradient with respect of y and using the saturation channels for the HLS color space AND the red channel for the RGB color space. The final binary iamge was the combination of gradient OR color binaries with the thresholds as follows:

```python
# Gradient thresholds
gradx_image = abs_sobel_threshold(undistorted_image, orient='x', sobel_kernel=9, thresh=(40, 200))
grady_image = abs_sobel_threshold(undistorted_image, orient='y', sobel_kernel=9, thresh=(40, 200))
   
# Color threshold
colorbinary = color_threshold(undistorted_image, r_thresh=(140, 255), s_thresh=(140, 255))
```
For the perspective transform, I had to choose which source (`src`) and destination (`dst`) points I had to use on the `cv2.getPerspectiveTransform()` to transform the original images to a bird's eye view of the vehicle. So I choose the points using the image of the straight line provided in the project repository ("straight_lines1.jpg") and the result is on the item 3 of this writeup report.

For the detection of lines, I used the sliding widows algorithm to find the position of the pixels for the left and right lines and after that I fit a 2nd order polynomial for both lines to find the equation. Then I determine the curvature of the lane and vehicle position with respect to center on `laneCurvaturesAndOffset()`.
Finally, I warped the detected lane boundaries back onto the original image including the radius of curvature and vehicle position em relation to center.

When I applied the pipeline to the video I noticed that in some areas (specifically when the road change the color) the lines detected abruptly change their positions and then came back to the right position. So, for each frame, I check if the detected lane has at least an approximately same distance for the center as the previous frame. If the distance is bigger than 0.3m, the previous detected lane is maintained.

To make the pipeline more robust, I could implement the sanity check suggested on the lesson to compare the curvature, horizontal distance and parallelism. With the sanity check, depending of the result of the check, I could applied the look-ahead filter algorithm provided on the lesson or reseting the detection and find the pixels positions using the sliding windows. This could make the pipeline faster. Also could smooth the detection of the lines with the last n good detections making the lines cleaner.
