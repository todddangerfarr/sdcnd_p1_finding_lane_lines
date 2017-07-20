**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

For this project the pipeline was built from the following 5 steps:

1. Convert the Image to Grayscale
2. Blur the image to reduce noise
3. Use Canny Edge Detection to create an edge gradient image
4. Mask the region of interest based on camera mounting location
5. Apply Hough Lines to find the Lanes

To build a robust pipeline I iteratively tested each of these steps to find the optima parameters for this particular application.  The first major operation in the pipeline was blurring the image.  This step is important as reducing the noise helps colligate jagged line segments into continuous lines in the later pipeline operations.  I tested odd kernel sizes from 1-9 with the following results:

[Gaussian Test]: ./writeup_images/gaussin_blur_test.png "Gaussian Blur Study"

I determine visually that a kernel_size value of 7 gave the best results.  The jagged pixel around the line lines started to smooth, but the image was not too blurry.  

The next step ended up being out of order from my pipeline but proved to be important to visualize the other iterative studies required for a robust process.  This was to setup masking vertices and test them across all the test images.   

[Masking Vertices]: ./writeup_images/masking_vertices_check.png "Masking"

In hindsight, these are simply hardcoded values and probably one of the weakest components in my pipeline as it requires the same camera mounting location and image size across all vehicles.  To improve this I'd try to build a set of vertices off a mathematical algorithm based on incoming images.

After the vertices we establish, the next step was to test and tune the Canny Edge Detection algorithm.  There are two main input variable for this algorithm: the lower and upper threshold values.  These determine which gradient values are returned in the output image and can range from 0-255.  For this I iterated the lower threshold value from 10 - 80 by steps of 10 and then calculated the upper threshold based on this value by using two industry rule of thumbs: 2X and 3X this lower threshold value.  My results were as follows:

[Canny Threshold Study]: ./writeup_images/canny_threshold_study.png "Canny"

Again, using visual inspection I determined the optimal values for this case are a lower threshold value of 60 and a upper threshold value calculated by the 3X rule of 180.  

The final step was to apply the Hough Lines algorithm.  For this algorithm on this application the influential variables are min_line_length and max_line_gap.  These variables are fairly explicit, but min_line_length is the consecutive number of pixels required to be a line and max_line_gap is the max gap between these "consecutive" pixels to be included in that line.  I iterated these values between 5 - 40 with a step size of 5.  The results can be seen below:

[Hough Lines Study]: ./writeup_images/hough_lines_study.png "Hough Lines"

I selected a min_line_length = 5 and a max_line_gap = 10.  This produced a nice long set of lines for the solid lane line and then kept the dashed line as a dashed line.  This is important because on the road these two types of lines mean different things, something a self-driving car should know.

Once I finished all of these studies I verified pipeline integrity first on my test image and then on the full set of test images.  The results can be seen below:

[Pipeline Quick Test]: ./writeup_images/pipeline_test_hough.png

[Pipeline Images Test]: ./writeup_images/finding_lines_test.png

Once it was verified that the pipeline was functioning correctly I went back and wrote the code for extrapolating these line segments into a set of continuous lines.  The Hough Lines algorithm returns a series of lines identified by a start and end point.  I iterated through each of these line segments and calculated the slope of each using (x2 - x1) / (y2 - y1).  Using this calculated slope it could be determined which lane line this line segment's points were associated with.  Due to the fact that our image coordinates (0, 0) are located in the upper left this slope value is actually counterintuitive.  Therefore points that are associated with the left lane line will have a negative slope while the right lane line will contain line segment points with the opposite condition.  I also had to enforce slope limits of 1 to 2 for the positive line slopes and -1 to -2 for the negative line slopes to remove points from horizontal lines.

After each of the line segment points were separated into their respective lists, I applied the OpenCV function fitLine() to each set of points.  This function returns the best fit line through the provided points in vector form [vx, vy, y, x].  In order to draw these lines on the image I needed to calculate and appropriate start and end point for each line.  This was done by calculating the slope (vx / vy) and intercept (y - (slope * x)) of this line and then calculating a set of points on that line constrained to the image space with the appropriate length.  The output of these new draw_extrapolated_lines method can be seen below:

[Extrapolated Lines Test]: ./writeup_images/pipeline_test_extrapolate.png

### 2. Identify potential shortcomings with your current pipeline

1. Vertices


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
