# **Finding Lane Lines on the Road** 

## 


**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/solidYellowCurve.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps:
1. Color mask the image to amplify the white and yellow lanes
2. Remove noise - apply gaussian blur filter
3. Find edges using canny edge algorithm
4. Remove all edges outside of ROI 
5. Detect lines using hough algorithm
6. Draw detected left and right lines (one line per side)

The pipeline itself is pritty straight forward. The tricky part is transforming the line segments coming out of the hough algorithm into a single line for each side. Ther are different approaches to achive this. I decided to do some research and found out that openCV provides a very handy function which exectly does this: **cv2.fitLine()**. 
With this function it is quite easy to derive the parameter *m* and *b* for the standard equation of a line *(y = m x + b)*. The other tricky part was to remove the line flickering between different frames within a video. I decided to address this issue by using a buffer for the line parameters *m,b* and calculating an average based on the current line and the past lines. This reduces the flickering.

In order to draw a single line on the left and right lanes, I introduced the get_lane_lines() function which is the core of the algorithm.
It performs the following steps:
1. Calculate the slope and length for each detected line
2. Filter out small lines (small 4 pixels)
3. Sort the lines into left and right lines
4. Calculate the avg slope for each side (to be able to do 5.)
5. Discard outlier
6. Use cv2.fitLine() function to calculate a mean line function for the given line segments per side
7. The result is stored in a line_buffer which is used when processing video frame to reduce flicker
8. return the coordinate of the left and right line


The following image shows an example output of the pipeline: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


Clear the current pipline works well in case of straigt street. Curves are not well handled due to the linear approach taken for line detection. One enhancement could be to use smaller line segements and modele the curves through this approach.

Handling situations in which you have multiple lane lines for one lane (i.e. constrcution work - at least in Germany - lead to mutiple lines on the street - one for the construction - one for the former lane). Handling this kind of scenario likely requires the system been able to process context information to detect the situation.

Edge case have not been tested, especialy different light condition like day vs night, direct sun, fog and so on. The alogorithums might work within these scenarios, but hasn't been tested. It is very likely that at least some of the constants need to be adjusted to produce the desired output for different lightning conditions



### 3. Suggest possible improvements to your pipeline

A possible improvement would be to further invest time to fine-tune the various parameter within the algorithm

Another shortcomming is the fixed and non adaptive ROI apprach. This approach will have challanges if the road is not plane, also the approach is cleary optimized for the given example videos. Also the color masking approach is limited to the given examples. Different light situation will negativly impact the result.
