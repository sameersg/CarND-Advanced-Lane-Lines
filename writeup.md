## Writeup Template

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

[image0]: ./output_images/camera_cal/13.jpg "Corners"
[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./output_images/undistorted2.png "Undistorted 2"

[image3]: ./output_images/binary.png "Warp Example"

[image4]: ./output_images/warp.png "Warp Example"
[image5]: ./output_images/better_find_lane.png "Fit Visual"
[image6]: ./output_images/lane_line.png "Output"

[image7]: ./output_images/warp_bianry.png "Binary Warp"
[image8]: ./output_images/histogram.png "Histogram"
[image9]: ./output_images/find_lane.png "find_lane"

[video1]: ./output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 1-4 code cell of the IPython notebook located in the /P4-CarND-Advanced-Lane-Lines.ipynb 


I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.

After that i started reading all the images in /camera_cal and graysacled them. The used this CV2 function to find the corners 
and drawed them.

```python
cv2.findChessboardCorners(gray, (9,6), None)
cv2.drawChessboardCorners(img, (9,6), corners, ret)
```
![alt text][image0]


I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

```python
cv2.calibrateCamera(objpoints, imgpoints, img_size,None,None)
cv2.undistort(img, mtx, dist, None, mtx)
```

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The fifth cell uses the def undistort(img): function which loads the pickle file and corrects the distortion.  
Result: ![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
The sixth code cell conatains the `def hls_pipeline(img, s_thresh=(170, 255), sx_thresh=(20, 100))` function with the code.

I used the HSL Color space and a combination of S-channel, sobel-x and Threshold x gradient to create a binary image.  
Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.


The code for my perspective transform is includes in the function called `def top_view(img) `, which appears in sixth code cell.

The `top_view()` function takes as inputs an image (`img`). An the result ist Birdview image 
I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[590, img_size[1] - 270], 
					[690, img_size[1] - 270], 
					[1120, img_size[1]], 
					[190, img_size[1]]])
					
dst = np.float32([[(img_size[0] / 4), 0], 
					[(img_size[0] * 3 / 4), 0], 
					[(img_size[0] * 3 / 4), img_size[1]], 
					[(img_size[0] / 4), img_size[1]]])
    
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 590, 450      | 320, 0        | 
| 690, 450      | 960, 0      |
| 1120, 720     | 960, 720      |
| 190, 720      | 320, 720       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.


![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Cell 22 `def find_lane(binary_warped)` uses the histogram method 
on a binary topview image.

![alt text][image7]

It uses the bottom half of the image to create a histogram. The place where the peaks are in the left and right of the histogram are uses for polynomial. It uses 9 windows margen and minpix of 50 to follow the line. My result:

![alt text][image8]

![alt text][image9]

In Cell 23 the `def better_find_lane(binary_warped, left_fit, right_fit)` ist the quicker method it uses the left_fit and right_fit found by `def find_lane(binary_warped)`  to find line pixels.

My result:
![alt text][image5]

 
#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in cell 25 through `def calc_curvature(left_fit, right_fit, ploty)` in my code. And cell 26 conatins the `def distance_center(left_fit, right_fit, img_size)` function whicht calulates the distance between car and the lane center.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the `def write_on_image() ` function

It uses all the functions i used before to detect the lane lines and draws it on the image.

Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
I implemented this step in the `def final_pipeline(image)` function. First it Process the image to combined_binary, creates a Birdview of it. Then uses the Find lane lines  to detect the lane lines. If previos frame exist then uses the better_find_lane function and uses a sanity_check to check if lane lines are separated by reasonable distance. If the sanity check is failed 5 times in a row, a reset is performed, and starts with the histogram method.
Smoothing is used to avg the left_fit, right_fit, left_fitx, right_fitx and ploty.

At last it uses `write_on_image` to draw the line and write the curvature and distance from center on the image.

![alt text][video1]



Here's a [link to my video result](./output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

A better binary conversion and more robust lane finde algorith is needed for it to work in the challenging video. And i am not really sure if my curvature in meters is right. It makes sense but not sure about it. I have no idea how to improve the lane finde algortihm.
