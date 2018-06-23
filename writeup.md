## Writeup

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

[image1]: ./output_images/UndistortedChessBoard.png "Undistorted"
[image2]: ./test_images/test4.jpg "Road Transformed"
[image3]: ./output_images/thresholding_image.png "Binary Example"
[image4]: ./output_images/warped_straight_line.png "Warp Example"
[image5]: ./output_images/fitting_test_image.png "Fit Visual"
[image6]: ./output_images/lane_tracking_result.png "Output"
[video1]: ./processed_project_video1.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd and 3rd code cell of the IPython notebook located in `Advanced-Lane-Line.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of gradients of both gray-scale and HLS's s space thresholds to generate a binary image (thresholding steps at 9th code cell's `extract_edge()` function in `Advanced-Lane-Line.ipynb`). I decided thresholds to check whether those thresholds can correctly identify lane lines in all of the test images.  Here's an example of my output for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform uses `cv2.warpPerspective` function (12th code cell in `Advanced-Lane-Line.ipynb`). My perspective transform warps images `src` coordinates to destination points `dst`. I chose the hardcode the source and destination points in the following manner:


```python
src = np.float32(
    [[(img_size[0] / 2) - 60, img_size[1] / 2 + 100],
     [((img_size[0] / 6) - 20), img_size[1]],
     [(img_size[0] * 5 / 6) + 20, img_size[1]],
     [(img_size[0] / 2 + 60), img_size[1] / 2 + 100]])

dst = np.float32(
    [[(img_size[0] / 4), 0],
     [(img_size[0] / 4), img_size[1]],
     [(img_size[0] * 3 / 4), img_size[1]],
     [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 580, 460      | 320, 0        |
| 193.3, 720      | 320, 720      |
| 1086.6, 720     | 960, 720      |
| 700, 460      | 960, 0        |

The camera is mounted on the center of the car, therefore I use symmetrical  `src` and `dst`.
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a straight line test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I coded window search to identify lane lines. The code is `process_image()` function in 14th code cell in `Advanced-Lane-Line.ipynb`. It has two ways to determine windows. First, if the fitting line is not detected in last image, it uses normal window search. By means of `histogram`, I got the peaks of thresholded points, that is the bottom center of window (`rightx_base` or `leftx_base`). Windows are the area whose height is `window_height` and whose width is double of `margin`. Average value of thresholded points in the window bacomes next windows' bottom center(`rightx_currnt` or `leftx_current`). This is the normal way to determine window. On the other hand, if the fitting line is detected in last image, I only searched fitting lines both side area apart from `margin`.
Then I used `np.polyfit()` to fit 2nd order polynominal of lane lines.  
Here is a sample example:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated radius of curvature in `Line.calcCurvature()` in 13th code cell in my code. In order to calculate radius of curvature, I get center line which is the middle line of right and left line. Also, the car center position respect to the lane lines is calculated by `Line.makePosText()` in 13th code cell in my code. Center position is the difference between image center and middle point between both right and left base points. This function also generates the position text in the image.



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in last 20 line through last 7 line in 14th code cell in the function `process_image()`. I used inverse transformation by means of `cv2.warpPerspective()` and draw half-transparent green on area between right and left lanes by using `cv2.fillPoly()`. Then I used `cv2.addWeighted()` function to merge that image and original image. I also used `cv2.putText` to draw the texts of the curvatures and the car center position.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

In order to pass line information during processing video, I define `Line` class in 13th code cell in my code. It includes the information whether line is detected, the current and best x values of fitting line, the x, y values of thresholded points, and radius of curvature.
Here's a [link to my video result](./processed_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My approach is the window search by means of gray-scale and HLS's s space gradients. This worked well in this project video, for some reason. First, it use not only gray-scale gradient detector but also  s space one, therefore, it becomes robust to lighting conditions. For example, test1's left line cannot be detected by gray-scale detector. Owing to s space gradient detector, I can detect lane lines on slightly lighter road color. Also, I don't use s space intact, this is because there are some places like test5.jpg image that have some block of blighter s space. Using gradient prevents me from detecting such blocks as lines.

There are some potential failure. First, if one lines, left or right, is completely invisible, my detector can fail, it uses both lines' information, therefore it cannot estimate radius of curvature from one line. Second, if the lines are blackout and during that time the radius of curvature changes rapidly, then my approach fail, this is because my program is using averaging and curvature thresholding, therefore it cannot deal with rapid change.

If I were going to pursue this project further, first, I introduce certainty of line in my `Line` class. This helps the detector when one of the lines detection is unstable. For example, when I calculate the radius of curvature, I can weight more certain one's curvature more, and it can make more robust detection. And, when one line is completely undetectable in one frame,  I can use last frame information and another line information, I can make better estimation of the line. Second, I use HLS's H space information, this helps my approach better because it prevents my detector from mistaking road edges or some straight object in my image as lane line.
In conclusion, those approach will help my detector be more robust.  
