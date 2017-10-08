# **Finding Lane Lines on the Road**

---

## Aim of this project

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

## Reflections

### 1. Description

My pipeline consisted of 5 steps. First, I converted the images to gray-scale
(using the `cv2.cvtColor` function), then I applied Gaussian blur (`cv2.GaussianBlur`
with a kernel size of 3) to smoothen the pixel intensities (which effectively
reduces rapid changes in the gradient).

Next, I used the `cv2.Canny` transform to detect edges in the image, which then
allowed me to perform the Hough transform, and find line candidates in the image
(all done by the `cv2.HoughLinesP` function).

Then, I used the `region_of_interest` function to cut out the lines lying in the
relevant region (the triangle in which we expect to find the lines).

The next step was the part that actually required my own work -- I needed to draw
a single line on the left and right lanes. As suggested, I modified the `draw_lines`
function, by:

  1. appending lines to either the list of left lines (`left_lines` variable), or
    the list of right lines (`right_lines`), based on the "slope coefficient" of the
    line (either less than -0.5 / greater than 0.5). I also assert that the
    slope isn't too high (less than 1 / greater than -1) -- I comment this in
    section 2.;

  2. finding the leftmost / rightmost points in the sets of lines (the
    `get_outer_points` function);

  3. extrapolating based on those points, and taking the



![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline

One potential shortcoming is the sensitivity of this approach to changes in the
environment:

  1. debris / leaves / other unexpected objects on the road;
  2. weather (heavy rain, for example);

Also, the region chosen in the `region_of_interest` function relies on very strong
assumptions: that the slope of the road is very small, and that the car is not
changing it's direction. The `"challenge.mp4"` showed, in particular, that the
"region of interest" should depend on the direction in which the car is moving. 

I struggled a bit with the choice of the parameters of the Hough
transform, which might indicate that my method (in particular, the part where I
extrapolate the lines) is not very robust -- it was susceptible to even small
changes in the values of `threshold`, `min_line_length`, and `max_line_gap`.

I extrapolated the lines by choosing leftmost / rightmost points, but I didn't
foresee that the choice of these points is pretty "variable" (I don't get what
I want for every possible frame). Thus, sometimes the extrapolated line "jumps",
and doesn't "stick" to the actual lane line.

Put shortly: there is a bit of "jitter" in the resulting lines.

### 3. Suggest possible improvements to your pipeline

One solution to the jitter I found (on the *p-lane-lines* channel on Slack) was
the use a sliding window, from which the next extrapolated lines depend on the n=10 previous ones. But that seemed like "cheating" in the sense that this solution
would go beyond the single-frame-analysis scheme.

I think I would get more stable results if I identified the relevant lines better.
The hack with choosing the slope between 0.5 and 1 should be more "general"
(perhaps by asserting that the lines have some desired properties?). Also, I think
I would get better results by extrapolating the lines using not only the leftmost
and rightmost points, but all in the middle, using the `np.polyfit` function.
Also, I could weight the points passed to `np.polyfit` by the lenghts of the lines.

Furthermore,
