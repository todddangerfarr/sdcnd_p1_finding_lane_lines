## Self Driving Car ND Project 1 - Finding Lane Lines on the Road

---

The goals / steps of this project are outlined below:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[gaussian_test]: ./writeup_images/gaussian_blur_study.png "Gaussian Blur Study"
[masking_vertices]: ./writeup_images/masking_vertices_check.png "Masking"
[canny_threshold_study]: ./writeup_images/canny_threshold_study.png "Canny"
[hough_lines_study]: ./writeup_images/hough_lines_study.png "Hough Lines"
[pipeline_quick_test]: ./writeup_images/pipeline_test_hough.png "Pipeline Quick Test"
[pipeline_images_test]: ./writeup_images/finding_lines_test.png "Pipeline Images Test"
[extrapolated_lines_test]: ./writeup_images/pipeline_test_extrapolate.png "Extrapolated Lines Test"

[solid_white_output]: ./writeup_images/solid_white_output.gif "Solid White Output"
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

In hindsight, these were originally hardcoded values and probably one of the weakest components in my pipeline as it requires the similar camera mounting locations and image resolution across all vehicles.  It instantly broke when I tried run my pipeline on the challenge video. To improve this I did end up  adding hardcoded multipliers that at least take into account the incoming image dimensions, but I still feel this can be improved.  

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


## Identify potential shortcomings with your current pipeline

---

The biggest shortcoming is that this pipeline will really only detect "straight-ish" lane lines, which makes it more ideal for these highway videos.  However, there will definitely be problems with winding and tightly curved roads.  

There will also be problems with steep or undulating road conditions as the region of interest is calculated from the image frame dimensions and not dynamically scaled via references in the images.   

Another shortcoming is that my original pipeline did not support color recognition of any kind.  It didn't seem to make a big difference on the two test videos required for this project, but on the challenge video the transitions from asphalt to concrete and changing lighting conditions did cause a lot of problems.  I later had to add color masking to the pipeline for the challenge video which helped a lot with these transitions but is still not perfect.  

## Suggest possible improvements to your pipeline

---

If this were a real world application we'd want to do much more regression testing and iterations of our parameters to really fine tune our detection algorithm variables for multiple road/lighting conditions and incoming video feeds. Lastly it would be helpful to reference our masking conditions to a detected horizon reference in the image so that we could shorten or elongate the mask with changing road inclines.  
