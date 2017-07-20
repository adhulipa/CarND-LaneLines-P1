# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

Briefly, my pipeline is:
1. Color selection
2. Grayscale conversion
3. Edge detection
4. Region of interest masking 
5. Hough transform for line detection
6. Line filtering and extrapolation
7. Generate final result - image with annotated lane lines

My pipeline consisted of the following steps:
##### 1. Color selection
I wrote code to perform both white selection & yellow selection. The white selection was straightforward. The yellow selection was a little different. RGB values don't linearly translate into singular colors (clearly, because yellow is a "mixture" of red & green). Therefore, I defined a range of possible RGB values that describe yellow lines. The specific values that I used are present in my project in the "pipeline" function.
I then performed the color masking and combined the outputs for both the color selections.

One possible improvement I thought of makign here was converting the image to a HSL format and fllter based on the hues. I didn't look too much into it but I feel that would be a useful avenue to explore. Part of the problem of the RGB based yellow selection was that some pixels neighboring the yellow pixels would be masked out even thought they are yellow-ish and would have tremndously improved the succeeding steps of the pipeline. This is because having more pixels would help define the line more prominently and would've aided in the hough transform step.

#### 2. Grayscale conversion to Hough transforms
These steps were straightforward. I tuned the parameters for many iterations and submitted the best values. Throughout the process I learned that fine-tuning the parameters does not necessarily translate into better outcomes. This made me realize the beauty of the various individual algorithms (Canny Edge detection & Hough transforms) becase they were designed to be resillient to large varitions in input and still extract the same features each time.

I also learned that the key insight and effort required for this project was in filtering & extrapolating the set of lines returned by Hough transforms.

#### 3. The draw_lines() function
My draw_lines() functions performed the following steps:
1. Bifurcate all the lines in the image into 'left' or 'right' lines
2. Group together similar lines (i.e. lines that have almost equal slopes)
3. Condense each group of lines to a single line segment

i. Bifurcate all the lines in the image into 'left' or 'right' lines:
It's easy to see that left edge of any lane has a negative valued slope and right edges have positive values. This is because the angles at which left & right edges of the lane meet the x-axis.
My pipeline explots this to bifurcate the detected lines into left/right lane edges

ii. Group together similar lines:
Given the set of all lines that are leaning left or leaning right, I'd like to now detect lines that are parallel to each other. This is because the lines detected for each border of a lane are usually parallel to each other. My pipeline takes advantage of this and groups together lines of similar slope. This filters out any unusual lines detected near the top-right edges of my region-of-interest. It also helped tremendously in filtering out line describing shadows of trees, other vehicles on the road and shrubs or bushed on shoulder.

iii. Condense each group of lines into a single line segment.
This step of the pipeline tries to extrapolate the line segment to cover the full height of the video. I applied a simple operation taking advantage of the line equation to determine the extended section of the line segment. This was very useful in stiching together the lines detected for the white line segments of the lane borders.

### 2. Identify potential shortcomings with your current pipeline

* The lane lines are not always sticking to the lane borders. If there is high curvature of the road, then my detected line tend to jump over the place
* My pipeline doesn't handle unreasonable values for the line endpoints. Ocassionally, my lane line would extend across the diagonal of the image. This is because the extrapolation function I've defined is very basic and can be thrown off by spurious points not filtered out in previous steps (such (0,0))
* The lane lines I've detected and annotated in my video are not smoothened. This implies that for the lane with broken lines, my pipeline would not present an line of constant length. It would expand and collapse as the car is moving. 
* The two edges of the lane I've detected are not always of the same length. Hence, the lane is bordered by unequal lines. This is inelegant

### 3. Suggest possible improvements to your pipeline

Potential avenues for improvement:
* It would be useful to convert the image to other color-spaces and extract specific colors to detect lanes. For example, it would be useful to explore color transformations to HSL and YUV spaces before color selection. It would also be useful to invert the image and apply a complementary masking so that areas such regions with trees or grass can be masked out without losing the yellow lanes.

* My draw_lines() function can implement more sophistcated line filtering algorithms. It would be useful to perform a least-squared-fit of a group of lines to find the most dominat line. This can then be used to define a single lane.

* My pipeline would benefit if the algorithms I implemented were all vectorized. Most of my code right now is iterative for loops. But, the nature of the problem and the numpy toolkit provide tremendous potential to speed up and improve the efficiency of the algorithm by taking advantage of matrix operations on the inputs.
