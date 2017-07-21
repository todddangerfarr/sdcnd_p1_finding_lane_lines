**Self Driving Car ND Project 1 - Finding Lane Lines on the Road**

The goals / steps of this project are outlined below:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[gaussian_test]: ./writeup_images/gaussian_blur_test.png "Gaussian Blur Study"
[masking_vertices]: ./writeup_images/masking_vertices_check.png "Masking"
[canny_threshold_study]: ./writeup_images/canny_threshold_study.png "Canny"
[hough_lines_study]: ./writeup_images/hough_lines_study.png "Hough Lines"
[pipeline_quick_test]: ./writeup_images/pipeline_test_hough.png "Pipeline Quick Test"
[pipeline_images_test]: ./writeup_images/finding_lines_test.png "Pipeline Images Test"
[extrapolated_lines_test]: ./writeup_images/pipeline_test_extrapolate.png "Extrapolated Lines Test"

[sold_white_output]: ./writeup_images/solid_white_output.gif "Solid White Output"
[solid_yellow_output]: ./writeup_images/solid_yellow_output.gif "Solid Yellow Output"

---

## Reflection
---

For this project the pipeline was built from the following 5 steps:

1. Convert the Image to Grayscale
2. Blur the image to reduce noise
3. Use Canny Edge Detection to create an edge gradient image
4. Mask the region of interest based on camera mounting location
5. Apply Hough Lines to find the Lane Lines

To build a robust pipeline I iteratively tested each of these steps to find the optimal parameters for this particular application.  The first major operation in the pipeline was blurring the image.  This step is important as reducing the noise helps colligate jagged line segments into continuous lines in the later pipeline operations.  I tested odd kernel sizes from 1-9 with the following results:

![Gaussian Test][gaussian_test]

I determine visually that a kernel_size value of 7 gave the best blurring results.  The jagged pixels around the angular lane lines started to smooth, but the image was also not too blurry.  

The next step ended up being out of order from my pipeline but proved to be important to visualize the other iterative studies required for a robust process.  This was to setup masking vertices and test the region across all the test images.   

![Masking Vertices Test][masking_vertices]

In hindsight, these are simply hardcoded values and probably one of the weakest components in my pipeline as it requires the same camera mounting location and image resolution across all vehicles.  To improve this I'd try to build a set of vertices from a mathematical algorithm based on incoming image properties.

After the vertices were establish, the next step was to test and tune the Canny Edge Detection algorithm.  There are two main input variables for this algorithm: the lower and upper threshold values.  These determine which gradient values are returned in the output image and can range from 0-255.  For this I iterated the lower threshold value from 10 to 80 by steps of 10 and then calculated the upper threshold based on this value by using two industry rules of thumb: 2X and 3X this lower threshold value.  My results were as follows:

![Canny Threshold Study][canny_threshold_study]

Again, using visual inspection I determined the optimal values for this case are a lower threshold value of 60 and a upper threshold value calculated by the 3X rule for a value of 180.  

The final step was to apply the Hough Lines algorithm.  For this algorithm on this application the influential variables are min_line_length and max_line_gap.  These variables are fairly explicit, but min_line_length is the consecutive number of pixels required to be a line and max_line_gap is the max gap between these "consecutive" pixels to be included in that line.  I iterated these values between 5 - 40 with a step size of 5.  The results can be seen below:

![Hough Lines Study][hough_lines_study]

I selected the image conditions with a min_line_length = 5 and a max_line_gap = 10.  This produced a nice long set of lines for the solid lane line and then kept the dashed line as a dashed line.  This is important because on the road these two types of lines mean different things, something a self-driving car should probably know.

Once I finished all of these studies I verified pipeline integrity first on the single test image and then on the full set of test images.  The results can be seen below:

Single image test:

![Pipeline Quick Test][pipeline_quick_test]

Testing on the full set of images:

![Pipeline Images Test][pipeline_images_test]

Once it was verified that the pipeline was functioning correctly I went back and wrote the code for extrapolating these line segments into a set of continuous lines.  The Hough Lines algorithm returns a series of lines identified by a set of start and end points.  I iterated through each of these line segments and calculated the slope of each using (x2 - x1) / (y2 - y1).  Using this calculated slope it can be determined which lane line this line segment is associated with.  However, one thing to note is that due to the fact that our image (0, 0) coordinates are located in the upper left this slope value is actually counterintuitive.  Meaning points that are associated with the left lane line will actually have a negative slope while the right lane line will contain line segment points with the opposite condition: a positive slope.  I also had to enforce slope limits of 1 to 2 for the positive line slopes and -1 to -2 for the negative line slopes to remove points from "horizontal" lines detecting the top and bottom of the dashed lines and the road reflectors.

After each of the line segment points were separated into their respective lists, I applied the OpenCV function fitLine() to each set of points.  This function returns the best fit line through the provided points in vector form [vx, vy, y, x].  In order to draw these lines on the image I needed to calculate and appropriate start and end point for each line.  This was done by calculating the slope (vx / vy) and intercept (y - (slope * x)) of this line and then calculating a set of points on that line constrained to the image space with the appropriate length.  The output of these new draw_extrapolated_lines method can be seen below:

![Extrapolated Lines Test][extrapolated_lines_test]

Finally the pipeline was applied to the videos.

Solid White Right Output:
![Solid White Right Output][solid_white_output]

Solid Yellow Left Output:
![Solid Yellow Left Output][solid_yellow_output]

### 2. Identify potential shortcomings with your current pipeline

As previously mentioned one of the shortcomings of this pipeline is the fact that the vertices for image masking are hardcoded and not mathematically calculated or referenced to the incoming images or video frames.  This instantly broke on the Challenge video as it has overall larger resolution in x and y.

Another shortcoming is that my pipeline does not support color recognition of any kind.  It didn't seem to make a big difference on the two test videos required for this project, but I imagine it would be nice to find the lines and also identify their color.  In addition to this maybe my pipeline is susceptible to error when road colors changes like in the challenge video there's a transition from asphalt to concrete.

### 3. Suggest possible improvements to your pipeline

As mentioned above a big improvement could be to constrain the masking shape mathematically to the incoming image or video frame shapes.  I'd also like to look at adding color detection to help mask unwanted pixels that are not in a specified range of yellows or whites. Finally, darkening the image during the day time might help with color and line sensitivity over the lighter concrete road sections.
