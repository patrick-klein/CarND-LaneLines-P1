# **Finding Lane Lines on the Road**

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./writeup_images/yellow/solidWhiteCurve.gif "Animated pipeline gif"

[image2]: ./writeup_images/incorrect_slope-2.png "example of incorrect slope"

[image3]: ./writeup_images/grayscale_vs_yellowscale.gif "comparison of grayscale & yellowscale"

[image4]: ./writeup_images/challenge.png "problem area in challenge video"

[image5]: ./writeup_images/missing_line.png "missing line in challenge video"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline consisted of 7 steps, each with a call to one of the helper functions.

1. ```yellowscale``` creates a copy of the image with only the yellow channel (which is really just an average of the red and green channels).  This is used instead of grayscale to create a better contrast for both yellow and white lines.

2. ```gaussian_blur``` applies a small blur to the image.

3. ```canny``` uses the Canny edge detection algorithm to create a map of the edges in the image.

4. ```region_of_interest```creates a mask of a trapezoidal region on the bottom of the screen, so that only edges within that region are kept.

5. ```hough_lines``` uses a Hough transform to generate a list of lines in the image, given the set of pixels that represent edges in the region of interest.

6. ```draw_lines``` loops through all of the lines in the list, and separates them based on the sign of their slope, while ignoring lines that are nearly flat.  Once all of the initial lines have been categorized, each side is averaged and drawn onto a new, blank array.  However, if the Hough transform didn't return a valid line, nothing is drawn for that side.

7. ```weighted_img``` then overlays the lines onto a copy of the original image.

![alt text][image1]
**Figure 1.** Each step of the pipeline, as described above, for the image *solidWhiteCurve.jpg*


### 2. Identify potential shortcomings with your current pipeline


While this pipeline correctly and adequately highlights the lane on the test images and videos, there were a few issues faced when designing it.

The first issue was caused by outlier line slopes that severely impacted the final lines being drawn onto the image.  An example of this is in Figure 2 below.  The first attempt I made in solving this issue was to weight the averages of the slopes in a way that de-emphasized outliers.  This proved to be difficult however, as there wasn't a straightforward or robust solution.  Most attempts I made at removing outlier slopes ended up breaking the algorithm for normal cases.

![alt text][image2]

**Figure 2.** An example of *draw_lines* output not aligning with the lanes.

I was able to work around the issue instead by tweaking the parameters to the Hough transform.  The most significant change was increasing max_line_gap to 300.  This helped ensure that dashed lines were recognized as only one line, instead of many smaller pieces.

Additionally, there are images in the challenge video that can cause the Hough transform to not find any valid lines.  When this occurs, the algorithm skips drawing the line on that frame.  This is still infrequent and never persists for more than a frame, but it still needs to be addressed.

![alt text][image5]

**Figure 3.** An example of the pipeline not finding any valid lines on the right side.

### 3. Suggest possible improvements to your pipeline


Problem images, such as Figure 3, are caused by two factors.  The first is that there are distractions, for example the change in pavement and shadows, which cause additional edges to show up, as seen in a similar frame in Figure 4.

![alt text][image4]

**Figure 4.**  A frame in the challenge video with many distracting edges.

Barring more advanced image-processing techniques, I believe the best solution to this is to find the best combination of parameters for the Gaussian blur, Canny edge detection, and Hough transform.  This process could be automated if a suitable metric for "correctness" could be calculated; perhaps by calculating the total percentage of edge pixels that are contained in the drawn lines, or even minimizing the number of frames that fail to find valid lines.

The second issue deals with the inability to detect the lane markings.  Partially, this is an issue with linear Hough transforms not being able to correctly model curved roads.  It may also be possible to use information from lines in previous frames to infer where the lane is, or even help smooth the annotations if they are jittery in the video.
