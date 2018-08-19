# Writeup: Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image0]: ./camera_cal/calibration1.jpg "distorted"
[image1]: ./output_images/calibration/undistorted_calibration1.jpg "Undistorted" 

[image20]: ./test_images/test3.jpg "Road Transformed"
[image2]: ./output_images/tracked4.jpg "Road Transformed"

[image3]: ./output_images/combined_04.jpg "Binary Example"

[image40]: ./output_images/warped_14.jpg "Warp Example"
[image4]: ./output_images/combined_14.jpg "Warp Example"

[image5]: ./output_images/combined_24.jpg "Fit Visual"

[image6]: ./output_images/combined_34.jpg "Output"

[video1]: ./output_tracked.mp4 "Video"


### [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points:
Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  
---


### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
The code for this step is contained in the first code cell of the IPython notebook located in "./carnd_advanced_lane_lines.ipynb".  

The OpenCV functions `cv2.findChessboardCorners` and  `cv2.drawChessboardCorners` are used to identify the locations of corners on a series of pictures of a chessboard taken from different angles. Then, using the locations of corners the camera calibration matrix and distortion coefficients are computed.

This distortion correction applied to the chessboard images using the `cv2.undistort()` function and obtained this result: 



### Distorted Image:
![alt text][image0]

### Undistorted Image:
![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.


The camera calibration matrix and distortion coefficients are used with `cv2.undistort` function to undistort from highway driving images.



### Original Image:
![alt text][image20]

### Undistorted Image:
![alt text][image2]


## 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

A combination of color and gradient thresholds are used to generate a binary image.  

```python
img = cv2.undistort(img, mtx, dist, None, mtx)
preprocessImage = np.zeros_like(img[:,:])
gradx = abs_sobel_thresh(img, orient='x', thresh=(12,255)) 
grady = abs_sobel_thresh(img, orient='y', thresh=(25,255)) 
c_binary = color_threshold(img, sthresh=(100,255), vthresh=(50,255)) 
preprocessImage[((gradx == 1) & (grady == 1) | (c_binary == 1) )] = 255
```
Here's an example of the output for this step.

### PreprocessImage:
![alt text][image3]

## 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for the perspective transform takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  

The source and destination points are chosen in the following manner:

```python
    img_size = (img.shape[1], img.shape[0])
    bot_width = .76
    mid_width = .08
    height_pct = .62
    bottom_trim = .935
    
    # Source coordinates
    src = np.float32([[img.shape[1]*(.5-mid_width/2),img.shape[0]*height_pct],
                      [img.shape[1]*(.5+mid_width/2),img.shape[0]*height_pct], 
                      [img.shape[1]*(.5+bot_width/2),img.shape[0]*bottom_trim],
                      [img.shape[1]*(.5-bot_width/2),img.shape[0]*bottom_trim]])
    
    offset = img_size[0]*.25
    
    # Destination coordinates
    dst = np.float32([[offset, 0], [img_size[0]-offset, 0], [img_size[0]-offset, img_size[1]], [offset, img_size[1]]])
```

The perspective transform is working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.


### Warped Image:
![alt text][image40]


### Warped Binary Image:
![alt text][image4]

## 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

### Finding the Lines:
![alt text][image5]

## 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`


### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  

Here is an example of my result on a test image:


### Identiﬁed Lane Area:
![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_tracked.mp4)

---

## Discussion

### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about:

the approach I took, 

what techniques I used, 

what worked and 

why, where the pipeline might fail and 

how I might improve it if I were going to pursue this project further. 





