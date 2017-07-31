# **Finding Lane Lines on the Road** 

The goals of this project are the following:
* Make a pipeline that finds lane lines on the road for several videos.
* Discuss the project in a written report (that's the purpose of this document).

---

## 1. Description of the pipeline

The "pipeline" takes a color .jpg image and after a series of steps outputs the original image superimposed with two line segments that indicate the left and right lanes.

The pipeline consists of 8 steps: 
1. First, the color image is converted to four grayscale images, the first image being a true conversion of the full color to grayscale and the remaining three images simply being the separate Red, Green, and Blue components of the original image.
2. Gaussian smoothing is applied to each grayscale image.
3. Canny edge detection is then applied to each image.
4. Next, each image is masked by a trapezoid (with the base along the horizontal at the bottom of the image). The trapezoid is wide enough such that both lanes are fully visible at all times in all sample videos.  The height of the trapezoidal mask is chosen such that the lanes are roughly straight within the mask (i.e. the road in the far distance is removed).
5. A Hough transform is applied to each image.  The parameters for the Hough transform were manually fine-tuned to accurately detect lanes in the optional "Challenge" video. The same parameters work well for the non-Challenge videos. The output of the Hough transform is a collection of lines segments.
6. The Hough line segments define lines (infinite extensions of the line segments) that are categorized as either "left lane," "right lane" or "neither" depending on whether the lines intersect the bottom and top of the mask within prescribed bounds.  The purpose of this step is to throw out lines with slopes or positions differing greatly from the expected values (e.g. roughly horizontal lines are removed). The output of this step is two collections of Hough line segments: one for the left lane and one for the right lane.
7. Each collection of line segments is used to construct a single line segment based on the average slope and the average position of the intersections with the bottom of the image (where the car is). The average is a weighted average with weights proportional to the length of the Hough line segments. The output is an image showing the average line segment for each lane over a black background. Rather than modify the draw_lines() function (which draws all Hough lines), I wrote a new function called draw_averaged_lines() that performs the described averaging.
8. Finally, these two line segments are overlaid on the original color image.


## 2. Potential shortcomings

1. Highly curved lanes cannot be accurately detected because the pipeline outputs two straight lane lines.
2. The lanes can only be correctly detected if they lie within the trapezoidal mask and have roughly the "correct" slopes and positions. Therefore, if the car turns, changes lanes, or drifts too far left or right, then the lane lines will not be accurately detected. 
3. The parameters were fine-tuned to specific conditions: daylight, highway, well-marked lanes, no cars directly in front, no lane changes, and no extraneous markings on the road. If any of these conditions were changed, the pipeline could fail catastrophically.

## 3. Possible improvements

Preferably, we would be able to detect lane lines even when the car turns or changes lanes. Instead of using pre-defined ranges of slopes and positions for the two lanes, we could use histograms of Hough line segments' slopes and positions to determine candidate lane lines.

The current pipeline only considers only one video frame at a time, but we could use multiple video frames to determine the lane lines. For example, if no lane line could be found in the current frame, then the lane line from the previous frame would be used as a good guess. Basically, something like a Kalman filter would be appropriate. The "jiggle" of the lane lines from one frame to the next would be reduced by a Kalman filter.
