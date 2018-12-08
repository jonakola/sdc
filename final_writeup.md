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

All functions mentioned are found in advanced_lane_finding_python.ipynb

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for camera calibration is in the cell titled 'Computing camera callibration matrix and distortion coefficeint' in advanced_lane_finding_pipeline.ipynb. 
The first step was to get the image points and object points, with the function get_img_points_and_obj_points, which takes in all of the images for callibration, calls the findChessBoardCorners method on all of the images, and returns the image points and the object points. 
After this, I call the calibrate camera method, which also takes in the callibration images folder, but with the img points and object points, returns the camera matrix and distortion coefficients, from calling OpenCVs calibrateCamera method.
![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

An example of an undistorted image can be found here - undistorted_image.jpg

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

To create a thresholded binary image, I use the saturation channel of the image, in addition to computing the gradients in the x direction with OpenCVs Sobel method. I then combine the results from the thresholded saturation channel, and the thresholded gradient results. 
I do all of this in the create_thresholded_binary_image method in section 3. This method takes in an image, and returns a thresholded binary image. 

A thresholded image can be found here - thresholded_image.jpg

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I compute a perspective transform in section 4, and provide a function, apply_perspective_transform, to do this once the transformation matrices have been found (M and Minv). 
My source points were [690, 450],[1042,675],[275,675],[590,450], and my destination points were [img_size[0]-350,10],[img_size[0]-350,(img_size[1]-10)], [350,img_size[1]-10], [350,10], representing a birds eye view perspective. 
I then apply a mask to the transformed image, to crop out the edges, as I discovered it was causing my lane detection to fail in some images. 

A succesful perspective transform can be seen here - warped_image.jpg

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identify lane pixels in the function draw_line_boundaries in section 5, which takes in a binary image and returns all of the points on the left and right lanes, all the y axis points, and all the coefficients of the fitted curves for the left and right lanes. I use the window method to identify the lines, starting from the points with the maximum vertical sum of pixels detected, and then working my way up, adjusting the window position accordingly.

An example of identifying the lane pixels can be found at - lane_identification.jpg

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculate the radius of curvature in section 6, with the function find_lane_curvature, which takes the left and right lane curve coefficients, and applies the standard curvature formula.
I calculate the position of the car with respect to the middle of the lane in the function find_car_position, which first computes the midpoint of the lane by substracting the left lane position from the right lane position, and then subtracting the mid point of the image from that. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

An example of an image with the lane area identified is - lanes_on_road.jpg


---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

My video can be found at final_video.mp4

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The biggest issue with the video is that it occasionally fails to identify the lanes as the car continues to take the bend. I think this can be solved by being smarter about the starting point for each subsequent frame. I could keep track of all of the window starting points, and possibly refer to the most recent starting point to as the starting point for a new frame, or perhaps calcuate an exponentially weighted average of the center of the starting middle points to make this more dynamic. This pipeline would also likely fail in conditions of low visibility, such as due to heavy rain or snow, or at night time. To improve this, I would want to look at other features, probably even just to the side of the road, so that I always have a view of the road, and also perhaps could adjust the threshold values for the image filters, as visibility changes in different conditions
