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
The code for this project is contained in "./carnd_advanced_lane_lines.ipynb".  

The OpenCV functions `cv2.findChessboardCorners` and  `cv2.drawChessboardCorners` are used to identify the locations of corners on a series of pictures of a chessboard taken from different angles. Then, using the locations of corners the camera calibration matrix and distortion coefficients are computed.

This distortion correction is applied to the chessboard images using the `cv2.undistort()` function and obtained this result: 


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

The code for the perspective transform takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  The source and destination points are chosen in the following manner:

```python
img_size = (img.shape[1], img.shape[0])
bot_width = .76
mid_width = .08
At this point I was able to use the combined binary image to isolate only the pixels belonging to lane lines. height_pct = .62
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

Transformed images are provided. 

### Warped Image:
![alt text][image40]


### Warped Binary Image:
![alt text][image4]
## 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Using the warped binary image and `class tracker()`,  lane-line pixels are identified and fit their positions with a polynomial are fit.

```python
window_width = 25
window_height = 80
 
curve_centers = tracker(Mywindow_width = window_width, Mywindow_height = window_height, Mymargin = 25, My_ym = 10/720, My_xm = 4/384, Mysmooth_factor = 15)

window_centroids = curve_centers.find_window_centroids(warped)
    
l_points = np.zeros_like(warped)
r_points = np.zeros_like(warped)
    
for level in range(0,len(window_centroids)):    l_mask = window_mask(window_width,window_height,warped,window_centroids[level][0],level)
    r_mask = window_mask(window_width,window_height,warped,window_centroids[level][1],level)        
    l_points[(l_points == 255) | ((l_mask == 1))] =255
The result is plotted back down onto the road such that the lane area is identified.    r_points[(r_points == 255) | ((r_mask == 1))] =255
        
template = np.array(r_points+l_points,np.uint8)
zero_channel = np.zeros_like(template)
template = np.array(cv2.merge((zero_channel,template,zero_channel)),np.uint8)
warpage = np.array(cv2.merge((warped,warped,warped)),np.uint8)
result = cv2.addWeighted(warpage, 1, template, 0.5, 0.0)

```

### Finding the Lines:
![alt text][image5]

## 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Using the following code, the radius of curvature of the lane and the position of the vehicle with respect to center are calculated.

```python

ym_per_pix = curve_centers.ym_per_pix
xm_per_pix = curve_centers.xm_per_pix
    
curve_fit_cr = np.polyfit(np.array(res_yvals,np.float32)*ym_per_pix, np.array(leftx,np.float32)*xm_per_pix,2)
curverad = ((1 + (2*curve_fit_cr[0]*yvals[-1]*ym_per_pix + curve_fit_cr[1])++2)**1.5) /np.absolute(2*curve_fit_cr[0])
   
camera_center = (left_fitx[-1] + right_fitx[-1])/2
center_diff = (camera_center-warped.shape[1]/2)*xm_per_pix
```


### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This step is implemented in the following code. The result is plotted back down onto the road such that the lane area is identified. 


```python
yvals = range(0,warped.shape[0])
    
res_yvals = np.arange(warped.shape[0]-(window_height/2),0,-window_height)
    left_fit = np.polyfit(res_yvals, leftx, 2)
left_fitx = left_fit[0]*yvals*yvals + left_fit[1]*yvals + left_fit[2]
left_fitx = np.array(left_fitx,np.int32)
    
right_fit = np.polyfit(res_yvals, rightx, 2)
right_fitx = right_fit[0]*yvals*yvals + right_fit[1]*yvals + right_fit[2]
right_fitx = np.array(right_fitx,np.int32)
    
left_lane = np.array(list(zip(np.concatenate((left_fitx-window_width/2,left_fitx[::-1]+window_width/2), axis=0),np.concatenate((yvals,yvals[::-1]),axis=0))),np.int32)
.right_lane = np.array(list(zip(np.concatenate((right_fitx-window_width/2,right_fitx[::-1]+window_width/2), axis=0),np.concatenate((yvals,yvals[::-1]),axis=0))),np.int32)
inner_lane = np.array(list(zip(np.concatenate((left_fitx+window_width/2,right_fitx[::-1]-window_width/2), axis=0),np.concatenate((yvals,yvals[::-1]),axis=0))),np.int32)
    
road = np.zeros_like(img)
road_bkg = np.zeros_like(img)
cv2.fillPoly(road,[left_lane],color=[255, 0, 0])
cv2.fillPoly(road,[right_lane],color=[0, 0, 255])
cv2.fillPoly(road,[inner_lane],color=[0, 255, 0])
cv2.fillPoly(road_bkg,[left_lane],color=[255, 255, 255])
cv2.fillPoly(road_bkg,[right_lane],color=[255, 255, 255])  
    
road_warped = cv2.warpPerspective(road,Minv,img_size,flags=cv2.INTER_LINEAR)
road_warped_bkg = cv2.warpPerspective(road_bkg,Minv,img_size,flags=cv2.INTER_LINEAR)
    
base = cv2.addWeighted(img, 1.0, road_warped_bkg, -1.0, 0.0)
result = cv2.addWeighted(base, 1.0, road_warped, 0.7, 0.0)
```

Here is an example of the result on a image.  

### Identiﬁed Lane Area:
![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to the project video result](./output_tracked.mp4)

---

## Discussion

### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline almost works well it might be because the road is in basically ideal conditions on a fine weather. But sometimes it fails. For example the other car is on next lane to the own car. The pipeline needs to be refined to work in such environments.

For further research, it is nesesarry to continue to refine the pipeline to work in various conditions/environments.

