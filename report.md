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

[image1]: ./pics_report/chessboard.png 
[image2]: ./pics_report/dissortion.png 
[image3]: ./pics_report/comb_1.png 
[image4]: ./pics_report/comb_2.png 
[image5]: ./pics_report/transformation_1.png 
[image6]: ./pics_report/wraped_1.png 
[image7]: ./pics_report/wraped_2.png
[image14]: ./pics_report/lanes_1.png 
[image15]: ./pics_report/lanes_2.png 
[image8]: ./pics_report/res_1.png 
[image9]: ./pics_report/res_2.png 
[image10]: ./pics_report/res_3.png 
[image11]: ./pics_report/res_4.png 
[image12]: ./pics_report/res_5.png 
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

All code is implemented in one IPython notebook, and here is a link to my [project code](https://github.com/dagrubi/Term1_P4/blob/master/P4.ipynb)

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. cv2.findCornders find the cordners of black and white fields of the chessboard. Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  cv2.drawChessboardCorners() draws the corners. The result shows 
<br />
![DrawChessboard][image1]

In the third cell the camera calibration is done for all chessboard images in the folder camera_cal.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. 3 of the 20 used chessboard images could not be used for calibration since the number of expected chessboard corners could not be found (images calibration1.jpg / calibration4.jpg and calibration5.jpg)

The conversion matrix for distotion correction is:
[[  1.15396093e+03   0.00000000e+00   6.69705359e+02]
 [  0.00000000e+00   1.14802495e+03   3.85656232e+02]
 [  0.00000000e+00   0.00000000e+00   1.00000000e+00]]
 
 
### Pipeline (single images)

The following steps are performed on all images in the folder  [folder] (https://github.com/dagrubi/Term1_P4/blob/master/test_images). First every step is explained step by step and at the end the whole pipeline on all images is described.


#### 1. Provide an example of a distortion-corrected image.

I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
 <br />
 ![Distortion Correction][image2].
 You can see the effect especially on the tree on the left and right side of the image.
 
 The code for disstortion correction on a test image is in cell 4 and in cell 5 of the notebook disstortion correction is applied on all pics in the [folder] (https://github.com/dagrubi/Term1_P4/blob/master/test_images).

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Cell 6 contains some methods for color gradients (source classroom)
- abs_sobel_thres: caculation of sobel operation
- mag_thres: threshold of sobel operator in x / y- direction (magnitude of gradient)
- dis_threshold: threshold direction of gradient (arctan of x-sobel and y-sobel operator)
- hls_selct: threshold for s-channel after conversion to hls color space
- combined_binary: combination of x-threshold and threshold of s-channel
- region of interest: application of an image mask

For final image i used the cominded binary method. The code on all test images can be found in cell 7 of the Notebook. I used a mask in order to apply the filter only on the relevant part of the image, where usually the lanes can be found.
Here are two examples:
 <br />
 ![Binary Image Example 1][image3]
 <br />
![Binary Image Example 2][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Some used methods for perspective transform can be found in cell 8.
- warp_all: do the wrap for an original image with following steps: 1) undistortion of the image (cv2.undistort(); 2)conversion to grayscale 3)get tranformation matrix (cv2.getPerspectiveTransform()); 4 warp picture (cv2.warpPerspective())
- warp: used for binary images (already undistorted) -> only getPerspectiveTransform und warpPerspective necessary

In cell 9 you can find the code in order to wrap an image und to select the source (`scr`) and destination (`dst`) points.
I used an image with straight lines (straight_lines1.jpg). As source points i selected 4 points on the lines (manually). After the perspective transformation the straight lines have to be parallel. In order to achieve this i selected the destination points, so that the lines are parallel.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 100, 720      | 
| 1100, 720     | 900, 720      |
| 650, 430      | 1100, 1       |
| 625, 430      | 300, 1        |

The result of the perspective transformation of the test image with straight lines can be found in 
 <br />
![Perspective Transformation][image5]
The lines appear parallel.

in cell 10 i applied the perspective transform on all test images. The original image, the warped original imagae and the warped combined binary image is shown. 

 <br />
![Wrapped test image 1][image6]

 <br />
![Wrapped test image 2][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
