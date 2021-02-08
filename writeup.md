# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report
---

### Reflection

### 1. Pipeline Description 
The pipeline is roughly divided into 7 steps.
1. Convert the input image into greyscale one.
2. Gaussian blur the greyscaled image.
3. Apply Canny transformation to get an image of edges.
4. Use a trapezoid shape, which top edge is near the diminishing point of the camera's view, to mask out the unnecessary lines
5. Hough transform to extract the coefficients of the lines
6. Draw the output lines as-is, using draw_lines() function provided
7. Cluster, average and extrapolate the lines output from the Hough transform step and produce a clear left and a clear right line.

The following image shows the pipeline flows:
![pipeline][./test_images_output/pipeline.png]

I did not modify the draw_lines function to do the extrapolation jobs, instead I made a delegate class called `DrawLinesExtrapolate` to calculate the extrapolation and manage possible improvements there.

The implementation of `DrawLinesExtrapolate` is mainly 5 steps as follows:
1. Sort all the lines based on its length, descending.
2. Cluster the lines into two categories using K-means method.
3. Remove noises by checking reasonable slopes.
4. Average or weighed sum(weighs are based on line length, longer lines has higher weighs) all the left lane lines and right lane lines, to get two lines.
5. Smooth out the two line coefficients using the previous estimations, which is used when generating lines for videos. For single image this step is essentially ignored.

Finally the code can draw the lines and blend in the original image using weighted_img() function provided.


### 2. Potential Shortcomings
There are a few things to note in this pipeline.
First the edge detection threshold needs hand tuning which isn't universally working, it will be especially bad when the lighting condition changes like entering a tunnel.

The second one I'm not satisfied with is the masking part. The fixed region means we need to carefully align the camera on the windshield centre which can results in errors. It will also have problem when driving uphill or downhill in a mountainous area.

Finally, in this pipeline if in one frame there's no left or right lane lines detected, it will resort to previous estimation to guess where the line is. This can work in a relatively smooth driving, assuming the line detection will work in a short time. That means if there's any lighting condition change or masking error as previously described which keeps the line detection from working for a long time, the algorithm will fail and not producing proper lane marks.

### 3. Possible Improvements
The edge detection and masking can be improved by supervisied machine learning, given carefully labelled data.

It will also help if the lane line is not detected as a straight line(1 degree polynomial). It can be detected as a curve so that interframe interpolation can do a better job at guess where the lane is when line detection fails.
