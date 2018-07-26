## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

### Outline
---
The objective of this project is to build a pipeline to identify lane lines in a video.  The following steps are included in the pipeline:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/chessboard_undist.png "Chessboard-undistorted"
[image2]: ./output_images/test4_undist.png "Raw image undistored"
[image3]: ./output_images/color_threshold.png "Color channel threshold"
[image4]: ./output_images/sobel_threshold.png "Sobel threshold"
[image5]: ./output_images/combined_threshold.png "Combined threshold"
[image6]: ./output_images/birds_eye.png "Birds-eye view"
[image7]: ./output_images/test4_detection.png "Lane line detection"
[image8]: ./output_images/final_image.png "Output of pipeline"
[video1]: ./processed_project_video.mp4 "Video"



### Camera Calibration
---
#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Today's cheap pinhole cameras introduces two major distortions: radial distortion and tangential distortion. A distortion correction can be made by a model with distiortion coefficients being the model parameters. To estimate the coefficients, we 
first need to compute a camera matrix. This can be done by using the chessboard.W
More specifically, we start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here we assume the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time we successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

We then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. We applied this distortion correction to a chessboard image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

Below we used the distortion correction to a test image `test4.jpg`:

![alt text][image2]


A detailed camera calibration tutorial using `OpenCV` can be found [here](http://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_calib3d/py_calibration/py_calibration.html).


### Color Channel and Gradient Threshold
---
The basic idea usning thresholds is to make the lane line pixels stand out by filtering out the noise pixels around the lane lines. We use a combination of color channel and gradient thresholds to generate a binary image.

#### 1.  Color channel threshold
We converted images from RGB (or else) to HLS using the `cv2.cvtColor()` function, and then applied thresholds to a single channel to create a threshold binary image. This can be done by using the funciton `hls_thresh()` we defined (see `Help Functions for Thresholded Binary Image Generation` cell in the Jupyter notebook).  Here are examples of S-channel and L-channel thresholded binary images (by applying `hls_thresh()` to the undistorted test image above):

![alt text][image3]

#### 2. Sobel threshold
We applied a gradient filter `Sobel`, a joint operataion of Gaussion smoothing and differetiation, to create a thresholded binary image.
For convenience, we wrote a wrapper funciton `abs_sobel_thresh()` (see `Help Functions for Thresholded Binary Image Generation` cell in the Jupyter notebook) in which the `OpenCV` funciton `cv2.Sobel()` was called. Below are Sobel x-derivative and y-derivative thresholded binary images we obtained:

![alt text][image4]


#### 3. Combination
We put together the color channel threshold and the Sobel threshold together on images to create combined thresholded binary images. We wrote a wrapper function `binary()` (see `Help Functions for Thresholded Binary Image Generation` cell in the Jupyter notebook) and applied it to the undistorted test image to create combined thresholded binary images.  Here is what we got:
 
![alt text][image5]

### Perspective Transform
---
Having obtained the thresholded binary image, we applied a perspective transform to rectify the binary image ("birds-eye view"). A wrapper function `warper()` was defined (see `Help Functions for Thresholded Binary Image Generation` cell in the Jupyter notebook) in which the 
`OpenCV` functions `cv2.getPerspectiveTransform()` and `cv2.warpPerspective()` were called. Below is what they look like:

![alt text][image6]

### Lane Line Pixel Detection
---
#### 1. Sliding window search
To identify lane line pixels, we used a sliding window and moved it based on a updated centroid by using histogram information. We then collected pixels with values 1 in the sliding window.  Those pixels were considered part of lane lines. 

#### 2. Fitting polynomials
After obtaining the left/right lane line pixels, we fit them with a quadratic polynomial respectively. We implemented the sliding window search and fitting polynomials in the function `detect_lines()` (see `Help Functions for Lane Line Finding` cell in the Jupyter notebook). Here is an example of identifying lane line pixels and fitting polynomials:

![alt text][image7]

#### 3. Radius of curvature and vehicle position
We used a formula which can be found [here](https://www.intmath.com/applications-differentiation/8-radius-curvature.php) to estimate the radius of curvature. For the vehicle position, we assumed it was in the midpoint of the image because we considered the camera was mounted at the center of the vehicle. The deviation is simply the distance between the vehicle postion and the center of two lane lines.

Note that the values we calucalated here are in pixels. We actually used the scales 3.7/700 (meters per pixel) in x dimension and 30/720  (meters per pixel) in y dimension for convertion in our calculations.  The function `compute_radius_and_deviation()` can be found in the
`Help Functions for Lane Line Finding` cell in the Jupyter notebook.


#### 4. Warping back
After identifying the lane line boundaries, we warped them back onto the original image.  To better display this finding, we used a polygon filled with green color and transform it back to the original image to see if its boundaries visually match the lane lines. The image (in the pipeline section below) shows this plotting.  The warping-back and visualization is implemented in the function `plot_lines()` (see `Help Functions for Lane Line Finding` cell in the Jupyter notebook). 


### Pipeline
---
#### 1.  Test on an imge
We ran the pipeline on the test image and obtained the following

![alt text][image8]

#### 2.  Test on a video

We tested this pipeline on the video `project_video.mp4` and generated the output video `processed_project_video.mp4`. 

### Discussion
---
The pipeline was built via two phases: (1) thresholded binary image generation, and (2) lane line detection and data fitting. The most challenging part, from my perspective, was from threshold binary image generation. The road environments could be very complex and tricky, and the techniques used here (e.g., color channel and gradient theshold) may fail for those environments. In the furture invetigation, other filtering techniques, for instance, may be taken into account to combine with the those we used. 
