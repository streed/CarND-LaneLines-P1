# **Finding Lane Lines on the Road** 

## Goals

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[original]: test_images/solidYellowLeft.jpg "Original"
[masked_canny]: my_examples/masked_canny.png "Masked and Canny"
[hough_lines_extrapolated]: my_examples/hough_lines_filtered_extrapolated.png "Hough Lines filtered and extrapolated"
[final_image]: my_examples/extrapolated_lines_on_original_image.png "Extrapolated lines on original image"
[lane_bug]: my_examples/lane_bug.png "Lane Bug"
[challenge_bug]: my_examples/challenge_bug.png "Challenge Bug"

---

### Reflection

### 1. Pipeline

The pipeline consisted of 5 image processing steps and 1 extra step to visualize the lane lines on the image itself.

    1. The first step was to take the color image and turn it to gray scale.
    2. The second step was to take the grayscale image and perform a gaussian blur with a kernel size of 3
    3. The third step was to perform the canny edge detection with a low threshold of 200 and a high threshold of 255
    4. The fourth step was to mask out the region of interest. This region used the bottom corners and the central point, this created a trianglular region in the image.
    5. The fifth step was to take the image with the edges found and transform this image into hough space.
    
 After the above image processing steps were completed, I had a list of lanes line segments that had been extracted from the image. The final step was to draw lines that showed where the lane lines were in the image. This final step, `draw_lines` has 3 steps inside of it.
 
     1. First the line segments are split based on their slope, a positive slope was the right lane line and a negative slope was the left lane line.
     2. Second the right lane lines are merged and extrapolated.
         1. First we find the average point locations for each side of each line segment.
         2. Using these new point endings we find the new slope as well as finding b and the x intercept with y value of the screen image height.
         3. Once we have this x intercept I can draw a new line towards the cut off of the masked region (middle pixel). The value `1.15` moves the Y point towards the bottom of the screen, this makes the output easier to examine.
         
The above steps are repeated for the left lane line segments and the lane line image is complete at this point. After this there is just one last step and that is to overlay the lane line's onto the original image.

Given the above steps the majority of the work was spent in tuning the parameters of the hough lines function and the gaussian blur function. I found that having a fairly high max line gap helped to make the lane lines more stable, smaller max line gaps created slightly more accurate lane lines, but they were also more erradict and I feel wouldn't be of much use. Along with creating longer lines there was the other issue where we didn't want to create too small of lines either, these smaller lines were probably noise or could the various reflectors in the image making it through the gaussian blur. While tweaking the gaussian there was a degredation in both accuracy of the pipeline as well as a drastic increase in time it took to process the videos for any kernel size greater than 3, so the value of 3 was choosen.

Some images of the pipeline can be seen here:

![original][original]
![masked_canny][masked_canny]
![hough_lines_extrapolated][hough_lines_extrapolated]
![final_image][final_image]


### 2. Identify potential shortcomings with your current pipeline

While tuning the various parameters to create a smoothly tracked left and right lane line I found that in some certain cases when the lane was a dashed line there the hough lines function would return no lines for those cases, this caused some issues and bugs in the code and in some cases the lane line may flicker, I'm not entirely sure if the values I am using are correct or optimal, but they produce a good result.

The other shortcoming is sometimes the lane lines do flicker and create the following situtation:

![lane_bug][lane_bug]

In this case I am not entirely sure what is occuring outside of possibly getting some really odd cases where the line segments were not found through the hough line function for the lane line and some other contrasty part of the image was found that my code thought was a lane line. In either case, in the following few frames it snaps back to the lane line and tracks it closely.

The other shortcomings cropped up in the challenge video. The challenge video has a very tight radius and this poses some problems for the pipeline. The pipeline really likes straight lines. So, in some cases when the road turns a lot the lane lines kind of hover on the inside of the curve next to the real lane line and not exactly on the lane itself. The next issue was shadows. My current pipeline ignores lighting completely and runs off various assumptions that the lane lines will have a lot of contrast with the road, but in the case of the challenge video this is not entirely true.

![challenge_bug][challenge_bug]

As is shown in the image the trees shadow is causing the lane lines to stop following the real lane lines. This could be a big issue in a real driving situation as there are a lot of trees along road sides.

### 3. Suggest possible improvements to your pipeline

Some improvements to the pipeline would be to use the HSV color space and pull out the exactly lane colors from the image and mask out everything that is not these colors. This would help mitigate the shadow problem described above, but also the issue where there might be some noise in the image that has high contrast and is in the region of interest. This could also lead to the code being faster as the gaussian step could possibly be removed from the pipeline.
