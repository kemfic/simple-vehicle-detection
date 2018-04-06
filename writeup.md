## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./writeup_imgs/car.png
[image2]: ./writeup_imgs/notcar.png
[image3]: ./writeup_imgs/hog.png
[image4]: ./writeup_imgs/road.png
[image5]: ./writeup_imgs/small.png
[image6]: ./writeup_imgs/medium.png
[image7]: ./writeup_imgs/large.png
[image8]: ./writeup_imgs/raw_heat.png
[image9]: ./writeup_imgs/filter_heat.png
[image10]: ./writeup_imgs/label.png
[image11]: ./writeup_imgs/out.png


## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the IPython notebook (or in lines # through # of the file called `some_file.py`).  

I started by reading in all the `vehicle` and `non-vehicle` images from the dataset Udacity provided.  Here is an example from each class:

![alt text][image1]  ![alt text][image2]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations` gave the best results with a minimal impact on performance).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YUV` color space and HOG parameters of `orientations=11`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image3]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters, and settled with 11 orientations, 8x8 pixels per cell, and 2x2 cells per block. 

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using the HOG features, and normalized my entire dataset with the training dataset. That way, my testing dataset remained sanitized, allowing for reliable, unbiased test results. My SVM had an accuracy of 97%, which is not bad, but can be improved upon. The best part about SVM's are that they train really quickly, even on my dataset of over 18000 images.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to keep things simple and implemented a sliding window search with 3 different window sizes in different areas. This was in order to reduce computation time, and because using smaller windows on closer points in the image is unnecessary, and only wastes time. I experimented with many different sizes, but mainly settled on 80x80, 100x100, and 120x120 window sizes. You can see the images below:

![alt text][image5] ![alt text][image6] ![alt text][image7]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

I searched on 3 different scales using an SVM that classified vehicles based on HOG features. I then converted the windows that detected cars into a heatmap, and added a bounding box around the labeled blobs. I didn't do much to optimize the performance of the classifier, due to the time constraint of the course deadline.
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap, then filtered out labelled blobs based on size and positiion. This tremendously diminished false positives.  I then assumed each blob corresponded to a vehicle, and constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap, labels, and bounding boxes:

### Here is an example of my pipeline:

|  ![original image][image4]  |
|:--:| 
|     *Original Image*      |
|   ![unfiltered heatmap][image8]   |
|     *Unfiltered Heatmap*      |
|   ![thresholded heatmap][image9]      | 
|     *Thresholded Heatmap*     |
| ![labels][image10]    |
|     *Labelled Blobs*|
| ![bounding boxes][image11]    |
|     *Bounding Boxes*      |
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had a lot issues with filtering out false positives and increasing the precision of the heatmap, as the detected blobs were too large, but my threshold had to be kept low to detect vehicles that were further away. I fixes this by increasing the overlap of windows, which allowed for much more votes per detection. Additionally, my blob filtration also greatly diminished false positives that couldn't be thresholded (I think this was an issue with my SVM model, as I didn't spend too much time optimizing it). My pipeline fails near areas with high gradients (changes in the road, shadows), and doesn't keep track of the vehicles over time. My goal for improving this project is to employ a method similar to SSD or YOLO in order to balance good performance with great accuracy. 

