# Finding Lane Lines on the Road

This is a write-up for Project 1 in Udacity's Self-Driving Car Nanodegree

Project repository: https://github.com/udacity/CarND-LaneLines-P1

Project Rubric: https://review.udacity.com/#!/rubrics/322/view

> When we drive, we use our eyes to decide where to go. The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle. Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.
> 
> In this project you will detect lane lines in images using Python and OpenCV. OpenCV means "Open-Source Computer Vision", which is a package that has many useful tools for analyzing images.

More about the nanodegree program: https://www.udacity.com/drive

## Pipeline

The purpose of the pipeline is to compose several different operations together, apply them to an image, and produce an annotated image that shows where a lane on a road would be.

My pipeline consists of multiple steps:

* Applying a color mask
* Performing edge detection
* Selecting regions to search for lane lines
* Using the Hough transform to find line segments
* Extrapolating the lane from the line segments provided by the Hough transform

### Applying the color mask

![solidyellowleft](https://cloud.githubusercontent.com/assets/712014/23243849/67b42b14-f936-11e6-93f4-46dddc3412ab.jpg)

Lanes on the road are typically found as a single color and designed to stand-out from the background. We can use this fact to help us filter out elements in the image that are irrelevant. To accomplish this, we use something called a color mask. A color mask can be applied to an image to remove all colors except for the ones we specify.

This can be done using OpenCV's [`cv2.inRange(src, lowerb, upperb[, dst]) → dst`](http://docs.opencv.org/3.0-beta/modules/core/doc/operations_on_arrays.html?highlight=inrange#cv2.inRange) thresholding function. You provide the image and a range of the colors you want to create a mask for.

For this project I created two masks: one for white lanes and one for yellow lanes. 

#### White lane

![solidyellowleft](https://cloud.githubusercontent.com/assets/712014/23243766/ef9437a0-f935-11e6-9452-5df981160549.jpg)

The white lane I manually found values that seemed close and looked to provide good results when ran against the example images. 

#### Yellow lane

![solidyellowleft](https://cloud.githubusercontent.com/assets/712014/23243816/287e77e2-f936-11e6-8ab2-ee5d5ffd38a0.jpg)

The yellow lane was a bit more involved since its not as simple as setting all the channels to the same value. I used a color picker to grab the RGB components and plugged it into [colorizer](http://colorizer.org/) to transform it to HSV space. Using the [`cv2.cvtColor()`](http://docs.opencv.org/3.0.0/d7/d1b/group__imgproc__misc.html#ga397ae87e1288a81d2363b61574eb8cab) function to convert the image from BGR colorspace to HSV, I was able to create a mask that isolated the yellow lane.

### Performing edge detection

![solidyellowleft](https://cloud.githubusercontent.com/assets/712014/23243941/fac4459c-f936-11e6-8104-09bbf28d1661.jpg)

I then use the canny edge detector to pull out edges in the image. [`cv.Canny(image, edges, threshold1, threshold2, aperture_size=3)`](http://docs.opencv.org/2.4/modules/imgproc/doc/feature_detection.html?highlight=canny#canny)

### Selecting regions to search for lane lines

![solidyellowleft](https://cloud.githubusercontent.com/assets/712014/23244093/e0ba98f8-f937-11e6-82a5-740f1f2ada56.jpg)

After performing edge detection, there is still a fair amount of irrelevant edges that need to be ignored if we are to find the lane lines. We remove a majority of the image and focus on a region that we would most likely find lane lines.

### Using the Hough transform to find line segments

![solidyellowleft](https://cloud.githubusercontent.com/assets/712014/23244234/a3610edc-f938-11e6-8e4d-bae9c679462b.jpg)

The hough transform is the operation in the pipeline that actually finds line segments in the image and provides the most information of where the lanes lines could be. Initially I performed the hough transform on a single region which provided some spurious results. When I adjusted the pipeline to instead use two regions (one of the left line and one for the right), it significantly increased the accuracy.

A large amount of time was spent tweaking the parameters and manually tuning the hough transform to provide good results.

### Extrapolating the lane from the line segments provided by the Hough transform

![solidyellowleft](https://cloud.githubusercontent.com/assets/712014/23244276/fdfc2250-f938-11e6-8d5b-2341ba9e6dfb.jpg)

Once we have the line segments produced by the hough transform, we can find a line that would be suitable for annotating the initial image. I found [`cv.FitLine(points, dist_type, param, reps, aeps)`](http://docs.opencv.org/2.4/modules/imgproc/doc/structural_analysis_and_shape_descriptors.html#fitline) to work relatively well at extrapolating the line from the lines found using the hough transform.

## Potential shortcomings with my current pipeline

The challenge video exposed a couple flaws with my pipeline:

* Highly sensitive to color
* Requires hard coded regions
* Highly dependent on lane location

## Possible improvements to my pipeline

* Use information from past frames
* Better tuned hough transform and edge detection
* Automatically calculate region
* Automatically calculate color mask range
