#**Finding Lane Lines on the Road** 

##Writeup Project-1


---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image0]: ./myexamples/0-original.jpg "Original"
[image1]: ./myexamples/1-grayscale.jpg "Grayscale"
[image2]: ./myexamples/2-gaussianblur.jpg "Gaussian Blur"
[image3]: ./myexamples/3-edges.jpg "Edges"
[image4]: ./myexamples/4-mask.jpg "Region of Interest"
[image5]: ./myexamples/5-hough.jpg "Hough Lines"
[image6]: ./myexamples/6-polyfit.jpg "Polyfit"
[image7]: ./myexamples/7-mask2.jpg "Polyfit"
[image8]: ./myexamples/8-result.jpg "Detected Lines"

---

### Reflection

###1. Pipeline For Lane-Line-Detection

My pipeline consisted of 7 steps.

Step 1: Converting to grayscale

![Original][image0]
![Grayscale][image1]

Step 2: Applying Gaussian Blur

![Gaussian Blur][image2]

Step 3: Detecting Edges

![Edges][image3]

Step 4: Masking Region of Interest

![Region of Interest][image4]

Step 5: Drawing Hough Lines

![Hough Lines][image5]

Step 6: The hough lines are used for generating (regression) two lines which aproximate the lane lines. Hough lines on the left half of the image are used for the left lane-line and the hough lines on the right half of the image are used for the right lane-line.

![Polyfit][image6]

Step 7: Masking the Regression Lines

![Mask2][image7]

Result: The Lane-Lines are properly detected

![Result][image8]

###2. Potential shortcomings with current pipeline


One potential shortcoming would be in sharp curves, when the lane lines are so heavily bend, that the corresponding hough lines cross the center line of the image. In this case, the regression/polyfit result will get increasingly inaccurate and the correct detection of the lane lines could fail.


###3. Possible improvements to pipeline

- Improving edge detection by reducing to different colorspaces instead of greyscale. This could be more effective on roads with difficult and high contrast surfaces.

- Improving the regression of hough lines. Perhaps could polyfit of a higher order improve the accuracy on curvy roads.



# THE CODE - Self-Driving Car Engineer Nanodegree


## Project: **Finding Lane Lines on the Road** 
***
In this project, you will use the tools you learned about in the lesson to identify lane lines on the road.  You can develop your pipeline on a series of individual images, and later apply the result to a video stream (really just a series of images). Check out the video clip "raw-lines-example.mp4" (also contained in this repository) to see what the output should look like after using the helper functions below. 

Once you have a result that looks roughly like "raw-lines-example.mp4", you'll need to get creative and try to average and/or extrapolate the line segments you've detected to map out the full extent of the lane lines.  You can see an example of the result you're going for in the video "P1_example.mp4".  Ultimately, you would like to draw just one line for the left side of the lane, and one for the right.

In addition to implementing code, there is a brief writeup to complete. The writeup should be completed in a separate file, which can be either a markdown file or a pdf document. There is a [write up template](https://github.com/udacity/CarND-LaneLines-P1/blob/master/writeup_template.md) that can be used to guide the writing process. Completing both the code in the Ipython notebook and the writeup template will cover all of the [rubric points](https://review.udacity.com/#!/rubrics/322/view) for this project.

---
Let's have a look at our first image called 'test_images/solidWhiteRight.jpg'.  Run the 2 cells below (hit Shift-Enter or the "play" button above) to display the image.

**Note: If, at any point, you encounter frozen display windows or other confounding issues, you can always start again with a clean slate by going to the "Kernel" menu above and selecting "Restart & Clear Output".**

---

**The tools you have are color selection, region of interest selection, grayscaling, Gaussian smoothing, Canny Edge Detection and Hough Tranform line detection.  You  are also free to explore and try other techniques that were not presented in the lesson.  Your goal is piece together a pipeline to detect the line segments in the image, then average/extrapolate them and draw them onto the image for display (as below).  Once you have a working pipeline, try it out on the video stream below.**

---

<figure>
 <img src="line-segments-example.jpg" width="380" alt="Combined Image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Your output should look something like this (above) after detecting line segments using the helper functions below </p> 
 </figcaption>
</figure>
 <p></p> 
<figure>
 <img src="laneLines_thirdPass.jpg" width="380" alt="Combined Image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Your goal is to connect/average/extrapolate line segments to get output like this</p> 
 </figcaption>
</figure>

**Run the cell below to import some packages.  If you get an `import error` for a package you've already installed, try changing your kernel (select the Kernel menu above --> Change Kernel).  Still have problems?  Try relaunching Jupyter Notebook from the terminal prompt.  Also, see [this forum post](https://carnd-forums.udacity.com/cq/viewquestion.action?spaceKey=CAR&id=29496372&questionTitle=finding-lanes---import-cv2-fails-even-though-python-in-the-terminal-window-has-no-problem-with-import-cv2) for more troubleshooting tips.**  

## Import Packages


```python
#importing some useful packages
import matplotlib.pyplot as plt
import matplotlib.image as mpimg
import numpy as np
import cv2
%matplotlib inline
```

## Read in an Image


```python
#reading in an image
image = mpimg.imread('test_images/solidWhiteRight.jpg')

#printing out some stats and plotting
print('This image is:', type(image), 'with dimesions:', image.shape)
plt.imshow(image)  # if you wanted to show a single color channel image called 'gray', for example, call as plt.imshow(gray, cmap='gray')
#plt.imshow(image, cmap='Accent')  # if you wanted to show a single color channel image called 'gray', for example, call as plt.imshow(gray, cmap='gray')
```

    This image is: <class 'numpy.ndarray'> with dimesions: (540, 960, 3)





    <matplotlib.image.AxesImage at 0x7f7a89a81b00>




![png](img/output_6_2.png)



## Ideas for Lane Detection Pipeline

**Some OpenCV functions (beyond those introduced in the lesson) that might be useful for this project are:**

`cv2.inRange()` for color selection  
`cv2.fillPoly()` for regions selection  
`cv2.line()` to draw lines on an image given endpoints  
`cv2.addWeighted()` to coadd / overlay two images
`cv2.cvtColor()` to grayscale or change color
`cv2.imwrite()` to output images to file  
`cv2.bitwise_and()` to apply a mask to an image

**Check out the OpenCV documentation to learn about these and discover even more awesome functionality!**

## Helper Functions

Below are some helper functions to help get you started. They should look familiar from the lesson!


```python
import math

def grayscale(img):
    """Applies the Grayscale transform
    This will return an image with only one color channel
    but NOTE: to see the returned image as grayscale
    (assuming your grayscaled image is called 'gray')
    you should call plt.imshow(gray, cmap='gray')"""
    return cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)
    # Or use BGR2GRAY if you read an image with cv2.imread()
    #return cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    
def canny(img, low_threshold, high_threshold):
    """Applies the Canny transform"""
    return cv2.Canny(img, low_threshold, high_threshold)

def gaussian_blur(img, kernel_size):
    """Applies a Gaussian Noise kernel"""
    return cv2.GaussianBlur(img, (kernel_size, kernel_size), 0)

def region_of_interest(img, vertices):
    """
    Applies an image mask.
    
    Only keeps the region of the image defined by the polygon
    formed from `vertices`. The rest of the image is set to black.
    """
    #defining a blank mask to start with
    mask = np.zeros_like(img)   
    
    #defining a 3 channel or 1 channel color to fill the mask with depending on the input image
    if len(img.shape) > 2:
        channel_count = img.shape[2]  # i.e. 3 or 4 depending on your image
        ignore_mask_color = (255,) * channel_count
    else:
        ignore_mask_color = 255
        
    #filling pixels inside the polygon defined by "vertices" with the fill color    
    cv2.fillPoly(mask, vertices, ignore_mask_color)
    
    #returning the image only where mask pixels are nonzero
    masked_image = cv2.bitwise_and(img, mask)
    return masked_image


def draw_lines(img, lines, color=[255, 0, 0], thickness=2):
    """
    NOTE: this is the function you might want to use as a starting point once you want to 
    average/extrapolate the line segments you detect to map out the full
    extent of the lane (going from the result shown in raw-lines-example.mp4
    to that shown in P1_example.mp4).  
    
    Think about things like separating line segments by their 
    slope ((y2-y1)/(x2-x1)) to decide which segments are part of the left
    line vs. the right line.  Then, you can average the position of each of 
    the lines and extrapolate to the top and bottom of the lane.
    
    This function draws `lines` with `color` and `thickness`.    
    Lines are drawn on the image inplace (mutates the image).
    If you want to make the lines semi-transparent, think about combining
    this function with the weighted_img() function below
    """
    for line in lines:
        for x1,y1,x2,y2 in line:
            cv2.line(img, (x1, y1), (x2, y2), color, thickness)

def hough_lines(img, rho, theta, threshold, min_line_len, max_line_gap):
    """
    `img` should be the output of a Canny transform.
        
    Returns an image with hough lines drawn.
    """
    lines = cv2.HoughLinesP(img, rho, theta, threshold, np.array([]), minLineLength=min_line_len, maxLineGap=max_line_gap)
    line_img = np.zeros((img.shape[0], img.shape[1], 3), dtype=np.uint8)
    draw_lines(line_img, lines)
    return line_img

# Python 3 has support for cool math symbols.

def weighted_img(img, initial_img, α=0.8, β=1., λ=0.):
    """
    `img` is the output of the hough_lines(), An image with lines drawn on it.
    Should be a blank image (all black) with lines drawn on it.
    
    `initial_img` should be the image before any processing.
    
    The result image is computed as follows:
    
    initial_img * α + img * β + λ
    NOTE: initial_img and img must be the same shape!
    """
    return cv2.addWeighted(initial_img, α, img, β, λ)
```

## Test Images

Build your pipeline to work on the images in the directory "test_images"  
**You should make sure your pipeline works well on these images before you try the videos.**


```python
import os
dirImages = "test_images/"
fileNameImages = os.listdir(dirImages)
print(fileNameImages)
```

    ['solidWhiteCurve.jpg', 'solidWhiteRight.jpg', 'solidYellowCurve.jpg', 'solidYellowCurve2.jpg', 'solidYellowLeft.jpg', 'whiteCarLaneSwitch.jpg']


## Build a Lane Finding Pipeline



Build the pipeline and run your solution on all test_images. Make copies into the test_images directory, and you can use the images in your writeup report.

Try tuning the various parameters, especially the low and high Canny thresholds as well as the Hough lines parameters.


```python
import matplotlib
# TODO: Build your pipeline that will draw lane lines on the test_images
# then save them to the test_images directory.
for i in range(len(fileNameImages)):
    fileNameImage = fileNameImages[i]
        
    # read image
    image = mpimg.imread(dirImages + fileNameImage)
    
    # process image
    resultImage = process_image(image)
    
    # write processed image
    plt.imshow(resultImage)
    matplotlib.image.imsave(dirImages + fileNameImage + ".linesdetected.jpg", resultImage)
    

```


![png](img/output_16_0.png)


## Test on Videos

You know what's cooler than drawing lanes over images? Drawing lanes over video!

We can test our solution on two provided videos:

`solidWhiteRight.mp4`

`solidYellowLeft.mp4`

**Note: if you get an `import error` when you run the next cell, try changing your kernel (select the Kernel menu above --> Change Kernel).  Still have problems?  Try relaunching Jupyter Notebook from the terminal prompt. Also, check out [this forum post](https://carnd-forums.udacity.com/questions/22677062/answers/22677109) for more troubleshooting tips.**

**If you get an error that looks like this:**
```
NeedDownloadError: Need ffmpeg exe. 
You can download it by calling: 
imageio.plugins.ffmpeg.download()
```
**Follow the instructions in the error message and check out [this forum post](https://carnd-forums.udacity.com/display/CAR/questions/26218840/import-videofileclip-error) for more troubleshooting tips across operating systems.**


```python
# Import everything needed to edit/save/watch video clips
from moviepy.editor import VideoFileClip
from IPython.display import HTML
```


```python
def process_image(image):
    # NOTE: The output you return should be a color image (3 channel) for processing video below
    # TODO: put your pipeline here,
    # you should return the final output (image where lines are drawn on lanes)
    
    # parameters
    imshape = image.shape
    kernel_size_gaussian = 5
    low_threshold_canny = 50
    high_threshold_canny = 150
    vertices = np.array([[(0,imshape[0]),(imshape[1]*5/10, imshape[0]*0.55), (imshape[1]*5/10, imshape[0]*0.55), (imshape[1],imshape[0])]], dtype=np.int32)
    rho = 2 # distance resolution in pixels of the Hough grid
    theta = np.pi/180 * 2 # angular resolution in radians of the Hough grid
    threshold = 30     # minimum number of votes (intersections in Hough grid cell)
    min_line_length = 40 #minimum number of pixels making up a line
    max_line_gap = 30    # maximum gap in pixels between connectable line segments
    
    # convert to grayscale
    imageMod = grayscale(image)
    
    # apply the gaussian blur
    imageMod = gaussian_blur(imageMod, kernel_size_gaussian)
    
    # apply canny to detect edges
    imageMod = canny(imageMod, low_threshold_canny, high_threshold_canny)
    
    # apply mask
    imageMod = region_of_interest(imageMod, vertices)
    
    # apply hough lines
    imageMod = hough_lines(imageMod, rho, theta, threshold, min_line_length, max_line_gap)
    
    # calculate lines
    leftLineX = []
    leftLineY = []
    rightLineX = []
    rightLineY = []
    
    coordNonzeros = np.argwhere(imageMod != 0)
    
    # sort hough lines left/right
    for y, x, _ in coordNonzeros:
        if x < imageMod.shape[1] * 0.5:
            leftLineX.append(x)
            leftLineY.append(y)
        elif x > imageMod.shape[1] * 0.5:
            rightLineX.append(x)
            rightLineY.append(y)
    
    # calculate slope/intersect with polyfit (y = m*x+b)
    leftLinePara = np.polyfit(leftLineX, leftLineY, 1)
    rightLinePara = np.polyfit(rightLineX, rightLineY, 1)
    
    # calc coordinates for left line
    ll = []
    
    # coordinate left line point 1
    ll.append(0)
    ll.append(int(leftLinePara[0] * ll[-1] + leftLinePara[1]))
    
    # coordinate left line point 1
    ll.append(image.shape[1])
    ll.append(int(leftLinePara[0] * ll[-1] + leftLinePara[1]))
    
    # calc coordinates for right line
    rl = []
    
    # coordinate right line point 1
    rl.append(0)
    rl.append(int(rightLinePara[0] * rl[-1] + rightLinePara[1]))
    
    # coordinate right line point 2
    rl.append(image.shape[1])
    rl.append(int(rightLinePara[0] * rl[-1] + rightLinePara[1]))
    
    # calculate intersection left/right lines
    
    
    # create empty image
    imageLines = np.copy(image)*0
    
    # draw lines
    cv2.line(imageLines, (ll[0], ll[1]), (ll[2], ll[3]), [255, 0, 0], 10)
    cv2.line(imageLines, (rl[0], rl[1]), (rl[2], rl[3]), [255, 0, 0], 10)
    
    # apply mask to imageLines
    imageLines = region_of_interest(imageLines, vertices)

    # combine original image and lines
    result = weighted_img(imageLines, image)
    
    return result
```

Let's try the one with the solid white lane on the right first ...


```python
white_output = 'white.mp4'
clip1 = VideoFileClip("solidWhiteRight.mp4")
white_clip = clip1.fl_image(process_image) #NOTE: this function expects color images!!
%time white_clip.write_videofile(white_output, audio=False)
```

    [MoviePy] >>>> Building video white.mp4
    [MoviePy] Writing video white.mp4


    100%|█████████▉| 221/222 [00:33<00:00,  5.91it/s]


    [MoviePy] Done.
    [MoviePy] >>>> Video ready: white.mp4 
    
    CPU times: user 1min 44s, sys: 4.12 s, total: 1min 48s
    Wall time: 35.1 s


Play the video inline, or if you prefer find the video in your filesystem (should be in the same directory) and play it in your video player of choice.


```python
HTML("""
<video width="960" height="540" controls>
  <source src="{0}">
</video>
""".format(white_output))
```





<video width="960" height="540" controls>
  <source src="white.mp4">
</video>




## Improve the draw_lines() function

**At this point, if you were successful with making the pipeline and tuning parameters, you probably have the Hough line segments drawn onto the road, but what about identifying the full extent of the lane and marking it clearly as in the example video (P1_example.mp4)?  Think about defining a line to run the full length of the visible lane based on the line segments you identified with the Hough Transform. As mentioned previously, try to average and/or extrapolate the line segments you've detected to map out the full extent of the lane lines. You can see an example of the result you're going for in the video "P1_example.mp4".**

**Go back and modify your draw_lines function accordingly and try re-running your pipeline. The new output should draw a single, solid line over the left lane line and a single, solid line over the right lane line. The lines should start from the bottom of the image and extend out to the top of the region of interest.**

Now for the one with the solid yellow lane on the left. This one's more tricky!


```python
yellow_output = 'yellow.mp4'
clip2 = VideoFileClip('solidYellowLeft.mp4')
yellow_clip = clip2.fl_image(process_image)
%time yellow_clip.write_videofile(yellow_output, audio=False)
```

    [MoviePy] >>>> Building video yellow.mp4
    [MoviePy] Writing video yellow.mp4


    100%|█████████▉| 681/682 [01:42<00:00,  7.08it/s]


    [MoviePy] Done.
    [MoviePy] >>>> Video ready: yellow.mp4 
    
    CPU times: user 5min 7s, sys: 12.6 s, total: 5min 20s
    Wall time: 1min 43s



```python
HTML("""
<video width="960" height="540" controls>
  <source src="{0}">
</video>
""".format(yellow_output))
```





<video width="960" height="540" controls>
  <source src="yellow.mp4">
</video>




## Writeup and Submission

If you're satisfied with your video outputs, it's time to make the report writeup in a pdf or markdown file. Once you have this Ipython notebook ready along with the writeup, it's time to submit for review! Here is a [link](https://github.com/udacity/CarND-LaneLines-P1/blob/master/writeup_template.md) to the writeup template file.


## Optional Challenge

Try your lane finding pipeline on the video below.  Does it still work?  Can you figure out a way to make it more robust?  If you're up for the challenge, modify your pipeline so it works with this video and submit it along with the rest of your project!


```python
challenge_output = 'extra.mp4'
clip2 = VideoFileClip('challenge.mp4')
challenge_clip = clip2.fl_image(process_image)
%time challenge_clip.write_videofile(challenge_output, audio=False)
```

    [MoviePy] >>>> Building video extra.mp4
    [MoviePy] Writing video extra.mp4


    100%|██████████| 251/251 [01:01<00:00,  4.19it/s]


    [MoviePy] Done.
    [MoviePy] >>>> Video ready: extra.mp4 
    
    CPU times: user 2min 39s, sys: 6.45 s, total: 2min 45s
    Wall time: 1min 3s



```python
HTML("""
<video width="960" height="540" controls>
  <source src="{0}">
</video>
""".format(challenge_output))
```





<video width="960" height="540" controls>
  <source src="extra.mp4">
</video>



