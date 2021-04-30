## Project Write-up

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

[image1]: ./output_images/writeup_images/chessboard_calibration.jpg 
[image2]: ./output_images/writeup_images/undistortion.jpg 
[image3]: ./output_images/writeup_images/components_combined.jpg 
[image4]: ./output_images/writeup_images/combined_transform.jpg 
[image5]: ./output_images/writeup_images/perspective_transform.jpg 
[image6]: ./output_images/writeup_images/check_perspective_transform.jpg 
[image7]: ./output_images/writeup_images/histogram.png 
[image8]: ./output_images/writeup_images/lane_lines.png 
[image9]: ./output_images/writeup_images/projected_back.png 
[image10]: ./output_images/writeup_images/image_pipeline.jpg 


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first few code cells of the IPython notebook located in [Camera_Cal_and_Perspective_Transform.ipynb](./Camera_Cal_and_Perspective_Transform.ipynb).


I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

Internal Corners (image points) of the chessboard are identified using cv2.findChessboardCorners() and drawn onto the board using cv2.drawChessboardCorners(). Below is a depiction of the undistortion process for one of the calibration images used. 

Note: There are 9*6 internal corners, which has been specified in the code. For some of the calibration images, all internal corners are not in the frame and hence there are not used for calibration (3/20)

![Camera Calibration][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image (test1.jpg) using the `cv2.undistort()` function and obtained this result: 

![Image Undistortion][image2]

The camera matrix (mtx) and undistortion co-efficients (dist) are saved in the [wide_dist_pickle.p](./camera_cal/wide_dist_pickle.p) file for later use.

### Pipeline (single images)

The pipeline for single images is contained in the IPython notebook [Image_Pipeline.ipynb](./Image_Pipeline.ipynb). Please run this notebook, and you will see all the steps involved. Below is a plot of all the steps performed. There are described step by step in the section ahead.

![Image Pipeline][image10]

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Image Undistortion][image2]

I defined a function - cal_undistort(image,mtx,dist) which outputs an undistorted image using previously saved camera matrix and undistortion co-efficients.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this was developed in the [Gradient_and_Colour_Thresholding.ipynb](./Gradient_and_Colour_Thresholding.ipynb) IPython notebook and the same functions have also been defined in the image pipeline notebook [Image_Pipeline.ipynb](./Image_Pipeline.ipynb). 

There are a total of 7 functions defined: 4 gradient and 3 colour transforms. Out of these, 4 combinations have been made and put in an expression for the combined transform. Each of the 4 combinations add a particular element to the combined transform:
1. gradx & grady: provide crucial curve information, especially towards the end of visible road
2. mag_binary & dir_binary: strengthen information provided above
3. s_binary & h_binary: saturation alone also detects imperfections/shadows on the road. Their combination removes these problematic elements
4. r_binary: Very effective in identifying white lanes, especially when lane line is not continuous.

The thresholds for these functions are the optimum ones found during the lesson. Threshold for r_binary is highly selective in order for it to provide only the specific information required.

Below is a depiction of the individual contribution of these components and their combined effect on the original image.

![Components_Combined][image3]

![Combined Threshold Binary Transform][image4]

A combining function "threshold_combined" has been defined to carry out this transformation in the [Image_Pipeline.ipynb](./Image_Pipeline.ipynb) file.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform has been developed in the second part of the IPython notebook [Camera_Cal_and_Perspective_Transform.ipynb](./Camera_Cal_and_Perspective_Transform.ipynb).

One of the test images (straight_lines1.jpg) was used to identify a trapezium on the road (source points) and transformed into a rectangle (destination points) in the warped image. These points were roughly selected by eye-balling them.

The result on the selected image is shown below:

![Perspective Transform][image5]
  
The Transform Matrix "M" and the Inverse Transform Matrix "Minv" have been saved in the file [persp_pickle.p](./persp_pickle.p) for usage in image/video pipeline.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image (test3.jpg) and its warped counterpart to verify that the lines appear parallel in the warped image.

![Verifying_Perspective_Transform][image6]

In the [Image_Pipeline.ipynb](./Image_Pipeline.ipynb) file, I have defined a "persp_transform" function that takes in the combined threshold binary image along with "M" and returns the warped image. The function uses cv2.warpPerspective() to make the transformation.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For this, please refer to the [Image_Pipeline.ipynb](./Image_Pipeline.ipynb) file again.

The histogram peaks and sliding windows method has been used to identify lane-line pixels. 

From the histogram, the two peaks give the base position of the lanes on either side as shown below:

![Histogram][image7]

The "fit_polynomial" function (defined in the lesson) is used to find lane pixels and fit a second-degree polynomial to each line.

![Lane Lines][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Refer to the function "measure_curvature_real()" :  It takes the left and right polyfits as inputs and returns their respective curvature as well as offset. Offset is the position of the vehicle with respect to center. 

The polyfit co-efficients are converted into suitable dimension for radii output to be in metres. Formula for curvature is taken from lesson.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step by calling the draw_lanes() function defined in the lesson.  Here is an example of my result on a test image:

![Projected back on road][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

##### There are 3 video pipelines I developed. Please refer to the [Project Video.ipynb](./Project Video.ipynb) for a well-explained code.

Below are the links to the 3 versions:

1. [Sliding Windows only](./output_videos/sliding_pipeline.mp4)
2. [Look-Ahead Filter + Sliding Windows as reset option](./output_videos/skipping_pipeline.mp4)
3. [Complete Pipeline: Sanity Check, Tracking, Look-Ahead Filter, Reset and Smoothing](./output_videos/smoothing_pipeline.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

##### Approach and Techniques Used

I enjoyed the part of the project where we had to determine a good combination of various binary threshold transforms in order to get a robust output. There was no set method to go about it and no particular standard to be compared with. The onus was on the individual to be creative, decide when their combination was satisfactory and move on. These learnings will be extremely valuable moving forward in the nanodegree program.

I split the pipeline into phases which helped me organise my thoughts. First, applied only sliding windows, then moved onto integrating the look-ahead filter and finally bringing it together with the smoothing and reset characteristics. This was an effective way to build the pipeline as it also helps you visualise the improvement in the output video one step at a time.

For the sanity checks, I have kept it simple. A total of 3 checks are performed and it is required that the frame passes all 3 of them in order to be considered a high-confidence measurement. There is a limit on the difference between radii of curvature of the two lines (except when both lanes have radii > 1000 m, as this means that lines are mostly straight and radii values measured can vary largely), a limit on the lane width and a check for parallel lane lines (if gap between lanes is maintained and radii of curvature is similar - they are ~ parallel)

A challenge I faced was when I initially got unrealistic results when moving to the video pipeline from single image pipeline. It turned out that my tracking code was saving the polynomial co-efficients of lane lines in different units from what I expected. It took me significant time to identify the problem and then to debug it. As the mistake was at a step I performed much earlier in the implementation, it was harder to trace back. Hence, I realised it is important to verify code at small checkpoints in the project.

Finally, I would like to share how I carried out the smoothing process. 

I defined a new class Lane(), apart from Line(), to hold common lane information like detection, sanity and offset. Lane.detected checks if a lane has been found through the fit_polynomial() or search_around_poly() functions. Lane.sanity is the sanity check and frame_count is the number of consecutive bad frames (failed sanity check) till the present iteration.

At the start of an iteration, a check is performed to see if present frame is either the first frame/ if last 3 previous frames were bad/ no lane detection/ sanity check failed. In this case, sliding window based detection is done. Otherwise, we first try to search around the polynomial and if this fails we use the sliding windows method. Next, the sanity check is performed. If passed, we store the values in the line class otherwise we use previous frame values. An averaging over the previous 5 (or less) is done by a call to the append_average() function (weighted average- giving more weight to recent frames). Using the average fit values, average radius of curvature and offset are computed and displayed along with the reprojected lane lines on the road.

##### Shortcomings and Scope for Improvement

1. Method used for averaging: We use a linear average for a quadratic polynomial which is not exactly a proper average. Instead, we could fit a polynomial over a collection of good lane pixels over multiple frames
2. Camera Angle: During the motion of the vehicle over a bump, the camera angle changes and hence the perspective transform matrix changes. However, as the matrix input to the algorithm remains the same, the width of the lane during these frames is altered. To address this, we could use a dynamic transform matrix
3. Sudden changes in curve: When a frame fails at the sanity check, the previous frame is reused. However, a fail at sanity can mean a significant change in the road direction. By appending the previous fits, instead of extrapolation/prediction we slow down the algorithm's responsiveness to changes in curve (in other words, increase inertia. While simple extrapolation might not be recommended as a solution (It would only further strengthen the influence of previous frames), machine learning could be applied to deal with the situation based on similar data during testing.
