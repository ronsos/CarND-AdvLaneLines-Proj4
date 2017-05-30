# Advanced Lane Finding

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

[image1]: ./camera_cal/calibration3.jpg "Calibration Image 3"
[image2]: ./output_images/calibration.png "Undistorted Calibration Image"
[image3]: ./test_images/test5.jpg "Test Image for Binary Example"
[image4]: ./output_images/binary.png "Binary Example"
[image5]: ./output_images/warped.png "Perspective Transform"
[image6]: ./output_images/lane_lines.png "Lane Lines"
[image7]: ./output_images/lane.png "Image with Lane Identified"
[video1]: ./P4_vid_lanelines.mp4 "Video"

### [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

This writeup follows the points in the rubric and addresses them individually. 

Please that the code for this project can be found in "lane_finding.ipynb".

## Camera Calibration

### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "lane_finding.ipynb".

First the image to be used in calibration is read in. 
![cal3][image1]

The image is converted to grayscale, then the chessboard corners are found for the image. 

These corners are then used to find the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. 

The distortion correction is applied to the image using the `cv2.undistort()` function. The undistorted image appears as follows:

![undistorted][image2]


## Pipeline (single images)

### 1. Provide an example of a distortion-corrected image.

The example above describes the process of distortion correction and shows and example. Here is the corrected image again:

![undistorted][image2]

### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The 2nd cell of the IPython notebook shows the thresholding that was used. Ultimately, a stacked binary of 5 different thresholds was chosen. From the RGB image, the red and green were used. From the HLS, saturation (S) and lightness (L) were used. The SobelX was added later to pick the yellow line in the midst of shadows, since none of the other channels were able to do it effectively, regardless of thresholding values. 

| Channel/Type  | Min    | Max  |
|:-------------:|:------:| :---:|
| R             | 220    | 250  | 
| G             | 220    | 255  |
| S             | 200    | 250  |
| L             | 210    | 255  |
| SobelX        |  35    |  90  | 

The original image prior to the binary thresholding (from 'test_images/test5.jpg')

![original binary][image3]

The resulting binary image is as follows:

![binary][image4]

### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image. 

The perspective transform code is in the 3rd cell of the IPython notebook. 

A set of source points were defined to determine the mask used for the source image. A second set of points was used for the resulting perspective transform. This points were chosen as follows:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 350, 720      | 
| 576, 450      | 350, 0        |
| 718, 450      | 980, 0        |
| 1130, 720     | 980, 720      |

The function 'cv2.getPerspectiveTransform' was used to calculate the transform matrix, M. The inverse, Minv, was also determined by swapping the order on the source and destination points. 

`M = cv2.getPerspectiveTransform(src, dst)
Minv = cv2.getPerspectiveTransform(dst, src)`

The perspective transform was applied using the function 
`cv2.warpPerspective(img, M, img_size, flags=cv2.INTER_LINEAR)`.

The resulting transformed image appears as follows: 

![warped][image5]

### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The 5th cell contains the code for identifying the lane lines and fitting the identified points with a polynomial curve. 

A histogram method was used. First, a histogram of the warped binary image above is taken across the horizontal axis. The x-location of the peaks is determined. The x-position of the left and right peak is used for initializing the sliding windows search. Nine vertical windows were used, along with a window pixel width of 100 pixels. The minimum number of pixels needed to recenter the window was set to 50. 

Pixels determined to be inside the windows are binned by side (left/right) and axis (x/y). A second order polynomial is fitted to the left and right data using the function `np.polyfit`. The resulting polynominals for the left and right lane are shown in the example image below:

![lanelines][image6]

### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The function `radius_of_curvature(left_fit, right_fit, ploty, left_fitx, right_fitx)` can be found at the bottom of the 5th cell. This function returns the radius of curvature for each lane line and also the vehicle offset. 

The equations used for the radius of curvature are:

`left_curverad = ((1 + (2*left_fit[0]*y_eval + left_fit[1])**2)**1.5) / np.absolute(2*left_fit[0])
right_curverad = ((1 + (2*right_fit[0]*y_eval + right_fit[1])**2)**1.5) / np.absolute(2*right_fit[0])`
 
Conversion to meters was 30/720 m/pixel in y, and 3.7/730 in x.     

Vehicle offset was calculated according to the following steps. First, the lane center is found using by averaging the bottom pixel of each lane line. This lane center value is subtracted from the value of the image center (half the width of the image). This gives the offset in pixels. Multiplying that value by 3.7 gives the distance in meters. 

Here is the code snippet that performs the offset calculation:

`lane_center = (left_fitx[-1] + right_fitx[-1]) / 2
lane_width_pixels = right_fitx[-1] - left_fitx[-1]
image_center = warped.shape[1] / 2
offset_pixels = lane_center - image_center
offset_m = offset_pixels * 3.7/lane_width_pixels`    


### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The 6th cell in the IPython notebook shows the lane area plotting example. The function `cv2.fillPoly()` is used to fill in a green area identified from the lane finding. 

Here is an example on a test image:

![laneimage][image7]

---

## Pipeline (video)

### 1. Link to your final video output. 

Here's a [link to my video result](./P4_vid_lanelines.mp4)

---

## Discussion

### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In order to get the pipeline to work with these settings on the main project video, two additional criteria were used for smoothing. 

First, if the radius of curvature of the left and right lines are not within a factor of two, the lane fit is discarded and the previous one retained. 

The second criteria is a check on the width of the lane at both the top and bottom of the image. If these are not within a factor of two, the solution is similarly discarded. 

The resulting solution works well on the primary project video. There is some occasional drift of the fit for a frame or two, but this is a worthwhile and manageable tradeoff for eliminating extremely poor solutions. 

This algorithm does not work well on the challenge video. One idea that might improve its performance is to dynamically scale the threshold values based on the average brightness of the image. Also, maintaining a running average of the lane line fit to fall back on (rather than just throwing out bad solutions and keeping the previous "good" one) may also help improve performance. 

