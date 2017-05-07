## Writeup 

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

[image1]: ./output_images/calibrate1.png "Undistorted"
[image2]: ./output_images/undist.png "Road Transformed"
[image3]: ./output_images/grad_color1.png "gradient, color"
[image4]: ./output_images/grad_color2.png "gradient, color"
[image5]: ./output_images/perspective.png "Warp Example"
[image6]: ./output_images/fitting.png "Fit Visual"
[image7]: ./output_images/curvature.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./lane_detect.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I applied cv2.undist() to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in `bin_threshold_for_lane()`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

I used a gradient of x direction. It captured much of the lanes, but with some noises. So I gave the low threshold to a gradient to get rid of noises. Instead I applid H channel and L channel. H channel captured the yellow lines well and L channel was good at detecting the while lines.
I combined a gradient of x with H+L channel.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `bird_eye_view_for_lane()` in the 4th code cell, which used src_pts and dst_pts like below.

```python
src_pts = np.array([
[img_w*(0.15),img_h],
[img_w*(0.5-0.040), img_h*(0.5+0.13)], 
[img_w*(0.5+0.040), img_h*(0.5+0.13)], 
[img_w*(1-0.15+0.01),img_] / 2 + 100]])

dst_pts = np.float32(
[[(img_size[0] / 4), 0],
[(img_size[0] / 4), img_size[1]],
[(img_size[0] * 3 / 4), img_size[1]],
[(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 192, 720      | 345, 720      | 
| 588, 453      | 345, 0        |
| 691, 453      | 934, 0        |
| 1100, 720     | 934, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used sliding window for dectecting lane-line pixels. First I get histogram of the points and started sliding window from the points at the high bin scores. I searched for lane pixels within the window area with a margin and made the window move left and right according to the position of the high bin scores again.

I applied polynomial fitting of order of 2 to those lane pixels.

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `cal_curvature()` function in the 5th code cell. I calculated the left and right curveture and get the mean of them, and then converted it into the real meters by multiplying xm_per_pix.
I got the position of the vehicle by comparing the center of the image with the center of the lanes. I multiplied the difference by xm_per_pix. I assumed that px_per_pix is 3.7/589 because the distance between the lanes was 589 pixels and it would be 3.7m.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 5th cell like this pipeline.
```python
    undist_img = undist_image(image)
    binary_thresh = bin_threshold_for_lane(undist_img)
    binary_warped = warp_perspective(binary_thresh, M)
    
    search = search_window_for_lane(binary_warped, fname=fname)
    #search = search_window_for_lane2(binary_warped)
    out = warp_perspective(search, Minv)
    out = cv2.addWeighted(image, 1, out, 0.5, 0)
```
Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced a problem when I applied video pipeline. It was good at still images but when it get to video stream, it failed. I found that video stream gave RGB images but I got the thresholds from BGR images. 

I tried a challenge video and it was not so good. I think the adjusting of the color space is important. Using the separate combination of thresholds respectively for yellow lanes and white lanes would be good.
