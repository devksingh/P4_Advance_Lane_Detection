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

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for performing calibration on the chessboard images provided is in Function calibrate_camera() in https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/P4_Advance_Lane_Detection.ipynb.
The steps are listed below:
1. Prepare "object points", which will be the 3-D (x, y, z) coordinates of the (10x6)chessboard corners. z axis is zero for all the corners because all are 2d images.  
2. Imagepoints are detected from all the chessboard images using cv2.findChessboardCorners function.
3. Compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  
4. This distortion correction can be applied to test image using the `cv2.undistort()` function and obtained this result: 

![alt text](https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/images/calibratedChessboard.png)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Calibrated Chessboard Image](https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/images/laneImage.png)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used various methods to come out with clean pipeline, and this process consumed most of the time, here is what worked for me.
I used following method (function draw_pipeline in the code ) for the binary image containing both the lane pipeline:
1.Color thershold for gray image for threshold (150, 255)
2. Gradient magnitude with x as s_channel of (HLS image) and y as gray with threshold (30, 100)
3. Combine both the binary images with AND condition to produce output binary image containing image pipeline


![ColrGradientTheshold](https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/images/color_gradient_theshold.png)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

the code for 
The code for performing perspective transform is in section get_perspective of the code. I used following source and destination points for perspective transform.
*image_size is size of the image using image.shape which is (720,1280)
src = np.float32([
    [(image_size[1]/4)-50,   image_size[0]-20],
    [ image_size[1]-80,   image_size[0]-20],
    [((image_size[1]/2)+140),   (2*image_size[0]/3)],
    [  (.4*image_size[1])-32,   (2*image_size[0]/3)]])

dst = np.float32([
    [0,image_size[0]],
    [image_size[1],image_size[0]],
    [image_size[1],0],
    [0,0]])



This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 270., 700.  | 0., 720.        | 
| 1200., 700. | 1280., 720.      |
| 780., 480.  | 1280., 0.      |
| 480, 480    | 0., 0.        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Perspective transform](https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/images/perspectiveLaneImg.png)
(https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/images/perspectiveLaneImage.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used following method to identify lane line:
For first frame (function fit_lines_first_time in the code):
1. Identify histogram peaks in righ and left side of the image.
2. Slide the window vertically on that histogram and recenter the window based on pixel position
3. After identifying all pixels in left and right lane, I used polyfit function to get second degree polynomial function
4. Draw the polynomial points on the lane
5. Fill the space between lines with green colow(0,255,0)
6. Project this image on the original undistorted image
From second frame onwards:
1. I performed steps 3-6 of the first frame lane identifictaion.
2. Stored the last frame's lane quardretic function and points
3. If no lane detected then use the last frame's lane
4. On every frame I took pixel by pixel difference using MSE (mean squared error) from current frame line with last frame lines
5. If MSE of the current frame is out by 5*margin(margin is 100) then use the last frame's lane because current lane is far away from where it should be. I arrived at 5*margin(500) after hit and trial and printing MSE for all the frame and distored frames had MSE more than 500. This step consumed most of the time in drawing pipeline

![pipeline](https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/images/filledLaneLineonPerspectiveBlankImage.png)


![pipeln1](https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/images/filledLaneLineonUndistortedOriginalBlankImage.png)

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I measured curvature and lane position function draw_lines in my code https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/P4_Advance_Lane_Detection.ipynb

![curvature](https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/images/curvature_code.png)

I used the formula and method provided by udacity during training.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in draw_lines functions of my code.  Here is an example of my result on a test image:

![final Image](https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/images/final_Image.png)

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://github.com/devksingh/P4_Advance_Lane_Detection/blob/master/challenge_video_output_diag_final.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced 2 problems while doing this project:
1. Identify best color and sobel threshold for identifying lane line. I ended up printing images of all the threshold (x,y,magnitude,direction,color threshold) for gray,HLS and HSV combinations. And finally chose to use gray and s channel, color threshold and gradient magnitude.
2. Identifying the bad lines, tried various methods and finally used MSE to identify which lines are far away from last lane lines so that I can use the frame line.
The project ended up well and looks like it's various with fair amount of accuracy.
