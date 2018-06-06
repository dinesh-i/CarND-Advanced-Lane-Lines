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

[image1]: camera_cal/calibration1.jpg "Distorted Image"
[image2]: ./output_images/calibrated_image1.jpg "Calibrated Image"
[image3]: ./output_images/grad_thresh_image.jpg "Gradient Threshold Image"
[image4]: ./output_images/test1_h_image.jpg "Hue Image"
[image5]: ./output_images/test1_s_image.jpg "Saturation Image"
[image6]: ./output_images/visualized_lane.jpg "Visualized Lane Image"
[image7]: ./output_images/warped_lane_image_combined.jpg "Combined Result of differenrt channels"
[image8]: ./output_images/warped_lane_image_dir_binary.jpg "Direction"
[image9]: ./output_images/warped_lane_image_gradx.jpg "Gradient X Image"
[image10]: ./output_images/warped_lane_image_grady.jpg "Gradient Y Image"
[image11]: ./output_images/warped_lane_image_hue_binary.jpg "Hue Image"
[image12]: ./output_images/warped_lane_image_mag_binary.jpg "Magnitude Image"
[image13]: ./output_images/warped_lane_image_saturation_binary.jpg "Saturation Image"
[image14]: ./output_images/warped_lane_image.jpg "Warped Lane Image"
[image15]: ./output_images/grad_thresh_image_cv2.jpg "Gradient Threshold Image"
[image16]: ./test_images/test3.jpg "Test Image"
[image17]: ./output_images/lane_highlighted_image.jpg "Lane Highlighted Image"

[video1]: ./project_video_output.mp4 "Project output video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
The file "advanced_lane_finder.ipynb" contains the complete source code of all the functionality defined in this document.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "advanced_lane_finder.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Distorted Image             |  Undistorted Image
:-------------------------:|:-------------------------:
![alt text][image1]        |  ![alt text][image2]

### Pipeline (single images)

#### Distortion corrected image
This is the first step in the pipeline to ensure that the given image is corrected for distortion so that the rest of the steps in the pipeline can work on a undistorted image. Steps followed are same as outlined in the section Camera Calibration"

#### Apply Perspective Transform
I defined the function "warp_image" to apply perpective transform on the lane portion of the image and warp the image. I referred to one of the references from the article https://medium.com/@vamsiramakrishnan/robust-lane-finding-using-python-open-cv-63eb66fa2616 to understand how to play around with the source pixel values. 

Below are the pixes values I have chosed as source points for perspective transform:
[200,660],[1200,660],[800,480],[540,480]
For Destination points, I have chosen the four corners of the image based on the image size.

Applying the warping step helped me to focus only on the lane area of the image when I proceeded further with color, gradient threshold steps.

Test Image                 |  Warped Image
:-------------------------:|:-------------------------:
![alt text][image16]       |  ![alt text][image14]


#### Apply Color and Gradient Threshold

I applied various color and gradient thresholds and tried out different threshold values. Images below shows the impact to the image when various color and gradient thresholds are applied:

Warped Lane Image          |  Gradient X Image        |Gradient Y Image                |  Magnitude Image |
:-------------------------:|:------------------------:|:-------------------------:|:-------------------------:
![alt text][image14]       |  ![alt text][image9]    |![alt text][image10]       |  ![alt text][image12]    |

Direction          |         S Channel Image                |  H Channel Image | Combined Threshold and Gradient|
:-------------------------:|:------------------------:|:-------------------------:|:-------------------------:
![alt text][image8]       |  ![alt text][image13]    |![alt text][image11]       |  ![alt text][image7]    |

I found that S channel was promising to detect lanes in most of the cases whereas magnitude and direction images were noisy in most of the cases. After trying out different combinations, I skipped magnitude & direction and used the following code to derive the combination of color and gradient thresholds:
    | combined[((gradx == 1) & (grady == 1)) | ((saturation_binary == 1) & (hue_binary == 1))] = 1 |

#### Apply sliding window search, fit a polynomial and highlight the lanes
I applied sliding window search(Refer the method sliding_window_search in cell#9) to identify the lanes. Generated a histogram of the binary thresholded image and used the following values for the sliding window search:
number of windows = 9
Min no.of pixels to recenter window = 50
Width of the window = 100

Once the left and right line pixels are identified, I applied second order polynomial function using np.polyfit function.

I also implemented the method "get_radius_of_curvature" to compute the radius of curvature of the left and right lanes in meter. The final image with the lanes highlighted is shown below:

Highlight Left and Right Lanes|
:-------------------------:
![alt text][image6]      |


#### Highlight Lanes in the original image and update details on radius of curvature and distance from lane centre
I implemented the method "highlight_lane"(Cell#10) to apply the image where the lane area is highlighted on the original image. 
The method "get_lane_center_info"(Cell #11) computer the position of the center of the car based on the pixel position of the left and right lanes. It compares this value against the horizontal center position of the image and checks how far the vehicle is from the center of the road in meter.


#### Pipeline
The method "pipeline" (Cell# 16) orchestrates all the above steps in order to produce the following result:

Unwarped Image with lanes highlighted|
:-------------------------:
![alt text][image17]      |


---

### Pipeline (video)

#### The above pipeline was applied to the project_video.mp4 file and it worked pretty well to highlight the lanes).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
This pipeline works well in well light conditions and light shadows. It should be improved further to handle situations like dark shadows, poor visibility etc. Need to check on different color and gradient thresholds to handle such scenarios.

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
