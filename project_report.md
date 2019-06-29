## Writeup Template

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

[distorted]: ./camera_cal/calibration1.jpg "Undistorted"
[undistchess]: ./output_images/undistorted_chessboard.jpg "Undistorted"
[original]: ./output_images/original_image.png "Undistorted"
[undist]: ./output_images/undistorted.png "Undistorted"
[edges]: ./output_images/edges.png "Undistorted"
[lane]: ./output_images/lane_region.png "Undistorted"
[masked]: ./output_images/masked_image.png "Undistorted"
[topview]: ./output_images/topview.png "Undistorted"
[topregion]: ./output_images/topview_region.png "Undistorted"
[stackwindows]: ./output_images/windowed_image.png "Undistorted"
[weighted]: ./output_images/weighted_image.png "Undistorted"
[annotated]: ./output_images/annotated_image.png "Undistorted"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The first preparatory step is to calibrate the camera to remove the effects of
distortions arising from the nature and alignment of the cameral lenses. This is
typically done by taking several photographs of a chessboard on a flat surface
and detecting the corners in the chessboard using OpenCVs built in fuctions.
Once four key corners are identified on the image (i.e., image points) they are
used, along with the corresponding points on the object (i.e., the object
points) to calibrate the camera. The procedure is well illustrated in this
github repository. The calibration step generates the calibration matrix which
can then be used to undistort any other images taken by the same camera. The key
code snippet is provided below:

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

```python

ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, img_size,None,None)
dst = cv2.undistort(img, mtx, dist, None, mtx)
```
The code for this step is contained in the XXXX code cell of the IPython
notebook located in "./P2.ipynb" . Shown below is a a distorted and undistorted image:

<p align="center">
<img src="camera_cal/calibration1.jpg" width="45%" alt>
<img src="report_images/undistorted_chessboard.jpg" width="45%">
</p>
<p align="center">
<em> Distorted (left) and undistorted (right) chessboard images </em>
</p>

### Pipeline (single images)

To describe the pipeline, the images generated at each step will be provided. The results will be shown for the following image:
![alt text][original]

#### 1. Provide an example of a distortion-corrected image.


When distortion correction is applied to the above image, the following undistorted image is obtained.

![alt text][undist]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In order to identify the lane lines, the edges in the image need to be
identified. The Sobel algorithm was used to identify gradients in the x and y
direction. The RGB color scale was used for performing these operations.
However, using the HLS color representation has been known offer more
flexibility in capturing lane markings while excluding sharp gradients such as
those associated with dark regions in the image. Shown below is the image obtained by
using the Sobel and color gradient approaches:

![alt text][edges]

The edge detection algorithm captures all edges. Since we are only interested in the lane lines we can eliminate edges associated with background and vehicles by choosing a region of intereest in the image where we expect the lane lines to be present. This is based on the assumption that the position of the lane lines with respect to the vehicle camera does not change much. A quadrilateral region was selected
and all edges outside this region were blanked out. The resultant image is shown below:

![alt text][masked]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The masked region obtained in the previous step is a warped image of the lanes. Because of the angle of the camera with respect to the road, the left lane typically appears to have a positive slope while the right lane appears to have a negative slope. In order to capture the lane properties such as the radius of curvature, it is necessary to have a top view of the road. This can be accomplished using a perspective transform.

In order to perform a perspective transform, a quadrilateral region in the
original image is mapped onto a rectangular region in the transformed image. One
way to choose four points (source points) is to take a sample image and identify
points on a straight line lane. We then map these points on to a rectangular
region, specified by four destination points. The `warpPerspective()` function
is then used to  extract the transformation matrix and the inverse
transformation matrix corresponding to this mapping. The relevant code for this
operation is provided below:

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

The top view image obtained by applying the perspective transform, is shown below:

![alt text][topview]

The fact that the lane lines in the transformed image are parallel to each other gives us confidence that the transformation behaves as expected.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Once the topview is obtained, the lane lines need to be identified. To do this,
a histogram of is generated, which sums the white pixels in the y-direction for
each x-location in the image. The two peaks in the histogram may be assumed to
be close to the location of the two lane lines. The lane pixels are analyzed by
using rectangular windows which are placed where we expect the lane lines to be
present. In the present work, 9 windows were staked one on top of the other to
track the lane lines along the image.

The first window is placed at the bottom of the image with its center at the
locations corresponding to the histogram peaks. If the number of pixels in a
window exceeds a threshold (which implies that there is a legitimate segment of
the lane in the window) the average x-coordinate of white pixels in the window
is computed. The center of the next lane window is then adjusted to match averaged
x-coordinate of its predecessor window. This ensures that the windows adjusts to
the lane location if there is significant curvature.

Once all the lane points are computed using the stacked windows, a polynomial is
fit to the left and right lane points. Since the lane is typically vertical, the
y-location is treated as the independent variable and the x-location is treated
as the independent variable for fitting the polynomial. The `polyfit()` function
is used to determine the coefficients of the polynomial.

The figure below shows the top view with the stacked windows along with
lane lines and lane points.

<p align="center">
<img src="output_images/windowed_image.png" width="75%">
</p>

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The previous section, described the approach used to fit a polynomial to the left
and right lanes. To compute the radius of curvature of the lane, the x-coordinates
of the left and right lanes were averaged to obtained the curve corresponding to
the center line of the lane. The center line so obtained was fit to a polynomials.
Once the coefficients of the polynomial for the center line were obtained, the
radius of curvature can be computed using the formula provided below:



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Once the lane regions are identified on the top view image, they are mapped
back onto a blank image that has the same dimensions as the original image, as
shown below:

![alt text][topregion]

Finally, the above image is combined with the original image with suitable weights.
The resultant image is shown below:

![alt text][weighted]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Once the procedure for processing a single image was formulated, the same
approach can be used to process a video by processing the individual images that
constitute the video. A link to the processed video is provided below:

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Outlined below are some of the issues faced while implementing this project
along with a description of how they were addressed:

*

While the current project achieves satisfactory results, it is not perfect. Some
of the shortcomings of the current approach are outlined below, along with ideas
for improving the algorithm:
