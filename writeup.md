## **Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # "Image References"
[image1]: ./output_images/Multi_scale.png
[image2]:  ./output_images/Detection_windows.png
[image3]:  ./output_images/Hog_subsampling.png
[image4]:  ./output_images/Heat_map.png
[image5]:  ./output_images/Examples.png
[image6]:  ./output_images/xxx.png
[image7]:  ./output_images/xxx.png

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the Jupyter notebook. I started by reading in all 5966 KITTI vehicle and all 8968 non-vehicle images. I decided to leave out GIT car images since these are time sequenced frames and would need further preparation (shuffling for training / test split is not recommended). Thus there are more non-vehicle images than vehicle images, but overall this gave a better training result.

In code cell 2 I then explored different color spaces and different HOG parameters (orientations, pixels_per_cell, and cells_per_block...). I started with an inital parameter set which I've got by exploring color spaces and HOG features with quizzes provided in course material.

#### 2. Explain how you settled on your final choice of HOG parameters.

To determine whether a parameter set is useful or not, I mainly looked at accuracy of following training step (code cell 3), but also at time consumption of feature extraction and training. Finally I decided to go on with these parameters:
| Parameter                  | Name           | Value |
| -------------------------- | -------------- | ----- |
| Color space                | color_space    | YCrCb |
| HOG orientations           | orient         | 13    |
| HOG pixels per cell        | pix_per_cell   | 8     |
| HOG cells per block        | cell_per_block | 8     |
| HOG channel(s) to use      | hog_channel    | all   |
| Spatial binning dimensions | spatial_size   | 16x16 |
| Number of histogram bins   | hist_bins      | 32    |
| Spatial features           | spatial_feat   | on    |
| Histogram features         | hist_feat      | on    |
| HOG features               | hog_feat       | on    |

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

In code cell 3 I trained a linear SVM using 80% of dataset for training and 20% for validation. The dataset has been normalized with a standard scaler. The parameters mentioned above yielded in a reasonable accuracy of over 99% at less than 100 seconds of feature extraction and training time.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I carefully looked at the video showing multi-scale windows and decided to implement an algorithm to get different window sizes for different planes:

![alt text][image1]

In code cell 4 you can see the final function `slide_window()`. Depending on plane number, it calculates a list of windows: the farer away a plane is, the smaller its windows are. The windows have an overlap of 50%. To test this method, I took an example image, calculated the features (function `single_img_features()` in code cell 5) and searched for cars (function `search_windows()` in same code cell). The result can be seen in code cell 6:

![alt text][image2]

Although the method itself sounded promising, the amount of found boxes was not great. As you can see, the white car on the right is barely detected.

I also tried a different approach which was presented as "Hog subsampling". I took the same example image and tested this method with success. Both cars are detected with a sufficient amount of windows:

![alt text][image3]

Scales and overlap were determined by exploration of different values. I tried to maximize amount of detected boxes on cars and minimize false positives. In the end I used four different scale values (0.75, 1.0, 1.5 and 2.0) and an overlap of 75%.

I then added a heat map and a thresholding function (code cell 9):

![alt text][image4]

Although this method looked better than my implementation, it turned out that computing time is about 5 times higher. Processing of project video took me over an hour, which was not reasonable, because I needed to tweak parameters several times. I therefore decided to go with my sliding window implementation, even if detection performance is not as good as with Hog subsampling.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on seven planes using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result. To optimize performance I tuned threshold value. It turned out, that a value of 2 is suitable for suppression of false positives. An even higher value would unnecessarily minimize vehicle detection. Here are the results of all example images (code cells 7 and 8):

![alt text][image5]



### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_final.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Additionally I sumed up the heatmaps of eight consecutive frames and applied a second, overall threshold.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Once again it turned out that my computer is a quite slow machine. Initial parameter settings can be deduced from course material or by thinking about, but sometimes you still need to do try-and-error. And then it's really bad, if you have to wait for over one hour to see your results...

Therefore I had to take a sliding window algorithm with only a few windows. It's faster, but also not as performant as other algorithms can be. A main drawback is, that there are not so much detection windows. So if a car is slightly off the window, it could happen, that it's also not be recognized be surrounding windows. This is mainly the case for vertical offset, because my algorithm hasn't a vertical overlap feature.

In the beginning of the video there are some detections to the left of the road. I am not sure, if they are false positives or if the cars to the left are detected. Unfortunately I am running out of time, otherwise I could look at heatmap to see, wether it's a strong detection or a weak one.

The time spent for lengthy project video processing was missing for other points which could be improved in future:

* Examination of training data: use GTI and Udacity images, compare dataset with project video (I feel a little bit, that they do not really match) and use augmentation (flipping images).
* Deeper inspection of HOG parameters, maybe there is some exotic combination with better results.
* Different method for training (now it's just SVC).
* Better sliding window algorithm: at least vertical overlap, but also HOG subsampling should be added.
* Tracking of vehicle movement over several video frames (centroids)