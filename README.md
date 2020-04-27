## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
This repository contains code for a project I did as a part of [Udacity's Self Driving Car Nano Degree Program](https://www.udacity.com/drive). The goal is to write a software pipeline to identify the road lane boundaries in a video.
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

[image1]: ./output_images/dist_udist_img.jpg "Undistorted"
[image2]: ./output_images/dist_udist_test_img.jpg "Road Transformed"
[image3]: ./output_images/binary_thresholded.jpg "Binary Thresholded"
[image4]: ./output_images/perspective_transform.png "Perspective Transform"
[image5]: ./output_images/sliding_window.jpg "Sliding Window Search"
[image6]: ./output_images/smart_search.jpg "Smart Search"
[image7]: ./output_images/final_result.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

All code sections for this project can be found in [Advanced_Lane_Finding_Pipeline.ipynb](Advanced_Lane_Finding_Pipeline.ipynb).

### Camera Calibration

The first part of this pipeline requires a camera calibration. This is done to correct for lens and perspective distortion from the camera sensor. The camera calibration uses 3 functions from openCV
and images of chessboards for reference from the `camera_cal` folder. You can see the calbration steps in the `camera_calibration` function in the first cell of this notebook.

![alt text][image1]

### Pipeline

#### 1. Distortion Correction applied to test image.

The `camera_calibration` function returns the camera matrix and distortion co-efficients. Using these returned values and applying the `cv2.undistort()` function. We can see the results below of undistorting a
test image from the camera.

![alt text][image2]

#### 2. Gradient and Color Thresholding

The next step is to apply Gradient and Color thresholding to get a binary representation of the image. The goal is to create a filter that retains the lane lines in various lighting conditions as well
as removing noise from the image to have robust lane detection. This part of the pipeline currently needs to be improved quite a bit to handle the harder challenge video in this project. To create a 
binary thresholded image as shown below I used a combination of color and gradient thresholding. I used the RGB and HLS color spaces for color thresholding. I then used the Sobel operator in the x direction
and created a directional gradient threshold between 30 and 90 dgrees. Here you can see the combination of sobel thresholding and color thresholding.

![alt text][image3]

#### 3. Perspective Transform (Birds Eye View)

Once the gradient and color thresholding has been applied to the image the next step is to warp the image performing a perspective transform using `cv2.warpPerspective()`. To use this function you must first
create a matrix transform using `cv2.getPerspectiveTransform()`. You must supply source and destination points to `cv2.getPerspectiveTransform()` to create the tranform and inverse transform matrix. These will be used to 
warp and unwarp the image. The code for this section is located in the 'Perspective Tranform' section of this notebook. The following source and distanation points were used for the perspective transform.

```python
    #Source point coordinates
    pt1 = [725, 455]
    pt2 = [555, 455]
    pt3 = [1280, 680]
    pt4 = [0, 680]
    src = np.float32([pt1, pt2, pt3, pt4])
    
    # Destination point coordinates
    dpt1 = [1080, 0]
    dpt2 = [0, 0]
    dpt3 = [1082, 720]
    dpt4 = [0, 720]
    dst = np.float32([dpt1, dpt2, dpt3, dpt4])
```



![alt text][image4]

#### 4. Sliding Window Search

Once the perspective transform has been done next we can search for the lane pixels using a sliding window search. To perform the sliding window search first we have to determine where the lane pixels
are concentrated from our binary image using a histogram on the bottom half of our image. This allows us to determine the base x position for the left and right lane line. Once the lane line pixels have
been found using the sliding window method we fit a 2nd order polynomial using `np.ployfit()`. Below is an illustration of the sliding window search.

![alt text][image5]


Once you have performed a blind search and validated the lane lines are valid you can perform a smart search using the previous fit coefficents to narraw in where to search. This is implemented in the `smart_search()` function.
You can see the illustration of the smart search below.

![alt text][image6]

#### 5. Radius of Curvature and Vehicle Offset.

In the Lane Curvature section of this notebook is where the function `get_lane_curvature()` is implemented. This returns the curvature in meters for the given left and right lane line. Once the 2nd order polynomial
equation is found then we can get the x intercept for the left and right lane line at the bottom of the image. We can then calculate the lane center using this information. We also know that the camera is in the 
very center of the image so the vehicle offset is calculated as following `lane_center - img.shape[1]/2`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
Here is the final output with lane lines drawn as well as the vehicle curvature, offset, and lane width overlayed on the image.


![alt text][image7]

---

### Pipeline (video)

#### 1. Final Output

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion
---

### Problems/Issues
---


#### 1. Gradient and Color Thresholding

The first major improvement to this pipeline needs to be improved thresholding. This implementation struggles with varied lighting conditions such as shadows, road surfaces with varying color, cracks, and tar strips. 
Exposing the light to dark contrast on each side of the lane line using adaptive thresholding would really help improve finding the lane pixels.


#### 2. Lost Frames

For frames with no detected lane lines I use a average of the last 3 frames to tell me where to search. If more then 3 lost frames are detected in a row then the lane line attributes are cleared. Then a blind search will be executed
to look for lane lines. So if no lane lines or detected in the blind search and there is no history from the previous 3 frames then there will be a frame with no detection. Currently for this pipeline both lanes must be detected for a valid output. This could be improved upon by showing only a single lane if the second lane has been lost. 

#### 3. Perspective Transform

The second main improvement I would add is making the perspective transofrm look at a smaller section of road. This could help with sharp turns by averaging over a smaller window of lane pixels in each lane.

#### 4 Sanity Checks

This pipeline would also be more robust if the sanity checks were improved. Each time a lane line is detected we could calculate a confidence of each detection by running through the lane sanity checks. 
This implementatoin currently only checks the lane width to determine if it is a valid lane line or the lane line information from the previous frame needs to be utilized. Lane curvature and the difference in polynomial coeffecients
from the previous and current frame could also be checked for higher confidence. 

#### Overall

This pipeline overall performs much better the my first project and has taught me a lot about different computer vision techniques. This project has also shown me there is a lot of room for improvement with this pipeline.

