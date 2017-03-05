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

###Pipeline (test_images)

####1. Undistorting image
Using the same function mentioned earlier, <b>undistort_image</b>, I first undistort the test image.
![alt tag](README_images/pipe_undist.png)

####2. Binary threshold of image
I have defined function <b>color_threshold</b> to find binary threshold of my input image. First I convert the image to HLS color space and using thresold values for 'S' channel I calculate s_binary. Then using SobelX gradients I find gradient binary image sxbinary. Finally I combine these two binary images by adding the two and I get below image:
![alt tag](README_images/pipe_bin_thresh.png)
