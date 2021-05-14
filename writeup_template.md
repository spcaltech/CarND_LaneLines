# **Finding Lane Lines on the Road** 


**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

This project is to detect and mark the lane lines in a static photo and then extend teh concept to a series of static images (video stream). A detetcion algorithm software pipeline is created to accomplish the task.

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I .... 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]
Convert Image to grayScale
Use CannyEdge to find the edges
Using OpenCV Morphological dilate operation to make edges more thick, because some edges were too much broken
Apply the region of interest and keep only the edges which are present in Region of Interest
Use hough algorithm to detect the lines from the image
From these lines get the left and right lines using Slope and also exclude lines below a specific thershold(.2 in our case)
Now using the points of left and right line get the m and c of y = mx + c by using np.polyfit and as we are using straight line we are using degree = 1
Now we are getting m and c for left and right lane.
As we know our region of interest and we know that value of y will be fixed in region of interest. We know y_min and y_max i.e. y1 and y2 of our lane line
Now converting the eq of line from y = mx + c to x = (y-c)/m we take absolute value of x from here
Now we have x1, y2 , x2, y2 using which we can draw lane lines
As we can see that lanes are less thicker at top we are using fillpoly to draw and fill lines and we are setting the width at the base of polygon more that the width at the top of the image

* Start with an image or frame from a video clip

<img src="test_images/solidWhiteCurve.jpg" width="480" alt="Combined Image" />

* Convert the image to grayscale

<img src="writeup_figs/solidWhiteCurve_gray.jpg" width="480" alt="Combined Image" />

* Apply Gaussian smoothing

<img src="writeup_figs/solidWhiteCurve_blur.jpg" width="480" alt="Combined Image" />

* Apply Canny edge detection function

<img src="writeup_figs/solidWhiteCurve_edges.jpg" width="480" alt="Combined Image" />

* Create mask on image to define working region

<img src="writeup_figs/solidWhiteCurve_masked.jpg" width="480" alt="Combined Image" />

* Apply Hough transform on the selected region

<img src="writeup_figs/solidWhiteCurve_hough.jpg" width="480" alt="Combined Image" />

* Manipulate the lines to get just two lane lines

<img src="writeup_figs/solidWhiteCurve_lines.jpg" width="480" alt="Combined Image" />

* Draw lines on top of original image


<img src="test_images_output/solidWhiteCurve.jpg" width="480" alt="Combined Image" />

Most of the steps above are straight forward and can be easily achieved with OpenCV.

I decided not to use color masking on my first attempt and it worked pretty well for the first part of the project.

The quadrilateral used to mask the image was hardcoded based on the images we received as an input. Ideally we want our function to work on any video stream, so that wasn't a good start. I will come back to this later on, when discussing the Challenge.

The other tricky part was on how to deal with all the lines coming from the Hough transform. For this part I decided to combine all the lines in a Pandas dataframe and split them based on angle thresholds that were fine tuned to isolate the lines that were interesting to us.

After splitting the data into left and right I fitted a line through each of these sets and got an equation for each proposed lane line. Now it was just a matter of finding 2 points for each line and plotting.

The results can be seen in this [video](test_videos_output/solidWhiteRight.mp4) and [video](test_videos_output/solidYellowLeft.mp4)

### 2. Identify potential shortcomings with your current pipeline


The proposed solution worked well for the first set of problems that we had to solve, but as expected it didn't work well in images that were significantly different than the ones we optimized the function to work on.

Needless to say that the approach also failed miserably when I used it on the challenge video stream.

I was particularly bothered by the fact that we were relying on predetermined mask coordinates and other hardcoded parameters such as the Canny Thresholds.


### 3. Suggest possible improvements to your pipeline

For the second part of this project I decided that i was going to go for a solution that generalizes better.

I replaced the mask with a series of constraints on how far the Hough lines are expected to be from the middle of the car hood, and introduced a new constraint on the expected angles of these lines.

I also had to  deal with more challenging light and contrast in the image, so color masking could no longer be ignored.

To solve that problem I converted the images from RGB into HSV to allow a better distinction between the colors in several light conditions.

The color masking combined with distance and angle constraints for the hough lines were enough to identify lines of interest and I could then drop the polygonal mask.

The last problem I faced was that it was still not calculating lines for all the frames and the lines could be significantly different from one frame to another depending on how visible the lane line was.

Since we expect this solution to work on public roads it is reasonable to assume that the lane line from the previous frame is a good approximation to where the lane line should be in the current frame. So for all frames that had no lines identified I used the line from the previous frame.

To solve the "shakiness" of the lines I added a smoothing parameter and would only allow lines to change by 15% per iteration on the final version.

The results can be seen in this [video](https://github.com/bguisard/CarND-LaneLines-P1/tree/master/test_videos_output/challenge.mp4)
