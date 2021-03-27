# CarND-Advanced-Lane-Lines-master
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

[image1]: ./output_images/camera_calibration.PNG "Chessboard"
[image2]: ./output_images/undistored.PNG "Undistorted"
[image3]: ./output_images/color_sobel.PNG "Thresholds"
[image4]: ./output_images/warped.PNG "Warped"
[image5]: ./output_images/lane_lines.PNG "Lane Lines"
[image6]: ./output_images/lane_detected.PNG "Lane detected"
[image7]: ./output_images/curvature.PNG "Curvature"
[video1]:  project_video_output.mp4 "Video"

### Overview

In this project, my goal is to write a software pipeline to identify the lane boundaries in a video from a front-facing camera on a car. 
[Here](Advance_Lane_Lines.ipynb) is my code.
Udacity Self-Driving Car Nanodegree - Project 2

### Process

#### 1. Camera Calibration

OpenCV functions were used to calculate the correct camera matrix and distortion coefficients using the calibration chessboard images provided in the repository. The distortion matrix should be used to un-distort one of the calibration images provided as a demonstration that the calibration is correct. Example of undistorted calibration image is Included in the writeup (or saved to a folder).

The code for this step is contained in the first code cell of the IPython notebook [camera_calibration.ipynb](./docs/camera_calibration.ipynb).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  
I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  

![alt text][image1]
#### 2. Distortion correction

Distortion correction that was calculated via camera calibration has been correctly applied to each image. 
I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

#### 3.  Creation a thresholded binary image.
I created a binary image containing lane pixels using gaussian blur, color transforms, and gradients
![alt text][image3]

#### 4.  Perspective transform, "birds-eye view".
- cv2.getPerspectiveTransform() to get M, the transform matrix
- cv2.warpPerspective() to apply M and warp your image to a top-down view

The code for my perspective transform includes a function called `warper()`. 
The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 5. Lane pixels detection

The first step I took is to split the histogram into two sides, one for each lane line. With this histogram we are adding up the pixel values along each column in the image. From that point, I used a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.
I used left_lane_inds and right_lane_indsto hold the pixel values contained within the boundaries of a given sliding window. The left and right line have been identified and fit with a curved functional form, a 2nd order polynomial. 

![alt text][image5]

#### 6. Curvature of the lane and vehicle position with respect to center determination

To take the measurements of where the lane lines are and estimate how much the road is curving and where the vehicle is located with respect to the center of the lane. The radius of curvature may be given in meters assuming the curve of the road follows a circle. For the position of the vehicle, I assumed the camera was mounted at the center of the car and the deviation of the midpoint of the lane from the center of the image is the offset we're looking for. As with the polynomial fitting, convert from pixels to meters.

![alt text][image7]

#### 7. The detected lane boundaries back onto the original image.

The fit from the rectified image has been warped back onto the original image and plotted to identify the lane boundaries. This should demonstrate that the lane boundaries were correctly identified. 

![alt text][image6]

#### 8. Pipeline (video)

The image processing pipeline that was established to find the lane lines in images processes the video. The output is a video where the lanes are identified in every frame, and outputs are generated regarding the radius of curvature of the lane and vehicle position within the lane. 

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

- Other gradients could be use to improve the line detection.
- There may be an incorrect frame where the lane is not detected.
- Lines with sharper turns and very short spacing still pose challenges. To improve this, I need to transform the mask calculation and session, and it seems that I need to fix it with different hyperparameters.
