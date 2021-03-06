## Advanced Lane Finding

---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

---
###Camera Calibration

I start by defining function <B>calibrate_camera</B> where I prepare "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (6, 9) plane at z=0, such that the object points are the same for each calibration image. Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output objpoints and imgpoints to undistort the image inside the function <b>undistort_image</b>. In this fucntion I first compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to the test image using the cv2.undistort() function and obtained this result
![alt tag](README_images/chessboard1.png)
![alt tag](README_images/chessboard1_undist.png)

###Pipeline (test_images)

####1. Undistorting image
Using the same function mentioned earlier, <b>undistort_image</b>, I first undistort the test image.
![alt tag](README_images/pipe_undist.png)

####2. Binary threshold of image
I have defined function <b>color_threshold</b> to find binary threshold of my input image. First I convert the image to HLS color space and using thresold values for 'S' channel I calculate s_binary. Then using SobelX gradients I find gradient binary image sxbinary. Finally I combine these two binary images by adding the two and I get below image:
![alt tag](README_images/pipe_bin_thresh.png)

####3. Transform Perspective of image
I defined function <b>transform_perspective</b> to transform perspective of my previous binary warped image. The source and destination polygon points are as below:

    src = np.array(
        [[(img_size[1] / 2) - 60, img_size[0] / 2 + 100],
        [((img_size[1] / 6) - 12), img_size[0]],
        [(img_size[1] * 5 / 6) + 50, img_size[0]],
        [(img_size[1] / 2 + 62), img_size[0] / 2 + 100]], np.float32)
    dst = np.array(
        [[(img_size[1] / 4), 0],
        [(img_size[1] / 4), img_size[0]],
        [(img_size[1] * 3 / 4), img_size[0]],
        [(img_size[1] * 3 / 4), 0]], np.float32)
        
Using these src and dst points, I calculate M = cv2.getPerspectiveTransform(src, dst) and also Minv = cv2.getPerspectiveTransform(dst, src). Finally I warp the image by using cv2.warpPerspective and return the warped image along with the inverse warp coefficients Minv whic will be used later on in <b>transform_inv_perspective</b>.

I also drew polygons for src and dst points to verify my warping function is working fine.
![alt tag](README_images/pipe_persp_src.png)

Below is the output of the warped binary image:
![alt tag](README_images/pipe_persp.png)

####4. Fitting lane lines
In my function <b>fit_lane_lines</b> I used sliding window to first calulate my left and right lane points. Once I had my points, I used np.polyfit to fit a second order polynomial on these points. Below is the output of this function:
![alt tag](README_images/pipe_fit_lines.png)

I also created function <b>fit_new_lane_lines</b> to fit lines for newer frames in video. Since once a line is calculated we need not do sliding window technique again, but just keep modifying the polynomial coefficients.

####5. Calculate Radius of Curvature and Distance from center.
I defined function <b>get_radius_curvature</b> to calculate radius of curvature. First I fit a new polynomial to the world space instead of image space. I use the pixels to space conversion parameters:
    ym_per_pix = 30/720 # meters per pixel in y dimension
    xm_per_pix = 3.7/700 # meters per pixel in x dimension
Once I have this new polynomial, I used the formula from the lecture to find the curvature of left and right lanes:
    left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])

For distance off center, I use function <b>get_off_center_distance</b>. Using image histogram I first get pixel value of left base and right base where we see the lane lines. Then I caluclate the lane center using these values. Image center is half of image shape in x direction. The difference between lane center and image center gives us the off center distance. Then I convert the pixel values to world space by using xm_per_pix = 3.7/700.

Finally in function <b>add_data_to_image</b> I add these two data to the final image. I use average of left and right curvature and print that on the image as below:
![alt tag](README_images/pipe_final.png)

###Pipeline (video)
I have uploaded the final video output to my git repository.

###Discussion
Problems with the pipeline:
* Pipeline will not work for challenge video, since there is a distinctive line inside the lane which is detected as the left edge of the lane. Thus gradient does not work here. Proper thresholding of yellow and white lines may work
* Pipleine also fails when there is a road divider. The bottom edge of the divider gets detected as the lane edge. This can also be avoided by color thresholding for white and yellow lines.
* The pipeline also does not work on the harder challenge video where there is lot of glare on the windshield. Some sort of polarization filtering can help reduce the glare.
* The pipleine gives very varying radius of curvature, a better formula or algorithm can be used to calculate the radius of curvature.
* Pipeline may fail for super bright roads like seen in harder challenge video. A good color scheme must be used to consider for the road brightness.
* Pipeline will also fail for very reflective roads, for example when it is raining. As that introduces too many reflective edges that will be detected as false positive for lane edge.
* Sometimes on the road lane line are modified and old lane lines might still be barely visible. This may also lead to false positive which can be remove by color thresholding for only white and yellow lane lines.
